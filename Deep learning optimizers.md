# Eight Popular Deep Learning Optimizers

A compact reference to eight established, widely available optimizers—not a measured popularity ranking. SGD includes its commonly used momentum variant.

## What an optimizer does

An optimizer updates model parameters to reduce a loss. **Backpropagation computes gradients; the optimizer uses them to choose the update.**

Let $\theta_t$ be the parameters before step $t$, $g_t = \nabla_{\theta}\mathcal{L}_t(\theta_t)$ the current minibatch gradient, and $\eta_t$ the learning rate. A minibatch is a small subset of training examples.

## Important elements and settings

Think of training as walking downhill on a landscape whose height is the loss.

| Element | Meaning | Setting / effect |
| --- | --- | --- |
| **Gradient** | Local uphill direction and steepness; computed by backpropagation. | An input to every optimizer, not a setting you choose. Updates generally move against it. |
| **Learning rate** | Overall step size. | $\eta$: too large can cause instability; too small can slow learning. |
| **Momentum** | Remembers past gradient directions, smoothing noise and reducing zigzagging. | SGD's $\mu$ or Adam's $\beta_1$: larger values retain direction longer but react more slowly. Their update conventions differ. |
| **Adaptive scaling** | Uses gradient history to adjust the effective step size separately for each parameter. | Adam's $\beta_2$ or RMSprop's $\rho$ controls squared-gradient memory; larger values give steadier but slower-changing scaling. |
| **Weight decay** | Encourages smaller weights to help control overfitting. | $\lambda$: stronger decay means stronger regularization. Its interaction with gradients depends on the implementation. |
| **Bias correction** | Compensates for moving averages starting at zero, especially in early steps. | Built into Adam and its variants; normally no separate knob. This is unrelated to a neural network's bias parameters. |
| **Epsilon** | Small denominator constant for numerical stability. | $\epsilon$: prevents division by zero and also affects scaling when gradients are very small. |
| **Learning-rate schedule** | Changes the learning rate over training. | Warmup raises it initially; decay lowers it later. A schedule works alongside an optimizer. |

An **exponential moving average (EMA)** blends old and new values, favoring recent observations. Its generic form is $a_t=\beta a_{t-1}+(1-\beta)x_t$. A larger $\beta$ means longer memory. For example, $0.9$ remembers roughly 10 steps and $0.999$ roughly 1,000, using the approximation $1/(1-\beta)$.

**What you choose vs. what is computed:** Learning rate, memory coefficients, weight decay, epsilon, and the schedule are hyperparameters. Gradients, moving-average states, adaptive scaling factors, and bias corrections are computed during training.

## The eight optimizers

| Optimizer | How it works | Main difference / tradeoff |
| --- | --- | --- |
| **1. [SGD](https://keras.io/api/optimizers/sgd/)** — stochastic gradient descent | Subtracts the gradient times the learning rate. Optional momentum accumulates past gradient directions; Nesterov momentum adds an anticipatory correction. | Simple and memory efficient. Uses one global learning rate without adaptive parameter scaling; often needs careful tuning and a schedule. |
| **2. [AdaGrad](https://keras.io/api/optimizers/adagrad/)** — adaptive gradient | Divides each gradient by the square root of the sum of its past squared gradients. | Infrequently updated parameters retain larger effective learning rates, useful for sparse features. Accumulation can make steps too small during long training. |
| **3. [RMSprop](https://keras.io/api/optimizers/rmsprop/)** — root mean square propagation | Divides gradients by the root of an EMA of squared gradients. | Forgets old gradients, avoiding AdaGrad's permanently growing denominator. Basic RMSprop has adaptive scaling without momentum; momentum is optional. |
| **4. [AdaDelta](https://keras.io/api/optimizers/adadelta/)** | Tracks EMAs of squared gradients and squared parameter updates; their root-ratio sets the update scale. | Also avoids unlimited accumulation, but uses past update sizes as well as gradients. Original AdaDelta needs no separate global learning rate; libraries may expose a multiplier. |
| **5. [Adam](https://keras.io/api/optimizers/adam/)** — adaptive moment estimation | Combines an EMA of gradients with an EMA of squared gradients, correcting their initial bias toward zero. | Adds momentum-like direction smoothing to adaptive scaling. A useful general baseline, with two state values per parameter. |
| **6. [AdamW](https://arxiv.org/abs/1711.05101)** — Adam with decoupled weight decay | Performs Adam's update and separately shrinks parameters toward zero. | Separates regularization from gradient scaling. Unlike adding an L2 penalty to conventional Adam's loss, decay does not enter the moment estimates. |
| **7. [Adamax](https://keras.io/api/optimizers/adamax/)** | Keeps Adam's gradient EMA but replaces its root-mean-square denominator with an exponentially weighted maximum absolute gradient. | Uses an infinity-norm-based scale. An alternative adaptive scaling rule, with no universal advantage over Adam. |
| **8. [Nadam](https://keras.io/api/optimizers/Nadam/)** — Nesterov-accelerated Adam | Combines Adam's adaptive scaling with Nesterov-style momentum. | Changes Adam's direction update to include an anticipatory correction; does not inherently add AdamW's decoupled decay. |

## Compare four common choices

SGD with momentum, RMSprop, Adam, and AdamW are four prominent choices from this reference; this is not a measured top-four ranking. The comparison uses ordinary momentum SGD, uncentered RMSprop, conventional Adam, and AdamW.

| Element | SGD + momentum | RMSprop | Adam | AdamW |
| --- | --- | --- | --- | --- |
| **Gradient** | Current minibatch gradient | Same | Same | Same |
| **Learning rate** | One global rate; no adaptive scaling | Global rate plus per-parameter scaling | Global rate plus per-parameter scaling | Same as Adam |
| **Direction memory** | Momentum $\mu$ | Optional momentum | Gradient EMA, $\beta_1$ | Same as Adam |
| **Scale memory** | None | Squared-gradient EMA, $\rho$ (PyTorch: `alpha`) | Squared-gradient EMA, $\beta_2$ | Same as Adam |
| **Bias correction** | No Adam-style correction | No Adam-style correction | Corrects both EMAs | Same as Adam |
| **Epsilon** | Not needed in the basic update | Stabilizes denominator | Stabilizes denominator | Same as Adam |
| **Weight decay** | Commonly coupled as an L2 gradient term | Commonly coupled as an L2 gradient term | Conventional coupled L2 changes both moment estimates | Decoupled shrinkage leaves moment estimates unchanged |
| **Schedule** | Supported; often important | Supported | Supported | Supported |
| **History per parameter** | 1 value | 1; 2 with momentum | 2 values | 2 values |

**Implementation note:** The decay row follows the usual [PyTorch SGD](https://docs.pytorch.org/docs/stable/generated/torch.optim.SGD.html), [RMSprop](https://docs.pytorch.org/docs/stable/generated/torch.optim.RMSprop.html), [Adam](https://docs.pytorch.org/docs/stable/generated/torch.optim.Adam.html), and [AdamW](https://docs.pytorch.org/docs/stable/generated/torch.optim.AdamW.html) conventions. Libraries can offer other decay modes; PyTorch Adam can also enable decoupled decay. State counts exclude parameters, gradients, and temporary buffers.

**AdamW in five parts:** $\eta$ controls overall step size; $\beta_1$ controls direction memory; $\beta_2$ controls scale memory; $\lambda$ controls weight shrinkage; $\epsilon$ stabilizes division. Tune learning rate and weight decay first, then investigate the other settings if needed.

## Key equations

All products, squares, square roots, and divisions below act element by element. State starts at zero; $t$ starts at 1. The constant $\epsilon > 0$ prevents division by zero, and $0 \leq \beta_1,\beta_2 < 1$ control memory.

**Plain SGD:**

$$
\theta_{t+1}=\theta_t-\eta_t g_t
$$

**Adam:** gradient memory $m_t$, squared-gradient memory $v_t$, and bias-corrected estimates:

$$
\begin{aligned}
m_t &= \beta_1m_{t-1}+(1-\beta_1)g_t, \\
v_t &= \beta_2v_{t-1}+(1-\beta_2)g_t^2, \\
\hat m_t &= \frac{m_t}{1-\beta_1^t}, \qquad
\hat v_t = \frac{v_t}{1-\beta_2^t}, \\
\theta_{t+1} &= \theta_t-\eta_t\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}.
\end{aligned}
$$

**AdamW:** use the same moments, calculated from the loss gradient, with weight-decay strength $\lambda \geq 0$:

$$
\theta_{t+1}=(1-\eta_t\lambda)\theta_t-\eta_t\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}
$$

## Remember the differences

- **Direction:** SGD follows the current gradient; momentum smooths direction; Nadam adds Nesterov-style anticipation.
- **Scale:** AdaGrad remembers all squared gradients; RMSprop remembers recent ones; AdaDelta also remembers update sizes; Adam combines direction and scale; Adamax changes the scale statistic.
- **Regularization:** AdamW separates weight decay from Adam's adaptive update. With zero decay, AdamW and Adam have the same core update.
- **Memory:** Plain SGD stores no per-parameter history; momentum SGD, AdaGrad, and basic RMSprop store one value per parameter; AdaDelta, Adam, AdamW, Adamax, and Nadam typically store two. This excludes parameters, gradients, and implementation overhead.

**Practical choice:** Compare AdamW and SGD with momentum as starting baselines; consider AdaGrad for sparse features. Tune the learning rate for each optimizer and compare validation results. Adaptive scaling does not remove the need for learning-rate tuning or scheduling. See the [PyTorch optimizer guide](https://docs.pytorch.org/cppdocs/api/optim/index.html) for common use cases.
