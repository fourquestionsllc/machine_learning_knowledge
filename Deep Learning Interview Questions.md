# Deep Learning Interview Reference: 16 Essential Questions

A practical reference covering core deep learning concepts, common
interview explanations, key equations, advantages, limitations, and
follow-up points.

------------------------------------------------------------------------

## 1. Vanishing and Exploding Gradients

**Question:** What are vanishing and exploding gradients? Why do they
happen, and how can they be addressed?

### Vanishing gradients

During backpropagation, gradients are multiplied through many layers or
time steps. If these multipliers are repeatedly smaller than 1, the
gradient can shrink toward zero.

**Consequences** - Earlier layers learn very slowly or stop making
meaningful progress. - In recurrent networks, information from distant
time steps becomes difficult to learn. - Training deep networks can
become difficult.

### Exploding gradients

If repeated multipliers are frequently larger than 1, gradients can grow
rapidly.

**Consequences** - Very large parameter updates. - Unstable loss values
or divergence. - Numerical overflow in extreme cases.

### Common solutions

-   **ReLU-family activations:** Reduce saturation-related vanishing
    gradients compared with sigmoid or tanh in many hidden layers.
-   **Careful initialization:** Xavier initialization is commonly used
    with tanh-like activations; He initialization is commonly used with
    ReLU-family activations.
-   **Normalization:** Batch normalization or layer normalization can
    improve training stability.
-   **Gradient clipping:** Limits gradient magnitude to help prevent
    exploding gradients.
-   **Residual connections:** Provide shorter paths for information and
    gradients.
-   **LSTM and GRU:** Gating mechanisms help recurrent networks preserve
    useful information over longer sequences.

**Interview takeaway:** Vanishing gradients make learning earlier layers
or distant time steps difficult; exploding gradients make updates
unstable. Initialization, architecture, normalization, and clipping
address different aspects of the problem.

------------------------------------------------------------------------

## 2. Weight Initialization: Xavier vs. He

**Question:** Why is weight initialization important? What is the
difference between Xavier and He initialization?

Neural networks begin with randomly initialized weights. If the initial
weights are poorly scaled, activations or gradients may become too small
or too large as they pass through layers.

### Xavier (Glorot) initialization

Designed to keep the scale of activations and gradients reasonably
stable across layers. It is commonly paired with tanh or sigmoid-like
activations.

For a uniform distribution:

\[ W
`\sim `{=tex}U`\left`{=tex}(-`\sqrt{\frac{6}{n_{in}+n_{out}}}`{=tex},`\sqrt{\frac{6}{n_{in}+n_{out}}}`{=tex}`\right`{=tex})
\]

### He (Kaiming) initialization

Designed for ReLU-family activations, which set negative inputs to zero.

A common normal form is:

\[ W
`\sim `{=tex}`\mathcal{N}`{=tex}`\left`{=tex}(0,`\frac{2}{n_{in}}`{=tex}`\right`{=tex})
\]

Here, (n\_{in}) is the number of input connections and (n\_{out}) is the
number of output connections.

**Rule of thumb** - Xavier: tanh and related activations. - He: ReLU and
many ReLU variants.

**Interview takeaway:** Initialization controls the scale of signals at
the start of training and helps avoid unstable activation and gradient
magnitudes.

------------------------------------------------------------------------

## 3. Gradient Descent, SGD, Mini-Batches, and Adam

**Question:** What are the differences between Batch Gradient Descent,
Stochastic Gradient Descent, Mini-Batch Gradient Descent, and Adam?

The goal of an optimizer is to update model parameters to reduce the
loss.

### Batch Gradient Descent

Computes the gradient using the entire training dataset for each update.

-   Stable gradient estimate.
-   Can be expensive for large datasets.
-   Usually requires processing the full dataset before each update.

### Stochastic Gradient Descent (SGD)

Computes an update using one training example at a time.

-   Frequent, inexpensive updates.
-   Noisy gradient estimates.
-   Noise can sometimes help optimization escape shallow local
    structures or saddle regions.

### Mini-Batch Gradient Descent

Computes each update using a small batch of examples.

-   Efficient matrix operations on modern hardware.
-   Less noisy than single-example SGD.
-   The standard approach in many deep learning workflows.

### Adam

Adam combines momentum-like averaging of gradients with adaptive scaling
based on squared gradients.

Let (g_t) be the gradient at step (t):

\[ m_t=`\beta`{=tex}*1m*{t-1}+(1-`\beta`{=tex}\_1)g_t \]

\[ v_t=`\beta`{=tex}*2v*{t-1}+(1-`\beta`{=tex}\_2)g_t\^2 \]

After bias correction, the parameter update is approximately:

\[
`\theta`{=tex}*t=`\theta`{=tex}*{t-1}-`\eta`{=tex}`\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}`{=tex}
\]

**Practical comparison** - SGD: simple and often effective with careful
learning-rate schedules. - Mini-batch SGD: the usual data-processing
approach. - Adam: often a strong default, especially when parameters
benefit from different effective learning rates.

**Interview takeaway:** Mini-batches describe how training examples are
grouped for gradient calculation; Adam and SGD describe how parameters
are updated.

------------------------------------------------------------------------

## 4. Overfitting and Regularization

**Question:** What is overfitting, and how can it be prevented?

Overfitting occurs when a model performs well on training data but
generalizes poorly to unseen data. It may learn noise or accidental
patterns in the training set.

### Common prevention techniques

**Dropout** - Randomly sets a fraction of activations to zero during
training. - Reduces reliance on particular neurons. - Dropout is
disabled during ordinary inference, with appropriate scaling handled by
the framework.

**L1 regularization** Adds a penalty proportional to the absolute values
of weights:

\[
L\_{`\text{total}`{=tex}}=L\_{`\text{task}`{=tex}}+`\lambda`{=tex}`\sum`{=tex}\_i
\|w_i\| \]

It can encourage sparse weights.

**L2 regularization / weight decay** Adds a penalty based on squared
weight magnitudes:

\[
L\_{`\text{total}`{=tex}}=L\_{`\text{task}`{=tex}}+`\lambda`{=tex}`\sum`{=tex}\_i
w_i\^2 \]

It discourages excessively large weights. Decoupled weight decay, as
used in AdamW, is not identical to adding an L2 penalty inside every
optimizer.

**Early stopping** - Monitor a validation metric, often validation
loss. - Stop when it has not improved for a chosen patience period. -
Restore the checkpoint with the best validation performance.

**Data augmentation** Creates varied training examples through
transformations that preserve the label, such as image crops, flips, or
rotations when appropriate.

**Other approaches** - Collect more representative data. - Reduce model
capacity. - Use cross-validation when appropriate. - Check for data
leakage.

**Interview takeaway:** Training data is used to fit the model;
validation data guides model selection and early stopping; the test set
is reserved for final evaluation.

------------------------------------------------------------------------

## 5. Batch Normalization

**Question:** What is batch normalization, how does it work, and why is
it useful?

Batch normalization normalizes activations using statistics calculated
from a mini-batch during training.

For an activation (x), a simplified description is:

\[ `\hat{x}`{=tex}=`\frac{x-\mu_B}{\sqrt{\sigma_B^2+\epsilon}}`{=tex} \]

Then it applies learnable scale and shift parameters:

\[ y=`\gamma`{=tex}`\hat{x}`{=tex}+`\beta`{=tex} \]

-   (`\mu`{=tex}\_B): mini-batch mean.
-   (`\sigma`{=tex}\_B\^2): mini-batch variance.
-   (`\gamma`{=tex}): learnable scale.
-   (`\beta`{=tex}): learnable shift.

### Why it helps

-   Often makes optimization more stable.
-   Can allow higher learning rates.
-   Can improve convergence in many architectures.
-   Mini-batch statistics introduce some noise, which can have a mild
    regularizing effect.

### Training vs. inference

During training, batch statistics are used and running estimates are
typically maintained. During inference, the stored running statistics
are generally used.

### Important distinction

Batch normalization does not directly guarantee that gradients will not
vanish. It can help keep activations in useful ranges, but its effects
depend on the architecture and activation function.

**Interview takeaway:** Batch normalization normalizes intermediate
activations and then restores flexibility through learned scale and
shift parameters.

------------------------------------------------------------------------

## 6. CNN vs. RNN

**Question:** What are CNNs and RNNs, and when are they used?

### Convolutional Neural Networks (CNNs)

CNNs use learned filters to detect local patterns.

Common applications: - Image classification. - Object detection. - Image
segmentation. - Video and spatial signal processing.

A convolutional layer applies filters across local regions. Early layers
may learn edges and textures; deeper layers can combine them into more
complex features.

### Recurrent Neural Networks (RNNs)

RNNs process sequential inputs one time step at a time. They maintain a
hidden state that carries information forward.

A simplified recurrence is:

\[ h_t=f(W_xx_t+W_hh\_{t-1}+b) \]

Where: - (x_t): input at time (t). - (h_t): hidden state. - (W_x,W_h):
learned weight matrices. - (f): activation function.

Common applications: - Time-series modeling. - Speech and language
tasks. - Sequential sensor data.

### Main difference

CNNs are especially effective at learning local spatial patterns. RNNs
explicitly maintain a recurrent state to process sequential information.
Modern sequence tasks often use Transformers instead of recurrent
networks.

------------------------------------------------------------------------

## 7. LSTM vs. GRU

**Question:** What problem do LSTM and GRU solve? How do their gates
differ?

Standard RNNs can struggle with long-term dependencies because gradients
may vanish as they are propagated through many time steps.

### LSTM: Long Short-Term Memory

LSTM maintains two states:

-   **Cell state:** A memory pathway that carries information across
    time.
-   **Hidden state:** The current output representation, also passed to
    the next time step.

It has three main gates:

1.  **Forget gate:** Controls how much of the previous cell state to
    retain.
2.  **Input gate:** Controls how much new candidate information to write
    into the cell state.
3.  **Output gate:** Controls how much of the cell state to expose as
    the hidden state.

### GRU: Gated Recurrent Unit

GRU has: - One hidden state; no separate cell state. - **Update gate:**
Controls the balance between retaining previous information and
incorporating new candidate information. - **Reset gate:** Controls how
much previous hidden-state information is used when computing the
candidate state.

### Comparison

-   LSTM has three gates and separate cell and hidden states.
-   GRU has two gates and one hidden state.
-   GRU often has fewer parameters and may train faster.
-   Neither is universally better; performance depends on the data,
    task, and implementation.

**Interview takeaway:** Both architectures use gates to control
information flow and help model longer dependencies than a basic RNN.

------------------------------------------------------------------------

## 8. Attention: Query, Key, and Value

**Question:** What is attention, and what do Query, Key, and Value
represent?

Attention lets a token gather information from other tokens according to
their relevance.

Each input representation is projected into three vectors:

\[ Q=XW_Q,`\quad `{=tex}K=XW_K,`\quad `{=tex}V=XW_V \]

-   **Query (Q):** What the current position is looking for.
-   **Key (K):** Features used to determine how well another position
    matches the query.
-   **Value (V):** Information that is combined after attention weights
    are calculated.

Scaled dot-product attention is:

\[ `\operatorname{Attention}`{=tex}(Q,K,V)=
`\operatorname{softmax}`{=tex}`\left`{=tex}(`\frac{QK^T}{\sqrt{d_k}}`{=tex}`\right`{=tex})V
\]

The steps are:

1.  Compare queries with keys using dot products.
2.  Scale the scores by the square root of the key dimension.
3.  Apply softmax to produce attention weights.
4.  Use those weights to calculate a weighted sum of value vectors.

### Important clarification

Keys and values are usually derived from the same input representations,
but through different learned projection matrices. They are not
generally the same vectors.

**Interview takeaway:** Queries and keys determine where to attend;
values provide the information that gets mixed together.

------------------------------------------------------------------------

## 9. Multi-Head Attention

**Question:** Why use multiple attention heads instead of one?

Multi-head attention runs several attention operations in parallel. Each
head has its own learned projections for queries, keys, and values.

Different heads can learn different relationships, such as: - Local
relationships between neighboring tokens. - Long-range dependencies. -
Connections between pronouns and possible references. - Relationships
between verbs and their objects. - Other syntactic or semantic patterns.

The head outputs are concatenated and projected to produce the final
representation.

**Why it helps** - Allows different representation subspaces to be
learned. - Captures multiple relationships in parallel. - Provides a
richer contextual representation than a single attention operation may
provide.

**Interview takeaway:** Multiple heads allow the model to attend to
different types of relationships simultaneously. It is not guaranteed
that each head learns a clean, human-interpretable linguistic role.

------------------------------------------------------------------------

## 10. Transformer Architecture

**Question:** What are the main components of a Transformer?

The original Transformer architecture uses attention rather than
recurrence as its central sequence-processing mechanism.

### Main components

**Token embeddings** Convert token IDs into learned vector
representations.

**Positional information** Provides information about token order. The
original Transformer uses sinusoidal positional encodings; other models
use learned or relative positional methods.

**Multi-head attention** Allows tokens to gather contextual information
from other positions.

**Feed-forward network** Applies a nonlinear transformation
independently at each sequence position.

**Residual connections** Add a sublayer's input to its output,
supporting information and gradient flow.

**Layer normalization** Normalizes hidden representations to support
stable optimization.

### Encoder

The original encoder layer contains self-attention and a feed-forward
network, with residual connections and normalization.

### Decoder

The original decoder includes: - Masked self-attention over previously
generated positions. - Cross-attention to encoder outputs. - A
feed-forward network.

**Why Transformers are effective** - Sequence positions can be processed
in parallel during training, subject to attention masks. - Attention
provides direct paths between distant tokens. - The architecture scales
effectively with data and compute.

**Limitation:** Standard full self-attention has quadratic complexity in
sequence length for its attention-score computation and memory, which
can be costly for long contexts.

------------------------------------------------------------------------

## 11. Encoder-Only vs. Decoder-Only vs. Encoder-Decoder

**Question:** What are the three common Transformer model architectures?

### Encoder-only: BERT

-   Uses bidirectional self-attention.
-   Builds contextual representations using information from both left
    and right context.
-   Commonly used for classification, extraction, and language
    understanding tasks.
-   BERT is pretrained with objectives such as masked language modeling.

### Decoder-only: GPT

-   Uses causal self-attention.
-   A position can attend only to itself and earlier positions.
-   Predicts the next token autoregressively.
-   Commonly used for text generation, dialogue, and code generation.

### Encoder-decoder: Original Transformer

-   Encoder reads and represents the input sequence.
-   Decoder generates the output sequence.
-   Decoder cross-attention accesses encoder representations.
-   Commonly used for translation and other sequence-to-sequence tasks.

**Interview takeaway:** Encoder-only models are commonly associated with
representation and understanding tasks; decoder-only models with
autoregressive generation; encoder-decoder models with input-to-output
sequence transformation.

------------------------------------------------------------------------

## 12. Self-Attention vs. Cross-Attention

**Question:** What is the difference, and why do we need both?

### Self-attention

Queries, keys, and values come from the same sequence.

It allows tokens to exchange information with other tokens in that
sequence.

Example: In "The animal didn't cross the street because it was tired,"
self-attention can help connect "it" with relevant context.

### Cross-attention

Queries come from one sequence, while keys and values come from another.

Example: Machine translation: 1. The encoder processes the English
input. 2. The decoder processes the partial French output. 3. The
decoder's current representations produce queries. 4. Encoder outputs
are projected into keys and values. 5. The decoder attends to relevant
source information while generating the next token.

### Why both?

-   Self-attention models relationships within a sequence.
-   Cross-attention lets one sequence retrieve information from another.

**Interview takeaway:** In encoder-decoder translation, decoder
self-attention models the generated prefix, while cross-attention
connects the decoder to the source sentence.

------------------------------------------------------------------------

## 13. LoRA: Low-Rank Adaptation

**Question:** How does LoRA fine-tune a large model efficiently? What
does low-rank decomposition mean?

Full fine-tuning updates pretrained model weights. For a large model,
storing gradients and optimizer states for all parameters can require
substantial memory.

LoRA freezes the original weights and learns a low-rank update.

Let the pretrained weight matrix be (W_0). LoRA represents the adapted
weight as:

\[ W=W_0+`\Delta `{=tex}W \]

Instead of directly training the full update matrix (`\Delta `{=tex}W),
LoRA approximates it using two smaller matrices:

\[ `\Delta `{=tex}W=BA \]

The dimensions of A and B are chosen so their shared rank (r) is much
smaller than the dimensions of the original weight matrix.

A common implementation includes a scaling factor:

\[ W=W_0+`\frac{\alpha}{r}`{=tex}BA \]

### How training works

1.  Keep pretrained weights (W_0) frozen.
2.  Insert trainable low-rank matrices A and B into selected layers.
3.  Calculate the layer output using the original weights plus the
    low-rank update.
4.  Backpropagate and update A and B only.
5.  Leave the original pretrained weights unchanged.

### Where is LoRA applied?

LoRA is commonly applied to attention projection matrices, such as query
and value projections. It can also be applied to other linear layers,
depending on the model and fine-tuning configuration.

### Why is it efficient?

-   Far fewer trainable parameters.
-   Lower optimizer-state memory.
-   Smaller adapter checkpoints.
-   Convenient task-specific adapters.

**Critical distinction:** LoRA does not decompose the original
pretrained matrix. It decomposes the additional weight update.

------------------------------------------------------------------------

## 14. QLoRA vs. LoRA

**Question:** What is QLoRA, and how does it reduce memory requirements?

LoRA freezes the pretrained model weights and trains small low-rank
adapter matrices.

QLoRA combines this approach with quantization.

### QLoRA workflow

1.  Quantize the pretrained base model, commonly to 4-bit.
2.  Keep the quantized base model frozen.
3.  Add LoRA adapters.
4.  Train the adapter parameters.
5.  Use the quantized base model together with the trained adapters.

### Benefits

-   Reduces memory needed to store base-model weights.
-   Allows fine-tuning larger models on more limited hardware.
-   Retains the parameter-efficient training approach of LoRA.

QLoRA uses techniques such as NF4 quantization, double quantization, and
paged optimizers in its original formulation.

**Interview takeaway:** LoRA reduces trainable parameters; QLoRA
additionally reduces base-model storage requirements through
quantization.

------------------------------------------------------------------------

## 15. Activation Functions: Advantages and Disadvantages

**Question:** What are common activation functions, and what are their
strengths and weaknesses?

Activation functions introduce nonlinearity. Without nonlinear
activations, stacking linear layers would still produce an overall
linear transformation.

### Sigmoid

\[ `\sigma`{=tex}(x)=`\frac{1}{1+e^{-x}}`{=tex} \]

**Output range:** 0 to 1.

Advantages: - Useful for binary classification output probabilities. -
Smooth and differentiable.

Disadvantages: - Saturates for large positive or negative inputs. - Can
cause vanishing gradients. - Not zero-centered.

### Tanh

\[ `\tanh`{=tex}(x) \]

**Output range:** -1 to 1.

Advantages: - Zero-centered. - Often useful in recurrent-network state
transformations.

Disadvantages: - Saturates at both extremes. - Can cause vanishing
gradients.

### ReLU

\[ `\operatorname{ReLU}`{=tex}(x)=`\max`{=tex}(0,x) \]

Advantages: - Simple and computationally efficient. - Positive-side
gradient does not shrink as the input grows. - Widely used in hidden
layers.

Disadvantages: - Negative inputs produce zero output and zero
gradient. - Some neurons can become inactive, causing the dying ReLU
problem.

### Leaky ReLU

\[ f(x)=`\max`{=tex}(`\alpha `{=tex}x,x) \]

For negative inputs, it uses a small fixed slope (`\alpha`{=tex}).

Advantages: - Maintains a nonzero negative-side gradient. - Can reduce
the dying ReLU problem.

Disadvantages: - The negative slope must be selected or tuned.

### PReLU

Parametric ReLU learns the negative slope during training.

Advantages: - The model can adapt the negative slope to the data. - Can
address inactive-neuron behavior.

Disadvantages: - Adds trainable parameters. - Learned slopes do not
guarantee better performance.

### GELU

Gaussian Error Linear Unit is commonly used in Transformer
architectures.

Advantages: - Smooth nonlinear activation. - Often performs well in
language models.

Disadvantages: - More computationally involved than ReLU.

### SiLU / Swish

\[ `\operatorname{SiLU}`{=tex}(x)=x`\sigma`{=tex}(x) \]

Advantages: - Smooth and non-monotonic. - Used in several modern neural
architectures.

Disadvantages: - More expensive than ReLU.

**Interview takeaway:** ReLU is a common hidden-layer default; sigmoid
is common for binary output probabilities; GELU is widely used in
Transformers. The best choice depends on the architecture and task.

------------------------------------------------------------------------

## 16. Residual Connections

**Question:** What are residual connections, and why do they help deep
networks?

A residual connection adds a layer's input directly to its transformed
output.

A simplified residual block is:

\[ y=F(x)+x \]

Where: - (x) is the input. - (F(x)) is the learned transformation. - (y)
is the output.

If dimensions differ, a projection may be used to make the shapes
compatible.

### Why residual connections help

-   Provide a shortcut path for information.
-   Improve gradient flow through deep networks.
-   Make optimization of very deep architectures easier.
-   Allow layers to learn residual changes rather than requiring every
    block to learn a complete transformation.

Residual connections are central to ResNet and are also used throughout
Transformer architectures.

**Interview takeaway:** A residual connection adds the input to a
block's output, creating a shortcut that helps information and gradients
flow through deep networks.

------------------------------------------------------------------------

# Quick Review: High-Value Interview Distinctions

-   **Vanishing vs. exploding gradients:** Gradients become too small
    vs. too large.
-   **Xavier vs. He initialization:** Commonly associated with tanh-like
    vs. ReLU-family activations.
-   **SGD vs. Adam:** Conventional gradient updates vs. adaptive
    per-parameter scaling with momentum-like averaging.
-   **Dropout vs. batch normalization:** Direct regularization through
    dropped activations vs. normalization using batch statistics, with
    possible incidental regularization.
-   **CNN vs. RNN:** Local spatial feature extraction vs. recurrent
    sequence processing.
-   **LSTM vs. GRU:** Three gates and two states vs. two gates and one
    state.
-   **Query vs. key vs. value:** What is sought vs. what is matched
    vs. what information is aggregated.
-   **Self-attention vs. cross-attention:** Same sequence vs. different
    source and query sequences.
-   **Encoder-only vs. decoder-only:** Bidirectional representation
    learning vs. causal autoregressive generation.
-   **LoRA vs. QLoRA:** Low-rank trainable updates vs. low-rank updates
    with a quantized frozen base model.
-   **ReLU vs. Leaky ReLU:** Zero negative output vs. small negative
    slope.
-   **Leaky ReLU vs. PReLU:** Fixed negative slope vs. learned negative
    slope.
-   **Residual connection:** Adds a shortcut from a block's input to its
    output.

# Practical Interview Answer Framework

When asked about a deep learning concept, structure your answer in four
steps:

1.  **Define it:** Explain what the technique or architecture is.
2.  **Explain the mechanism:** Describe how it works.
3.  **Identify the problem:** Explain why it was introduced.
4.  **Discuss trade-offs:** Mention advantages, limitations, and typical
    use cases.

For example, when asked about LoRA:

"LoRA is a parameter-efficient fine-tuning method. It freezes the
pretrained model weights and learns a low-rank approximation of the
weight update using two smaller matrices. This reduces trainable
parameters and optimizer memory, making fine-tuning large models more
practical. Its effectiveness depends on adapter rank, target layers, and
the task."

------------------------------------------------------------------------

# Final Preparation Checklist

-   [ ] Explain backpropagation and gradient problems.
-   [ ] Compare Xavier and He initialization.
-   [ ] Explain SGD, mini-batches, and Adam.
-   [ ] Describe dropout, weight decay, and early stopping.
-   [ ] Explain batch normalization and inference statistics.
-   [ ] Compare CNN, RNN, LSTM, and GRU.
-   [ ] Explain attention using Q, K, and V.
-   [ ] Describe multi-head attention.
-   [ ] Draw the main components of a Transformer.
-   [ ] Compare BERT, GPT, and encoder-decoder models.
-   [ ] Explain self-attention and cross-attention.
-   [ ] Explain LoRA's low-rank weight update.
-   [ ] Compare LoRA and QLoRA.
-   [ ] Compare common activation functions.
-   [ ] Explain residual connections.

**Final advice:** Focus on understanding mechanisms rather than
memorizing definitions. Strong candidates can explain not only what a
technique does, but also why it works, when it is useful, and what
trade-offs it introduces.
