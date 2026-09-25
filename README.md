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

All scripts are run from the repository root; paths inside them are relative to it.

- **Analysis** (fetches, CN/Kc builds, comparisons, figures): a Python environment with
  `requirements.txt`.
- **Model runs** (`09*`): Docker. Build the image from `Python_scripts/swy_borneo_run/Dockerfile`
  (tag used here: `swy_borneo_run:rainfix4`), start Docker Desktop, then run the command given in
  each `09*` script's docstring. A Philippines run takes about 1.5 hours.

## Data

`data/` is not in the repository (about 150 GB). It is rebuilt from:

- **Public sources, fetched by scripts:** SRTM DEM, MODIS MOD13A3 NDVI/EVI (NASA AppEEARS,
  needs an Earthdata login), CHIRPS, TerraClimate, NASA POWER net radiation, MapSPAM 2020,
  Sacks et al. crop calendar, GCN250, HYSOGs250m, Köppen-Geiger (Beck et al. 2018).
- **Partner data, not redistributable:** WWF-SIPA's Philippines inputs and baseline outputs
  (`data/swy/philippines/INPUTS_SP/`, `data/swy/philippines/rich_shared/`). Request from WWF-SIPA.
