# Spatio‑temporal Evolution of Glacial Lakes & GLOF Susceptibility — Koshi River Basin (1994–2024)

> **Internship Project (Centre for Rivers, Atmosphere and Land sciences, Marine Resources and Natural Hazards Lab)** — Remote sensing & GIS workflow to (1) map glacial lakes across 1994/2004/2014/2024 and (2) rank GLOF susceptibility for ≥0.05 km² lakes using an Analytic Hierarchy Process (AHP).

---

## 🔎 Overview

This repository contains code, data schema, and reproducible steps for assessing **Glacial Lake Outburst Flood (GLOF) susceptibility** in the **Koshi River Basin** using multi-temporal satellite imagery and DEMs. We first updated a 2024 lake inventory, quantified decadal changes since 1994, and then computed a multi-factor **GLOF Susceptibility Index** for a curated set of lakes using **AHP**.

**Key highlights**
- Updated inventory of **755 lakes (≥0.01 km²)** for 2024 and tracked change across **1994 → 2004 → 2014 → 2024**.  

- Detailed susceptibility analysis for **299 lakes (≥0.05 km²)** using **morphometric, topographic, seismic, and climatic** factors.  

- AHP weights emphasize **moraine W/H ratio**, **freeboard**, and **extreme precipitation events** as leading controls.  

- Output includes **ranked lakes** and **risk class maps** (Low, Medium, High, Very High).

> This work supports monitoring, early warning, and climate adaptation planning across the Himalayan headwaters.

---

## 🗂️ Repository Structure

```
.
├── data/
│   ├── raw/                 # Unprocessed vectors/rasters (not tracked: large files)
│   ├── interim/             # Intermediates (tiles, masks, DEM derivatives)
│   └── processed/           # Final shapefiles/GeoPackages/CSVs (inventories & scores)
├── notebooks/
│   ├── 01_inventory_update_2024.ipynb
│   ├── 02_change_analysis_1994_2024.ipynb
│   └── 03_glof_susceptibility_ahp.ipynb
├── src/
│   ├── lakes/
│   │   ├── extract_ndwi.py
│   │   ├── update_inventory.py
│   │   └── change_metrics.py
│   ├── topo/
│   │   ├── dem_derivatives.py      # slope, profiles, flow-path
│   │   └── morphometry.py          # W/H ratio, freeboard, distal slope
│   ├── susceptibility/
│   │   ├── ahp.py                  # pairwise → weights → CR
│   │   └── score_lakes.py          # weighted linear combination + Jenks classes
│   └── viz/
│       ├── maps.py                 # quick map composer
│       └── charts.py               # elevation bands, growth trends
├── requirements.txt
└── README.md
```

> Tip: Use a `.gitignore` to keep large rasters, intermediate tiles, and cache files out of the repo.

---

## 📦 Dependencies

- Python ≥ 3.10
- Packages: `geopandas`, `rasterio`, `numpy`, `pandas`, `scipy`, `matplotlib`, `richdem`, `pyproj`, `mapclassify`, `shapely`, `scikit-image`
- Optional: `rtree` (spatial index), `pyogrio` (fast I/O)

Create a virtual environment:
```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Example `requirements.txt` (you may refine versions):
```
geopandas
rasterio
numpy
pandas
scipy
matplotlib
richdem
pyproj
mapclassify
shapely
rtree
pyogrio
```

---

## 🛰️ Data Sources

- **Satellite imagery**: Sentinel‑2 MSI (10 m) for 2024; Landsat‑8 OLI (30 m) for 2014; Landsat (30 m) for 2004/1994.- **DEM**: ASTER GDEM v3 (~30 m).- **Baselines**: High Mountain Asia glacial lake inventory (e.g., Wang et al., 2020) used as a starting reference.- **Validation**: High‑res Google Earth imagery (for moraine features & crest profiles).

> Ensure licensing/compliance for each dataset. Large rasters should be stored externally (e.g., Zenodo/Drive) and referenced via a `data/README.md` with download links.

---

## 🧭 Methodology (Short)

1. **Inventory Update (2024)**   - Post‑ablation images (Sep–Oct) to minimize snow/cloud.   - NDWI‑assisted delineation with manual QA.   - Consistent minimum area: **≥0.01 km²**.

2. **Change Analysis (1994–2024)**   - Harmonize lake IDs; remove drained/merged duplicates.   - Compute **absolute** and **% change** by elevation bands (e.g., 5000–5500 m).

3. **Susceptibility Factors (for ≥0.05 km² lakes)**   - Morphometry: **Moraine W/H**, **Freeboard**, **Distal face slope**.   - Setting/Triggers: **Glacier–lake distance**, **Lake–glacier slope**, **Seismicity**, **Extreme precipitation**, **Elevation**, **Current area**, **Change rate**.

4. **AHP & Scoring**   - Pairwise matrix → weights → **Consistency Ratio (CR ≈ 0.018 < 0.1)**.   - Weighted linear combination → **composite index**.   - **Jenks natural breaks** → {Low, Medium, High, Very High}.

5. **Outputs**   - Ranked lake table (ID, index, class, lon/lat).   - Thematic **risk maps**; charts for elevation‑wise dynamics.

---

## ▶️ Quick Start (CLI)

Update 2024 inventory from prepared scenes:
```bash
python -m src.lakes.update_inventory   --scenes data/raw/s2_2024/   --aoi data/raw/koshi_basin.gpkg   --min_area_km2 0.01   --out data/processed/lakes_2024.gpkg
```

Compute decadal change metrics:
```bash
python -m src.lakes.change_metrics   --historical data/processed/lakes_{1994,2004,2014}.gpkg   --current data/processed/lakes_2024.gpkg   --out data/processed/lakes_change_1994_2024.csv
```

Derive morphometric & topo factors:
```bash
python -m src.topo.morphometry   --lakes data/processed/lakes_2024_ge_0p05.gpkg   --dem data/raw/aster_gdem_v3.tif   --out data/interim/factors.parquet
```

AHP → weights, score lakes, classify:
```bash
python -m src.susceptibility.ahp   --pairwise data/raw/ahp_pairwise.csv   --out data/interim/ahp_weights.json

python -m src.susceptibility.score_lakes   --factors data/interim/factors.parquet   --weights data/interim/ahp_weights.json   --out data/processed/glof_scores.gpkg   --classes data/processed/glof_classes.csv
```

Make maps and charts:
```bash
python -m src.viz.maps --scores data/processed/glof_scores.gpkg --out figs/maps/
python -m src.viz.charts --change data/processed/lakes_change_1994_2024.csv --out figs/charts/
```

---

## 📊 Key Findings (from the study)

- **755 lakes** (≥0.01 km²) mapped for 2024; strongest clustering at **5000–5500 m**.- **299 lakes** (≥0.05 km²) assessed for susceptibility.- AHP highlights **Moraine W/H (≈0.249)**, **Freeboard (≈0.164)**, **Extreme precipitation (≈0.164)** as top‑weighted factors.- Final classes: **Low (150)**, **Medium (89)**, **High (45)**, **Very High (15)**.- Eastern sub‑basins show a higher density of **Very High** susceptibility lakes.

> See the internship report for full context, assumptions, and limitations.

---

## 🧪 Reproducibility

- Use the same **post‑ablation window** for imagery (Sep–Oct).- Keep the **area thresholds** identical for comparability (0.01 km² for inventory; 0.05 km² for susceptibility).- Document any manual edits in `data/CHANGELOG.md`.- For DEM profiles/freeboard, record the **profile lines** used for crest elevation extraction.

---

## 📜 Citation

If you use this repository or the results, please cite the internship report and the sources referenced therein.

> *Spatio‑temporal Evolution of Glacial Lakes and GLOF Susceptibility in the Koshi River Basin — Internship Report, CORAL, IIT Kharagpur (2024).*

---

## 🙏 Acknowledgements

- **Supervisor**: Dr. Abhishek K. Rai (CORAL, IIT Kharagpur)- **Research support**: Marine Resources & Natural Hazards Lab (CORAL)- **Tools**: QGIS, Google Earth Pro, Python (GeoPandas/Rasterio), ASTER GDEM, Sentinel‑2, Landsat

---

## 🛡️ License

Choose a license (e.g., MIT/CC‑BY‑4.0) and add it as `LICENSE` in this repo.
