# Deep Learning Loss Functions: Quick Reference

Popular loss functions for **TensorFlow/Keras, PyTorch, and JAX/Optax**, grouped by ML task.

## 1. Regression

Let $N$ be the number of examples, $y_i$ the target, $\hat{y}_i$ the prediction, and $e_i = y_i - \hat{y}_i$.

### MSE

$$
\mathcal{L}_{\mathrm{MSE}} = \frac{1}{N}\sum_{i=1}^{N}(y_i-\hat{y}_i)^2
$$

Strongly penalizes large errors; general regression.

### MAE

$$
\mathcal{L}_{\mathrm{MAE}} = \frac{1}{N}\sum_{i=1}^{N}\lvert y_i-\hat{y}_i\rvert
$$

Less sensitive to outliers than MSE; absolute-error regression.

### Huber

$$
\mathcal{L}_{\mathrm{Huber}} = \frac{1}{N}\sum_{i=1}^{N}h_{\delta}(e_i)
$$

$$
h_{\delta}(e) =
\begin{cases}
\frac{1}{2}e^2, & \lvert e\rvert \leq \delta, \\
\delta\left(\lvert e\rvert-\frac{1}{2}\delta\right), & \lvert e\rvert > \delta.
\end{cases}
$$

Quadratic for small errors and linear for large errors; balances smoothness and robustness to outliers. The threshold is $\delta > 0$.

### Log-Cosh

$$
\mathcal{L}_{\mathrm{LogCosh}} = \frac{1}{N}\sum_{i=1}^{N}\log\left(\cosh(\hat{y}_i-y_i)\right)
$$

Smooth and less sensitive to outliers than MSE.

### MSLE

$$
\mathcal{L}_{\mathrm{MSLE}} = \frac{1}{N}\sum_{i=1}^{N}\left[\log(1+y_i)-\log(1+\hat{y}_i)\right]^2
$$

Emphasizes relative errors; nonnegative targets/predictions.

### Quantile

$$
\mathcal{L}_{q} = \frac{1}{N}\sum_{i=1}^{N}\max\left(qe_i,(q-1)e_i\right), \qquad 0 < q < 1
$$

Predicts a chosen quantile; forecasting and prediction intervals.

### Library implementations

| Loss | TensorFlow / Keras | PyTorch | JAX / Optax |
| --- | --- | --- | --- |
| MSE | `MeanSquaredError` | `MSELoss` | `l2_loss`* |
| MAE | `MeanAbsoluteError` | `L1Loss` | `absolute_error` |
| Huber | `Huber` | `HuberLoss` | `huber_loss` |
| Log-Cosh | `LogCosh` | Custom | `log_cosh` |
| MSLE | `MeanSquaredLogarithmicError` | Custom | Custom |
| Quantile | Custom | Custom | `quantile_regression_loss` |

*Optax `l2_loss` returns element-wise half-squared error; average it and account for the factor of 1/2 if matching conventional MSE.

## 2. Classification

Let $p_{ic}$ be the predicted probability for class $c$, and $y_{ic}$ the one-hot target. The expressions below are for a single example; subscripts $i$ are omitted. Training losses are typically averaged over examples.

### Binary Cross-Entropy (BCE)

$$
\mathcal{L}_{\mathrm{BCE}} = -\left[y\log p+(1-y)\log(1-p)\right]
$$

Binary or multilabel classification. For multilabel tasks, apply this expression independently to each label and then reduce over labels.

### Categorical Cross-Entropy (CCE)

$$
\mathcal{L}_{\mathrm{CCE}} = -\sum_{c}y_c\log p_c
$$

Multiclass classification with one-hot labels.

### Sparse Categorical CE

$$
\mathcal{L}_{\mathrm{SparseCCE}} = -\log p_y
$$

Same objective as CCE, but the target $y$ is an integer class index rather than one-hot encoded.

### Focal Loss

$$
\mathcal{L}_{\mathrm{Focal}} = -\alpha_t(1-p_t)^{\gamma}\log(p_t)
$$

Here $p_t$ is the probability assigned to the true class, $\alpha_t$ is a class weight, and $\gamma \geq 0$ controls down-weighting of easy examples. Useful for class-imbalanced tasks and object detection.

### Hinge Loss

$$
\mathcal{L}_{\mathrm{Hinge}} = \max\left(0,1-yf(x)\right), \qquad y\in\{-1,+1\}
$$

Margin-based binary classification; SVM-style objectives.

### KL Divergence

$$
D_{\mathrm{KL}}(P\parallel Q) = \sum_{c}P_c\log\left(\frac{P_c}{Q_c}\right)
$$

Compares probability distributions; distillation and distribution matching.

**CCE vs. Sparse CCE:** The loss value, objective, and gradients are the same when labels represent the same class. Only the target encoding differs. Example: for probabilities `[0.1, 0.8, 0.1]` and true class Bird (zero-based index 2), both calculate $-\log(0.1) \approx 2.3026$. CCE target: `[0, 0, 1]`; sparse target: `2`.

### Library implementations

| Loss | TensorFlow / Keras | PyTorch | JAX / Optax |
| --- | --- | --- | --- |
| BCE | `BinaryCrossentropy` | `BCEWithLogitsLoss` | `sigmoid_binary_cross_entropy` |
| CCE | `CategoricalCrossentropy` | `CrossEntropyLoss`** | `softmax_cross_entropy` |
| Sparse CCE | `SparseCategoricalCrossentropy` | `CrossEntropyLoss`** | `softmax_cross_entropy_with_integer_labels` |
| Focal | `BinaryFocalCrossentropy` / `CategoricalFocalCrossentropy` | Custom | `sigmoid_focal_loss` |
| Hinge | `Hinge` | `MultiMarginLoss` / custom | `hinge_loss` |
| KL divergence | `KLDivergence` | `KLDivLoss` | `kl_divergence` |

**PyTorch `CrossEntropyLoss` expects integer class indices by default; it also supports probability targets. It combines log-softmax and negative log-likelihood. For BCE, use logits with `BCEWithLogitsLoss`; do not apply sigmoid first. In Keras, set `from_logits=True` when supplying logits.

## 3. Ranking and metric learning

### Pairwise Hinge / Margin Ranking

**Type:** Pairwise.

$$
\mathcal{L}_{\mathrm{PairwiseHinge}} = \max\left(0,m-y(s^{+}-s^{-})\right)
$$

Learn that a relevant item should score above an irrelevant item. Here $m$ is the margin and $y\in\{-1,+1\}$ specifies the desired ordering; for a relevant item scored as $s^{+}$ and an irrelevant item scored as $s^{-}$, use $y=+1$.

### Pairwise Logistic

**Type:** Pairwise.

$$
\mathcal{L}_{\mathrm{PairwiseLogistic}} = \log\left(1+\exp\left(-(s^{+}-s^{-})\right)\right)
$$

Search and recommendation ranking.

### Triplet Loss

**Type:** Triplet.

$$
\mathcal{L}_{\mathrm{Triplet}} = \max\left(0,d(a,p)-d(a,n)+m\right)
$$

Here $a$ is an anchor, $p$ a positive example, $n$ a negative example, $d$ a distance function, and $m$ a margin. Used for similarity learning, face/image retrieval, and embeddings.

### ListMLE

**Type:** Listwise.

Negative log-likelihood of the ground-truth permutation under a Plackett–Luce model. Optimizes an ordered list for search/recommendation.

### Softmax Ranking Loss

**Type:** Listwise.

Softmax cross-entropy over candidate item scores and relevance targets. Used for ranking a set of candidates.

### Library implementations

| Loss | TensorFlow / Keras | PyTorch | JAX / Optax |
| --- | --- | --- | --- |
| Pairwise hinge / margin | TF-Ranking (pairwise losses) | `MarginRankingLoss` | Custom |
| Pairwise logistic | TF-Ranking | Custom | Custom |
| Triplet | TF-Ranking / custom | `TripletMarginLoss` | `optax.losses` triplet-related APIs vary by version; custom if unavailable |
| ListMLE | TF-Ranking `ListMLELoss` | Custom | Custom |
| Softmax ranking | TF-Ranking `SoftmaxLoss` | Custom | Custom |

Ranking APIs vary more across versions and libraries than standard regression/classification losses. Check the installed version's documentation; many ranking objectives require custom code or a specialist library such as TensorFlow Ranking.

## 4. Practical starting points

- **Regression:** MSE as a baseline; Huber when outliers matter; quantile loss for interval/quantile forecasts.
- **Classification:** BCE for binary/multilabel; categorical or sparse categorical CE for multiclass; focal loss for severe imbalance.
- **Ranking:** Pairwise logistic or margin ranking for ordering pairs; triplet loss for embedding similarity; ListMLE for listwise ranking.

**Loss vs. metric:** A loss is the training objective. Evaluation metrics measure task performance—for example, RMSE/MAE (regression), precision/recall/F1 (classification), and NDCG@K/MRR/MAP (ranking).
