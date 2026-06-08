# GeoFly NEXUS — AI-Enhanced 3D Geospatial Intelligence Platform

**Semantic segmentation and urban analytics from high-resolution aerial imagery.**
M.Sc. thesis project · Karlsruhe Institute of Technology (KIT) × GeoFly GmbH · 2026

🌍 **Live platform:** http://geofly-nexus-sanju.s3-website.eu-central-1.amazonaws.com/
📄 **Portfolio / case study:** https://sanjusajimon220.github.io/geofly-nexus/

> **Note on source code.** This repository documents the architecture, methodology, and
> results of the project. The model code, training pipeline, and datasets are proprietary to
> GeoFly GmbH and are not publicly available. The live platform and this documentation are
>  shared for portfolio and demonstration purposes.

---

<!--
ADD SCREENSHOTS HERE — biggest single improvement to this README.
Take them from the public live site (no IP issue) and drop the image files in a /docs or /assets
folder in this repo, then reference them like the examples below.

![3D viewer](docs/viewer.png)
![Segmentation overlay](docs/segmentation.png)
![Derived heatmap layers](docs/heatmaps.png)
-->

## Overview

GeoFly NEXUS is an end-to-end system that turns raw aerial imagery and LiDAR into a live,
browsable urban-intelligence platform. A modified **SegFormer-B3** transformer performs
**7-class land-cover segmentation** on a custom 5-channel input (RGB + Near-Infrared +
normalised Digital Surface Model). Its per-pixel predictions drive five derived analytical
layers and a per-tile intelligence engine, all served through an interactive CesiumJS 3D
web viewer.

## Key results

- **89.42% mIoU** and **96.11% pixel accuracy** (deployed v4 model, 7 classes, 3 cm GSD)
- Monotonic improvement across four error-driven training rounds: 67.89% → 80.84% → 87.51% → 89.42%
- Six of seven classes exceed 86% IoU

### Per-class IoU (deployed v4 model)

| Class               | IoU (%) |
| ------------------- | ------- |
| Agricultural land   | 98.39   |
| Building            | 94.98   |
| Water               | 93.48   |
| Low vegetation      | 91.28   |
| Bare soil           | 88.93   |
| Impervious surface  | 86.48   |
| Tree / canopy       | 72.39   |
| **Mean IoU**        | **89.42** |

Tree is the honest weak point — canopy boundaries are genuinely ambiguous at 3 cm
resolution, where tree pixels are most often confused with low vegetation.

## How it works

**Multi-channel model adaptation.** SegFormer-B3 was extended from its standard 3-channel
RGB input to a 5-channel input (RGB + NIR + nDSM). The pretrained ImageNet weights were
preserved for the three RGB channels, and the two additional input channels were initialised
and trained from the multi-modal data — giving the model both spectral (NIR) and elevation
(nDSM) cues to separate visually similar classes such as water vs. dark asphalt and trees vs.
low vegetation.

**Error-driven iterative training.** The model was trained across four rounds. Each round used
confusion-matrix analysis and direct inspection in the 3D viewer to identify a specific failure
mode, which was then addressed through targeted relabelling and class-weight adjustment. The
clearest example is the water class: in v1 it scored **0% IoU** (harbour and coastal water were
misclassified as building or impervious surface); after adding harbour training tiles and
correcting water/impervious boundaries over the following rounds, water reached **93.48% IoU**
with full recall in v4.

**Multi-region generalisation.** The deployed v4 checkpoint was applied to a second, distinct
region (Saarland) with **no retraining and no code changes**. Each region is configured through
a single YAML file covering the coordinate reference system, LiDAR specification, DSM grid
resolution, and nDSM encoding. Two generalisable engineering findings emerged: an auxiliary
nDSM product for regions whose building heights exceed the model's input ceiling, and a
per-tile morphology safeguard that prevents tall mining spoil heaps (Bergehalden) from being
misclassified as buildings.

**Serving.** Segmentation outputs and the five derived layers are rendered as TMS tile
pyramids (zoom 14–19), served from AWS S3 + CloudFront and exposed through OGC services
(WMS / WMTS / TMS) to a CesiumJS 3D viewer.

## What it does

- **Segmentation** — building, tree, low vegetation, impervious surface, bare soil, water, agricultural land
- **Five derived layers** — building height, NDVI vegetation, solar suitability, surface sealing, flood susceptibility
- **Per-tile analytics** across eight themes — land cover, building morphology, BauNVO regulatory compliance, environmental/ecological metrics, asset valuation, energy, hydrology, scenario modelling
- **Scene captioning** — a BLIP vision-language model adds a natural-language description per tile

## Production & scale

- Same v4 checkpoint deployed across **two regions** (Rostock + Saarland, ~352 km², ~98,000 tiles) with **no retraining or code changes** — per-region setup is a single YAML file
- Served as TMS tile pyramids on **AWS S3 + CloudFront**, ~€2.18/month steady-state

## Stack

`PyTorch` · `HuggingFace Transformers (SegFormer, BLIP)` · `GDAL` · `Rasterio` · `GeoPandas` · `laspy` · `Weights & Biases` · `Docker` · `AWS (S3, CloudFront)` · `CesiumJS` · `OGC services (WMS/WMTS/TMS)`

## Author

**Sanju Sajimon** — Geospatial ML Engineer · Remote Sensing & Computer Vision
M.Sc. Remote Sensing & Geoinformatics, KIT (2026) · Karlsruhe, Germany
✉️ sanjusajimon76@gmail.com · 🔗 linkedin.com/in/sanju-sajimon
