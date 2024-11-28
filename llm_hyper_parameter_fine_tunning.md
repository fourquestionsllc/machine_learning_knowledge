# Fine-Tuning Hyperparameters of LLMs

This guide provides an overview of hyperparameters for LLMs and demonstrates how to fine-tune these models using Python code.

---

## Hyperparameters of LLMs

### **Training Hyperparameters**
1. **Learning Rate**: Controls how much the model weights are adjusted during each training step.
2. **Batch Size**: The number of samples processed before updating the model.
3. **Epochs**: The number of complete passes through the training dataset.
4. **Weight Decay**: A regularization parameter to prevent overfitting by penalizing large weights.
5. **Warmup Steps**: The number of steps for the learning rate to increase linearly before decaying.
6. **Gradient Clipping**: Limits the gradient values to avoid exploding gradients.

### **Model Hyperparameters**
1. **Number of Layers**: Controls the depth of the transformer network.
2. **Number of Attention Heads**: Determines the granularity of attention mechanisms.
3. **Hidden Size**: The dimensionality of the model's hidden states.
4. **Dropout Rate**: Adds noise to prevent overfitting by randomly zeroing activations during training.
5. **Vocabulary Size**: Size of the tokenizer vocabulary.

---

## Fine-Tuning Hyperparameters

Fine-tuning adjusts a pretrained LLM to a specific task using smaller, task-specific datasets. Common fine-tuning hyperparameters include:
- Learning rate
- Batch size
- Epochs
- Optimizer type (e.g., AdamW)
- Scheduler type (e.g., linear decay)

---

## Example: Fine-Tuning an LLM (Hugging Face Transformers)

### **Setup**
Install the necessary libraries:
```bash
pip install transformers datasets accelerate
```

### **Code for Fine-Tuning***

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, Trainer, TrainingArguments
from datasets import load_dataset

# Load pretrained model and tokenizer
model_name = "gpt2"
model = AutoModelForCausalLM.from_pretrained(model_name)
tokenizer = AutoTokenizer.from_pretrained(model_name)

# Load and preprocess dataset
dataset = load_dataset("wikitext", "wikitext-2-raw-v1")
def preprocess_function(examples):
    return tokenizer(examples['text'], truncation=True, padding="max_length", max_length=128)

tokenized_datasets = dataset.map(preprocess_function, batched=True)
tokenized_datasets = tokenized_datasets.remove_columns(["text"])
tokenized_datasets.set_format("torch")

# Define training arguments
training_args = TrainingArguments(
    output_dir="./results",
    evaluation_strategy="epoch",
    learning_rate=5e-5,
    per_device_train_batch_size=8,
    per_device_eval_batch_size=8,
    num_train_epochs=3,
    weight_decay=0.01,
    warmup_steps=500,
    logging_dir="./logs",
    logging_steps=10,
    save_strategy="epoch",
    fp16=True,  # Use mixed precision for faster training
)

# Create Trainer object
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_datasets["train"],
    eval_dataset=tokenized_datasets["validation"],
)

# Fine-tune the model
trainer.train()

# Save the model
model.save_pretrained("./fine_tuned_gpt2")
tokenizer.save_pretrained("./fine_tuned_gpt2")
```

### **Explanation of Key Hyperparameters in Code**

* learning_rate=5e-5: A commonly used learning rate for fine-tuning.
* num_train_epochs=3: Fine-tune for 3 passes over the training data.
* per_device_train_batch_size=8: Process 8 samples per training step on each GPU.
* weight_decay=0.01: Helps regularize the model to avoid overfitting.
* warmup_steps=500: Gradually increases the learning rate for the first 500 steps.

# Hyperparameter Tuning

To optimize hyperparameters, you can use grid search or libraries like Optuna or Ray Tune.

## Example: Using Optuna for Hyperparameter Search

```python
import optuna
from transformers import TrainingArguments

def objective(trial):
    # Define the hyperparameter search space
    learning_rate = trial.suggest_loguniform("learning_rate", 1e-5, 1e-3)
    batch_size = trial.suggest_categorical("batch_size", [8, 16, 32])
    
    training_args = TrainingArguments(
        output_dir="./results",
        learning_rate=learning_rate,
        per_device_train_batch_size=batch_size,
        num_train_epochs=3,
        evaluation_strategy="epoch",
    )
    
    trainer = Trainer(
        model=model,
        args=training_args,
        train_dataset=tokenized_datasets["train"],
        eval_dataset=tokenized_datasets["validation"],
    )
    
    # Evaluate the model
    eval_result = trainer.evaluate()
    return eval_result["eval_loss"]

# Run Optuna optimization
study = optuna.create_study(direction="minimize")
study.optimize(objective, n_trials=10)

# Best hyperparameters
print(study.best_params)
```
# Summary

This guide demonstrates how to fine-tune LLMs like GPT-2 using the Hugging Face library. The hyperparameter tuning example uses Optuna to optimize learning rate and batch size for improved performance. Adjustments can be made for larger models or tasks such as classification and summarization.
