# GeoFly NEXUS — AI-Enhanced 3D Geospatial Intelligence Platform

**Semantic segmentation and urban analytics from high-resolution aerial imagery.**
M.Sc. thesis project · Karlsruhe Institute of Technology (KIT) × GeoFly GmbH · 2026

🌍 **Live platform:** http://geofly-nexus-sanju.s3-website.eu-central-1.amazonaws.com/
📄 **Portfolio / case study:** https://sanjusajimon220.github.io/geofly-nexus/

---

## Overview

GeoFly NEXUS is an end-to-end system that turns raw aerial imagery and LiDAR into a live, browsable urban-intelligence platform. A modified **SegFormer-B3** transformer performs **7-class land-cover segmentation** on a custom 5-channel input (RGB + Near-Infrared + normalised Digital Surface Model). Its per-pixel predictions drive five derived analytical layers and a per-tile intelligence engine, all served through an interactive 3D web viewer.

## Key results

- **89.42% mIoU** and **96.11% pixel accuracy** (deployed v4 model, 7 classes, 3 cm GSD)
- Monotonic improvement across four error-driven training rounds: 67.89% → 80.84% → 87.51% → 89.42%
- Six of seven classes exceed 86% IoU; water improved from 0.00% (v1) to 93.48% (v4)

## What it does

- **Segmentation** — building, tree, low vegetation, impervious surface, bare soil, water, agricultural land
- **Five derived layers** — building height, NDVI vegetation, solar suitability, surface sealing, flood susceptibility
- **Per-tile analytics** across eight themes — land cover, building morphology, BauNVO regulatory compliance, environmental/ecological metrics, asset valuation, energy, hydrology, scenario modelling
- **Scene captioning** — a BLIP vision-language model adds a natural-language description per tile

## Production & scale

- Same v4 checkpoint deployed across **two regions** (Rostock + Saarland, ~352 km², ~98,000 tiles) with **no retraining or code changes** — per-region setup is a single YAML file
- Served as TMS tile pyramids on **AWS S3 + CloudFront**, ~€2.18/month steady-state

## Stack

`PyTorch` · `HuggingFace Transformers (SegFormer, BLIP)` · `GDAL` · `Rasterio` · `GeoPandas` · `laspy` · `Weights & Biases` · `Docker` · `AWS (S3, CloudFront)` · `OGC services (WMS/WMTS/TMS)`

## Author

**Sanju Sajimon** — Data Scientist, Remote Sensing & Computer Vision
M.Sc. Remote Sensing & Geoinformatics, KIT (2026) · Karlsruhe, Germany
✉️ sanjusajimon76@gmail.com · 🔗 linkedin.com/in/sanju-sajimon
