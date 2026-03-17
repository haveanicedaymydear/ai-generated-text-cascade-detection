# Detecting Machine-Generated Text in the Wild with a Two-Stage Cascade (DTD → LRC)

A final year project on **open-world AI-generated text detection**.  
This repository implements a practical two-stage cascade for detecting machine-generated text under changing domains, model families, and writing styles.

The core idea is simple:

- **Stage 1 (DTD)** handles most inputs quickly using lightweight statistical and linguistic evidence.
- **Stage 2 (LRC)** is only triggered for uncertain cases and performs deeper analysis using multi-model response patterns.

This design aims to balance **speed, robustness, and interpretability** for real-world use.

---

## Overview

Traditional AI-text detectors often fail when the generator, domain, or writing style changes.  
A single detector may perform well on one benchmark but become brittle when exposed to paraphrased text, platform text, student writing, or outputs from newer models.

This project addresses that problem with a **two-stage cascade**:

1. **DTD (Document/Text Detector)**  
   A lightweight first-stage detector based on:
   - TF-IDF features
   - lexical features
   - syntactic features
   - punctuation features
   - repetition features
   - structural / style indicators  
   These features are combined and passed into a traditional classifier to produce:
   - AI probability
   - label prediction
   - confidence score

2. **LRC (Ladder Response Curve Detector)**  
   A second-stage detector used only when Stage 1 is uncertain.  
   It measures how the **normalised negative log-likelihood (NLL/byte)** changes across multiple model scales and families, then extracts low-dimensional curve features such as:
   - early drop
   - late drop
   - overall drop
   - drop ratio
   - concavity / shape indicators

The system routes a text to Stage 2 only when the Stage 1 confidence falls inside a gate band of **[0.41, 0.61]**. This preserves throughput while reserving expensive computation for hard cases.

---

## Why This Project Matters

This project was designed for **practical deployment**, not just benchmark fitting.

It focuses on three goals:

- **Accuracy**: maintain strong detection performance under open-world variation
- **Efficiency**: avoid running expensive multi-model analysis on every sample
- **Interpretability**: expose evidence such as feature importance, confidence band behaviour, LRC curves, and family-scale response matrices

Instead of relying on one fragile signal, the system combines multiple evidence sources and degrades more gracefully when shallow features become unreliable.

---

## Method

### Stage 1 — DTD

The first stage is a lightweight detector using:

- **TF-IDF** vectors
- **lexical features**  
  e.g. average word length, long-word ratio, vocabulary diversity
- **syntactic features**  
  e.g. sentence length statistics
- **punctuation features**
- **repetition features**
- **structural/style features**  
  e.g. paragraph statistics, character variety, academic-style markers

These features are concatenated and sent to a traditional classifier for a fast first decision.

### Stage 2 — LRC

The second stage treats language models as **measurement instruments**.

For the same text, the system computes **NLL/byte** across multiple model scales and families, then examines the resulting response pattern.  
Rather than using a raw absolute score, it extracts shape-based evidence such as:

- early drop
- late drop
- overall drop
- drop ratio
- concavity flag

This stage is intended to capture cross-family behaviour that shallow stylistic features may miss.

### Cascade Routing

The routing rule is:

- if Stage 1 confidence is **outside** `[0.41, 0.61]` → return Stage 1 result directly
- if Stage 1 confidence is **inside** `[0.41, 0.61]` → send the text to Stage 2

This keeps the system fast on easy cases and more careful on uncertain ones.

---

## System Architecture

```text
Input Text
   │
   ▼
Preprocessing / Cleaning / Normalisation
   │
   ▼
Stage 1: DTD
   ├─ TF-IDF features
   ├─ linguistic & stylistic features
   ├─ traditional classifier
   └─ AI probability + confidence
   │
   ├─ if confidence outside [0.41, 0.61] ──► Final Output
   │
   └─ if confidence inside [0.41, 0.61]
          │
          ▼
Stage 2: LRC
   ├─ multi-model NLL/byte scoring
   ├─ ladder response curve feature extraction
   ├─ calibrated second-stage classification
   └─ interpretable evidence artefacts
          │
          ▼
Final Output
