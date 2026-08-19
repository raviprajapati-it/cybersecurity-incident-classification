# Feature Engineering

## Overview

This document describes the feature-engineering pipeline used for the cybersecurity incident classification experiment.

The project combined three complementary sources of information:

1. **TF-IDF textual features**
2. **Transformer-based sentence embeddings**
3. **Structured cybersecurity metadata**

The objective was to create a hybrid representation capable of capturing both explicit security terminology and contextual relationships within cybersecurity incident records.

The reported experiment produced a combined feature space containing **694 predictive features**.

---

## Feature Engineering Pipeline

The overall feature-engineering workflow was:

```text
Microsoft GUIDE Dataset
        │
        ▼
Selected Security Attributes
        │
        ▼
Missing-Value Handling
        │
        ▼
TicketText Construction
        │
        ├─────────────────────┐
        ▼                     ▼
     TF-IDF             Sentence Transformer
   Features               Embeddings
        │                     │
        └──────────┬──────────┘
                   │
                   ▼
        Structured Metadata
                   │
                   ▼
         Combined Feature Set
                   │
                   ▼
         694 Predictive Features
                   │
                   ▼
        Machine-Learning Models
```

This design allowed the classifiers to learn from several different representations of the same cybersecurity incidents.

---

## Source Features

The research selected the following fields from the Microsoft Security Incident Prediction (GUIDE) dataset:

```text
Category
MitreTechniques
EntityType
EvidenceRole
SuspicionLevel
LastVerdict
ThreatFamily
OSFamily
CountryCode
AlertTitle
IncidentGrade
```

These fields contained a mixture of:

- Textual information
- Security classifications
- MITRE ATT&CK context
- Threat-family information
- Entity information
- Operating-system information
- Incident labels

The target variable used in the reported classification experiment was:

```text
IncidentGrade
```

---

## Missing-Value Handling

Several selected fields contained missing values.

Rather than removing every incomplete observation, missing values were represented using:

```text
Unknown
```

This approach was intended to preserve potentially useful cybersecurity records while providing a consistent representation for subsequent processing.

Removing every record containing missing values could have significantly reduced the available dataset and potentially removed useful incident information.

---

## TicketText Construction

A derived feature named:

```text
TicketText
```

was created to provide a unified textual representation of each cybersecurity incident.

The reported methodology combined information including:

```text
AlertTitle
Category
MitreTechniques
EntityType
ThreatFamily
LastVerdict
```

Conceptually:

```text
AlertTitle
    +
Category
    +
MITRE ATT&CK Techniques
    +
EntityType
    +
ThreatFamily
    +
LastVerdict
        │
        ▼
     TicketText
```

The purpose of `TicketText` was to expose more security context to the NLP pipeline than would be available from a single textual field.

For example, an incident record could contain information describing:

- The alert itself
- Its security category
- Associated ATT&CK techniques
- The affected entity
- Threat-family information
- Previous security verdicts

Combining these fields created a richer representation for downstream feature extraction.

---

## Why Combine Multiple Fields?

Cybersecurity incidents are rarely described completely by a single attribute.

An `AlertTitle` might contain a short description such as:

```text
Suspicious login detected
```

but additional fields may provide important context such as:

```text
Category: Credential Access
EntityType: User
ThreatFamily: Unknown
LastVerdict: Suspicious
MITRE Technique: T1078
```

Combining several attributes allows the feature-engineering pipeline to represent more of the available incident context.

This is particularly useful when individual text fields are short, ambiguous, or incomplete.

---

# TF-IDF Representation

## Purpose

Term Frequency–Inverse Document Frequency (**TF-IDF**) was used to transform cybersecurity text into numerical features.

TF-IDF assigns greater importance to terms that are informative within a document while reducing the influence of words that appear very frequently across the entire dataset.

The technique provides a sparse representation of explicit textual patterns.

---

## Configuration

The reported TF-IDF configuration used:

```text
Maximum features: 300
Stop words:       English
N-gram range:     (1, 2)
```

Equivalent configuration in scikit-learn is conceptually:

```python
TfidfVectorizer(
    max_features=300,
    stop_words="english",
    ngram_range=(1, 2)
)
```

The repository notebook contains the implementation used during the project.

---

## Unigrams and Bigrams

The configuration:

```text
ngram_range=(1, 2)
```

allows the vectorizer to consider:

- Individual words
- Two-word combinations

Examples of individual cybersecurity terms could include:

```text
phishing
malware
credential
ransomware
login
suspicious
```

Potential multi-word patterns could include expressions such as:

```text
suspicious login
credential access
malicious file
phishing email
```

Bigrams can provide additional context that may be lost when words are considered independently.

---

## Advantages of TF-IDF

TF-IDF offers several benefits for cybersecurity text classification:

- Computational efficiency
- Straightforward implementation
- Sparse numerical representation
- Ability to identify explicit security terminology
- Compatibility with traditional machine-learning models
- Partial interpretability through vocabulary inspection

For example, security-specific words may provide strong signals when differentiating incident categories.

---

## Limitations of TF-IDF

TF-IDF primarily represents lexical occurrence.

It does not naturally understand that differently worded descriptions may have similar meanings.

For example:

```text
User credentials were stolen
```

and:

```text
Account authentication information was compromised
```

may describe related security events while sharing relatively few identical terms.

This limitation motivated the addition of contextual sentence embeddings.

---

# Transformer-Based Sentence Embeddings

## Model

The experiment generated contextual sentence embeddings using Sentence Transformers with:

```text
all-MiniLM-L6-v2
```

This model converts text into dense numerical vectors intended to represent semantic meaning.

---

## Terminology Note

The original dissertation sometimes refers to this feature representation as **DistilBERT embeddings**.

The preserved implementation uses:

```python
SentenceTransformer("all-MiniLM-L6-v2")
```

Therefore, this repository describes the implementation more precisely as:

> **Sentence Transformer embeddings generated using `all-MiniLM-L6-v2`.**

References to `BERT_*` feature names in the original analysis are retained where necessary because those labels were used for embedding dimensions in the experimental feature matrix.

They should not be interpreted as evidence that the implementation directly used a DistilBERT classifier.

---

## Why Sentence Embeddings?

Sentence embeddings complement TF-IDF by representing contextual similarity.

Consider two security descriptions:

```text
A user entered credentials into a fraudulent login page.
```

and:

```text
Authentication details were captured through a fake portal.
```

A lexical representation may treat these sentences as substantially different because they use different words.

A contextual embedding model can potentially represent them as semantically related.

This is useful in cybersecurity because similar incidents can be described using many different phrases.

---

## Security Context Captured

The dissertation discusses semantic relationships involving incidents such as:

- Credential compromise
- Phishing attempts
- Suspicious authentication
- Malware activity
- Login anomalies

Sentence embeddings were intended to capture contextual relationships between these types of security descriptions even when their wording differed.

---

## Dense Representation

Unlike TF-IDF, which produces sparse features, sentence-transformer models generate dense numerical vectors.

Conceptually:

```text
Cybersecurity Incident Text
            │
            ▼
Sentence Transformer
            │
            ▼
Dense Semantic Vector
```

The numerical dimensions collectively represent characteristics of the input text.

Individual embedding dimensions generally do not correspond to simple human-readable concepts.

---

# Structured Cybersecurity Metadata

The project did not rely exclusively on text.

Structured cybersecurity attributes were also incorporated into the feature representation.

Examples included information relating to:

- Entity type
- Threat family
- Operating-system family
- Security verdict
- Incident category
- MITRE ATT&CK techniques

Categorical information was converted into numerical representations so it could be combined with the NLP-derived features.

---

## Label Encoding

Categorical attributes were transformed into numerical form using label encoding.

Conceptually:

```text
User        → 0
Device      → 1
Mailbox     → 2
IP Address  → 3
```

The exact numerical value assigned to a category does not imply that one category is inherently greater or more important than another.

The encoding simply allows categorical values to be represented numerically for the machine-learning pipeline.

---

## Encoding Limitation

Label encoding can introduce an artificial numerical ordering between categories.

For example:

```text
User   = 0
Device = 1
File   = 2
```

does **not** mean that:

```text
File > Device > User
```

in any meaningful cybersecurity sense.

This is an important methodological limitation.

Alternative approaches such as one-hot encoding or learned categorical representations could be investigated in future versions of the project.

---

# Hybrid Feature Representation

The central feature-engineering strategy was to combine:

```text
TF-IDF
   +
Sentence Transformer Embeddings
   +
Structured Cybersecurity Metadata
```

into a single machine-learning feature matrix.

Each component contributes different information.

| Feature Type | Primary Contribution |
| --- | --- |
| TF-IDF | Explicit words and phrases |
| Sentence embeddings | Semantic/contextual relationships |
| Structured metadata | Security-specific categorical context |

---

## Complementary Representations

### TF-IDF answers:

> Which explicit words or phrases appear to be important?

### Sentence embeddings answer:

> What does the incident description mean in context?

### Structured metadata answers:

> What additional security context is associated with the incident?

Combining these representations was intended to give the classifiers access to all three perspectives.

---

## Final Feature Space

The dissertation reports a final feature matrix containing:

```text
50,000 observations
694 predictive features
```

The features were derived from the combined textual and structured representation.

The reported dataset was subsequently divided into:

```text
Training set: 40,000 records
Testing set:  10,000 records
```

before model evaluation.

---

# Feature Importance Analysis

Random Forest feature importance was used to examine which dimensions contributed most strongly to prediction.

The reported analysis identified contributions from several feature families.

Examples included:

### TF-IDF-Derived Features

```text
TFIDF_293
TFIDF_294
TFIDF_171
TFIDF_295
TFIDF_286
```

### Transformer-Derived Features

```text
BERT_215
BERT_193
BERT_202
BERT_188
BERT_91
```

### Structured Features

```text
EntityType
```

The presence of multiple feature types among the highly ranked variables suggests that the model used information from more than one representation family.

---

## Important Interpretability Limitation

Feature names such as:

```text
TFIDF_293
```

and:

```text
BERT_215
```

are numerical feature identifiers.

They are not automatically meaningful cybersecurity concepts.

For TF-IDF features, interpretation requires mapping the feature index back to the original vocabulary.

For example:

```python
vectorizer.get_feature_names_out()
```

could be used to determine which term corresponds to a particular TF-IDF position.

Transformer embedding dimensions are less directly interpretable because an individual embedding dimension generally does not correspond to one specific semantic concept.

Therefore:

> Feature importance measures predictive contribution, not causal importance or direct cybersecurity meaning.

---

# MITRE ATT&CK Context

MITRE ATT&CK information was included among the cybersecurity attributes available in the dataset.

This provided standardised context relating incidents to documented adversary techniques.

The research also examined the frequency distribution of ATT&CK techniques represented within the dataset.

The distribution was highly uneven, with some techniques appearing much more frequently than others.

This has implications for feature engineering and classification because models have fewer examples from which to learn underrepresented attack behaviours.

---

## ATT&CK Mapping Limitation

The presence of `MitreTechniques` within the dataset should not be interpreted as evidence that this project created a complete independent MITRE ATT&CK mapping engine.

The project used ATT&CK-related information available within the source dataset as part of the experimental cybersecurity classification workflow.

This distinction is important when describing the project professionally.

---

# Rule-Based Features vs Learned Features

The project also implemented a simple rule-based baseline using predefined cybersecurity keywords.

Example phishing-related terms included:

```text
phishing
credential
login
fake
email
spoof
```

Example malware-related terms included:

```text
malware
trojan
ransomware
worm
virus
```

This provides a useful contrast between manually defined and learned representations.

---

## Rule-Based Approach

Advantages:

- Simple
- Fast
- Easy to explain
- Predictable

Limitations:

- Depends on predefined vocabulary
- Limited semantic understanding
- Vulnerable to wording variation
- Requires manual maintenance

---

## TF-IDF

Advantages:

- Automatically learns important vocabulary
- Efficient
- Partially interpretable
- Captures explicit textual patterns

Limitations:

- Limited contextual understanding
- Sparse representation
- Vocabulary-dependent

---

## Sentence Embeddings

Advantages:

- Context-sensitive
- Dense representation
- Better representation of semantic similarity
- Less dependent on identical wording

Limitations:

- More computationally expensive
- Less directly interpretable
- Performance depends on embedding quality

---

# Why the Hybrid Approach Matters

Cybersecurity incident data contains multiple forms of information.

A ticket may contain:

```text
Explicit vocabulary
        +
Semantic meaning
        +
Structured security context
```

No single representation captures all of these perfectly.

The hybrid approach attempted to combine their strengths:

```text
Lexical Signal
    +
Semantic Signal
    +
Security Metadata
        │
        ▼
More Complete Incident Representation
```

This representation was then provided to:

- Random Forest
- Gradient Boosting
- XGBoost

for comparative evaluation.

---

# Findings

The feature-engineering analysis supports several conclusions.

### 1. Textual information contributed predictive value

TF-IDF-derived features appeared among the highly ranked Random Forest features.

### 2. Contextual embeddings also contributed

Several transformer-derived embedding dimensions appeared among important features.

### 3. Structured metadata remained useful

Structured security attributes such as `EntityType` also contributed to model prediction.

### 4. Multiple feature families were complementary

The model did not appear to depend entirely on either traditional NLP or transformer-derived representations.

### 5. Feature engineering did not eliminate class imbalance

Even with the richer representation, minority classes remained difficult to classify reliably.

Feature quality therefore represents only one component of overall model performance.

---

# Potential Improvements

Future feature-engineering work could investigate:

- Mapping TF-IDF feature indices back to vocabulary terms
- One-hot encoding for categorical variables
- Target encoding where appropriate
- Feature scaling where required
- Dimensionality reduction
- Feature selection
- Alternative sentence-transformer models
- Fine-tuned cybersecurity embeddings
- Full transformer classification
- SHAP-based explainability
- Additional ATT&CK-derived features
- Security-event temporal features
- Organisation-specific ticket context

These approaches could improve both predictive performance and interpretability.

---

# Reproducibility Note

The dissertation reports a feature matrix generated from:

```text
50,000 observations
694 predictive features
```

The preserved notebook currently contains a smaller configurable dataset-loading limit.

Therefore, exact feature-matrix dimensions and resulting model performance may differ when the notebook is executed without reproducing the original dissertation configuration.

The repository intentionally distinguishes between:

- Reported dissertation results
- Preserved notebook implementation
- Future reproduced results

---

# Scope

The feature-engineering pipeline demonstrates an experimental approach to representing cybersecurity incidents for machine-learning classification.

It does **not** establish:

- A production feature pipeline
- Real-time SIEM feature extraction
- Production MITRE ATT&CK mapping
- Universal cybersecurity text representations
- Causal relationships between features and incidents
- Production-ready model explainability

The project is intended to demonstrate how traditional NLP, contextual embeddings, and structured cybersecurity information can be combined for experimental incident classification.

---

## Related Documentation

- [Methodology](methodology.md)
- [Model Evaluation](model-evaluation.md)
- [Dataset Documentation](../data/README.md)
- [Original Notebook](../notebooks/cybersecurity-incident-classification.ipynb)
