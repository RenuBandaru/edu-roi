# EduROI: Mining Post-Graduation Career Outcomes to Assess Returns to Education

A data mining project (Penn State **DAAN-545**) that tracks a cohort of entry-level
Google hires from 2018 across employers, geographies, and seniority levels over a
five-year horizon. The goal is to study early-career return on investment (ROI) and
labor mobility using large-scale résumé/employment data from
[Revelio Labs](https://www.reveliolabs.com/) accessed through
[WRDS](https://wrds-www.wharton.upenn.edu/).

> **Status:** Data-extraction pipeline. This repository contains the reproducible
> extraction and configuration code only. No proprietary WRDS/Revelio records or
> empirical results are committed to version control (see [Data & Privacy](#data--privacy)).

---

## Table of Contents
- [Research Question](#research-question)
- [Cohort Definition](#cohort-definition)
- [Repository Structure](#repository-structure)
- [Pipeline Overview](#pipeline-overview)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Outputs](#outputs)
- [Data & Privacy](#data--privacy)
- [Reproducibility](#reproducibility)
- [Tech Stack](#tech-stack)
- [Author](#author)
- [License](#license)

---

## Research Question

How do entry-level employees who started at Google in 2018 progress over their first
five years? The project follows each individual across **all** subsequent employers
and countries to examine advancement (seniority changes), retention versus attrition,
and compensation trajectories, while holding education background comparable.

## Cohort Definition

The candidate cohort is defined precisely to keep the study reproducible and defensible:

| Dimension | Rule |
|---|---|
| Employer | Google LLC (verified company `RCID`, not the parent entity) |
| Entry year | Position **starts** during calendar year 2018 (`2018-01-01` to `2018-12-31`) |
| Level | Entry / junior seniority codes |
| Location (salary model) | U.S. positions retained for comparability |
| Follow-up | Five years from entry, tracking **all** employers and countries |
| Education | U.S. bachelor's requirement applied during preprocessing |

Design notes baked into the pipeline:
- These are **first observed** Google entrants. Earlier employment history is pulled
  so transfers/internal moves can be distinguished from genuine new hires.
- Entry-level codes may include internships; the raw extract keeps them and any
  internship rule is applied later as a documented preprocessing step.
- Records with unknown start dates are retained for manual review rather than
  silently dropped, to avoid biasing the follow-up history.

## Repository Structure

```
edu-roi/
├── Revelio_Data_Extraction.ipynb   # Main extraction pipeline (run cell by cell)
├── data/                           # Local-only extracted data (git-ignored)
├── results/                        # Local-only figures/tables (git-ignored)
├── requirements.txt                # Python dependencies
├── docs/
│   └── DATA_DICTIONARY.md          # Output column definitions
├── .gitignore
├── LICENSE
└── README.md
```

## Pipeline Overview

The notebook `Revelio_Data_Extraction.ipynb` runs as a sequence of guarded steps:

1. **Configuration** – set the cohort window, follow-up horizon, Google `RCID`, and
   junior seniority codes.
2. **Metadata discovery** – list the authorized Revelio library, tables, and columns
   on WRDS (no records pulled yet).
3. **Column mapping** – map source columns to the project's output schema, with
   assertions that block extraction until every field and the `RCID` are supplied.
4. **Candidate extraction** – pull positions that *start* in 2018 at junior level in
   the U.S., saving the raw candidate set and the unique user-ID lookup list.
5. **Follow-up extraction** – for every candidate, pull all education records and all
   positions across all employers/countries through the fifth-anniversary cutoff.
6. **Provenance** – write a manifest capturing the exact configuration so the extract
   can be reproduced.

Safety features built into the code:
- SQL identifiers are validated against a strict pattern and quoted; value filters are
  passed as query parameters (no string interpolation of user values).
- ID columns are cast to text to avoid floating-point precision loss.
- Extraction is gated behind explicit coverage-confirmation and mapping assertions.

## Getting Started

### Prerequisites
- Python 3.11
- A **WRDS account with authorized access** to the Revelio Labs individual data.
  Access is required to run the extraction; the code alone does not grant access.

### Installation

```bash
git clone https://github.com/<your-username>/edu-roi.git
cd edu-roi
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Run

Open the notebook and execute one cell at a time, filling in the configuration
checkpoints as prompted:

```bash
jupyter lab Revelio_Data_Extraction.ipynb
```

You will be asked for your WRDS credentials at connection time. **Never store your
password in the notebook.**

## Configuration

Key parameters live near the top of the notebook:

| Parameter | Meaning |
|---|---|
| `START_FROM` / `START_BEFORE` | Cohort entry window (2018) |
| `FOLLOWUP_YEARS` | Follow-up horizon (default 5) |
| `RCID` | Verified Google LLC company ID |
| `JUNIOR` | Entry/junior seniority codes |
| `US_JOB` | Country value used for the salary model |
| `HISTORY_COVERAGE_CONFIRMED` | Must be `True` once you confirm access to earlier and cross-employer history |

Column mapping dictionaries `P` (positions) and `E` (education) must be completed
before the extraction cells will run.

## Outputs

Written to a local `revelio_google_2018_entrants/` (or `data/`) folder — **not**
committed:

| File | Contents |
|---|---|
| `google_2018_candidates_raw.csv` | Raw 2018 junior U.S. Google positions |
| `user_ids.txt` | Unique candidate user IDs |
| `education_raw.csv` | All education records for candidates |
| `positions_all_employers_raw.csv` | All positions across all employers/countries |
| `manifest.json` | Extraction configuration and provenance |

See [`docs/DATA_DICTIONARY.md`](docs/DATA_DICTIONARY.md) for column definitions.

## Data & Privacy

- Revelio Labs data accessed via WRDS is **proprietary and license-restricted**.
  No records, extracts, or derived datasets are committed to this repository.
- The `data/` and `results/` folders are git-ignored by design.
- WRDS credentials are entered interactively at runtime and never stored in code.
- This repository shares only the extraction *method*, not the underlying data.

## Reproducibility

Given authorized WRDS access, the study is reproducible by cloning the repo,
installing dependencies, supplying the verified configuration, and running the
notebook top to bottom. The generated `manifest.json` records the exact parameters
used for each extract.

## Tech Stack

- **Python 3.11**, Jupyter
- **pandas**, **numpy** for data handling
- **wrds** for database access
- **scikit-learn**, **statsmodels**, **mlxtend**, **matplotlib** for the downstream
  modeling and analysis stages

## Author

**Venkata Sai Renusree Bandaru**
Graduate coursework — DAAN-545, Data Mining, The Pennsylvania State University.

## License

Released under the MIT License for the **code**. See [LICENSE](LICENSE).
This license does **not** cover the Revelio Labs / WRDS data, which remains subject to
its own licensing terms.
