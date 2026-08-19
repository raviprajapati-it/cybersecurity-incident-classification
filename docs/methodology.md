# Methodology

## Overview

This project evaluates a hybrid Natural Language Processing and machine-learning pipeline for classifying cybersecurity incidents using structured security metadata, textual ticket information, transformer-based sentence embeddings, and MITRE ATT&CK context.

The methodology was developed as part of a BSc Cyber Security dissertation focused on automated classification of phishing- and malware-related security incidents.

The original reported experiment used a sample of **50,000 records** from the Microsoft Security Incident Prediction (GUIDE) dataset.

---

## Research Pipeline

The experimental workflow consisted of:

1. Dataset collection
2. Feature selection
3. Data preprocessing
4. Text construction
5. Feature engineering
6. Train-test splitting
7. Class-imbalance handling
8. Model training
9. Cross-validation
10. Performance evaluation

The models were compared using accuracy, precision, recall, F1-score, confusion matrices, ROC analysis, precision-recall curves, and feature-importance analysis.

---

## Dataset

The research used the Microsoft Security Incident Prediction (GUIDE) dataset.

The dissertation reports a working sample of:

* **50,000 cybersecurity incident records**
* **40,000 training records**
* **10,000 testing records**

The following fields were selected for analysis:

* `Category`
* `MitreTechniques`
* `EntityType`
* `EvidenceRole`
* `SuspicionLevel`
* `LastVerdict`
* `ThreatFamily`
* `OSFamily`
* `CountryCode`
* `AlertTitle`
* `IncidentGrade`

The target variable used in the reported experiment was `IncidentGrade`.

The dataset itself is not redistributed in this repository.

See:

[Dataset Documentation](../data/README.md)

---

## Missing-Value Handling

Some selected attributes contained missing values.

Rather than discarding incomplete records, missing values were represented using:

```text
Unknown
```

This approach preserved potentially useful cybersecurity observations while keeping the dataset structurally consistent for downstream processing.

---

## TicketText Construction

A derived text feature called `TicketText` was created by combining several security-related attributes into a unified textual representation.

The reported methodology combined information including:

* Alert title
* Category
* MITRE ATT&CK techniques
* Entity type
* Threat family
* Last verdict

The objective was to provide NLP models with richer contextual information than would be available from `AlertTitle` alone.

Conceptually:

```text
AlertTitle
    +
Category
    +
MITRE Techniques
    +
Entity Type
    +
Threat Family
    +
Last Verdict
        │
        ▼
     TicketText
```

---

## Rule-Based Baseline

Before applying machine-learning models, a simple keyword-based baseline classifier was constructed.

Example phishing-related keywords included:

```text
phishing
credential
login
fake
email
spoof
```

Example malware-related keywords included:

```text
malware
trojan
ransomware
worm
virus
```

This baseline provided a simple reference point against which more advanced classification techniques could be considered.

Rule-based methods are easy to interpret but have limited ability to capture semantic meaning or adapt to variations in incident descriptions.

---

## TF-IDF Feature Engineering

Term Frequency–Inverse Document Frequency (TF-IDF) was used to transform incident text into numerical features.

The reported configuration used:

```text
Maximum features: 300
Stop words: English
N-gram range: (1, 2)
```

This representation captures both individual terms and two-word combinations.

TF-IDF is particularly useful for identifying explicit cybersecurity vocabulary such as:

* phishing
* malware
* credential
* login
* suspicious activity
* malicious behaviour

---

## Transformer-Based Sentence Embeddings

Contextual sentence embeddings were generated using Sentence Transformers with:

```text
all-MiniLM-L6-v2
```

This model converts textual incident descriptions into dense numerical vectors that capture semantic relationships between sentences.

Unlike TF-IDF, which focuses primarily on lexical importance, sentence embeddings can represent similar meanings even when incidents use different wording.

Examples include relationships between descriptions involving:

* credential compromise
* phishing attempts
* suspicious authentication
* malware activity
* account anomalies

> The original dissertation sometimes refers to this component as DistilBERT. In this portfolio repository it is described more precisely as the Sentence Transformer model `all-MiniLM-L6-v2`.

---

## Combined Feature Representation

The research combined multiple feature sources into a single machine-learning representation.

These included:

* TF-IDF features
* Sentence-transformer embeddings
* Encoded structured cybersecurity metadata

The reported final feature space contained:

```text
694 predictive features
```

This hybrid approach was intended to capture:

* explicit cybersecurity terminology
* semantic relationships
* structured security context

---

## Label Encoding

Categorical attributes were converted into numerical form using label encoding so they could be processed by the machine-learning classifiers.

This enabled structured cybersecurity fields to be combined with NLP-derived features.

---

## Train-Test Split

The reported experiment used:

```text
Training set: 80%
Testing set: 20%
```

For the 50,000-record dissertation experiment, this corresponded to:

```text
Training records: 40,000
Testing records: 10,000
```

A stratified split was used to preserve class proportions.

---

## Class Imbalance

The dataset contained imbalanced incident classes.

Synthetic Minority Oversampling Technique (**SMOTE**) was applied to the training data to increase representation of minority classes.

SMOTE creates synthetic minority-class examples rather than simply duplicating existing observations.

This was intended to reduce model bias toward majority classes.

The dissertation nevertheless found that minority classes remained more difficult to classify accurately.

---

## Machine-Learning Models

Three supervised classifiers were evaluated.

### Random Forest

Random Forest was configured with:

```text
n_estimators = 150
max_depth = 15
min_samples_split = 5
class_weight = balanced
```

It was selected because of its ability to work effectively with:

* high-dimensional features
* mixed structured/text-derived features
* noisy cybersecurity data
* nonlinear relationships

Random Forest produced the strongest reported overall performance.

---

### Gradient Boosting

Gradient Boosting was configured with:

```text
n_estimators = 100
learning_rate = 0.08
```

The model sequentially builds weak learners and attempts to correct errors produced by earlier stages.

---

### XGBoost

XGBoost was configured with:

```text
n_estimators = 150
learning_rate = 0.05
max_depth = 6
subsample = 0.8
```

XGBoost was included because of its scalability, regularisation support, and ability to process high-dimensional classification data.

---

## Validation

The research used:

```text
3-fold cross-validation
```

Cross-validation provided an additional estimate of model reliability beyond the single held-out test split.

Hyperparameters such as:

* number of estimators
* learning rate
* tree depth

were adjusted during experimentation.

---

## Evaluation Metrics

The classifiers were evaluated using:

* Accuracy
* Precision
* Recall
* Weighted F1-score
* Cross-validation accuracy
* Classification reports
* Confusion matrices
* ROC curves
* AUC
* Precision-recall curves
* Feature importance

Using several metrics was important because the dataset contained class imbalance and accuracy alone would not fully describe model performance.

---

## Reported Model Results

The dissertation reports the following comparative results:

| Model             |   Accuracy | Weighted F1 | Cross-Validation Accuracy |
| ----------------- | ---------: | ----------: | ------------------------: |
| Random Forest     | **69.28%** |  **0.6963** |                **75.96%** |
| XGBoost           |     67.99% |      0.6862 |                    74.97% |
| Gradient Boosting |     65.61% |      0.6653 |                    71.98% |

Random Forest achieved the strongest overall reported result.

The results should be interpreted as experimental research findings rather than production-level performance guarantees.

---

## Reproducibility Note

The original dissertation reports experiments using **50,000 records**.

The notebook currently preserved in this repository contains a configurable dataset-loading step and may use a smaller row limit, such as:

```python
nrows=10000
```

depending on the saved execution version and available computational resources.

Therefore:

> Running the notebook exactly as currently configured may not reproduce the dissertation's 50,000-record metrics.

The dissertation results and the preserved notebook implementation are both retained as separate evidence of the original project.

---

## Scope

This project demonstrates an offline experimental classification workflow.

It does **not** represent:

* A production SOC classifier
* A live SIEM integration
* Real-time incident ingestion
* Automated incident containment
* Production MITRE ATT&CK mapping
* A deployed enterprise ML service
* Real-time phishing or malware detection

The objective is to evaluate the feasibility and limitations of combining NLP, transformer-based embeddings, structured cybersecurity metadata, and ensemble machine learning for cybersecurity incident classification.
