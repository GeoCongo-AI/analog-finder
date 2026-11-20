# Analog Finder

**Discovering & characterizing terrestrial analog sites for lunar and Martian exploration using Optimal Transport — GeoCongo AI · NASA Space Apps Challenge 2026**

![Project](https://img.shields.io/badge/project-Analog%20Finder-183153)
![Team](https://img.shields.io/badge/team-GeoCongo%20AI-2f6f5e)
![Challenge](https://img.shields.io/badge/challenge-NASA%20Space%20Apps%202026-173b63)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Tests](https://img.shields.io/badge/tests-42%20passed-brightgreen)
![License](https://img.shields.io/badge/license-MIT-green)

## Live deployments

| Service | Link |
|---|---|
| Interactive dashboard | [analog.geocongoai.com](https://analog.geocongoai.com/) |
| Open Science REST API (Swagger) | [api.analog.geocongoai.com/docs](https://api.analog.geocongoai.com/docs) |
| ReDoc | [api.analog.geocongoai.com/redoc](https://api.analog.geocongoai.com/redoc) |

## Earth Analog (Mode A) Case study Tab
![Analog Finder dashboard](app/assets/dashboard.png)
---
## Earth Analog (Mode A) Run new study Tab
![Analog Finder dashboard](app/assets/dashboard2.png)

## Problem statement

Selecting future lunar base locations and Martian landing sites depends on a combination of physical factors — topography, geology, environmental context, and in-situ resources. Earth sites that share comparable conditions (volcanic fields, hyper-arid deserts, lava tube systems, collapse calderas) allow mission teams to train, test equipment, and validate mission protocols before actual deployment. However, the catalog of characterized terrestrial analog sites remains incomplete, especially for lesser-known high-fidelity sites in Africa and the Middle East.

**Analog Finder** addresses this gap by comparing continuous terrain and environmental indicator distributions using one- and two-dimensional Wasserstein Optimal Transport ($W_1$). The open-source pipeline automatically discovers, scores, and characterizes candidate sites from open satellite catalogs (STAC), enriched by in-situ field collection and locally preprocessed geospatial rasters. Results are presented in an interactive ranked dashboard and a fully documented REST API, enabling reproducible, data-driven analog site selection.

## Dual operational capabilities

| Track | Purpose |
|---|---|
| **A — Earth Analog (Moon & Mars)** | Characterize terrestrial analog candidates — volcanic fields, deserts, lava tubes, basalt flows, collapse calderas — against JAXA SELENE Kaguya lunar and NASA MRO Martian digital terrain models using Optimal Transport similarity scoring. |
| **B — East African Rift Geohazard Monitor** | Monitor landslide indicators in Bukavu (DR Congo) and volcanic activity in Nyiragongo and Virunga through multi-temporal change detection between baseline $T_0$ and observation $T_1$. |

## Architecture

```
STAC Catalogs (Planetary Computer / USGS Astrogeology)
         │
         ▼
 ┌───────────────────────────────────┐
 │  Hybrid Ingestion Pipeline        │
 │  - Satellite COG streaming (STAC) │
 │  - Local field rasters (priority) │
 └───────────────┬───────────────────┘
                 │
                 ▼
 ┌───────────────────────────────────────────────┐
 │  Optimal Transport Engine                     │
 │  - 1D Wasserstein distance W₁(u,v)            │
 │  - 2D sliding-window patch differencing       │
 │  - Criterion scoring & confidence index       │
 └────────────┬──────────────────────────────────┘
              │
   ┌──────────┴───────────┐
   ▼                      ▼
scores.json (Mode A)   changes.json (Mode B)
   │                      │
   ▼                      ▼
┌──────────────────┐  ┌──────────────────────┐
│ Streamlit        │  │  FastAPI REST API     │
│ Dashboard :8501  │  │  :8000 (Swagger/ReDoc)│
└──────────────────┘  └──────────────────────┘
```

## Case studies

The pipeline includes two **in-depth case studies** grounded in original field collection and local data preprocessing, alongside seven reference analog and control sites:

| Site | Role | Lead Collector | Key Processes |
|---|---|---|---|
| **Virunga Volcanic Province (DR Congo)** | Case study | Nicolas | Active alkaline volcanism, fresh basaltic lava flows, Nyamuragira lava tubes, collapse pits |
| **Tibesti Volcanic Province (Chad)** | Case study | Gérard | Tarso Tousside basaltic flows, Trou au Natron collapse caldera (8 km / 1 000 m), hyper-arid Saharan erosion |
| Harrat Khaybar (Saudi Arabia) | Candidate (lesser-known) | Nicolas | Intraplate basaltic lava tubes (Umm Jirsan), hyper-arid preservation |
| Mount Suswa Caldera (Kenya) | Candidate (lesser-known) | Diego | Double phonolitic-trachytic caldera with braided lava tube networks |
| Hawaii — Kilauea / Mauna Loa (USA) | Known analog | Diego | Active tholeiitic shield basalt, NASA analog test site |
| Lanzarote — Timanfaya (Spain) | Known analog | Bosworth | Historical basaltic fissure eruptions, ESA PANGAEA astronaut training site |
| Lava Beds National Monument (USA) | Known analog | Bosworth | Highest concentration of accessible lava tube caves in North America |
| Atacama Desert (Chile) | Cross-validation | Nicolas | Extreme hyper-aridity, Mars analog control |
| Death Valley (USA) | Cross-validation | Diego | Extensional tectonic basin, extreme thermal amplitude |
| Central Congo Rainforest (DR Congo) | Negative control | Bosworth | Dense equatorial canopy — zero planetary analog fidelity (validation baseline) |

## Open data pipeline

| Source | Data | Use |
|---|---|---|
| Copernicus DEM GLO-30 | Global elevation, 30 m | Earth topography and terrain derivatives |
| Sentinel-2 L2A | Multispectral imagery, 10–20 m | Bare ground, vegetation, and mineral indicators |
| TerraClimate | Climate and precipitation | Aridity and environmental context |
| JAXA SELENE Kaguya | Lunar terrain-camera DTMs, 26.2 m | Lunar target topography |
| NASA MRO CTX / HiRISE | Martian terrain models, 20 m / 1 m | Mars target topography |
| **In-situ & local rasters** | Field DEMs, basalt masks, fault traces | Ground-truth calibration enriching satellite distributions |

Earth collections are accessed through Microsoft Planetary Computer STAC; lunar and Martian collections use USGS Astrogeology STAC. Data products are open and accessed as Cloud-Optimized GeoTIFFs where available. For case study sites, locally preprocessed rasters (placed in `data/raw/<site_id>/`) take priority over API-streamed data to eliminate cloud contamination and micro-relief uncertainties.

## Mathematical core

For one-dimensional distributions with quantile functions $F_P^{-1}$ and $F_Q^{-1}$, the Wasserstein-1 distance is:

$$W_1(P,Q)=\int_0^1\left|F_P^{-1}(u)-F_Q^{-1}(u)\right|\,du$$

The similarity transform is $S_i=\exp(-W_1/\sigma_i)$, where $\sigma_i$ is the scale for indicator $i$. For temporal monitoring, localized two-dimensional patch comparisons preserve the spatial pattern of change between $T_0$ and $T_1$.

## Quickstart

### Local dashboard

Requires Python 3.10 or later.

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip install -e .

# Generate offline demo inputs for both operational tracks (synthetic test data)
python -m analog_finder.pipeline --mode all --synthetic
streamlit run app/streamlit_app.py
```

> **Note:** `--synthetic` generates invented values for local pipeline and interface testing only.
> To compute real scores from live satellite data (requires internet for STAC access):
> ```bash
> python -m analog_finder.pipeline --mode analog
> ```

### Docker Compose

```bash
docker compose up -d
```

Open the dashboard at [localhost:8501](http://localhost:8501), or the local API documentation at [localhost:8000/docs](http://localhost:8000/docs). The Compose ports bind to localhost.

## REST API examples

Search lunar terrain data around Marius Hills:

```bash
curl --fail-with-body \
  "https://api.analog.geocongoai.com/v1/planetary/search?body=moon&min_lon=-57&min_lat=13&max_lon=-55&max_lat=15&data_type=dem"
```

Analyze a terrestrial candidate against the lunar target:

```python
import requests

response = requests.post(
    "https://api.analog.geocongoai.com/v1/analyze",
    json={
        "mode": "analog",
        "site_id": "hawaii",
        "target_ref": {"target_id": "marius_hills"},
        "options": {"synthetic": True},
    },
    timeout=60,
)
response.raise_for_status()
print(response.json())
```

## Team

 **GeoCongo AI** is a dynamic, multidisciplinary team driven by a common goal: innovating for the **NASA Space Apps Challenge 2026**.
 
| Member | Role |
|---|---|
| **Gérard** | Lead Backend Developer and Planetary Data Curator · Field Collector (Tibesti, Chad) · Planetary targets (Marius Hills / Moon, Jezero Crater / Mars) |
| **Nicolas** | Lead Field Collector · Virunga Volcanic Province (DR Congo) · Harrat Khaybar (Saudi Arabia) · Atacama Desert |
| **Diego** | Lead Field Collector · Suswa Caldera (Kenya) · Hawaii (Kilauea) · Death Valley |
| **Bosworth** | Lead Data Collector · Lanzarote Timanfaya · Lava Beds National Monument · Negative control (Congo Rainforest) |
| **Birusha** | Lead Frontend Developer and UI/UX designer |
| **Remy** | Storyteller and Graphic Designer |


## Citation

```bibtex
@software{geocongoai_analog_finder_2026,
  author       = {GeoCongo AI},
  title        = {Analog Finder — Terrestrial Planetary Analog Discovery via Optimal Transport},
  year         = {2026},
  note         = {NASA Space Apps Challenge 2026}
}
```
