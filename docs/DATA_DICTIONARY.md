# Data Dictionary

Output column definitions for the extraction pipeline. These are the project's
**output** names; the actual Revelio/WRDS source columns are mapped to them in the
notebook's `P` (positions) and `E` (education) dictionaries.

## Positions (`P`)

| Output column | Description |
|---|---|
| `user_id` | Individual identifier (stored as text to preserve precision) |
| `position_id` | Unique position/role identifier |
| `rcid` | Company identifier for the employer of the position |
| `company_name` | Employer name (identifies destination employers in follow-up) |
| `startdate` | Position start date |
| `enddate` | Position end date (null = ongoing) |
| `country` | Country of the position |
| `state` | State/region of the position |
| `title` | Job title |
| `occupation` | Occupation taxonomy value (one documented level used throughout) |
| `seniority` | Seniority level code |
| `salary` | Estimated/observed compensation |

## Education (`E`)

| Output column | Description |
|---|---|
| `user_id` | Individual identifier (joins to positions) |
| `school` | Institution name/identifier |
| `degree` | Degree type (e.g., bachelor's) |
| `major` | Field of study |
| `grad_date` | Graduation date |

## Generated artifacts

| File | Description |
|---|---|
| `google_2018_candidates_raw.csv` | Raw positions starting in 2018 (junior, U.S.) at Google |
| `user_ids.txt` | Unique candidate `user_id` values |
| `education_raw.csv` | All education records for candidate users |
| `positions_all_employers_raw.csv` | All positions for candidates across all employers/countries |
| `manifest.json` | Extraction configuration and provenance metadata |

## Notes

- ID fields (`user_id`, `position_id`, `rcid`, `school`) are cast to text during
  extraction to avoid floating-point precision loss.
- End dates are treated as inclusive; a null end date denotes an ongoing position.
- No cleaning, deduplication, or analytical preprocessing happens during extraction —
  those steps are documented separately in the analysis stage.
