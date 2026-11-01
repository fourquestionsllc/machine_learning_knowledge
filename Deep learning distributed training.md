# Distributed Training: Interview Cheat Sheet

**DDP replicates; FSDP and DeepSpeed ZeRO shard; Ray orchestrates.**

The snippets below show the key changes to describe in an interview. They omit routine model/data definitions and full training scripts. Assume Linux/CUDA, one process per GPU, and current PyTorch FSDP2 APIs. First-person answers are templates to adapt to work you have actually done.

## 1. Choose the right tool

| Tool | What happens on each GPU | Main reason to use it |
| --- | --- | --- |
| **DDP** | Full model, gradients, and optimizer state; different minibatches; gradients synchronized by all-reduce. | Improve throughput when training state fits per GPU. |
| **FSDP2** | Shards parameters, gradients, and optimizer state; gathers a module's parameters when needed. | Reduce per-GPU memory using native PyTorch. |
| **DeepSpeed ZeRO** | Progressively shards optimizer state, gradients, and parameters according to the ZeRO stage. | Control memory savings, accumulation, precision, and optional offload through an engine/config. |
| **Ray Train** | Schedules GPU workers and coordinates execution/reporting; DDP or another strategy handles model communication. | Manage training across a cluster. |

FSDP/ZeRO are sharded **data parallelism**: ranks process different examples and execute the model. Pipeline parallelism instead assigns different layers to stages. Ray can orchestrate jobs using these strategies; it is not itself a replacement for gradient synchronization. See the [PyTorch overview](https://docs.pytorch.org/tutorials/beginner/dist_overview.html) and [Ray Train guide](https://docs.ray.io/en/latest/train/getting-started-pytorch.html).

## 2. DDP — “I replicated the model and synchronized gradients”

**What I would say:**
> “I used one process per GPU, selected the GPU using local rank, and initialized NCCL. I wrapped the model with DDP and used a DistributedSampler so ranks processed different data. DDP synchronized gradients during backward; every rank then applied the same optimizer update.”

**Key code to explain:**

```python
import os
import torch
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP
from torch.utils.data import DataLoader, DistributedSampler

local_rank = int(os.environ['LOCAL_RANK'])  # Set by torchrun.
torch.cuda.set_device(local_rank)
dist.init_process_group('nccl')

model = DDP(model.cuda(), device_ids=[local_rank])
sampler = DistributedSampler(dataset, shuffle=True)
loader = DataLoader(dataset, batch_size=32, sampler=sampler)
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-4)

for epoch in range(epochs):
    sampler.set_epoch(epoch)  # Reshuffle consistently across ranks.
    for x, y in loader:
        optimizer.zero_grad(set_to_none=True)
        loss = criterion(model(x.cuda()), y.cuda())
        loss.backward()      # DDP hooks synchronize gradient buckets.
        optimizer.step()     # Every rank updates its own replica.
```

```bash
torchrun --standalone --nproc-per-node=4 train.py
```

**Remember:** DDP does not pool GPU memory. The full training state must fit per GPU. The sampler shards data; DDP itself does not. [DDP tutorial](https://docs.pytorch.org/tutorials/intermediate/ddp_tutorial.html).

## 3. FSDP2 — “I sharded training state to reduce memory”

**What I would say:**
> “When replicated training state became the memory bottleneck, I sharded transformer blocks with FSDP. It all-gathered parameters for computation and reduce-scattered gradients. I wrapped blocks bottom-up and created the optimizer after sharding.”

**Replace DDP wrapping with this; keep process setup, data sharding, and the normal training loop:**

```python
from torch.distributed.fsdp import fully_shard

# Assume a transformer whose blocks are in model.layers.
for block in model.layers:
    fully_shard(block, reshard_after_forward=True)
fully_shard(model, reshard_after_forward=True)

optimizer = torch.optim.AdamW(model.parameters(), lr=1e-4)
loss = criterion(model(x), y)  # Inputs are already on the local GPU.
loss.backward()
optimizer.step()
```

**Explain these operations:** all-gather temporarily assembles a module's parameters; resharding releases full parameter copies; reduce-scatter combines gradients and leaves each rank its shard. With resharding after forward, parameters are gathered again for backward. Each rank updates its own parameter shard.

**Tradeoff:** less persistent state per GPU, but additional communication and temporary full-module memory. Block-level wrapping controls gather size. Sharding does not eliminate activation memory; activation checkpointing saves that memory by recomputing forward work. [FSDP2 tutorial](https://docs.pytorch.org/tutorials/intermediate/FSDP_tutorial.html).

## 4. DeepSpeed — “I selected a ZeRO stage and used the engine”

**What I would say:**
> “I configured ZeRO, mixed precision, and gradient accumulation in JSON. I initialized the DeepSpeed engine and used its backward and step methods so it managed synchronization and optimizer-step boundaries.”

| ZeRO stage | Sharded across GPUs | Still replicated |
| --- | --- | --- |
| **1** | Optimizer state | Parameters and gradients |
| **2** | Optimizer state and gradients | Parameters |
| **3** | Optimizer state, gradients, and parameters | Parameters are gathered temporarily for computation |

**Configuration I should be able to explain — `ds_config.json`:**

```json
{
  "train_micro_batch_size_per_gpu": 2,
  "gradient_accumulation_steps": 8,
  "bf16": {"enabled": true},
  "optimizer": {
    "type": "Adam",
    "params": {"lr": 0.0001}
  },
  "zero_optimization": {"stage": 2}
}
```

```python
import deepspeed

engine, optimizer, _, _ = deepspeed.initialize(
    model=model, model_parameters=model.parameters(), config='ds_config.json')

for x, y in loader:  # Loader must already shard data across ranks.
    x = x.to(engine.device, dtype=next(engine.parameters()).dtype)
    y = y.to(engine.device)
    loss = criterion(engine(x), y)
    engine.backward(loss)
    engine.step()  # Call each microbatch; engine manages accumulation/zeroing.
```

```bash
deepspeed --num_gpus=4 train.py
```

**Explain the numbers:** four GPUs give an effective batch of `2 × 4 × 8 = 64`. BF16 reduces precision-related memory/compute cost on supported GPUs. Change stage 2 to 3 if parameter replication is also a bottleneck. CPU offload saves GPU memory but adds CPU-memory and transfer costs. [DeepSpeed setup](https://www.deepspeed.ai/getting-started/), [ZeRO config](https://www.deepspeed.ai/docs/config-json/).

## 5. Ray Train — “I managed workers; DDP did the synchronization”

**What I would say:**
> “I put the training loop inside TorchTrainer and requested GPU workers. Ray established the worker environment; prepare_model applied DDP, and prepare_data_loader handled distributed sampling and device transfer. I reported metrics from every worker.”

```python
import ray.train as train
from ray.train import ScalingConfig
from ray.train.torch import TorchTrainer, prepare_model, prepare_data_loader

def train_worker(config):
    model = prepare_model(build_model())  # Assigned GPU + DDP wrapper.
    loader = prepare_data_loader(build_loader())  # Sampler + device transfer.
    optimizer = torch.optim.AdamW(model.parameters(), lr=config['lr'])
    for epoch in range(config['epochs']):
        if train.get_context().get_world_size() > 1:
            loader.sampler.set_epoch(epoch)
        # Ordinary zero_grad -> forward -> backward -> optimizer.step loop.
        train_one_epoch(model, loader, optimizer)
        train.report({'epoch': epoch + 1})  # All workers, same report points.

trainer = TorchTrainer(
    train_worker,
    train_loop_config={'lr': 1e-4, 'epochs': 3},
    scaling_config=ScalingConfig(num_workers=4, use_gpu=True),
)
trainer.fit()  # In a script, put driver execution under __main__.
```

**Remember:** `num_workers=4` requests four training workers, one GPU each here; it does not shard the model. For FSDP/DeepSpeed integration, control wrapping yourself rather than accidentally nesting it inside DDP. Ray reporting does not automatically average custom loss metrics across ranks. [Ray PyTorch guide](https://docs.ray.io/en/latest/train/getting-started-pytorch.html).

## 6. Multi-node operations I should recognize

**Rank** is a global process ID; **local rank** identifies the GPU within a node; **world size** is the total process count. Two nodes with four processes each give world size 8.

```bash
# DDP/FSDP: run on BOTH nodes; use node-rank=0 on node0, 1 on node1.
torchrun --nnodes=2 --nproc-per-node=4 --node-rank=0 \
  --master-addr=node0 --master-port=29500 train.py
```

**DeepSpeed:** a hostfile such as `node0 slots=4` and `node1 slots=4` declares GPU slots. With passwordless SSH, launch once: `deepspeed --hostfile=hostfile train.py`.

**Ray:** start the head with `ray start --head --port=6379`; join worker nodes with `ray start --address=<head-ip>:6379`. Connect the driver with `ray.init(address='auto')`, then request the desired workers in `ScalingConfig`.

**Operational checks:** matching code/environments, correct GPU counts, reachable network addresses, and shared/durable checkpoint storage. Starting a process on each GPU is only the launch step; synchronization and data sharding still need to be correct.

## 7. Follow-up questions: short, practical answers

| Interview question | Answer I should be able to defend |
| --- | --- |
| **Why didn't adding GPUs fix OOM?** | DDP replicates state. Reduce microbatch, use supported mixed precision, shard state with FSDP/ZeRO, or checkpoint activations depending on what occupies memory. |
| **What is global batch size?** | Microbatch per GPU × data-parallel world size × accumulation steps. Match it when comparing runs; consider LR/schedule changes if it changes. |
| **Why did training hang?** | First inspect every rank for an earlier exception, mismatched collective calls/step counts, or network issues. Enable `NCCL_DEBUG=INFO` and `TORCH_DISTRIBUTED_DEBUG=DETAIL`. |
| **Why were GPUs idle?** | Check data-loading time, CPU/I/O saturation, and communication time before increasing GPU count. |
| **How did you checkpoint?** | DDP replicated state can be saved on rank 0. FSDP uses sharding-aware distributed checkpoints; DeepSpeed uses engine checkpoints. All ranks participate in those sharded saves. Ray workers report checkpoints for persistence. |
| **How did you recover?** | Restore model, optimizer, scheduler/scaler when used, step, RNG, and data progress. A saved file or worker retry alone does not resume training correctly. |
| **How did you measure success?** | Compare samples/tokens per second after warmup, peak memory, and validation quality at controlled effective batch. Report actual measured results. |

**A concise closing answer:**
> “I chose DDP for replicated training throughput, FSDP or ZeRO when model-state memory required sharding, and Ray when worker orchestration was the main need. I checked data partitioning, effective batch, communication, and checkpoint recovery rather than judging the job only by whether it launched.”

For checkpoint API details, see the [PyTorch distributed checkpoint recipe](https://docs.pytorch.org/tutorials/recipes/distributed_checkpoint_recipe.html). Code here is illustrative interview material, not a complete application or a claimed benchmark.
