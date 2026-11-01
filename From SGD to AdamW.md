# From SGD to AdamW: How Optimization Algorithms Improve Neural Network Training

Training a neural network means repeatedly adjusting its weights to minimize a loss function. The optimizer determines how those weights are updated based on the gradients calculated through backpropagation.

Several popular optimizers have evolved to address limitations in earlier approaches. The progression from SGD to AdaGrad, Adam, and AdamW illustrates how optimization has become more adaptive and efficient.

All equations below use element-wise operations: each weight has its own gradient and, where applicable, accumulated history. Let $g_t = \nabla_W\mathcal{L}_t(W_t)$ be the gradient of the current mini-batch loss, $\alpha > 0$ the learning rate, and $\epsilon > 0$ a small stability constant. Initialize $G_0=m_0=v_0=0$ and start the step index at $t=1$. The memory coefficients satisfy $0 \leq \beta_1,\beta_2 < 1$; $\lambda \geq 0$ controls regularization strength.

## 1. Stochastic Gradient Descent (SGD)

SGD is one of the most fundamental optimization algorithms in deep learning.

Instead of calculating gradients using the entire training dataset, SGD typically uses a randomly selected mini-batch of examples to estimate the gradient and update the model's weights.

Its update rule is:

$$
W_{t+1}=W_t-\alpha g_t
$$

Where:

- $W_t$: Current model weights.
- $\alpha$: Learning rate.
- $g_t$: Gradient calculated from the current mini-batch.

**The limitation of SGD**

Traditional SGD generally uses the same learning rate for every parameter.

This creates two challenges:

1. Parameters with large gradients may take excessively large steps.
2. Parameters with small or infrequent gradients may learn too slowly.

A single global learning rate may not be suitable for every parameter, especially in models with many different features and gradient scales.

## 2. AdaGrad: Adaptive Learning Rates for Individual Parameters

AdaGrad improves SGD by assigning each parameter its own adaptive learning rate.

Instead of using one fixed learning rate for every parameter, AdaGrad tracks the historical squared gradients for each parameter.

Its accumulated squared-gradient value is:

$$
G_t=G_{t-1}+g_t^2
$$

The parameter update becomes:

$$
W_{t+1}=W_t-
\frac{\alpha}{\sqrt{G_t}+\epsilon}g_t
$$

Here, $G_t$ accumulates the squared gradients, and $\epsilon$ prevents division by zero.

**How does AdaGrad improve SGD?**

- Parameters that consistently receive large gradients get smaller effective learning rates.
- Parameters with smaller or infrequent gradients retain relatively larger effective learning rates.
- Different parameters can learn at different speeds.

This is particularly useful for sparse data, where some features appear much less frequently than others.

**The limitation of AdaGrad**

AdaGrad continuously accumulates squared gradients throughout training.

Because the accumulated value keeps increasing, the effective learning rate continually decreases.

Eventually, learning rates can become extremely small, preventing the model from making meaningful progress.

## 3. Adam: Fixing AdaGrad's Learning-Rate Decay Problem

Adam stands for Adaptive Moment Estimation.

It combines two ideas:

1. Momentum, which smooths gradient direction.
2. Adaptive learning rates, which adjust update sizes for individual parameters.

Unlike AdaGrad, Adam does not accumulate all historical squared gradients indefinitely. It uses exponential moving averages, giving more importance to recent gradients.

Adam maintains two moving averages.

**First moment: Average of gradients**

$$
m_t=\beta_1m_{t-1}+(1-\beta_1)g_t
$$

This smooths the gradient direction and helps reduce noisy updates.

**Second moment: Average of squared gradients**

$$
v_t=\beta_2v_{t-1}+(1-\beta_2)g_t^2
$$

This tracks the recent magnitude of gradients and helps determine the effective learning rate for each parameter.

Adam also applies bias correction to both moving averages:

$$
\hat m_t=\frac{m_t}{1-\beta_1^t}
$$

$$
\hat v_t=\frac{v_t}{1-\beta_2^t}
$$

The final parameter update is:

$$
W_{t+1}=W_t-
\alpha\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}
$$

**How does Adam improve AdaGrad?**

The key difference is how they handle historical squared gradients.

- AdaGrad accumulates squared gradients indefinitely.
- Adam uses an exponential moving average that gradually forgets older gradients.

This allows Adam to adapt to more recent gradient behavior instead of continuously shrinking learning rates based on the entire training history.

Adam also incorporates momentum, helping stabilize updates and accelerate progress in consistent directions.

These mechanisms can help Adam make progress on noisy gradients and uneven gradient scales. Performance still depends on the task and hyperparameter tuning.

## 4. AdamW: Fixing the Weight-Decay Problem in Adam

AdamW builds on Adam by changing how weight decay is applied.

First, what is weight decay?

Weight decay is a regularization technique that encourages model weights to remain small. It helps prevent overfitting by discouraging unnecessarily large parameter values.

**The problem with traditional Adam**

In the conventional coupled-L2 version of Adam, an L2 regularization term is added to the loss, giving the modified gradient:

$$
g_t'=g_t+\lambda W_t
$$

Adam then uses this combined gradient in both moving-average calculations. For adaptive optimizers, this L2 penalty is not equivalent to decoupled weight decay.

Consequently, the adaptive scaling also affects the weight-decay contribution. This can make the regularization behavior less predictable.

**How AdamW fixes the problem**

AdamW decouples weight decay from the gradient update.

It calculates the Adam moments using only the loss gradient $g_t$ and applies parameter shrinkage separately from those moment calculations.

The combined update equation is:

$$
W_{t+1}=
W_t-\alpha\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}
-\alpha\lambda W_t
$$

The final term represents the independent weight-decay operation.

This separation makes regularization more predictable and easier to tune.

AdamW is widely used in modern deep learning, particularly when training Transformer models and large language models.

## 5. Summary: The Evolution of Optimization

| Optimizer | Main improvement | Limitation addressed |
| --- | --- | --- |
| SGD | Mini-batch gradient updates | Baseline optimization |
| AdaGrad | Adaptive LR per parameter | One global LR in SGD |
| Adam | Momentum + adaptive LR | AdaGrad's shrinking LR |
| AdamW | Decoupled weight decay | Adam's coupled decay |

The overall evolution can be summarized as follows:

- **SGD:** Uses gradients to update model weights with a global learning rate.
- **AdaGrad:** Gives each parameter an adaptive learning rate based on accumulated squared gradients.
- **Adam:** Combines momentum with adaptive learning rates and uses moving averages to avoid AdaGrad's indefinitely accumulating history.
- **AdamW:** Improves Adam's regularization by applying weight decay independently of the gradient update.

**Practical takeaway:** SGD remains useful, particularly when carefully tuned. Adam is a popular general-purpose optimizer, while AdamW is often the preferred starting point for modern Transformer and LLM training.
