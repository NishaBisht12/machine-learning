# Satellite-Based Oil Spill Detection & Ship Attribution System

## Project Overview

This system detects oil spills from satellite radar imagery and cross-references ship traffic data to identify likely source vessels. It combines remote sensing, machine learning, geospatial analysis, and a web dashboard into a single end-to-end pipeline.

**Core idea:** Sentinel-1 radar imagery tells us *where* a spill is. AIS ship-tracking data tells us *which ships were nearby*. Correlating the two in space and time surfaces the most likely culprit vessels.

---

## 1. Data Sources

| Source | Purpose | Notes |
|---|---|---|
| **Sentinel-1 (SAR satellite)** | Detects oil slicks on the ocean surface | Oil dampens surface waves, so slicks appear as dark patches in radar backscatter. Free, publicly available via Copernicus. |
| **AIS (Automatic Identification System)** | Ship location, speed, heading, and identity broadcasts | Functions like a real-time "who was where" log for large vessels. |

Together: Sentinel-1 answers *"where is the spill?"* and AIS answers *"which ships were nearby?"*

---

## 2. Backend / Machine Learning

| Component | Role |
|---|---|
| **Python** | Primary language for the entire backend and ML pipeline. |
| **PyTorch / TensorFlow** | Deep learning framework used to build and train the spill-segmentation model. |
| **U-Net / DeepLabV3 (pretrained)** | Semantic segmentation architectures fine-tuned on oil-spill imagery rather than trained from scratch — leverages existing "shape outlining" capability and drastically reduces data/time requirements. |
| **OpenCV** | Pre-processes raw SAR images — reduces speckle noise inherent to radar imagery before it's fed to the model. |
| **GDAL / rasterio** | Reads and parses satellite imagery in geospatial raster formats (e.g. GeoTIFF), including embedded geographic coordinates. |
| **Geopandas + Shapely** | Performs spatial operations on detected spill geometries — e.g. "find all points within 20 km of this polygon." |

---

## 3. Correlation Logic

Custom Python logic (no external library required) that:
1. Takes the spill's detected location and timestamp.
2. Queries the AIS dataset for vessels near that location within a relevant time window.
3. Ranks candidate vessels by proximity, heading, and time correlation to the spill event.

Conceptually similar to cross-referencing a crime scene's time and location against a list of suspects' known movements.

---

## 4. API / Backend Service

| Component | Role |
|---|---|
| **FastAPI / Flask** | Serves as the interface layer between the frontend and the ML model — receives uploaded imagery, triggers detection, and returns results. |
| **PostgreSQL + PostGIS** | Relational database with geospatial query support — enables efficient queries like "list all ships within X km of this point." |

---

## 5. Frontend

| Component | Role |
|---|---|
| **React** | Builds the dashboard/web interface. |
| **Leaflet.js / Mapbox** | Renders the interactive map showing spill location and nearby ships — the primary visual centerpiece for a demo. |
| **Chart.js** *(optional)* | Displays supplementary metrics such as model confidence scores. |

---

## 6. Deployment

| Component | Role |
|---|---|
| **Docker (local)** | Packages the full stack so it runs identically across machines, avoiding environment-dependent failures during a demo. Local execution is sufficient for a hackathon; cloud hosting is optional. |

---

## Architecture Flow Summary

```
Sentinel-1 Imagery ──► OpenCV (denoise) ──► GDAL/rasterio (geo-parse)
                                                     │
                                                     ▼
                                    U-Net / DeepLabV3 (spill segmentation)
                                                     │
                                                     ▼
                                        Geopandas/Shapely (spill geometry)
                                                     │
                                                     ▼
AIS Data ──────────────────────────► Correlation Logic (ship matching)
                                                     │
                                                     ▼
                                   PostgreSQL + PostGIS (storage/query)
                                                     │
                                                     ▼
                                    FastAPI/Flask (API layer)
                                                     │
                                                     ▼
                              React + Leaflet.js/Mapbox (dashboard)
```

---

## Tech Stack Summary

- **Data:** Sentinel-1 (SAR imagery), AIS (ship tracking)
- **ML/Backend:** Python, PyTorch/TensorFlow, U-Net/DeepLabV3, OpenCV, GDAL/rasterio, Geopandas/Shapely
- **API:** FastAPI/Flask, PostgreSQL + PostGIS
- **Frontend:** React, Leaflet.js/Mapbox, Chart.js
- **Deployment:** Docker (local)
