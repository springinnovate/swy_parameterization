# SWY parameterization

Building curve number (CN) and crop coefficient (Kc) parameters for the InVEST Seasonal Water
Yield model (run through [`inspring`](https://github.com/springinnovate/inspring)) from global
data, and testing them against a regionally parameterized run. Moved out of
[`global_NCP`](https://github.com/springinnovate/global_NCP) in September 2026 so the work can be
handed over and continued on its own.

## Where it stands

The Philippines comparison (WWF-SIPA's regional run as the benchmark) established:

- **The pipeline is sound.** WWF-SIPA's own CN and Kc run through this pipeline at 90m reproduce
  their 30m run (r=0.92 for quickflow and baseflow). That run is the reference for all comparisons.
- **Global data gets most land cover close.** GCN250 CN plus per-pixel monthly Kc from published
  calibrations gives baseflow 1.03x the reference overall (r=0.66).
- **The remaining gap is cropland CN**, mostly paddy rice (annual crop baseflow 2.0x; the
  reference's CN alone brings it to 1.25x). No global CN source describes flooded paddies.
- **Proposed path to a global run:** global defaults, a cropland lookup table keyed by climate zone
  x crop group x water regime (9 combinations cover 80% of global crop area), sensitivity ranges
  instead of a single parameter set. See `docs/swy/global_cropland_parameterization_plan.md`.

Read first: `docs/reports/swy/swy_status_report.html` (rendered report with interactive map),
`docs/reports/swy/FIGURES.md` (guide to the runs and figures), and
`Python_scripts/swy_philippines_run/README.md` ("Current state" section: which scripts form the
current chain).

## Layout

| Path | Contents |
|---|---|
| `Python_scripts/swy_philippines_run/` | Philippines pipeline: input fetches (`02`-`06`), CN/Kc builds (`07*`), model runs (`09*`), maps (`10*`, `11`), comparisons (`12`-`20`) |
| `Python_scripts/swy_borneo_run/` | Earlier Borneo pipeline, and the `Dockerfile` for the model-run image |
| `Python_scripts/swy_global_scoping/` | Global cropland scoping: MapSPAM crop area over Köppen-Geiger climate |
| `docs/swy/` | Methods reference (`swy_methods.qmd`), research log (`research_notes.md`), CN/Kc coverage registry, workflow diagram, global cropland plan, references |
| `docs/reports/swy/` | Status report, figures, interactive map, run/figure guide |
| `analysis/` | Small diagnostic checks (EVI vs NDVI Kc) |

## Running

Commands below are for Git Bash on Windows (the setup this was built on), run from the repository
root; every script uses paths relative to the root. On macOS/Linux, drop `MSYS_NO_PATHCONV=1` and
use `.venv/bin/` instead of `.venv/Scripts/`.

### 1. Python environment (everything except the model runs)

```bash
python -m venv .venv
.venv/Scripts/python -m pip install -r requirements.txt
```

### 2. Docker image (model runs only)

Start Docker Desktop, then build the image. The tag matters: the run scripts' docstrings use
`swy_borneo_run:rainfix4`.

```bash
docker build -t swy_borneo_run:rainfix4 -f Python_scripts/swy_borneo_run/Dockerfile Python_scripts/swy_borneo_run
```

The image starts from `therealspring/global_ncp-computational-environment` (Docker Hub), clones
`inspring`, and patches two bugs in its spatially distributed rain-events code (details in the
Dockerfile's header).

### 3. A model run

Each `09*` script is one run, with its own workspace under `data/swy/philippines/`. Example, our
current parameter set:

```bash
MSYS_NO_PATHCONV=1 docker run --rm -w /scripts_root -e SWY_DATA_ROOT=/scripts_root/data     --mount type=bind,source="$(pwd)",target=/scripts_root swy_borneo_run:rainfix4     micromamba run -n geopy311 python /scripts_root/Python_scripts/swy_philippines_run/09n_run_swy_ph_paddy_kc.py
```

A Philippines run takes about 1.5 hours. Tracebacks about `currentThread` / `cannot join current
thread` at the end of the log are harmless shutdown noise from `taskgraph`; a successful run ends
with `DONE — outputs in ...`.

### 4. Reproducing the current results

All in `Python_scripts/swy_philippines_run/`, assuming `data/` is in place (see Data):

| Step | Scripts | Environment |
|---|---|---|
| Our CN table | `07b` | Python |
| Our Kc rasters | `07d` → `07e` → `07f` → `07g` → `07h` (optional check: `07i`) | Python |
| Model runs | `09h` (reference), `09n` (ours), `09m`, `09f` (CN/Kc swaps) | Docker |
| Comparison figures and tables | `20` | Python |
| Interactive map | `10c` → `10e` → (`10f`, test layers) → `11` | Python |
| Report | `quarto render docs/reports/swy/swy_status_report.qmd` | Quarto |

Global scoping (`Python_scripts/swy_global_scoping/01` → `02`) only needs the MapSPAM and
Köppen-Geiger data in `data/swy/shared/`. Which run is which, and what each figure shows:
`docs/reports/swy/FIGURES.md`.

## Data

`data/` is not in the repository (about 150 GB). It is rebuilt from:

- **Public sources, fetched by scripts:** SRTM DEM, MODIS MOD13A3 NDVI/EVI (NASA AppEEARS,
  needs an Earthdata login), CHIRPS, TerraClimate, NASA POWER net radiation, MapSPAM 2020,
  Sacks et al. crop calendar, GCN250, HYSOGs250m, Köppen-Geiger (Beck et al. 2018).
- **Partner data, not redistributable:** WWF-SIPA's Philippines inputs and baseline outputs
  (`data/swy/philippines/INPUTS_SP/`, `data/swy/philippines/rich_shared/`). Request from WWF-SIPA.
