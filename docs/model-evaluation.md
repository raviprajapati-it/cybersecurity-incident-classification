# Model Evaluation

## Overview

This document presents the experimental evaluation of the machine-learning classifiers developed for cybersecurity incident classification.

Three supervised ensemble models were evaluated:

- Random Forest
- Gradient Boosting
- XGBoost

The models were trained using a hybrid feature representation combining:

- TF-IDF textual features
- Sentence-transformer embeddings
- Encoded structured cybersecurity metadata
- MITRE ATT&CK-related context

Because the dataset contained significant class imbalance, model performance was evaluated using multiple metrics rather than accuracy alone.

---

## Evaluation Metrics

The experiment considered:

- Accuracy
- Precision
- Recall
- Weighted F1-score
- Cross-validation accuracy
- Classification reports
- Confusion matrix analysis
- Receiver Operating Characteristic (ROC) curves
- Area Under the Curve (AUC)
- Precision-recall curves
- Feature importance

Using multiple evaluation methods was important because overall accuracy can conceal poor performance on underrepresented incident classes.

---

## Model Performance Comparison

The dissertation reports the following comparative results:

| Model | Accuracy | Weighted F1 | Cross-Validation Accuracy |
| --- | ---: | ---: | ---: |
| **Random Forest** | **69.28%** | **0.6963** | **75.96%** |
| XGBoost | 67.99% | 0.6862 | 74.97% |
| Gradient Boosting | 65.61% | 0.6653 | 71.98% |

Random Forest achieved the strongest overall reported performance across the three tested classifiers.

The differences between the models were relatively small, however, so these results should not be interpreted as evidence that Random Forest will universally outperform Gradient Boosting or XGBoost on cybersecurity classification problems.

---

## Random Forest

Random Forest was the highest-performing classifier in the reported experiment.

### Reported Performance

```text
Accuracy:                  69.28%
Weighted F1-score:         0.6963
Cross-validation accuracy: 75.96%
```

The model processed the combined high-dimensional feature representation containing:

- TF-IDF features
- Sentence-transformer embeddings
- Encoded cybersecurity metadata

Its ensemble structure was suitable for heterogeneous and potentially noisy cybersecurity incident data.

The Random Forest configuration used in the reported methodology included:

```text
Number of estimators:     150
Maximum tree depth:       15
Minimum samples split:    5
Class weight:             Balanced
```

---

## Gradient Boosting

Gradient Boosting produced the lowest overall reported performance of the three tested classifiers.

### Reported Performance

```text
Accuracy:                  65.61%
Weighted F1-score:         0.6653
Cross-validation accuracy: 71.98%
```

The reported configuration included:

```text
Number of estimators: 100
Learning rate:        0.08
```

Although the classifier learned predictive relationships from the feature space, it did not outperform Random Forest in the reported experiment.

---

## XGBoost

XGBoost achieved the second-highest overall reported performance.

### Reported Performance

```text
Accuracy:                  67.99%
Weighted F1-score:         0.6862
Cross-validation accuracy: 74.97%
```

The reported configuration included:

```text
Number of estimators: 150
Learning rate:        0.05
Maximum depth:        6
Subsample ratio:      0.8
```

XGBoost performed competitively but remained slightly below Random Forest across the reported overall metrics.

---

## Class-Level Performance

The dissertation reports the following Random Forest class-level results:

| Class | Precision | Recall | F1-score |
| --- | ---: | ---: | ---: |
| Class 0 | 0.72 | 0.80 | 0.76 |
| Class 1 | 0.56 | 0.49 | 0.52 |
| Class 2 | 0.80 | 0.68 | 0.74 |
| Class 3 | 0.12 | 0.69 | 0.21 |

These results demonstrate why overall accuracy should not be considered in isolation.

Classes 0 and 2 showed substantially stronger classification performance than Classes 1 and 3.

Class 3 is particularly important because it achieved relatively high recall but very low precision.

This means that although the classifier identified a substantial proportion of the true Class 3 observations, many predictions assigned to Class 3 were incorrect.

The result highlights the difficulty of evaluating minority classes using a single metric.

---

## Confusion Matrix Analysis

The reported Random Forest confusion matrix was:

```text
                 Predicted
              0      1      2      3
          ┌──────────────────────────
Actual 0  │ 3450   478    220    157
Actual 1  │  649  1058    369     80
Actual 2  │  710   358   2382     34
Actual 3  │   12     3      2     38
```

### Class 0

The model correctly classified:

```text
3450 / 4305
```

Class 0 therefore demonstrated comparatively strong classification performance.

### Class 1

Class 1 showed greater overlap with neighbouring classes.

The classifier correctly identified 1,058 observations but also produced substantial confusion with Classes 0 and 2.

This suggests that the feature patterns associated with Class 1 were less clearly separated.

### Class 2

The model correctly classified:

```text
2382 / 3484
```

Class 2 was another comparatively strong class.

### Class 3

The displayed confusion matrix contained only 55 true Class 3 observations:

```text
12 + 3 + 2 + 38 = 55
```

Of these, 38 were correctly classified.

However, the class-level precision of only `0.12` demonstrates that many observations from other classes were also incorrectly predicted as Class 3.

This is an important example of why confusion matrices and class-level metrics provide information that overall accuracy cannot.

---

## Class Imbalance

Class imbalance was one of the major challenges identified during the experiment.

The dataset contained substantially different numbers of observations across incident classes.

Synthetic Minority Oversampling Technique (**SMOTE**) was applied to the training data to increase minority-class representation.

SMOTE creates synthetic examples for minority classes rather than simply duplicating existing observations.

The objective was to reduce model bias toward majority classes.

However, the evaluation demonstrates that oversampling did not completely resolve the imbalance problem.

Minority classes continued to exhibit:

- Lower predictive stability
- Greater misclassification
- Poorer precision
- Greater sensitivity to overlapping incident characteristics

Class 3 provides the clearest example.

The experiment therefore demonstrates an important machine-learning limitation:

> Balancing training data can improve minority-class learning, but it does not guarantee reliable minority-class classification.

---

## ROC Analysis

Receiver Operating Characteristic analysis was performed for the Random Forest classifier.

The dissertation reports the following AUC values:

| Class | AUC |
| --- | ---: |
| Class 0 | 0.88 |
| Class 1 | 0.81 |
| Class 2 | 0.90 |
| Class 3 | 0.97 |

All four reported ROC curves performed above the random-classification reference line.

However, ROC AUC requires careful interpretation when evaluating imbalanced datasets.

For example, Class 3 achieved a reported AUC of `0.97` while simultaneously producing very low precision in the classification report.

This demonstrates why ROC AUC should not be interpreted independently of:

- Precision
- Recall
- F1-score
- Class support
- Confusion matrices
- Precision-recall behaviour

---

## Precision-Recall Analysis

Precision-recall curves were also used because they provide useful insight when class distributions are uneven.

The reported analysis showed stronger performance for the more highly represented classes and weaker behaviour for the minority class.

This reinforces the classification report and confusion-matrix findings.

Precision-recall analysis is particularly valuable in cybersecurity because operational datasets often contain significantly fewer examples of certain attack or incident categories.

---

## Feature Importance

Random Forest feature-importance analysis identified predictive contributions from several feature families.

### TF-IDF Features

Highly ranked TF-IDF-derived features included examples such as:

```text
TFIDF_293
TFIDF_294
TFIDF_171
TFIDF_295
TFIDF_286
```

### Transformer Embedding Features

Highly ranked embedding dimensions included examples such as:

```text
BERT_215
BERT_193
BERT_202
BERT_188
BERT_91
```

The original experiment used Sentence Transformer embeddings generated with `all-MiniLM-L6-v2`.

The `BERT_*` labels above are retained because they were used as feature names in the original analysis. They should not be interpreted as evidence that these dimensions correspond directly to human-readable BERT concepts.

### Structured Cybersecurity Metadata

Structured metadata also contributed to prediction.

One example appearing among the highly ranked features was:

```text
EntityType
```

The presence of TF-IDF, transformer-derived, and structured features among the important variables supports the hybrid feature-engineering design.

The classifier was not relying solely on a single representation type.

---

## Interpreting Feature Importance

Feature identifiers such as:

```text
TFIDF_293
BERT_215
```

represent numerical positions in the generated feature space.

They should not automatically be interpreted as directly meaningful cybersecurity concepts.

For TF-IDF features, meaningful interpretation requires mapping the numerical feature index back to the vectorizer vocabulary.

Transformer embedding dimensions are even less directly interpretable because individual dimensions generally do not correspond to simple human-readable concepts.

Feature importance therefore indicates predictive contribution rather than causal significance.

---

## MITRE ATT&CK Distribution

MITRE ATT&CK technique information was included within the cybersecurity dataset and analysed as part of the research.

The distribution analysis showed that technique frequencies were highly uneven.

Some ATT&CK techniques occurred far more frequently than others.

This has an important machine-learning implication:

> Underrepresented techniques provide fewer examples from which a classifier can learn.

MITRE ATT&CK information therefore provided both:

- Cybersecurity context for incident representation
- Insight into the distribution of attack behaviours within the dataset

The project does not claim that the model performs comprehensive production-grade MITRE ATT&CK technique classification.

---

## Key Findings

The evaluation produced several important findings.

### 1. Random Forest performed best overall

Random Forest achieved:

```text
Accuracy:                  69.28%
Weighted F1-score:         0.6963
Cross-validation accuracy: 75.96%
```

This was the strongest overall reported performance among the three tested classifiers.

### 2. XGBoost remained competitive

XGBoost achieved:

```text
Accuracy: 67.99%
```

Its performance remained relatively close to Random Forest.

### 3. Gradient Boosting ranked third

Gradient Boosting achieved:

```text
Accuracy: 65.61%
```

It produced the lowest overall result of the three tested models.

### 4. Overall accuracy concealed class-level weaknesses

Classes 0 and 2 were classified considerably more effectively than the weaker classes.

The results demonstrate why class-level precision, recall, F1-score, and confusion matrices are necessary.

### 5. Class imbalance remained significant

SMOTE improved representation of minority classes during training, but minority-class prediction remained challenging.

### 6. Hybrid feature engineering contributed predictive information

Important features originated from:

- TF-IDF
- Sentence-transformer embeddings
- Structured cybersecurity metadata

This supports the use of multiple representation types within the experimental pipeline.

### 7. No single evaluation metric was sufficient

Accuracy, F1, ROC AUC, confusion matrices, and precision-recall analysis revealed different aspects of classifier behaviour.

---

## Practical Interpretation

The reported Random Forest accuracy of **69.28%** should be treated as an experimental research result rather than a production-ready performance level.

The experiment demonstrates that meaningful classification patterns can be learned from cybersecurity incident data.

It also reveals several limitations that would need to be addressed before operational deployment.

These include:

- Minority-class performance
- Overlapping class characteristics
- Dataset representativeness
- Hyperparameter optimisation
- Model calibration
- Generalisation to unseen organisational data
- Integration with realistic SOC workflows
- Human analyst validation

---

## Potential Improvements

Future experimentation could investigate:

- Larger training datasets
- More balanced class distributions
- Alternative resampling strategies
- Class-specific threshold optimisation
- Additional feature-selection techniques
- Hyperparameter optimisation
- Logistic Regression and SVM baselines
- Larger transformer models
- Fine-tuned transformer classifiers
- Model calibration
- Explainability techniques such as SHAP
- Additional cybersecurity datasets
- External validation
- Realistic SOC ticket datasets

These improvements would help determine whether the observed experimental performance generalises beyond the original research environment.

---

## Reproducibility Note

The metrics documented here are the results reported in the original dissertation experiment.

The dissertation describes an experiment using:

```text
50,000 records
40,000 training records
10,000 testing records
```

The notebook preserved in this repository currently contains a smaller configurable dataset-loading limit:

```python
nrows=10000
```

Therefore:

> Running the preserved notebook exactly as currently configured should not be assumed to reproduce the dissertation's 50,000-record results.

The repository intentionally distinguishes between:

1. The original reported dissertation results
2. The preserved implementation notebook
3. Results produced by any future re-execution

This distinction prevents reproduced results from being presented as identical when the experimental configurations differ.

---

## Scope and Limitations

This evaluation represents an offline experimental machine-learning study.

The results do **not** establish:

- Production SOC performance
- Real-time threat-detection accuracy
- Generalisation to every cybersecurity dataset
- Generalisation to every MITRE ATT&CK technique
- Production-ready automated incident response
- A deployed enterprise classifier
- Replacement of human security analysts

The purpose of the project is to evaluate the opportunities and limitations of applying NLP, transformer-based representations, and ensemble machine learning to cybersecurity incident classification.

---

## Related Documentation

- [Methodology](methodology.md)
- [Dataset Documentation](../data/README.md)
- [Original Notebook](../notebooks/cybersecurity-incident-classification.ipynb)
