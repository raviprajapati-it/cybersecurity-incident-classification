# Dataset

## Microsoft Security Incident Prediction (GUIDE)

This project uses the Microsoft Security Incident Prediction dataset (GUIDE) as the source of cybersecurity incident records.

The dataset contains structured and textual security-event information suitable for cybersecurity incident classification, Natural Language Processing (NLP), and MITRE ATT&CK-oriented analysis.

## Features Used

The original research selected the following fields:

- `Category`
- `MitreTechniques`
- `EntityType`
- `EvidenceRole`
- `SuspicionLevel`
- `LastVerdict`
- `ThreatFamily`
- `OSFamily`
- `CountryCode`
- `AlertTitle`
- `IncidentGrade`

`AlertTitle` provides textual security-event information used as part of the NLP feature-engineering pipeline.

Several cybersecurity attributes were also combined into a derived `TicketText` representation for text classification.

## Research Dataset Size

The dissertation reports experiments using a sample of:

- **50,000 cybersecurity incident records**
- **40,000 training records (80%)**
- **10,000 testing records (20%)**

The repository notebook may use a smaller row limit depending on its execution configuration and available computational resources.

Therefore, results reproduced by running the notebook as currently configured may differ from the metrics reported in the dissertation.

## Dataset Availability

The dataset is **not redistributed in this repository**.

Users wishing to reproduce or extend the analysis should obtain the Microsoft Security Incident Prediction dataset from its original distribution source and configure the notebook to reference the locally downloaded training data.

## Why the Dataset Is Not Included

Excluding the dataset keeps this repository focused on:

- Analysis code
- Feature engineering
- Machine-learning methodology
- Model evaluation
- Reproducibility instructions

It also avoids unnecessarily duplicating a large externally maintained dataset.

## Scope

This project uses the dataset for offline experimental cybersecurity incident classification.

It does not represent:

- Live SOC ticket ingestion
- Production SIEM integration
- Real-time threat detection
- Automated production incident response
- A deployed enterprise classification service
