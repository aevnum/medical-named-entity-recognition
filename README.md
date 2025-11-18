# Medical Named Entity Recognition

A comparison of general-purpose and biomedical language models for clinical Named Entity Recognition (NER).
The project fine-tunes several transformer models on the **MACCROBAT 2018/2020** case-report dataset, then evaluates their robustness on two external benchmark datasets (NCBI Disease and BC5CDR).

## Overview

This repo contains:

* Training and evaluation code for clinical NER
* Fine-tuning on MACCROBAT case reports (83 tags → 41 entity types + “O”)
* External evaluation on NCBI Disease and BC5CDR (collapsed to Disease / Chemical)
* Experiment tracking with Weights & Biases (wandb)
* Benchmarks across five models (DistilBERT, ModernBERT, BioMedBERT, Bioformer, BioBERT)

## Datasets

### **Primary Fine-Tuning Dataset**

| Dataset                   | Source                         | Notes                                                                                           |
| ------------------------- | ------------------------------ | ----------------------------------------------------------------------------------------------- |
| **MACCROBAT (2018–2020)** | `ktgiahieu/maccrobat2018_2020` | Case-report domain. 400 docs total. Highly imbalanced distribution of 41 clinical entity types. |

Approximate split (iterative stratification used to ensure all entities appear in train/val):

* **Train:** ~350
* **Val:** ~50

Entity counts (top few for reference):

| Entity               | Count  |
| -------------------- | ------ |
| Diagnostic_procedure | 11,747 |
| Sign_symptom         | 6,563  |
| Lab_value            | 6,303  |
| Biological_structure | 5,119  |
| Detailed_description | 3,269  |
| Disease_disorder     | 2,894  |
| …                    | …      |

Total tags: **83 BIO tags** (41 entity types + “outside”).

### **External Evaluation Datasets**

| Dataset          | Source                | Entities          | Notes                              |
| ---------------- | --------------------- | ----------------- | ---------------------------------- |
| **NCBI Disease** | `bigbio/ncbi_disease` | Disease           | Abstract domain, not case reports. |
| **BC5CDR**       | `bigbio/bc5cdr`       | Disease, Chemical | Abstract domain.                   |

Since MACCROBAT → case-report domain, external evaluation tests domain transfer and robustness.

During evaluation, all predicted MACCROBAT entity types are *collapsed* to:

* **Disease**
* **Chemical** (only for BC5)

## Models

All models loaded from HuggingFace Hub:

| Model Name    | HF Model                                                        |
| ------------- | --------------------------------------------------------------- |
| DistilBERT    | `distilbert/distilbert-base-uncased`                            |
| ModernBERT    | `answerdotai/ModernBERT-base`                                   |
| BioMedBERT    | `microsoft/BiomedNLP-BiomedBERT-base-uncased-abstract-fulltext` |
| Bioformer-16L | `bioformers/bioformer-16L`                                      |
| BioBERT v1.1  | `dmis-lab/biobert-v1.1`                                         |

## Training Setup

* **Epochs:** 70
* **Learning rate:** 2e-5
* **Batch size:** 8 (ModernBERT used 4 due to memory)
* **Precision:** fp32 (ModernBERT in fp16)
* **Early stopping** enabled
* **Runs logged to wandb**
* **Output dir:** `/kaggle/working/ner_runs`

## Results

### **MACCROBAT Validation Performance**

| Model             | Precision | Recall | F1        | Accuracy |
| ----------------- | --------- | ------ | --------- | -------- |
| **DistilBERT**    | 0.819     | 0.917  | **0.865** | 0.9569   |
| **ModernBERT**    | 0.893     | 0.935  | **0.913** | 0.9694   |
| **BioMedBERT**    | 0.881     | 0.952  | **0.915** | 0.9669   |
| **Bioformer-16L** | 0.802     | 0.905  | **0.850** | 0.9527   |
| **BioBERT v1.1**  | 0.884     | 0.963  | **0.922** | 0.9675   |

**Top performer on MACCROBAT:** **BioBERT (F1 = 0.922)**
Close behind: BioMedBERT (0.9158) and ModernBERT (0.9136).

---

### **NCBI Disease (External Evaluation)**

| Model       | Precision  | Recall     | F1         |
| ----------- | ---------- | ---------- | ---------- |
| DistilBERT  | 0.4007     | 0.4009     | **0.4008** |
| ModernBERT  | 0.0661     | 0.1737     | **0.0958** |
| BioMedBERT  | 0.6162     | 0.3217     | **0.4227** |
| Bioformer   | 0.5357     | 0.3092     | **0.3921** |
| **BioBERT** | **0.6016** | **0.4317** | **0.5027** |

**Top performer on NCBI:** **BioBERT (0.5027 F1)**

---

### **BC5CDR (External Evaluation)**

All models struggle heavily due to:

* domain shift (case reports → biomedical abstracts)
* entity schema mismatch (83 labels → 2 labels)
* conservative predictions after collapsing labels

| Model      | Precision | Recall | F1     |
| ---------- | --------- | ------ | ------ |
| DistilBERT | 0.0164    | 0.0160 | 0.0162 |
| ModernBERT | 0.0154    | 0.0320 | 0.0208 |
| BioMedBERT | 0.0169    | 0.0096 | 0.0122 |
| Bioformer  | 0.0156    | 0.0094 | 0.0118 |
| BioBERT    | 0.0168    | 0.0098 | 0.0124 |

Performance is uniformly low — expected given domain differences and label collapse.

## Observations

* **BioBERT consistently performs best**, both in-domain (MACCROBAT) and out-of-domain (NCBI).
* **BioMedBERT and ModernBERT** follow closely in-domain, but struggle more during domain transfer.
* **Label granularity mismatch** (83 clinical tags → 2 biomedical tags) severely limits transfer performance on BC5CDR.
* **MACCROBAT annotations are noisy**, which likely caps overall scores.
* Case report → abstract shift significantly affects the external performance.

## Confidence Score Analysis and Error Analysis

* Present in individual model notebooks.

Models, metrics, and logs are all synced to Weights & Biases.

## License

MIT
