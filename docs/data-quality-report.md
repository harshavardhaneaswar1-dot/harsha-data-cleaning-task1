# Data-Quality Report — Netflix Titles 2021

Source: `data/netflix_titles_2021.csv` — 8807 rows × 12 columns.
Method: `Import-Csv` full scan; blank/whitespace counted as missing;
`show_id` uniqueness checked via `Sort-Object -Unique`.

## 1. Missing-value table

| Column | Missing count | Missing % | Verdict |
|--------|---------------|-----------|---------|
| `show_id` | 0 | 0.00% | Clean, unique (8807 distinct) |
| `type` | 0 | 0.00% | Clean — Movie 6131, TV Show 2676 |
| `title` | 0 | 0.00% | Clean |
| `director` | 2634 | 29.91% | High but expected (see §2) |
| `cast` | 825 | 9.37% | Expected |
| `country` | 831 | 9.44% | Expected |
| `date_added` | 10 | 0.11% | Genuine gaps — keep NULL |
| `release_year` | 0 | 0.00% | Clean (stored as text, cast to int) |
| `rating` | 4 | 0.05% | Genuine gaps |
| `duration` | 3 | 0.03% | Genuine gaps |
| `listed_in` | 0 | 0.00% | Clean |
| `description` | 0 | 0.00% | Clean |

## 2. Rationale per column

- **`director` (29.9%):** Netflix source data routinely omits director for
  stand-up, documentaries, and reality titles. Deleting 2634 rows would bias
  the catalogue; imputing a real name would fabricate data. Decision: replace
  blanks with `"No Information"` for BI readability; exclude the sentinel from
  any `DISTINCTCOUNT`/ML encoding.
- **`cast` (9.4%) / `country` (9.4%):** same logic — missing means "not listed",
  not "zero". Sentinel replacement, same caveat.
- **`date_added` (10):** a date cannot be invented. Blanks stay `null` after
  Text→Date conversion so time filters honestly exclude them.
- **`rating` (4) / `duration` (3):** rare gaps; same sentinel treatment in the
  `.pbix` for slicer visibility. For modelling, prefer `NULL` + indicator.
- **Numbers vs text:** `release_year` Text→Whole Number is lossless;
  `date_added` Text→Date uses `en-US` locale `MMMM d, yyyy` (e.g.
  `September 25, 2021`).

## 3. Duplicates

- 0 exact duplicate rows; `show_id` 8807/8807 unique.
- `Table.Distinct` retained as an idempotent guard step.

## 4. Type & consistency checks

- Column names already consistent snake_case — verified, no rename.
- `duration` mixes `min` (movies) and `Season(s)` (TV) — left unsplit;
  splitting is a modelling decision, not a data error.
- `listed_in` / `country` are multi-valued (`", "` separated) — left as-is
  pending a bridge-table design.

## 5. Reproduce

```powershell
$csv = "data/netflix_titles_2021.csv"
$data = Import-Csv $csv
$data.Count  # 8807
($data | Where-Object { [string]::IsNullOrWhiteSpace($_.director) }).Count  # 2634
($data.show_id | Sort-Object -Unique | Measure-Object).Count  # 8807
$data | Group-Object type | Select-Object Name, Count
# Movie: 6131, TV Show: 2676
```
