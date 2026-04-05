# Land Use Land Cover (LULC) Classification — Madhya Pradesh

> **Status: In Progress** — Classification pipeline operational; state-wide deployment ongoing.

Automated LULC classification system for Madhya Pradesh (~308,000 km²) using LISS-IV satellite imagery, object-based image analysis (OBIA), and machine learning. Built for the Madhya Pradesh State Electronics Development Corporation (MPSEDC).

---

## Problem Statement

Manual GIS-based land cover mapping for a state the size of Madhya Pradesh is slow, inconsistent across districts, and difficult to reproduce. This project automates the full pipeline — from raw satellite imagery to district-wise classified maps and area statistics — reducing weeks of manual effort to a reproducible, scalable workflow.

---

## Workflow Overview

The pipeline runs in two phases: **Training** and **Testing (Inference)**.

```
LISS-IV Imagery (district-wise tiles)
        │
        ▼
┌─────────────────────┐     ┌──────────────────────────┐
│   TRAINING PHASE    │     │     TESTING PHASE         │
│                     │     │                           │
│  Ground truth       │     │  Full MP LISS-IV tiles    │
│  polygons (6 class) │     │  + District boundaries    │
│         │           │     │         │                 │
│  Zonal Statistics   │     │  Clip per district        │
│  (NDVI, NDWI, etc.) │     │         │                 │
│         │           │     │  OBIA Segmentation        │
│  Feature Engineering│     │  (MeanShift / kMeans)     │
│         │           │     │         │                 │
│  RF / SVM Training  │     │  Zonal Stats on segments  │
│  + GridSearchCV     │     │         │                 │
│         │           │     │  Load trained model       │
│  Save model (.pkl)  │────▶│  Predict class per polygon│
└─────────────────────┘     │         │                 │
                            │  Export maps + CSV stats  │
                            └──────────────────────────┘
```

---

## Key Technical Components

### Satellite Data
- **Sensor:** LISS-IV (ResourceSat-2/2A) — 5.8m resolution
- **Coverage:** All districts of Madhya Pradesh
- **Preprocessing:** District-wise tile merging for complete coverage

### LULC Classes (6)
| Class | Description |
|-------|-------------|
| Agriculture | Cropland, cultivated areas |
| Forest | Dense and open forest cover |
| Wasteland | Degraded, barren, scrubland |
| Water Body | Rivers, reservoirs, ponds |
| Built-up | Urban and rural settlements |
| Others | Mixed / unclassified land |

### Feature Engineering
Spectral indices and statistics computed per segment:
- **Indices:** NDVI, NDWI
- **Statistics:** Mean, Standard Deviation, Min, Max per band

### Segmentation (OBIA)
Object-based segmentation applied using Orfeo ToolBox (OTB):
- `MeanShiftSegmentation`
- `LargeScaleMeanShift`
- `KMeans`
- SAM (Segment Anything Model) — experimental

Segmentation groups spectrally similar pixels into homogeneous objects before classification, significantly improving accuracy over pixel-based approaches.

### Machine Learning
| Model | Tuning |
|-------|--------|
| Random Forest | GridSearchCV (n_estimators, max_depth, min_samples_split) |
| SVM | GridSearchCV (C, kernel, gamma) |

- Label encoding for categorical classes
- Train/validation split with cross-validation
- Models saved in `.pkl` / `.joblib` format for reuse

### Output
- District-wise classified maps (vector shapefile + raster GeoTIFF)
- Area statistics per class per district (CSV / Excel)
- Ready for GIS visualization in QGIS / GeoServer

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Satellite Processing | QGIS, Orfeo ToolBox (OTB), GDAL |
| Machine Learning | scikit-learn, Random Forest, SVM |
| Feature Computation | GDAL, Rasterio, GeoPandas, NumPy |
| Geospatial | PostGIS, Shapely, Fiona |
| Model Persistence | joblib, pickle |
| Output | GeoTIFF, Shapefile, CSV, Excel |

---

## Scale

- **State coverage:** ~308,252 km² (all districts of MP)
- **Approach:** District-wise processing → state-level aggregation
- **Deployment:** MPSEDC internal GIS infrastructure

---

## Project Structure

```
lulc-mp-classification/
│
├── docs/
│   └── workflow_documentation.pdf     # Full methodology document
│
├── training/
│   ├── feature_extraction.py          # Zonal stats + index computation
│   ├── train_model.py                 # RF/SVM training + GridSearchCV
│   └── models/                        # Saved .pkl model files
│
├── inference/
│   ├── segment.py                     # OTB segmentation wrapper
│   ├── predict_district.py            # Per-district classification
│   └── export_stats.py               # Area statistics export
│
├── utils/
│   ├── indices.py                     # NDVI, NDWI computation
│   └── preprocessing.py              # Tile merging, clipping
│
└── README.md
```

> **Note:** Core pipeline code is maintained within MPSEDC's secure deployment environment. Scripts uploaded here are reference implementations.

---

## Results (In Progress)

- Pipeline validated on pilot districts
- Object-based segmentation shows significant improvement over pixel-based classification
- Full state-wide deployment ongoing as part of MPSEDC's GIS modernisation programme

---

## Author

**Nitesh Singh Bhadoriya**
AI & ML Engineer — MPSEDC, Bhopal
[LinkedIn](https://www.linkedin.com/in/nitesh-singh-bhadoriya-917411226) · [GitHub](https://github.com/nitesh-bhadoriya)

---

*Developed under the Madhya Pradesh State Electronics Development Corporation (MPSEDC) as part of state-level geospatial intelligence initiatives.*
