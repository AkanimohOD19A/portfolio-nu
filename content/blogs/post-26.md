---
title: 'My "Full-Stack" MLOps Journey: From Broken Models to Actually Working AI'
date: '2025-08-10T07:34:32+01:00'
draft: false
description: "A sentiment analysis model that classifies movie reviews as positive or negative. Sounds straightforward, right?"
tags: ["ai", "mlops", "debugging", "huggingface", "tutorial"]
author: "AfroLogicInsect"
image: /images/placeholder.jpg
---

*Or: How I learned to stop worrying and love debugging neural networks*

## The Grand Plan (That Almost Failed Spectacularly)

I recently completed what I'd call my first "full-stack" MLOps project – and by "full-stack" I mean the entire pipeline from raw data to deployed model. The goal was simple: build a sentiment analysis model, train it, and deploy it to Hugging Face. The reality? Well, let's just say my first model was so broken it thought "I absolutely loved this movie!" was negative sentiment with 99.9% confidence. 

But that's exactly why this became such a valuable learning experience.

## The Technical Stack (Free Tier Heroes)

The whole project was designed to be **completely cost-free** and maintainable as a proof of concept:

- **Google Colab** (Free GPU access)
- **Hugging Face Hub** (Free model hosting and datasets)
- **Weights & Biases** (Free experiment tracking)
- **IMDB Dataset** (Stanford's free sentiment dataset)

No AWS bills, no surprise charges – just pure learning.

## What I Built
A sentiment analysis model that classifies movie reviews as positive or negative. Sounds straightforward, right? 

**First attempt result**: [AfroLogicInsect/sentiment-analysis-model](https://huggingface.co/AfroLogicInsect/sentiment-analysis-model)

**Spoiler alert**: This model was completely broken. It predicted everything as negative with ridiculous confidence.

## The Learning Curve (AKA: Everything That Went Wrong)

### Issue #1: The "Always Negative" Model

My first model was like that pessimistic friend who finds fault with everything:

```python
# What I expected:
"I loved this movie!" → Positive (85% confidence)

# What I got:
"I loved this movie!" → Negative (99.9% confidence)
"This movie was amazing!" → Negative (99.5% confidence)
"Best film ever!" → Negative (99.8% confidence)
```

### Issue #2: Data Poisoning (Accidentally)

Here's where it gets interesting. The IMDB dataset is perfectly balanced (50% positive, 50% negative), but my data loading was catastrophically wrong:

```python
# My broken approach:
dataset["train"].select(range(15000))  # First 15k samples

# What I actually got:
# Training: 12,500 negative, 2,500 positive (83% negative!)
# Validation: 1,500 negative, 0 positive (100% negative!)
```

**The problem**: The IMDB dataset is structured with all negative reviews first, then positive ones. So my "balanced" sampling was actually creating a heavily skewed dataset. No wonder my model learned to always predict negative!

### Issue #3: Tiny Batch Syndrome

I was training with a batch size of 4. For context, that's like trying to learn patterns from looking at 4 movie reviews at a time instead of 16-32. The gradients were all over the place, making stable learning nearly impossible.

### Issue #4: Label Confusion

Even after fixing everything else, I had a label mapping bug where I was interpreting the model's outputs backwards. The model was actually working, but I was reading its outputs wrong!

## The Debugging Journey

The breakthrough came when I built a diagnostic tool to test the model systematically:

```python
test_cases = [
    "This movie is absolutely amazing!",
    "This movie is terrible.",
    "Great film, highly recommend!",
    "Boring movie, waste of time."
]

# Result: Everything → LABEL_0 (negative) with 99%+ confidence
# Translation: The model had collapsed completely
```

This revealed that the model wasn't just mislabeling – it had learned to always predict one class, which is a classic sign of severe training issues.

## The Fix: Proper MLOps Practices

### 1. Balanced Data Sampling
```python
# Fixed approach: Explicitly balance the dataset
train_pos_indices = [i for i, label in enumerate(data["label"]) if label == 1]
train_neg_indices = [i for i, label in enumerate(data["label"]) if label == 0]

# Take equal numbers: 7500 positive + 7500 negative
train_indices = train_pos_indices[:7500] + train_neg_indices[:7500]
```

### 2. Proper Training Parameters
```python
TrainingArguments(
    per_device_train_batch_size=16,  # Up from 4
    learning_rate=2e-5,              # Explicit learning rate
    num_train_epochs=3,
    eval_steps=200,                  # Frequent evaluation
    save_strategy="steps",
    load_best_model_at_end=True,
    seed=42                          # Reproducibility
)
```

### 3. Explicit Label Configuration
```python
model = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased",
    num_labels=2,
    id2label={0: "NEGATIVE", 1: "POSITIVE"},  # Crystal clear mapping
    label2id={"NEGATIVE": 0, "POSITIVE": 1}
)
```

### 4. Comprehensive Testing
```python
# Always test on obvious examples before deployment
test_cases = [
    ("Amazing movie, loved it!", "positive"),
    ("Terrible film, hated it", "negative"),
    ("Best movie ever!", "positive"),
    ("Worst movie I've seen", "negative")
]
```

## Technical Deep Dives
### Tokenizers and Text Processing

Working with transformers taught me about the intricacies of text preprocessing:

- **Tokenization**: How "I love this movie!" becomes `[101, 1045, 2293, 2023, 3185, 999, 102]`
- **Padding/Truncation**: Managing variable-length inputs in batches
- **Special tokens**: `[CLS]` and `[SEP]` tokens for sequence classification
- **Attention masks**: Telling the model which tokens to pay attention to

### Hardware and Compute

Google Colab's free tier taught me about GPU memory management:

```python
# Memory management became crucial
import gc
gc.collect()
torch.cuda.empty_cache()

# Monitoring GPU usage
nvidia-smi
```

With limited GPU memory, I learned to optimize batch sizes, use gradient accumulation, and implement efficient data loading.

### Model Architecture Choices

I chose DistilBERT over BERT for practical reasons:
- **6x faster** training and inference
- **40% smaller** model size
- **97% of BERT's performance** on most tasks
- **Perfect for experimentation** and resource constraints

## The Deployment Pipeline

The final deployment involved:

1. **Model saving**: Proper serialization with tokenizer
2. **Hugging Face integration**: Model cards, tags, and documentation
3. **API testing**: Ensuring the inference API works correctly
4. **Version control**: Git-based model versioning

```python
# Clean deployment code
model.push_to_hub("AfroLogicInsect/sentiment-analysis-model_v*") # *:1,2, .., n
tokenizer.push_to_hub("AfroLogicInsect/sentiment-analysis-model_v*")

# Test the deployed model
from transformers import pipeline
classifier = pipeline("sentiment-analysis", model="AfroLogicInsect/sentiment-analysis-model_v*")
result = classifier("I loved this movie!")
```

## Interview-Ready Insights

This project prepared me for several common ML engineering questions:

**"How do you debug a poorly performing model?"**
- Start with data quality and class distribution
- Use diagnostic tests with obvious examples
- Check label mappings and model configuration
- Monitor training metrics and convergence

**"Describe your experience with MLOps tools."**
- End-to-end pipeline from data to deployment
- Experiment tracking with W&B
- Version control for models and data
- Automated testing and validation

**"How do you handle limited compute resources?"**
- Efficient model selection (DistilBERT vs BERT)
- Memory management and batch size optimization
- Free tier resource management (Colab, HF Hub)

**"What's your approach to model validation?"**
- Balanced train/validation splits
- Comprehensive test cases covering edge cases
- Statistical significance testing
- Real-world performance monitoring

## Key Takeaways

1. **Data quality trumps model complexity** – A simple model with good data beats a complex model with bad data
2. **Always validate your data loading** – My biggest bugs were in data preprocessing, not model architecture
3. **Test early and often** – Don't wait until deployment to discover your model is broken
4. **Monitor class distributions** – Imbalanced data leads to biased models
5. **MLOps is mostly about debugging** – The actual model training is often the easy part

## The Working Model

After all the fixes, the model now properly distinguishes sentiments:

```python
# Current results:
"I loved this movie!" → Positive (91% confidence) ✅
"This movie was terrible" → Negative (89% confidence) ✅
"Amazing acting!" → Positive (94% confidence) ✅
"Complete waste of time" → Negative (92% confidence) ✅
```

You can try the working model here: [https://huggingface.co/spaces/AfroLogicInsect/sentiment-analysis-model-gradio]

## What's Next?

This experience showed me that MLOps is as much about software engineering practices as it is about machine learning. The debugging skills, systematic testing approach, and deployment pipeline knowledge I gained here directly translate to production ML systems.

Next up: Building a more complex multi-class classification system, experimenting with model serving infrastructure, and diving deeper into A/B testing for ML models.

## Resources and Code

- **Full training code & resources**: [https://drive.google.com/drive/folders/1zajKoNLsDoRtj6UJqtlyfaAnZD0Ajj1S?usp=sharing]
- **Experiment logs**: [https://wandb.ai/afrologicinsect/huggingface/runs/kjtg4oby?nw=nwuserafrologicinsect]
- **Model card**: [https://huggingface.co/AfroLogicInsect/sentiment-analysis-model_v2]
- **Dataset**: [Stanford IMDB Dataset](https://huggingface.co/datasets/stanfordnlp/imdb)

---

*Have you built any MLOps projects? I'd love to hear about your debugging war stories – especially the embarrassing ones where the fix was just a single line of code!*