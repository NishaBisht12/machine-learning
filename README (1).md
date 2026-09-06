# 🌊 AquaTrace — Oil Spill Detection & Vessel Attribution System

AquaTrace is an end-to-end system that detects oil spills in Synthetic Aperture Radar (SAR) satellite imagery using a trained CNN, and correlates detections with real AIS vessel-tracking data to identify likely responsible vessels — all surfaced through a live, interactive dashboard.

> **Status:** Fully functional end-to-end demo — real dataset, trained model, real AIS data, live backend, and connected frontend.

---

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Stage 1: Dataset](#stage-1-dataset-acquisition--verification)
- [Stage 2: Model Training](#stage-2-cnn-model-development--training)
- [Stage 3: AIS Vessel Correlation](#stage-3-ais-vessel-correlation-logic)
- [Stage 4: Backend API](#stage-4-backend-api)
- [Stage 5: Frontend Dashboard](#stage-5-frontend-dashboard)
- [Setup & Installation](#setup--installation)
- [Project Structure](#project-structure)
- [Results Summary](#results-summary)
- [Future Improvements](#future-improvements)

---

## Overview

Real oil spill detection combines two very different problems: (1) **spotting** a spill in satellite imagery, which is hard because natural look-alikes (wind slicks, algae, rain) mimic oil's radar signature, and (2) **attributing** that spill to a responsible vessel using ship tracking data. This project builds a working pipeline for both, using real public datasets throughout — no synthetic/fabricated data.

**Pipeline:** SAR image → CNN classifier → confidence score → AIS proximity/time correlation → ranked candidate vessels → dashboard visualization.

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌─────────────┐
│  SAR Image  │ --> │  CNN Model   │ --> │  Flask Backend │ --> │  React       │
│  (upload)   │     │ (TensorFlow) │     │  /predict      │     │  Dashboard   │
└─────────────┘     └──────────────┘     │  /find_vessels │     │  (AquaTrace) │
                     ┌──────────────┐     └───────┬───────┘     └─────────────┘
                     │  AIS Vessel  │ ─────────────┘
                     │  Data (NOAA) │
                     └──────────────┘
```

---

## Stage 1: Dataset Acquisition & Verification

**Objective:** Acquire a labeled SAR imagery dataset to train a binary oil/no-oil classifier.

- Dataset: [CSIRO Sentinel-1 SAR Oil/No-Oil Dataset](https://doi.org/10.25919/4v55-dn16) — 5,538 grayscale 400×400px image chips
- Classes: `Class_0` (no oil / look-alikes, 3,695 images) and `Class_1` (oil, 1,843 images)
- Verified dataset integrity via file counts and visual inspection (`check_data.py`)

**Tools:** Python, OpenCV, Matplotlib

## Stage 2: CNN Model Development & Training

**Objective:** Train a CNN to classify SAR chips as oil / no-oil.

- Preprocessed all images to 128×128, normalized, split 70/15/15 (train/val/test)
- Applied class weighting to correct for the ~2:1 class imbalance
- Trained a 3-block CNN (Conv2D + MaxPooling ×3, Dense, Dropout, Sigmoid output — 3.3M parameters) with early stopping

**Results (test set, n=831):**

| Metric | No Oil (0) | Oil (1) |
|---|---|---|
| Precision | 0.85 | 0.69 |
| Recall | 0.84 | 0.71 |
| F1-score | 0.85 | 0.70 |

**Overall accuracy: 80%**

**Tools:** TensorFlow/Keras, NumPy, scikit-learn

## Stage 3: AIS Vessel Correlation Logic

**Objective:** Given a spill's location/time, identify and rank nearby vessels from real AIS tracking data.

- Source: [NOAA Marine Cadastre AIS data](https://marinecadastre.gov/ais/) (Jan 28, 2023)
- Filtered ~10M+ raw records down to 2,042,399 rows for the Gulf of Mexico region
- Matching logic: Haversine distance formula + ±3 hour time window + 20nm proximity cutoff, deduplicated per vessel (MMSI), ranked by distance

**Tools:** Python, Pandas, NumPy

## Stage 4: Backend API

**Objective:** Serve the trained model and AIS logic as callable endpoints.

- `POST /predict` — accepts an image, returns `{ is_oil_spill, confidence }`
- `POST /find_vessels` — accepts `{ latitude, longitude, detected_at }`, returns ranked `candidate_vessels`

**Tools:** Flask, Flask-CORS, TensorFlow, Pandas

## Stage 5: Frontend Dashboard

**Objective:** Visualize incidents and vessel attribution in a live, interactive UI.

- React + Vite + Tailwind CSS dashboard ("AquaTrace")
- KPI summary cards, SAR map view, incident log, AIS vessel correlation table, live detection pipeline sidebar
- "Run live detection" uploads an image → calls `/predict` → calls `/find_vessels` → updates the dashboard in real time

**Tools:** React, Vite, Tailwind CSS, lucide-react

---

## Setup & Installation

### Prerequisites
- Python 3.9+
- Node.js 18+
- ~2GB free disk space (dataset + AIS data)

### 1. Clone the repo
```bash
git clone https://github.com/<your-username>/aquatrace.git
cd aquatrace
```

### 2. Backend setup
```bash
cd backend
python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt
python app.py
```
Runs on `http://localhost:5000`

> **Note:** `oil_spill_model.h5` and `ais_gulf_filtered.csv` are not committed to this repo (see `.gitignore`) due to file size. See [Project Structure](#project-structure) for how to regenerate them from the training scripts.

### 3. Frontend setup
```bash
cd frontend
npm install
npm run dev
```
Runs on `http://localhost:5173`

### 4. Use it
Open `http://localhost:5173`, click **Run live detection**, and upload a SAR image chip (e.g. from `dataset/data/Class_1/`).

---

## Project Structure

```
aquatrace/
├── dataset/                  # CSIRO SAR dataset (not committed — see below)
│   └── data/
│       ├── Class_0/
│       └── Class_1/
├── model/
│   ├── load_data.py
│   ├── split_data.py
│   ├── check_imbalance.py
│   ├── build_model.py
│   ├── train_model.py
│   └── evaluate_model.py
├── ais_data/
│   ├── filter_ais.py
│   ├── spill_scenario.py
│   └── match_vessels.py
├── backend/
│   ├── app.py
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── AquaTraceDashboard.jsx
│   │   └── App.jsx
│   └── package.json
└── README.md
```

## Results Summary

| Stage | Key Output |
|---|---|
| Dataset | 5,538 real SAR image chips verified |
| Model | 80% test accuracy, 71% recall on oil class |
| AIS | 2M+ real vessel records processed, distance/time correlation working |
| Backend | Two working REST endpoints, tested end-to-end |
| Frontend | Live dashboard, real predictions and vessel matches confirmed working |

## Future Improvements

- [ ] Weighted vessel scoring (vessel type, speed anomalies, AIS signal gaps) instead of distance-only ranking
- [ ] Pixel-level spill segmentation (U-Net) instead of whole-chip classification, to estimate spill area/shape
- [ ] Drift-back modeling using ocean current/wind data to estimate spill origin point instead of using the detection point directly
- [ ] Functional Overview / Incidents / Vessels tabs in the dashboard nav
- [ ] Deploy backend + frontend so the demo is publicly accessible (not just localhost)

---

## License / Data Attribution

- SAR dataset: CSIRO, *Sentinel-1 SAR image dataset of oil- and non-oil features for machine learning*, [DOI: 10.25919/4v55-dn16](https://doi.org/10.25919/4v55-dn16)
- AIS data: [NOAA Office for Coastal Management, Marine Cadastre](https://marinecadastre.gov/ais/)
