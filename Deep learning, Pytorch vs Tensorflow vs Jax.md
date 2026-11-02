# Deep Learning Frameworks: PyTorch, TensorFlow, and JAX

## 1. Overview

PyTorch, TensorFlow, and JAX are Python frameworks for building, training, and evaluating machine-learning models. They provide tensor operations, automatic differentiation, accelerator support, and tools for organizing training.

| Topic | PyTorch | TensorFlow | JAX |
|---|---|---|---|
| Core style | Imperative, Python-first | Eager execution plus graph execution | Functional, transformation-based |
| Computation graph | Commonly built dynamically as operations run; `torch.compile` can capture/compile parts | Eager by default; `tf.function` traces functions into graphs | Functions are transformed and compiled, commonly with `jax.jit` |
| Automatic differentiation | `torch.autograd` | `tf.GradientTape` | `jax.grad`, `jax.value_and_grad` |
| Typical optimizer API | `torch.optim` | Keras optimizers / TensorFlow APIs | Commonly Optax |
| Distributed training | DDP, FSDP, plus other ecosystem tools | `tf.distribute`, DTensor | Device meshes and explicit sharding APIs |
| Common strengths | Flexible model development and a broad research/production ecosystem | Integrated model APIs and graph-based deployment options | Composable transformations, compilation, and explicit accelerator sharding |

**Important nuance:** Saying “PyTorch and TensorFlow build graphs, while JAX is functional” is an oversimplification. PyTorch is typically imperative and builds autograd graphs dynamically during execution. TensorFlow supports both eager execution and traced graphs. JAX encourages pure functions and uses transformations such as `grad` and `jit`; it can trace functions into compiled computation graphs.

## 2. The Standard Model-Training Workflow

The broad training process is similar across all three frameworks:

1. **Prepare data** — load examples, preprocess features, create batches, and split training/validation/test sets.
2. **Define the model** — specify layers and the forward computation.
3. **Initialize parameters** — create the model's trainable weights.
4. **Choose a loss function** — measure the difference between predictions and targets.
5. **Choose an optimizer** — for example, SGD, Adam, or AdamW.
6. **Run the forward pass** — compute predictions from a batch.
7. **Calculate the loss** — evaluate the predictions against targets.
8. **Compute gradients** — use automatic differentiation to find how the loss changes with each parameter.
9. **Update parameters** — the optimizer uses gradients and its state to adjust weights.
10. **Repeat over batches and epochs** — monitor training and validation metrics.
11. **Evaluate and save** — evaluate on held-out data and save the model/checkpoint.

A compact view:

```text
Batch of data
    ↓
Forward pass → Predictions
    ↓
Loss function → Loss
    ↓
Automatic differentiation → Gradients
    ↓
Optimizer step → Updated parameters
    ↓
Next batch / next epoch
```

## 3. What the Training Loop Looks Like

### PyTorch

```python
for x, y in train_loader:
    optimizer.zero_grad()
    predictions = model(x)       # forward pass
    loss = loss_fn(predictions, y)
    loss.backward()               # compute gradients
    optimizer.step()              # update parameters
```

PyTorch typically records operations during the forward pass so `backward()` can compute gradients. `DistributedDataParallel` (DDP) synchronizes gradients across model replicas; FSDP can shard model states across GPUs.

### TensorFlow / Keras

```python
with tf.GradientTape() as tape:
    predictions = model(x, training=True)
    loss = loss_fn(y, predictions)

grads = tape.gradient(loss, model.trainable_variables)
optimizer.apply_gradients(zip(grads, model.trainable_variables))
```

Keras can also manage much of this loop through `model.compile(...)` and `model.fit(...)`. TensorFlow runs eagerly by default, while `tf.function` can trace a function into a graph for optimization and execution.

### JAX

```python
def loss_fn(params, x, y):
    predictions = model_apply(params, x)
    return loss_function(predictions, y)

loss, grads = jax.value_and_grad(loss_fn)(params, x, y)
updates, opt_state = optimizer.update(grads, opt_state, params)
params = optax.apply_updates(params, updates)
```

JAX code commonly passes parameters and optimizer state explicitly. `jax.grad` computes derivatives, `jax.jit` can compile functions, and `jax.vmap` can vectorize computations. Optax is a commonly used optimizer library. The exact code depends on the model library, such as Flax or another JAX-compatible library.

## 4. Graphs, Eager Execution, and Functional Programming

- **Eager execution:** Operations execute as Python code runs, which can make debugging straightforward.
- **Computation graph:** A representation of operations and their dependencies that a runtime can optimize or execute.
- **Dynamic autograd graph:** In typical PyTorch execution, the graph used for differentiation is recorded as operations run and can reflect Python control flow.
- **Traced/compiled graph:** TensorFlow `tf.function` and JAX `jit` trace functions and can compile their computation for more efficient execution.
- **Functional programming:** Functions are ideally expressed without hidden side effects; inputs determine outputs. JAX emphasizes this style and commonly passes parameters and state explicitly.

These concepts are related but distinct: a framework can support eager execution and graphs, and functional programming does not mean “no computation graph.”

## 5. Distributed Training

Distributed training uses multiple GPUs and/or machines to increase throughput or fit larger models.

- **Data parallelism:** Workers process different batches. In classic DDP, each GPU holds a full model replica and gradients are synchronized with All-Reduce.
- **Model-state sharding:** FSDP and DeepSpeed ZeRO distribute parameters, gradients, and/or optimizer states across workers to reduce per-GPU memory.
- **Tensor parallelism:** Splits individual operations, such as large matrix multiplications, across devices.
- **Pipeline parallelism:** Places different model layers or stages on different devices.
- **Ray Train:** Can schedule and coordinate training workers across a cluster; a backend such as DDP, FSDP, or DeepSpeed performs the corresponding training synchronization.

Framework examples:
- PyTorch: `DistributedDataParallel` (DDP), FSDP, and ecosystem tools such as DeepSpeed.
- TensorFlow: `tf.distribute` strategies and DTensor APIs.
- JAX: device meshes and explicit array sharding, with collectives such as All-Reduce, All-Gather, and Reduce-Scatter.

## 6. Choosing a Framework

- **Choose PyTorch** when you want a flexible, Python-first workflow and a broad ecosystem for research, model development, and production ML.
- **Choose TensorFlow/Keras** when its integrated model APIs, training workflow, or existing TensorFlow deployment stack fits the project.
- **Choose JAX** when functional transformations, compilation, vectorization, and explicit sharding are a good fit for the team and workload.

There is no universally best framework. The decision depends on model architecture, team experience, deployment requirements, hardware, distributed-training needs, and ecosystem compatibility.

## 7. Key Takeaways

1. All three frameworks support tensors, automatic differentiation, model training, and accelerator execution.
2. PyTorch is commonly imperative with a dynamically recorded autograd graph.
3. TensorFlow supports eager execution and graph execution through tools such as `tf.function`.
4. JAX emphasizes functional transformations such as `grad`, `jit`, and `vmap`; it can compile traced functions.
5. A training step follows the same core pattern: forward pass → loss → gradients → optimizer update.
6. Distributed orchestration and distributed model training are different concerns: Ray can coordinate workers, while a framework/backend implements the training strategy.
