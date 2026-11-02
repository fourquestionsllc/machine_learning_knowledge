# Distributed Deep Learning: Training and Parallelism

## 1. Core Concepts

Distributed training uses multiple GPUs, and sometimes multiple machines, to train a model faster or to fit models that exceed one GPU's memory.

Two common forms of parallelism:

- **Data parallelism:** Each worker processes different examples. In classic DDP, every GPU holds a full model replica.
- **Model parallelism:** The model's parameters or computation are split across devices. Tensor parallelism splits operations such as matrix multiplications; pipeline parallelism assigns different layers to different devices.
- **Sharding:** Divides model state—parameters, gradients, and/or optimizer state—among devices to reduce per-GPU memory.

## 2. PyTorch DDP

**DDP** means `DistributedDataParallel`.

1. Each GPU has a complete model replica.
2. Each worker processes a different mini-batch.
3. Each worker computes gradients locally.
4. DDP synchronizes gradients using **All-Reduce**.
5. Each replica applies the same update and remains in sync.

DDP is a strong choice when the model and its training state fit on each GPU. It improves throughput, but it does not shard the model parameters.

## 3. PyTorch FSDP

**FSDP** means `FullyShardedDataParallel`.

It combines data-parallel training with sharding of model state:

- Parameters are sharded across workers.
- Gradients are sharded.
- Optimizer states are sharded.
- Parameter shards are gathered temporarily when a wrapped module needs its full parameters for computation.

Typical communication pattern:

1. **All-Gather:** Collect parameter shards so a module can run.
2. **Forward pass:** Each worker processes its own data.
3. **Backward pass:** Compute gradients.
4. **Reduce-Scatter:** Reduce gradients across workers and distribute the resulting gradient shards.
5. **Optimizer step:** Each worker updates its local parameter shard.

FSDP can lower per-GPU memory use substantially. The exact memory and communication costs depend on wrapping strategy, precision, and configuration.

**Important:** FSDP is not the same as tensor parallelism. FSDP primarily shards model state, while tensor parallelism splits individual operations.

## 4. DeepSpeed and ZeRO

**DeepSpeed** is a PyTorch-oriented optimization and distributed-training library. Its best-known memory technique is **ZeRO (Zero Redundancy Optimizer)**.

ZeRO progressively shards training state:

| Stage | Optimizer states | Gradients | Parameters |
|---|---|---|---|
| ZeRO-1 | Sharded | Replicated | Replicated |
| ZeRO-2 | Sharded | Sharded | Replicated |
| ZeRO-3 | Sharded | Sharded | Sharded |

- **ZeRO-1:** Shards optimizer states, such as Adam's first- and second-moment estimates.
- **ZeRO-2:** Also shards gradients.
- **ZeRO-3:** Also shards model parameters; parameters are gathered when needed for computation.

The stage number identifies the sharding stage; it does **not** mean the number of GPUs or the number of model pieces. With four workers, for example, a sharded state may be divided among four workers.

DeepSpeed also offers features such as CPU offload, mixed precision, gradient accumulation, and support for other parallelism strategies.

### FSDP versus ZeRO

FSDP and DeepSpeed ZeRO-3 are conceptually similar: both shard parameters, gradients, and optimizer states, and both use communication such as All-Gather and Reduce-Scatter. They are different implementations with different configuration, integration, and performance characteristics.

## 5. Optimizer State

An **optimizer state** is information retained by an optimizer to help update model weights.

For Adam, this usually includes:

- **First moment:** An exponential moving average of gradients, similar to momentum.
- **Second moment:** An exponential moving average of squared gradients, used to adapt the effective step size per parameter.

Optimizer state is separate from the model parameters and the current gradients, but it can consume substantial memory.

## 6. Collective Communication

Collective communication operations coordinate data exchange among a group of workers.

| Operation | Main idea | Common use |
|---|---|---|
| **All-Reduce** | Reduce values across workers and make the result available to all | DDP gradient synchronization |
| **All-Gather** | Collect pieces from all workers so each can access the combined result | FSDP / ZeRO parameter gathering |
| **Reduce-Scatter** | Reduce values across workers, then distribute different portions of the result | FSDP / ZeRO gradient sharding |

These are communication primitives, not separate training frameworks.

NVIDIA GPU training commonly uses NCCL to implement them.

## 7. Ray's Role

**Ray** is a distributed-computing framework. **Ray Train** can launch and manage training workers across GPUs and machines.

Ray commonly handles:

- Resource scheduling and worker placement.
- Launching workers and setting up the distributed job.
- Coordinating training runs and integrating with supported training frameworks.
- Failure handling and recovery workflows, depending on configuration.

Inside the workers, PyTorch DDP, FSDP, or DeepSpeed handles the relevant model training and synchronization.

Ray and these tools can be used together:

**Ray coordinates the job; the training backend implements the distributed training strategy.**

### Example: Two Machines, Four GPUs Each

Ray can allocate eight workers, one per GPU.

| Worker | Machine | GPU |
|---|---|---|
| 0–3 | Machine 1 | 0–3 |
| 4–7 | Machine 2 | 0–3 |

Ray schedules and launches the workers. If the training code uses DDP, DDP synchronizes gradients among the workers. If it uses FSDP, FSDP manages parameter and gradient sharding.

Ray is optional: PyTorch distributed training can also be launched without Ray, for example with `torchrun`.

Ray is useful when Python-native resource scheduling and orchestration across a cluster are valuable.

### Related Ray Components

- **Ray Core:** General distributed tasks and actors.
- **Ray Train:** Distributed model training.
- **Ray Tune:** Hyperparameter tuning.
- **Ray Serve:** Model serving.

## 8. TensorFlow, Keras, and JAX Equivalents

| Framework | Data-parallel / distributed training | Model sharding / parallelism |
|---|---|---|
| PyTorch | `torch.nn.parallel.DistributedDataParallel` | FSDP; tensor- and pipeline-parallel approaches |
| TensorFlow | `tf.distribute.MirroredStrategy`; `MultiWorkerMirroredStrategy` across machines | DTensor APIs, including mesh and layout concepts |
| Keras 3 | `keras.distribution.DataParallel` | `keras.distribution.ModelParallel`, `DeviceMesh`, and `LayoutMap` |
| JAX | Distributed execution and sharding APIs | `Mesh`, `PartitionSpec`, and sharding APIs |

`tf.distribute` is TensorFlow's overall distributed-training API.

`MirroredStrategy` is a particular data-parallel strategy: each GPU keeps a full model replica.

DTensor provides lower-level tensor distribution capabilities. Keras 3 also offers higher-level distribution APIs. These APIs are related but are not interchangeable names for `tf.distribute`.

Exact API availability and recommended usage can vary by framework version.

## 9. Quick Comparison

| Tool | Primary role | Model-state sharding? |
|---|---|---|
| DDP | Data-parallel training with full replicas | No |
| FSDP | Data parallelism with sharded model state | Yes |
| DeepSpeed ZeRO | Memory-efficient distributed training and related optimizations | Yes, depending on ZeRO stage |
| Ray Train | Orchestration and management of distributed training jobs | Not by itself; uses a training backend |

### Key Distinctions to Remember

- **Ray versus DDP:** Ray manages resources and worker execution; DDP synchronizes gradients.
- **DDP versus FSDP:** DDP replicates model state; FSDP shards model state to save memory.
- **FSDP versus tensor parallelism:** FSDP shards training state; tensor parallelism splits individual model operations.
- **FSDP versus ZeRO-3:** Similar full-sharding goals, implemented in different systems.
- **ZeRO stages:** Progressively shard optimizer states, then gradients, then parameters.
- **Ray is optional:** It can orchestrate a distributed training job, but it does not replace the underlying training strategy.

## 10. JAX and Flax

Flax is a neural-network library commonly used with JAX. JAX emphasizes functional transformations and explicit model state.

A typical Flax training step:

1. Define the model.
2. Initialize and explicitly hold parameters.
3. Define a loss function.
4. Use `jax.value_and_grad` to calculate gradients.
5. Use an optimizer such as Optax to produce updated parameters and optimizer state.

JAX distributed training uses concepts such as:

- **Device mesh:** A logical arrangement of devices.
- **Sharding:** Rules describing where tensor data is placed.
- **`jax.jit`:** Compiles computations.
- **Collectives:** Communication such as All-Gather, All-Reduce, and Reduce-Scatter.

**Important:** `jax.jit` alone does not automatically distribute a model across GPUs. Distributed execution requires appropriate distributed setup, device mesh, and sharding configuration.

## 11. Concise Technical Summary

Distributed training has two layers.

The **orchestration layer** places workers on available machines and GPUs; Ray Train can provide this layer.

The **training layer** defines how model state and computation are distributed:

- **DDP** replicates models and synchronizes gradients.
- **FSDP and DeepSpeed ZeRO** shard training state to reduce memory.
- **TensorFlow** provides distributed-training strategies and tensor-sharding APIs.
- **JAX** provides mesh-based sharding and compiler-supported distributed execution.

All-Reduce, All-Gather, and Reduce-Scatter are collective communication operations used to exchange or combine data across workers.
