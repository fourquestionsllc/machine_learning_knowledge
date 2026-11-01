# Deep Learning Loss Functions: Quick Reference

Popular loss functions for **TensorFlow/Keras, PyTorch, and JAX/Optax**,
grouped by ML task.

## 1. Regression

  -------------------------------------------------------------------------------------------------------------------------------------------------
  Loss                    Formula / principle                                                                               Characteristics & use
                                                                                                                            cases
  ----------------------- ------------------------------------------------------------------------------------------------- -----------------------
  **MSE**                 (`\frac{1}{N}`{=tex}`\sum`{=tex}\_i(y_i-`\hat `{=tex}y_i)\^2)                                     Strongly penalizes
                                                                                                                            large errors; general
                                                                                                                            regression.

  **MAE**                 (`\frac{1}{N}`{=tex}`\sum`{=tex}\_i                                                               y_i-`\hat `{=tex}y_i

  **Huber**               (`\frac{1}{N}`{=tex}`\sum`{=tex}*i h*`\delta`{=tex}(e_i)), where                                  e
                          (h\_`\delta`{=tex}(e)=`\frac`{=tex}12e\^2) if (                                                   

  **Log-Cosh**            (`\frac{1}{N}`{=tex}`\sum`{=tex}\_i`\log`{=tex}(`\cosh`{=tex}(`\hat `{=tex}y_i-y_i)))             Smooth and less
                                                                                                                            sensitive to outliers
                                                                                                                            than MSE.

  **MSLE**                (`\frac{1}{N}`{=tex}`\sum`{=tex}\_i\[`\log`{=tex}(1+y_i)-`\log`{=tex}(1+`\hat `{=tex}y_i)\]\^2)   Emphasizes relative
                                                                                                                            errors; nonnegative
                                                                                                                            targets/predictions.

  **Quantile**            (`\frac{1}{N}`{=tex}`\sum`{=tex}\_i`\max`{=tex}(qe_i,(q-1)e_i)), (e_i=y_i-`\hat `{=tex}y_i)       Predicts a chosen
                                                                                                                            quantile; forecasting
                                                                                                                            and prediction
                                                                                                                            intervals.
  -------------------------------------------------------------------------------------------------------------------------------------------------

  Loss       TensorFlow / Keras              PyTorch       JAX / Optax
  ---------- ------------------------------- ------------- ----------------------------
  MSE        `MeanSquaredError`              `MSELoss`     `l2_loss`\*
  MAE        `MeanAbsoluteError`             `L1Loss`      `absolute_error`
  Huber      `Huber`                         `HuberLoss`   `huber_loss`
  Log-Cosh   `LogCosh`                       Custom        `log_cosh`
  MSLE       `MeanSquaredLogarithmicError`   Custom        Custom
  Quantile   Custom                          Custom        `quantile_regression_loss`

\*Optax `l2_loss` returns element-wise half-squared error; average it
and account for the factor of 1/2 if matching conventional MSE.

## 2. Classification

Let (p\_{ic}) be the predicted probability for class (c), and (y\_{ic})
the one-hot target.

  --------------------------------------------------------------------------------------------------------------
  Loss                    Formula / principle                                            Characteristics & use
                                                                                         cases
  ----------------------- -------------------------------------------------------------- -----------------------
  **Binary Cross-Entropy  (-\[y`\log `{=tex}p+(1-y)`\log`{=tex}(1-p)\])                  Binary or multilabel
  (BCE)**                                                                                classification.

  **Categorical           (-`\sum`{=tex}\_c y_c`\log `{=tex}p_c)                         Multiclass
  Cross-Entropy (CCE)**                                                                  classification with
                                                                                         one-hot labels.

  **Sparse Categorical    (-`\log `{=tex}p_y)                                            Same objective as CCE,
  CE**                                                                                   but target is an
                                                                                         integer class index
                                                                                         rather than one-hot
                                                                                         encoded.

  **Focal Loss**          (-`\alpha`{=tex}\_t(1-p_t)\^`\gamma`{=tex}`\log`{=tex}(p_t))   Down-weights easy
                                                                                         examples;
                                                                                         class-imbalanced tasks
                                                                                         and object detection.

  **Hinge Loss**          (`\max`{=tex}(0,1-yf(x))), (y`\in`{=tex}{-1,+1})               Margin-based binary
                                                                                         classification;
                                                                                         SVM-style objectives.

  **KL Divergence**       (`\sum`{=tex}\_c P_c`\log`{=tex}(P_c/Q_c))                     Compares probability
                                                                                         distributions;
                                                                                         distillation and
                                                                                         distribution matching.
  --------------------------------------------------------------------------------------------------------------

**CCE vs. Sparse CCE:** The loss value, objective, and gradients are the
same when labels represent the same class. Only the target encoding
differs. Example: for probabilities `[0.1, 0.8, 0.1]` and true class
Bird (index 2), both calculate `-log(0.1) = 2.3026`. CCE target:
`[0, 0, 1]`; sparse target: `2`.

  --------------------------------------------------------------------------------------------------------------------------
  Loss              TensorFlow / Keras                PyTorch                  JAX / Optax
  ----------------- --------------------------------- ------------------------ ---------------------------------------------
  BCE               `BinaryCrossentropy`              `BCEWithLogitsLoss`      `sigmoid_binary_cross_entropy`

  CCE               `CategoricalCrossentropy`         `CrossEntropyLoss`\*\*   `softmax_cross_entropy`

  Sparse CCE        `SparseCategoricalCrossentropy`   `CrossEntropyLoss`\*\*   `softmax_cross_entropy_with_integer_labels`

  Focal             `BinaryFocalCrossentropy` /       Custom                   `sigmoid_focal_loss`
                    `CategoricalFocalCrossentropy`                             

  Hinge             `Hinge`                           `MultiMarginLoss` /      `hinge_loss`
                                                      custom                   

  KL divergence     `KLDivergence`                    `KLDivLoss`              `kl_divergence`
  --------------------------------------------------------------------------------------------------------------------------

\*\*PyTorch `CrossEntropyLoss` expects integer class indices by default;
it also supports probability targets. It combines log-softmax and
negative log-likelihood. For BCE, use logits with `BCEWithLogitsLoss`;
do not apply sigmoid first. In Keras, set `from_logits=True` when
supplying logits.

## 3. Ranking and metric learning

  ---------------------------------------------------------------------------------------------------------
  Loss              Type              Formula / principle                          Common use cases
  ----------------- ----------------- -------------------------------------------- ------------------------
  **Pairwise Hinge  Pairwise          (`\max`{=tex}(0,m-y(s^+-s^-)))               Learn that a relevant
  / Margin                                                                         item should score above
  Ranking**                                                                        an irrelevant item.

  **Pairwise        Pairwise          (`\log`{=tex}(1+`\exp`{=tex}(-(s^+-s^-))))   Search and
  Logistic**                                                                       recommendation ranking.

  **Triplet Loss**  Triplet           (`\max`{=tex}(0,d(a,p)-d(a,n)+m))            Similarity learning,
                                                                                   face/image retrieval,
                                                                                   embeddings.

  **ListMLE**       Listwise          Negative log-likelihood of the ground-truth  Optimize an ordered list
                                      permutation under a Plackett--Luce model     for
                                                                                   search/recommendation.

  **Softmax Ranking Listwise          Softmax cross-entropy over candidate item    Ranking a set of
  Loss**                              scores and relevance targets                 candidates.
  ---------------------------------------------------------------------------------------------------------

  ---------------------------------------------------------------------------
  Loss              TensorFlow /      PyTorch               JAX / Optax
                    Keras                                   
  ----------------- ----------------- --------------------- -----------------
  Pairwise hinge /  TF-Ranking        `MarginRankingLoss`   Custom
  margin            (pairwise losses)                       

  Pairwise logistic TF-Ranking        Custom                Custom

  Triplet           TF-Ranking /      `TripletMarginLoss`   `optax.losses`
                    custom                                  triplet-related
                                                            APIs vary by
                                                            version; custom
                                                            if unavailable

  ListMLE           TF-Ranking        Custom                Custom
                    `ListMLELoss`                           

  Softmax ranking   TF-Ranking        Custom                Custom
                    `SoftmaxLoss`                           
  ---------------------------------------------------------------------------

Ranking APIs vary more across versions and libraries than standard
regression/classification losses. Check the installed version's
documentation; many ranking objectives require custom code or a
specialist library such as TensorFlow Ranking.

## 4. Practical starting points

-   **Regression:** MSE as a baseline; Huber when outliers matter;
    quantile loss for interval/quantile forecasts.
-   **Classification:** BCE for binary/multilabel; categorical or sparse
    categorical CE for multiclass; focal loss for severe imbalance.
-   **Ranking:** Pairwise logistic or margin ranking for ordering pairs;
    triplet loss for embedding similarity; ListMLE for listwise ranking.

**Loss vs. metric:** A loss is the training objective. Evaluation
metrics measure task performance---for example, RMSE/MAE (regression),
precision/recall/F1 (classification), and NDCG@K/MRR/MAP (ranking).
