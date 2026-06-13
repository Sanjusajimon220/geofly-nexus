# Sanju Sajimon — Earth Observation & Geospatial ML

Production geospatial ML and remote-sensing work across **aerial imagery, radar, optical satellite, and LiDAR** — from raw signal to shipped system.
M.Sc. Remote Sensing & Geoinformatics, Karlsruhe Institute of Technology (KIT), 2026 · Karlsruhe, Germany.

🌍 **Live GeoFly platform:** http://geofly-nexus-sanju.s3-website.eu-central-1.amazonaws.com/
📄 **Portfolio (all projects):** https://sanjusajimon220.github.io/geofly-nexus/

This repository hosts the portfolio site and supporting material for three projects:

| Project | Domain | In this repo |
|---|---|---|
| **GeoFly NEXUS** | Aerial imagery · semantic segmentation · production | `index.html` (case study) + live platform |
| **PS-InSAR — La Palma** | Radar · Sentinel-1 · ground deformation | `SAR-INSAR_REPORT.pdf` |
| **NDVI change detection — Amazon** | Optical satellite · Google Earth Engine | `deforestation_analysis.ipynb` |

---

## 1 · GeoFly NEXUS — AI-Enhanced 3D Geospatial Intelligence Platform

Semantic segmentation and urban analytics from high-resolution aerial imagery. M.Sc. thesis project · KIT × GeoFly GmbH · 2026.

> **Note on source code.** This repository documents the architecture, methodology, and results of the project. The model code, training pipeline, and datasets are proprietary to GeoFly GmbH and are not publicly available. The live platform and this documentation are shared for portfolio and demonstration purposes.

### Overview
GeoFly NEXUS is an end-to-end system that turns raw aerial imagery and LiDAR into a live, browsable urban-intelligence platform. A modified **SegFormer-B3** transformer performs 7-class land-cover segmentation on a custom 5-channel input (RGB + Near-Infrared + normalised Digital Surface Model). Its per-pixel predictions drive five derived analytical layers and a per-tile intelligence engine, all served through an interactive CesiumJS 3D web viewer.

### Key results
- **89.42% mIoU** and **96.11% pixel accuracy** (deployed v4 model, 7 classes, 3 cm GSD)
- Monotonic improvement across four error-driven training rounds: 67.89% → 80.84% → 87.51% → 89.42%
- Six of seven classes exceed 86% IoU

| Class | IoU (%) |
|---|---|
| Agricultural land | 98.39 |
| Building | 94.98 |
| Water | 93.48 |
| Low vegetation | 91.28 |
| Bare soil | 88.93 |
| Impervious surface | 86.48 |
| Tree / canopy | 72.39 |
| **Mean IoU** | **89.42** |

Tree is the honest weak point — canopy boundaries are genuinely ambiguous at 3 cm resolution, where tree pixels are most often confused with low vegetation.

### How it works
**Multi-channel model adaptation.** SegFormer-B3 was extended from its standard 3-channel RGB input to a 5-channel input (RGB + NIR + nDSM). Pretrained ImageNet weights were preserved for the three RGB channels, and the two additional input channels were initialised and trained from the multi-modal data — giving the model both spectral (NIR) and elevation (nDSM) cues to separate visually similar classes such as water vs. dark asphalt and trees vs. low vegetation.

**Error-driven iterative training.** The model was trained across four rounds. Each round used confusion-matrix analysis and direct inspection in the 3D viewer to identify a specific failure mode, addressed through targeted relabelling and class-weight adjustment. The clearest example is the water class: in v1 it scored 0% IoU (harbour and coastal water misclassified as building/impervious surface); after adding harbour training tiles and correcting water/impervious boundaries, water reached 93.48% IoU with full recall in v4.

**Multi-region generalisation.** The deployed v4 checkpoint was applied to a second, distinct region (Saarland) with no retraining and no code changes. Each region is configured through a single YAML file (CRS, LiDAR spec, DSM grid resolution, nDSM encoding). Two generalisable engineering findings emerged: an auxiliary nDSM product for regions whose building heights exceed the model's input ceiling, and a per-tile morphology safeguard that prevents tall mining spoil heaps (Bergehalden) from being misclassified as buildings.

**Serving.** Segmentation outputs and the five derived layers are rendered as TMS tile pyramids (zoom 14–19), served from AWS S3 + CloudFront and exposed through OGC services (WMS / WMTS / TMS) to a CesiumJS 3D viewer.

### Production & scale
- Same v4 checkpoint deployed across two regions (Rostock + Saarland, ~352 km², ~98,000 tiles) with no retraining or code changes — per-region setup is a single YAML file
- Served as TMS tile pyramids on AWS S3 + CloudFront, ~€2.18/month steady-state

---

## 2 · PS-InSAR — La Palma volcanic deformation

Measuring millimetre-scale ground motion from space. Persistent-Scatterer InSAR on a four-year Sentinel-1 stack, tracking surface deformation across La Palma through the 2021 Cumbre Vieja eruption. **Full technical report:** [`SAR-INSAR_REPORT.pdf`](SAR-INSAR_REPORT.pdf).

### Result
- **Up to +30 mm/yr of uplift** concentrated on the south-western Cumbre Vieja ridge, consistent with magma intrusion during the 2021 eruption.
- The line-of-sight time series shows a stable trend through 2021, a sharp step at the eruption, and continued uplift into 2023.
- **Validated:** non-deforming northern and eastern coasts show no motion after correction, and the reference area was cross-checked against **GNSS** and **EGMS**.

### Data & method
- **Data:** Sentinel-1 SLC, descending pass 169, Jan 2020 – late 2023 (~110 acquisitions). Master scene 1 Feb 2022, selected for low, stable atmospheric water vapour.
- **Pipeline:** ESA **SNAP** (TOPSAR split, precise-orbit, calibration, back-geocoding + ESD coregistration, interferogram formation with DEM topographic-phase removal) → **StaMPS** for persistent-scatterer selection by the amplitude-dispersion index (Dₐ ≈ 0.42), phase unwrapping, and weeding.
- **Correction:** atmospheric delays and orbital ramps removed (`v-dso`) to isolate true displacement; LOS time series extracted at stable scatterers.

---

## 3 · NDVI change detection — Colombian Amazon reserve (Google Earth Engine)

An end-to-end Earth Engine workflow for vegetation-change detection over the **Resguardo Indígena Llanos del Yarí (Yaguará II)** reserve. **Code:** [`deforestation_analysis.ipynb`](deforestation_analysis.ipynb) — a corrected, runnable GEE Python notebook.

### Approach
- **AOI:** 312 km², clipped to the real reserve geometry from a shapefile asset.
- **Data:** Sentinel-2 (2019–2024) and Landsat-8 (2014–2018), kept **separate** so a sensor-bandpass difference is never mistaken for real change.
- **Cloud handling:** per-pixel masking (S2 SCL, L8 QA_PIXEL) plus monthly median composites to suppress tropical cloud noise.
- **Change detection:** a **sustained-decline** test (windowed before vs after each date) flags persistent drops rather than a single noisy step; before/after and difference maps export as GeoTIFF.

### Result — reported honestly
- The 2014–2018 record is essentially flat (no significant change).
- The strongest *sustained* signal is a **~January 2022 dip of ΔNDVI ≈ −0.15** in the Sentinel-2 series. Crucially, NDVI **recovers** afterwards — so this reads as a seasonal / transient effect rather than confirmed permanent forest loss, and is reported as a **candidate** event for year-over-year confirmation.
- What the workflow deliberately avoids: the sensor-level offset between Landsat-8 and Sentinel-2 is never reported as a trend, and a one-month 2019 cloud outlier is correctly ignored instead of being flagged as deforestation.

This project is as much a study in doing optical change detection *honestly* — correct geometry, per-pixel cloud masking, sensors separate, and reporting candidate change for confirmation rather than over-claiming.

---

## Stack
PyTorch · HuggingFace Transformers (SegFormer, BLIP) · GDAL · Rasterio · GeoPandas · laspy · Google Earth Engine · ESA SNAP · StaMPS · QGIS · Weights & Biases · Docker · AWS (S3, CloudFront) · CesiumJS · OGC services (WMS/WMTS/TMS)

## Repository contents
- `index.html` — portfolio site covering all three projects (served at the GitHub Pages URL above)
- `SAR-INSAR_REPORT.pdf` — full PS-InSAR technical report
- `deforestation_analysis.ipynb` — corrected NDVI change-detection notebook (Google Earth Engine, Python)
- `README.md` — this file

## Author
**Sanju Sajimon** — Geospatial ML Engineer · Remote Sensing & Computer Vision
M.Sc. Remote Sensing & Geoinformatics, KIT (2026) · Karlsruhe, Germany
✉️ sanjusajimon76@gmail.com · 🔗 linkedin.com/in/sanju-sajimon
