# TryHackMe — The Building Blocks of AI

> **Understanding the foundations of AI, Machine Learning, Neural Networks, Deep Learning, and Large Language Models.**

**Platform:** TryHackMe
**Category:** AI Security / AI Fundamentals
**Difficulty:** Beginner
**Duration:** ~30 minutes

---

## Overview

**The Building Blocks of AI** introduces the core technologies behind modern AI systems before moving into their security implications.

The room covers:

* Artificial Intelligence
* Machine Learning
* ML learning paradigms
* Neural Networks
* Deep Learning
* Large Language Models
* Transformers
* Attention
* Backpropagation
* RLHF
* AI-agent interaction

The key takeaway is understanding how these technologies relate to each other and why that knowledge matters when studying **AI Security**.

---

## AI → ML → DL → LLM

A useful mental model from the room:

```text
Artificial Intelligence
        │
        └── Machine Learning
                │
                └── Deep Learning
                        │
                        └── Neural Networks
                                │
                                └── Transformers
                                        │
                                        └── LLMs
```

These terms are related, but they are **not interchangeable**.

* **AI** — broad field of systems performing tasks associated with intelligence.
* **ML** — systems learn patterns from data.
* **Deep Learning** — ML based on multi-layer neural networks.
* **LLMs** — large deep-learning models specialised in language, commonly built using Transformer architectures.

---

# Task 1 — Introduction

The room begins by establishing why AI knowledge is important for cybersecurity.

AI is increasingly relevant to both:

* **Attackers**, who can use AI to automate or enhance attacks.
* **Defenders**, who can use AI for detection, analysis and automation.

Before studying attacks against AI systems, it is necessary to understand how the underlying technology works.

---

# Task 2 — TryHackMe AI Agent Platform

TryHackMe introduces an interactive **AI Agent** as an alternative to the traditional terminal-based environment.

An agent can be configured with:

* A role
* Behavioural instructions
* Objectives
* Information it should protect
* Rules governing its responses

The interaction is conversational:

```text
User
  │
  ▼
Prompt
  │
  ▼
AI Agent
  │
  ├── Interpret input
  ├── Apply instructions
  └── Generate response
  │
  ▼
Output
```

This provides an introduction to an important AI-security concept:

> **The way an AI system interprets instructions can become part of its attack surface.**

---

# Task 3 — AI and Machine Learning

## Machine Learning

**Machine Learning (ML)** is a subfield of AI where models learn patterns from data instead of relying entirely on explicitly programmed rules.

A simplified ML lifecycle:

```text
Problem Definition
       ↓
Data Collection
       ↓
Data Preparation
       ↓
Training
       ↓
Evaluation
       ↓
Optimisation
       ↓
Deployment
       ↓
Monitoring
       ↓
Retraining
```

The lifecycle is iterative because model performance can change as real-world data changes.

---

## Overfitting

**Overfitting** occurs when a model learns the training data too specifically and fails to generalise effectively to unseen data.

```text
Training Data
     ↓
Model learns too specifically
     ↓
High training performance
     ↓
Poor unseen-data performance
```

### Answers

| Question                                      | Answer             |
| --------------------------------------------- | ------------------ |
| Model becomes too familiar with training data | `Overfitting`      |
| AI subfield that learns from data             | `Machine Learning` |

---

# Task 4 — Machine Learning Algorithms

The room introduces four major learning paradigms.

## Supervised Learning

Uses **labelled data** where the expected output is known.

```text
Input → Known Label
```

**Examples:** spam classification, malware classification and image classification.

---

## Unsupervised Learning

Uses **unlabelled data** and attempts to discover patterns or structure.

```text
Unlabelled Data
      ↓
Pattern Discovery
      ↓
Clusters / Anomalies / Relationships
```

**Cybersecurity example:** identifying unusual network behaviour.

---

## Semi-Supervised Learning

Uses a combination of:

* A small labelled dataset
* A larger unlabelled dataset

This is useful when obtaining labels is expensive or time-consuming.

---

## Reinforcement Learning

An agent learns by interacting with an environment and receiving rewards or penalties.

```text
Environment
     ↓
   State
     ↓
   Agent
     ↓
  Action
     ↓
Environment
     ↓
Reward / Penalty
     ↓
Agent learns
```

### Answers

| Question                                                       | Answer                     |
| -------------------------------------------------------------- | -------------------------- |
| Learns through rewards and penalties                           | `Reinforcement Learning`   |
| Uses a small labelled dataset with a larger unlabelled dataset | `Semi-Supervised Learning` |
| flag                                                           | THM{4lg0r1thm_4g3nt}       |

---

## Practical Challenge

The practical exercise required selecting the appropriate ML paradigm for different scenarios.

The decision process:

| Given Scenario                            | Learning Type   |
| ----------------------------------------- | --------------- |
| Labelled examples                         | Supervised      |
| Unlabelled data / pattern discovery       | Unsupervised    |
| Small labelled + large unlabelled dataset | Semi-Supervised |
| Actions + rewards/penalties               | Reinforcement   |

The final flag was obtained after correctly completing the mission sequence.



---

# Task 5 — Neural Networks and Deep Learning

## Neural Network Structure

A basic neural network consists of:

```text
Input Layer
     ↓
Hidden Layer(s)
     ↓
Output Layer
```

### Input Layer

Receives the raw input data.

### Hidden Layers

Transform the input and learn increasingly complex representations.

### Output Layer

Produces the final prediction or classification.

---

## Weights

Connections between neurons contain numerical **weights**.

Weights determine how strongly information from one neuron influences another.

During training, these values are adjusted to improve the model's predictions.

```text
Neuron A
   │
   │ Weight
   ▼
Neuron B
```

---

## Deep Learning

Deep Learning uses neural networks with multiple processing layers to learn complex representations from data.

A simplified image-classification example:

```text
Pixels
  ↓
Edges
  ↓
Shapes
  ↓
Complex Features
  ↓
Classification
```

This ability to learn increasingly complex representations is one reason deep learning is effective for large and complex datasets.

---

## NEURON-1

The practical challenge demonstrates the flow of information through a neural network:

```text
Raw Input
    ↓
Input Layer
    ↓
Feature Processing
    ↓
Hidden Layer
    ↓
Pattern Recognition
    ↓
Output Layer
    ↓
Classification
```

### Answers

| Question                           | Answer        |
| ---------------------------------- | ------------- |
| First layer receiving raw input    | `Input Layer` |
| Weighted connections between nodes | `Synapses`     |
| flag                               | `THM{n3ur0n_1_0nl1n3}` |


The practical flag was obtained after successfully completing the NEURON-1 classification exercise.


---

# Task 6 — Large Language Models

## LLM Fundamentals

Large Language Models are deep-learning models designed to process and generate language.

At a high level, generation can be viewed as repeated next-token prediction:

```text
"The server is"
       ↓
   Prediction
       ↓
"online"
       ↓
"The server is online"
       ↓
Next prediction
```

---

## Pre-Training

During pre-training, the model processes very large datasets and learns statistical patterns in language.

A simplified training loop:

```text
Training Data
     ↓
Prediction
     ↓
Calculate Error
     ↓
Backpropagation
     ↓
Update Parameters
     ↓
Repeat
```

---

## Parameters

**Parameters** are numerical values learned during training.

They influence how the model transforms input into predictions.

Large language models contain extremely large numbers of these learned parameters.

---

## Backpropagation

Backpropagation is used during neural-network training to propagate error information backward through the network so that model parameters can be updated.

```text
Prediction
    ↓
Error
    ↓
Backpropagation
    ↓
Parameter Updates
```

### Answer

**Algorithm used to adjust parameters based on prediction error:**

`Backpropagation`

---

# Transformers

Modern LLMs are largely based on the **Transformer architecture**.

The architecture was introduced in the 2017 paper:

> **Attention Is All You Need**

Transformers made large-scale sequence processing more practical and introduced the attention mechanism that became fundamental to modern language models.

### Answer

**Neural-network architecture introduced in 2017 that powers modern LLMs:**

`Transformer`

---

# Attention

The **attention mechanism** allows a Transformer to determine which parts of a sequence are more relevant when processing a particular token.

For example:

```text
The bank approved the loan because it was financially stable.
```

The model needs to use context to understand relationships between words such as **"it"** and the surrounding tokens.

Attention allows the model to assign different importance to different parts of the input.

### Answer

**Mechanism used to assign different levels of importance to words/tokens:**

`Attention`

---

# RLHF

**RLHF — Reinforcement Learning from Human Feedback** — is a technique used to further shape model behaviour using human preferences or evaluations.

Simplified:

```text
Pre-trained Model
       ↓
Generate Responses
       ↓
Human Evaluation
       ↓
Feedback / Preferences
       ↓
Further Training
       ↓
Improved Behaviour
```
### Answer

**What is the name of the process where humans review and flag model outputs to refine its behaviour after pre-training?**
`RLHF`

---

# Task 7 — Practical

The final practical exercise combines the concepts introduced throughout the room.

The challenge requires operating the NEURON-1 environment and correctly following the neural-network processing flow:

```text
Input
  ↓
Input Layer
  ↓
Hidden Layer
  ↓
Pattern Recognition
  ↓
Output Layer
  ↓
Classification
```

The final flag was obtained after successfully completing the classification challenge.

## Answer the questions below

**What's the flag?**

**Answer:** `THM{y0u_tr41n3d_th3_n3tw0rk}`

---

# Task 8 — Conclusion

This room establishes the technical foundation required for the rest of the AI Security learning path.

The main concepts can be summarised as:

| Concept                  | Key Idea                                                |
| ------------------------ | ------------------------------------------------------- |
| AI                       | Broad field of machine intelligence                     |
| ML                       | Learning patterns from data                             |
| Supervised Learning      | Learns from labelled data                               |
| Unsupervised Learning    | Discovers patterns in unlabelled data                   |
| Semi-Supervised Learning | Combines labelled and unlabelled data                   |
| Reinforcement Learning   | Learns through rewards and penalties                    |
| Neural Networks          | Layered computational models using weighted connections |
| Deep Learning            | Multi-layer neural-network-based ML                     |
| Transformer              | Architecture behind modern LLMs                         |
| Attention                | Determines contextual importance                        |
| Backpropagation          | Propagates error for parameter updates                  |
| RLHF                     | Uses human feedback to shape model behaviour            |
| LLM                      | Large-scale language model based on deep learning       |

---

# Cybersecurity Relevance

The concepts from this room have direct applications in cybersecurity.

### Machine Learning

Can assist with:

* Network anomaly detection
* Malware classification
* Phishing detection
* User/entity behaviour analytics
* Threat detection

### Deep Learning

Can process complex security data such as:

* Network traffic
* Endpoint telemetry
* Malware features
* Security logs
* Authentication behaviour

### LLMs

Can assist security teams with:

* Alert summarisation
* Threat-intelligence analysis
* Incident investigation
* Security automation
* Detection engineering
* Security documentation

However, AI also introduces new security risks.

Understanding the underlying technology is therefore essential before studying threats such as:

* Prompt injection
* Data poisoning
* Model manipulation
* Sensitive information disclosure
* Adversarial attacks
* AI supply-chain risks

---

# Key Takeaways

The biggest takeaway from this room is the relationship between the technologies:

```text
AI
 │
 └── ML
      │
      └── Deep Learning
           │
           └── Neural Networks
                │
                └── Transformers
                     │
                     └── LLMs
```

Rather than treating AI as a black box, this room provides the foundation for understanding **how models learn, process information and produce predictions**.

That foundation becomes important when analysing the security of AI systems.

---

# Skills Demonstrated

* AI fundamentals
* Machine Learning fundamentals
* ML learning paradigms
* Neural Networks
* Deep Learning
* LLM fundamentals
* Transformer architecture
* Attention mechanisms
* Backpropagation
* RLHF
* AI-agent interaction
* AI Security fundamentals

---

