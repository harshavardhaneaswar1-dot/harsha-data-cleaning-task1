# Data Cleaning & Preprocessing — Netflix Titles (2021)

> Task 1: Data Cleaning and Preprocessing using Power BI (Power Query).

## 1. Objective

Take the raw Netflix Titles dataset (Kaggle, 2021 snapshot) and produce an
analysis-ready table in Power BI by:

1. Profiling data quality (missing values, duplicates, wrong types).
2. Applying reproducible cleaning steps in **Power Query (M)**.
3. Validating the result (row counts, type checks, spot checks).

Original submission note (kept verbatim in spirit — see §6 for the exact
original steps):

> *I have completed Task 1: Data Cleaning and Preprocessing using Power BI.*
> *Files: original Kaggle CSV + Power BI project with cleaned dataset and*
> *all transformation steps.*

## 2. Dataset Description

- **Source:** Netflix Titles 2021, downloaded from Kaggle.
- **Raw file:** `data/netflix_titles_2021.csv`
- **Shape:** **8807 rows × 12 columns** (verified with `Import-Csv`).
- **Grain:** one row per title (`show_id` is unique — 8807 distinct values).

| # | Column | Example | Notes |
|---|--------|---------|-------|
| 1 | `show_id` | `s1` | Unique ID, no missing |
| 2 | `type` | `Movie` / `TV Show` | No missing |
| 3 | `title` | `Dick Johnson Is Dead` | No missing |
| 4 | `director` | `Kirsten Johnson` | 29.9% missing — normal for Netflix (many titles list no director) |
| 5 | `cast` | `…` | 9.4% missing |
| 6 | `country` | `United States` | 9.4% missing |
| 7 | `date_added` | `September 25, 2021` | Text in raw CSV, 10 missing (0.11%) |
| 8 | `release_year` | `2020` | Text in raw CSV, no missing |
| 9 | `rating` | `PG-13` / `TV-MA` | 4 missing (0.05%) |
| 10 | `duration` | `90 min` / `2 Seasons` | 3 missing (0.03%) |
| 11 | `listed_in` | `Documentaries` | Genre list, no missing |
| 12 | `description` | `…` | No missing |

Key distribution (raw):

- **Movie: 6131 (69.6%) / TV Show: 2676 (30.4%)**

## 3. Data-Quality Issues Found

Full audit in [`docs/data-quality-report.md`](docs/data-quality-report.md).
Summary:

| Column | Missing | % | Issue |
|--------|---------|---|-------|
| `director` | 2634 | 29.91% | Expected — documentaries/stand-up often lack director credit |
| `cast` | 825 | 9.37% | Same — no cast listed |
| `country` | 831 | 9.44% | No production country listed |
| `date_added` | 10 | 0.11% | Blank — title never stamped with add date |
| `rating` | 4 | 0.05% | Blank maturity rating |
| `duration` | 3 | 0.03% | Blank runtime/seasons |
| others | 0 | 0% | Clean |

Other issues:

- **Duplicates:** 0 exact duplicate rows; `show_id` fully unique. Dedup step
  kept as a guard (idempotent).
- **`date_added` type:** stored as free text e.g. `September 25, 2021`.
  Must be parsed to `Date` for time analysis; parsing fails loudly on typos,
  which is desirable.
- **`release_year` type:** stored as text; must be `Whole Number` for
  sorting/arithmetic.
- **Naming:** column names already consistent snake_case — verified, no rename
  needed.
- **Inconsistencies:** mixed `duration` units (`min` vs `Seasons`) — kept as-is
  because splitting them is a modelling choice, not a cleaning error.

## 4. Cleaning Steps & Theory (what + why)

These are the exact steps from the original submission, expanded with rationale:

1. **Remove duplicate rows.**
   *Why:* duplicates inflate counts and break `show_id` uniqueness.
   `Table.Distinct` on all columns is safe and idempotent — with 0 duplicates
   found it changes nothing but protects against future appends.

2. **Convert `date_added` Text → Date.**
   *Why:* text cannot be filtered by year/month/quarter, sorted chronologically,
   or used in time-intelligence. Power Query `Date.FromText` with locale
   `en-US` parses `MMMM d, yyyy`. The 10 blanks stay `null` (a real date
   cannot be invented).

3. **Convert `release_year` Text → Whole Number.**
   *Why:* prevents lexicographic sort bugs (`"1999" > "2000"` as text in some
   contexts) and enables numeric visuals. Values are all 4-digit years, so
   `Int64.Type` is lossless.

4. **Replace missing text (`director`, `cast`, `country`, + `rating`/`duration` blanks) with `"No Information"`.**
   *Why / trade-off:* Power BI slicers and `COUNT` visuals drop `null`/blank
   silently, confusing end users ("where did 29.9% of directors go?").
   An explicit sentinel keeps rows visible and countable.
   *Trade-off acknowledged:* `"No Information"` is **not** real data — any
   `DISTINCTCOUNT(director)` must exclude it, and for ML work `NULL` + a
   `is_missing` flag is preferable. For a BI report aimed at non-technical
   readers, the sentinel is the pragmatic choice. Applied only to categorical
   text columns, never to dates/numbers.

5. **Verify consistent column naming.**
   *Why:* downstream DAX, merges, and refreshes break on `Title` vs `title`
   vs `Title `. All 12 names already snake_case — check is documented so a
   future schema change is caught.

6. **Review for inconsistencies and correct where appropriate.**
   *Why:* catch-alls for leading/trailing spaces, double spaces, inconsistent
   capitalisation in `type`/`rating`/`country`. No destructive edits were made
   to `duration` units or `listed_in` multi-values — those need a star-schema
   decision first.

### Concepts learned

- Missing Values · Duplicate Rows · Data Types · Date Conversion ·
  Data Cleaning · Data Preprocessing · Replacing Missing Text Values using
  Power Query (from the original submission).

## 5. Tools

- **Power BI Desktop + Power Query (M)** — all transforms are stored as
  applied steps inside `reports/harsha.pbix`. No Python needed.
- Verification here used PowerShell `Import-Csv` (row/column/missing counts).

## 6. Original Submission Steps (preserved)

> - Removed duplicate rows.
> - Converted the "date_added" column from Text to Date.
> - Converted the "release_year" column from Text to Whole Number.
> - Replaced missing values in text columns (such as "director", "cast", and "country") with "No Information".
> - Verified that all column names follow a consistent naming convention.
> - Reviewed the dataset for inconsistencies and corrected data where appropriate.

## 7. How to Reproduce

1. Open `reports/harsha.pbix` in Power BI Desktop.
2. **Home → Transform data** to open Power Query Editor.
3. In **Query Settings → Applied Steps** you will see, in order:
   `Source → Removed Duplicates → Changed Type (date_added → Date, release_year → Whole Number) → Replaced Value (null → "No Information" on director/cast/country) → Renamed/verified columns`.
4. Raw source points at `data/netflix_titles_2021.csv` (8807 rows).
5. Refresh (`Home → Refresh`) should return the same 8807 rows with 0 errors
   on type conversion (except the 10 expected `null` dates).

## 8. Repository Structure

```text
.
├── data/
│   └── netflix_titles_2021.csv   # raw dataset (8807 × 12)
├── reports/
│   └── harsha.pbix               # Power BI project with Power Query steps
├── docs/
│   └── data-quality-report.md    # missing-value audit + rationale
├── README.md
└── .gitignore
```

## 9. Limitations & Future Work

- The `.pbix` is binary — the M code is not diffable here. Future work: paste
  the **Advanced Editor** M script into `docs/power-query-steps.m` and export
  a `data/netflix_titles_cleaned.csv` so reviewers without Power BI can diff.
- `duration` (`90 min` vs `2 Seasons`) and multi-value `listed_in`/`country`
  are intentionally left unsplit; proper handling means a bridge/genre table.
- `"No Information"` sentinel vs `NULL` trade-off documented above — revisit
  if this feeds an ML pipeline.

## 10. Author

Harsha — Data Cleaning Task 1.
Thank you.
