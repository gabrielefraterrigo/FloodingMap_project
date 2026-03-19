Ecco la versione inglese del README:
markdown# Flood Detection with Sentinel-1 SAR — Emilia-Romagna, May 2023

## Project Overview

This project implements a complete **flood detection pipeline** using 
radar imagery from ESA's Sentinel-1 satellite to automatically map 
flooded areas during the May 2023 Emilia-Romagna flood event — one of 
the most severe flooding disasters in recent Italian history, with 
approximately 500-600 km² of territory inundated across the provinces 
of Bologna, Forlì-Cesena, and Ravenna.

The analysis focuses on the **province of Ravenna**, one of the most 
affected areas, identifying approximately **69 km²** of flooded zones 
through SAR-based change detection techniques.

---

## Why Radar Instead of Optical Imagery?

During a flood event, the sky is covered with clouds — optical satellites 
like Sentinel-2 cannot photograph the ground. Sentinel-1 uses radar waves 
that penetrate clouds with no issues, operating 24/7 regardless of weather 
conditions.

The underlying physics is straightforward: calm water reflects radar waves 
away from the satellite (low backscatter, ~-20/-25 dB), while dry soil 
scatters them in all directions (higher backscatter, ~-10/-15 dB). By 
comparing a pre-flood image with a post-flood image, flooded areas emerge 
as regions where backscatter has dropped significantly.

---

## Analysis Pipeline
```
Sentinel-1 imagery (GEE)
        ↓
Preprocessing (speckle filtering, dB conversion)
        ↓
Change detection (absolute backscatter threshold)
        ↓
Morphological cleaning (opening + closing)
        ↓
Logical masking (permanent water exclusion)
        ↓
Flood map + statistics
```

---

## Project Structure
```
FloodingMap_project/
├── data/
│   ├── raw/              # original Sentinel-1 images (not included)
│   ├── processed/        # preprocessed images (not included)
│   └── aoi/              # area of interest (GeoJSON)
├── notebooks/
│   ├── 01_data_download.ipynb      # Google Earth Engine download
│   ├── 02_preprocessing.ipynb      # speckle filter, normalization
│   └── 03_change_detection.ipynb   # analysis, classification, output
└── outputs/
    ├── flood_mask_final.tif         # georeferenced binary mask
    ├── flood_map_emilia_romagna_2023.png   # Python technical map
    ├── flood_map_qgis_final.png     # QGIS cartographic map
    ├── flood_map_qgis_final.pdf     # printable version
    └── flood_statistics.csv         # quantitative statistics
```

---

## Results

| Parameter | Value |
|---|---|
| Sensor | Sentinel-1 SAR (C-band) |
| Polarisation | VV |
| Resolution | 30 metres per pixel |
| Pre-flood period | April 2023 (mean of 5 images) |
| Post-flood period | May 2023 (mean of 4 images) |
| Backscatter threshold | -15 dB |
| Identified flooded area | 69 km² |
| Area of interest | Province of Ravenna |

![Flood map](outputs/flood_map_qgis_final.png)

---

## Methodological Note

During the analysis, a **seasonal vegetation bias** was identified — 
between April and May, plant growth increases the mean backscatter 
across the area, masking the water signal in the pre/post difference. 
This was addressed by adopting an **absolute threshold** on post-flood 
backscatter** instead of relative change detection, combined with 
**logical masking** to exclude permanent water bodies (sea, lakes, 
canals):
```python
permanent_water   = pre < -15.0
final_flood_mask  = flood_mask & ~permanent_water
```

This approach is more robust in the presence of seasonal biases and 
is documented in the SAR literature as an alternative to classical 
change detection.

---

## How to Reproduce

1. Clone the repository
```bash
   git clone https://github.com/your-username/FloodingMap_project.git
```

2. Install dependencies
```bash
   pip install -r requirements.txt
```

3. Authenticate Google Earth Engine
```python
   import ee
   ee.Authenticate(auth_mode='notebook')
   ee.Initialize(project='your-gee-project')
```

4. Run notebooks in order
```
   01_data_download.ipynb → 02_preprocessing.ipynb → 03_change_detection.ipynb
```

---


## Tech Stack

![Python](https://img.shields.io/badge/Python-3.12-blue)
![GEE](https://img.shields.io/badge/Google_Earth_Engine-API-green)
![QGIS](https://img.shields.io/badge/QGIS-3.x-brightgreen)

- **Python 3.12** — main language
- **Google Earth Engine** — Sentinel-1 data access and download
- **rasterio** — GeoTIFF read/write
- **NumPy / SciPy** — array processing and binary morphology
- **matplotlib** — visualization and technical map
- **QGIS** — final cartographic layout

---

## Author

**Gabriele Fraterrigo**  
Personal portfolio project for learning Python applied to GIS 
and satellite remote sensing.