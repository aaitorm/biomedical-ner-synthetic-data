# Can LLMs Replace Annotated Data? Synthetic Data Generation for Biomedical NER

**Individual project — Text Data Mining (3rd year, AI Degree)**  
Universidad del País Vasco / Euskal Herriko Unibertsitatea (UPV/EHU)  
Author: Aitor Milicua Fernandez

---

## Overview

This project investigates whether LLM-generated synthetic data can substitute human-annotated corpora for training biomedical Named Entity Recognition (NER) models. A BioBERT model is fine-tuned exclusively on synthetic data generated via instruct prompting and evaluated against the BC5CDR benchmark.

**Research questions:**
- Can a model trained exclusively on LLM-generated data approach the performance of one trained on human-annotated data?
- How does the volume of synthetic data affect downstream NER performance?
- Does the prompting strategy used for generation affect data quality?

---

## Task

Given a biomedical text, identify and classify named entities into two categories using IOB2 format:
- **Chemical**: drugs, compounds, and biomedical substances
- **Disease**: diseases, disorders, and medical conditions

**Input:** tokenised biomedical sentence  
**Output:** IOB2 label sequence (`O`, `B-Chemical`, `I-Chemical`, `B-Disease`, `I-Disease`)

---

## Dataset

| Split | Source | Examples |
|---|---|---|
| Evaluation and baseline training | BC5CDR (tner/bc5cdr) | 5,228 train / 5,865 test |
| Synthetic training | Generated via Qwen2.5-3B-Instruct | 100 / 500 / 800 per prompt |

BC5CDR contains 1,500 PubMed abstracts manually annotated with Chemical and Disease mentions.

---

## Experimental Design

**Axis 1 — Data volume** (Few-shot B prompt)

| Corpus | Examples |
|---|---|
| FSB-100 | 100 |
| FSB-500 | 500 |
| FSB-800 | 800 |

**Axis 2 — Prompting strategy** (800 examples each)

| Corpus | Prompt type |
|---|---|
| ZS-800 | Zero-shot: instructions and entity definitions |
| FSA-800 | Few-shot A: instructions and isolated entity examples |
| FSB-800 | Few-shot B: instructions and full annotated BC5CDR sentence |

---

## Results

| Model | F1 | Precision | Recall |
|---|---|---|---|
| Baseline (BC5CDR real) | 0.883 | 0.865 | 0.902 |
| ZS-800 | 0.318 | 0.693 | 0.206 |
| FSA-800 | 0.339 | 0.627 | 0.233 |
| FSB-100 | 0.036 | 0.040 | 0.032 |
| FSB-500 | 0.322 | 0.554 | 0.227 |
| FSB-800 | 0.371 | 0.677 | 0.255 |

Key findings:
- Synthetic data enables above-chance NER performance but a gap of 0.51 F1 points remains vs baseline
- More data consistently improves performance; richer prompts produce better training examples
- All synthetic models show high Precision but low Recall (conservative prediction behaviour)

---

## Models

| Role | Model |
|---|---|
| NER fine-tuning | BioBERT dmis-lab/biobert-base-cased-v1.2 |
| Synthetic data generation | Qwen2.5-3B-Instruct (local GPU) |

---

## Repository Structure

    biomedical-ner-synthetic-data/
    |
    |-- README.md
    |-- llmcreated-vs-annotated-data-bioner.ipynb
    |
    |-- corpus/
    |   |-- corpus_fewshot_b_100.json
    |   |-- corpus_fewshot_b_500.json
    |   |-- corpus_fewshot_b_800.json
    |   |-- corpus_zeroshot_800.json
    |   |-- corpus_fewshot_a_800.json
    |
    |-- results/
    |   |-- resultados_finales.csv
    |   |-- resultados_graficos.png
    |   |-- comparativa_corpus.png
    |   |-- confusion_matrix.png
    |
    |-- report/
        |-- milicua_MDT_report.pdf

---

## Environment

- **Platform:** Kaggle (GPU P100) for training and Google Colab (T4 GPU) for generation
- **Key libraries:** transformers, datasets, seqeval, torch

To install dependencies:

    pip install transformers datasets seqeval

---

## Generation Pipeline

    Qwen2.5-3B-Instruct (local GPU)
            |
            v instruct prompting (ZS / FSA / FSB)
            |
            v JSON output: {text, entities}
            |
            v automatic IOB2 conversion
            |
            v synthetic corpus (.json)
            |
            v BioBERT fine-tuning
            |
            v evaluation on BC5CDR test set

Note on generation: Gemini 2.0 Flash API and HuggingFace Inference API (Qwen2.5-72B) were tested but discarded due to rate limits and credit exhaustion. Local generation with Qwen2.5-3B-Instruct was adopted as the final approach.

---

## References

- Li et al. (2016). BioCreative V CDR task corpus. Database.
- Lee et al. (2020). BioBERT: a pre-trained biomedical language representation model. Bioinformatics.
