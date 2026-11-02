# ProjectSample

Annual SSR event-study pipeline for studying director mobility, interfirm board connections, and pharmaceutical product outcomes.

The project builds annual leadership rosters and firm-level event tables, links annual events to SSR product outcomes, constructs balanced cohorts, prepares common-ownership controls, and estimates dynamic and ATC-sharing effects.

## Workflow

```text
BoardEx affiliations and SSR firm universe
                ↓
Annual leadership rosters
                ↓
Director-movement and interlock events
                ↓
Firm-product event-study panels
                ↓
Balanced cohorts and Kappa controls
                ↓
Dynamic event studies and ATC-sharing DDD
```

Annual movement events are dated at the calendar-year level. The estimation programs anchor those events in Q1 when they are merged with quarterly SSR product outcomes.

## Repository layout

```text
python/
  build_ssr_roster.py       Build imputed and sentinel-dropped annual rosters
  build_raw_events.py       Construct movement candidates and firm interlocks
  build_event_tables.py     Create req0/req1/req2 firm-event tables
  build_firm_panels.py      Merge events with SSR firm-product outcomes
  build_cohorts.py          Build balanced event cohorts and diagnostics
  build_kappa_controls.py   Prepare common-ownership controls
  pipeline_config.py        Paths, roster variants, and sample definitions
stata/
  event_study.do            Dynamic did_imputation specifications
  atc_sharing_ddd.do        ATC3-sharing triple-difference specifications
```

## Required inputs

Keep confidential and licensed data outside version control. Point `PROJECTSAMPLE_DATA_ROOT` to a directory containing:

```text
InterimData/
  boardex_pharma.dta
  boardex_ssr_price_sample.csv
  cro_bname_boardex_citeline.dta
  cro_bname_boardex_within.dta
RawData/
  boardex/boardex_na/individual_employment.csv
  boardex/boardex_na/organization_composition.csv
  common_ownership/processed_13f_data.csv
  common_ownership/cikmap.csv
crosswalks/
  ssr_cusip_map.csv
  ssr_company_states_size.csv
```

The event and panel builders also consume the annual roster files created in the first step. Generated outputs are written under `data/`, `csv/`, `figures/`, `logs/`, and `tex/`.

## Setup

Python 3.10 or newer is recommended.

```bash
python -m venv .venv
python -m pip install -r requirements.txt
```

Set the data location before running the pipeline:

```powershell
$env:PROJECTSAMPLE_DATA_ROOT = "D:\path\to\project-data"
```

Optional variables:

- `PROJECTSAMPLE_OUTPUT_ROOT`: generated Python outputs; defaults to the repository root.
- `PROJECTSAMPLE_ROSTER_OUTPUT_ROOT`: annual roster output directory; defaults to `data/`.
- `PROJECTSAMPLE_VARIANT_OUTPUT_ROOT`: variant-specific panel and cohort root.
- `PROJECTSAMPLE_ROSTER_VARIANTS`: comma-separated annual variants; defaults to `imputed_year,drop_sentinel_year`.
- `PROJECTSAMPLE_PERSONNEL_DEFINITIONS`: comma-separated sample definitions; defaults to `narrow,medium,broad`.

## Run order

From the repository root:

```powershell
python python/build_ssr_roster.py
python python/build_raw_events.py
python python/build_event_tables.py
python python/build_firm_panels.py
python python/build_cohorts.py
python python/build_kappa_controls.py --help
```

The Kappa builder exposes command-line options for the holdings inputs, CUSIP map, and selected variants. Review `--help` before the first run.

Run the Stata programs after cohort and Kappa outputs are available:

```stata
do stata/event_study.do
do stata/atc_sharing_ddd.do
```

Both programs accept optional positional filters for roster variant, personnel definition, requirement level, fixed effects, outcome, treatment side, event type, control group, and Kappa specification.
