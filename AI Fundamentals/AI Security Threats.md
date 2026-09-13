# TryHackMe — AI Security Threats

> Exploring AI-specific vulnerabilities, AI-enhanced attacks, defensive AI, and secure AI adoption.

**Platform:** TryHackMe
**Room:** AI Security Threats
**Difficulty:** Easy

🔗 [TryHackMe — AI Security Threats](https://tryhackme.com/room/aisecuritythreats)

---

## Overview

AI introduces a new attack surface alongside traditional cybersecurity risks.

This room focuses on four areas:

* Security vulnerabilities specific to AI/ML systems
* Traditional attacks enhanced by AI
* Defensive applications of AI in security operations
* Security controls and standards for AI adoption

The room builds on the concepts covered in **The Building Blocks of AI**.

---

# Task 1 — Introduction

The room introduces the security implications of deploying AI in organisational environments.

The key objectives are to understand:

* How AI models can be attacked
* How AI can enhance existing attacks
* How defenders can use AI for security operations
* How AI systems should be secured throughout their lifecycle


---

# Task 2 — Vulnerabilities in AI Models

AI systems introduce vulnerabilities that differ from traditional application vulnerabilities.

The room uses **MITRE ATLAS** to provide a structured view of adversarial techniques targeting AI systems.

🔗 [MITRE ATLAS](https://atlas.mitre.org/)

## Key Vulnerabilities

### Prompt Injection

An attacker crafts input designed to manipulate an AI model into ignoring or overriding its original instructions.

Potential consequences include:

* System prompt disclosure
* Sensitive information exposure
* Unintended model behaviour
* Bypassing model restrictions

The practical challenge demonstrates this against the **MENTOR** AI assistant.

---

### Data Poisoning

An attacker manipulates training data so that the resulting model behaves incorrectly or produces biased results.

The attack occurs during the data/training stage rather than through normal user interaction.

---

### Model Theft

An attacker can repeatedly query a model through an exposed API and collect its outputs.

Those outputs can then be used to build a model that attempts to reproduce the behaviour of the original system.

This creates risks involving:

* Intellectual property
* Proprietary models
* Computational investment
* Competitive advantage

---

### Privacy Leakage

AI models may unintentionally expose sensitive information contained within their training data.

This makes training-data governance and privacy protection important parts of AI security.

---

### Model Drift

Model drift occurs when a model's effectiveness decreases as the environment or underlying data changes.

For security systems, changing attacker behaviour and network conditions can cause previously effective models to become less accurate.

Continuous monitoring is therefore important after deployment.

---

## Task 2 — Answers

| Question                                       | Answer             |
| ---------------------------------------------- | ------------------ |
| MITRE framework for AI threats                 | `ATLAS`      |
| User input overriding model instructions       | `Prompt Injection` |
| Manipulating training data                     | `Data Poisoning`   |
| Repeated API queries to reproduce a model      | `Model Theft`      |
| Gradual degradation as the environment changes | `Model Drift`      |



---

# Task 3 — AI-Enhanced Attacks

AI doesn't only introduce new vulnerabilities. It can also significantly increase the effectiveness and scalability of existing attacks.

The room focuses on:

* AI-generated malware
* Deepfakes
* AI-enhanced phishing
* AI-assisted social engineering

---

## AI-Generated Malware

Generative AI can reduce the time and technical effort required to create or modify malicious code.

Attackers can potentially use AI to:

* Generate code
* Modify existing malware
* Create variants
* Automate repetitive development tasks

This lowers the barrier to entry for some forms of malicious activity.

---

## Deepfakes

Deepfake technology can generate convincing representations of real people.

This includes:

* Synthetic voices
* Facial impersonation
* AI-generated video

From a security perspective, deepfakes can support impersonation and social-engineering attacks.

For example, an attacker could impersonate an executive and request an urgent financial transaction.

---

## AI-Enhanced Phishing

AI can make phishing campaigns more convincing by generating fluent, personalised, and context-aware messages at scale.

Traditional indicators such as poor grammar and awkward wording are therefore becoming less reliable.

Security awareness must increasingly focus on:

* Sender verification
* Domain verification
* Link inspection
* Context
* Out-of-band verification

---

## Practical — Secure Inbox

The practical challenge presented three messages containing AI-enhanced threats.

The objective was to identify:

1. The attack type
2. How AI contributed to the attack

The relevant categories were:

| Threat                         | AI Contribution                           |
| ------------------------------ | ----------------------------------------- |
| AI-enhanced phishing           | Generates convincing targeted messages    |
| Deepfake                       | Replicates a person's appearance or voice |
| AI-assisted social engineering | Enables more personalised manipulation    |


---

## Task 3 — Answers

| Question                                                         | Answer     |
| ---------------------------------------------------------------- | ---------- |
| AI-generated replica of a person's voice/appearance              | `Deepfake` |
| Common initial-access technique enhanced by AI-generated content | `Phishing` |

---

# Task 4 — Defensive AI

AI can also provide defenders with significant advantages when analysing large volumes of security data.

The room focuses on four major defensive capabilities:

### Analysis

AI/ML can identify patterns and anomalies across large datasets such as:

* Network traffic
* Authentication events
* Endpoint telemetry
* Security logs

### Prediction

Historical security data can be used to identify patterns associated with potential future attacks.

### Summarisation

LLMs can reduce large volumes of security information into concise summaries for analysts and incident responders.

### Investigation

LLMs can assist analysts by analysing raw logs, explaining suspicious events, suggesting investigation queries, and supporting threat hunting.

---

## Practical — AEGIS

The AEGIS exercise demonstrates how an AI assistant can support a security analyst.

### Firewall Log Analysis

The provided log showed a blocked TCP connection targeting port `22`.

Important fields included:

```text
SRC=185.220.101.47
DST=10.0.0.5
PROTO=TCP
DPT=22
```

The exercise demonstrates how AI can help an analyst quickly interpret raw security telemetry.

### Phishing Triage

The provided email contained several indicators associated with phishing:

* Urgency
* Account suspension threat
* Suspicious domain
* External verification link

### Incident Summarisation

AEGIS was used to consolidate the available investigation information into a concise incident summary.

### Threat Hunting

The final step involved asking the AI assistant what additional threats might exist based on the observed activity.


---

## Task 4 — Answers

| Question                                               | Answer                            |
| ------------------------------------------------------ | --------------------------------- |
| Faster identification and containment according to IBM | `108 days`                        |
| Microsoft security product mentioned                   | `Microsoft Defender for Endpoint` |
| LLM capability used with raw logs during an incident   | `Investigation`                   |

---

# Task 5 — Securing AI

Deploying AI without securing the AI infrastructure creates another attack surface.

The room highlights several important security controls.

---

## Access Control

AI systems should use strong authentication and authorisation controls.

Important measures include:

* **RBAC** — Role-Based Access Control
* **MFA** — Multi-Factor Authentication
* Least privilege
* Strict permissions

RBAC ensures that users receive access according to their role rather than receiving unnecessary permissions.

---

## Training Data Protection

Training data may contain sensitive information.

Organisations should therefore apply controls such as:

* Data minimisation
* Auditing
* Encryption
* Access control
* Proper data governance

---

## AI Security Standards

The room introduces **ISO/IEC 27090**, which provides guidance related to security threats specific to AI systems.

Security standards help organisations incorporate security into the AI lifecycle rather than treating it as an afterthought.

---

## Model Monitoring

AI models should be monitored after deployment for:

* Unexpected behaviour
* Anomalous outputs
* Performance degradation
* Statistical drift
* Potential attacks

The room also introduces two model explainability techniques:

* **SHAP**
* **LIME**

These techniques can help security teams understand why models produce particular outputs.

---

## Task 5 — Answers

| Question                                    | Answer          |
| ------------------------------------------- | --------------- |
| Generative AI initiatives currently secured | `24%`           |
| Recommended access-control model            | `RBAC`          |
| AI security ISO standard                    | `ISO/IEC 27090` |

---

# Task 6 — Practical

The final challenge is an **AI Security Analyst Orientation** assessment.

It combines concepts from the AI Fundamentals rooms and tests understanding of:

* AI and ML fundamentals
* AI-specific vulnerabilities
* AI-enhanced attacks
* Defensive AI
* Secure AI adoption


---

# Key Takeaways

| Area                 | Key Concept                                        |
| -------------------- | -------------------------------------------------- |
| AI Threat Framework  | MITRE ATLAS                                        |
| Model Vulnerability  | Prompt Injection                                   |
| Training Attack      | Data Poisoning                                     |
| Model Protection     | Model Theft                                        |
| Privacy Risk         | Privacy Leakage                                    |
| Model Reliability    | Model Drift                                        |
| AI-Enhanced Attack   | Phishing                                           |
| Synthetic Media      | Deepfakes                                          |
| Defensive AI         | Analysis, Prediction, Summarisation, Investigation |
| Access Control       | RBAC + MFA                                         |
| AI Security Standard | ISO/IEC 27090                                      |
| Model Explainability | SHAP + LIME                                        |

---

# Security Perspective

The most important takeaway from this room is that AI changes the security landscape in **both directions**.

Attackers can use AI to increase:

* Scale
* Speed
* Personalisation
* Automation
* Social-engineering effectiveness

Defenders can use the same technology to improve:

* Detection
* Investigation
* Threat hunting
* Alert triage
* Incident analysis

However, AI should be treated as another critical technology that requires security controls from design through deployment and monitoring.

> **AI can amplify both offensive and defensive capabilities. Securing the AI system itself is therefore just as important as using it to secure other systems.**


## Skills & Concepts Practised

`AI Security` · `Prompt Injection` · `MITRE ATLAS` · `Data Poisoning` · `Model Theft` · `Privacy Leakage` · `Model Drift` · `Phishing` · `Deepfakes` · `Threat Detection` · `Incident Investigation` · `RBAC` · `AI Security Standards`
