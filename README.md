# Lab 3 – Clean the NYC Taxi Data, Then Stop

INF4058 Data Engineering. This project profiles and cleans the NYC Yellow Taxi
trip data for **January 2021** (`yellow_tripdata_2021-01.csv.gz`, 1,369,765 rows).

**Scope:** the work stops at cleaning. The cleaned data is **not** written to disk
and **not** loaded into PostgreSQL. That connection is made in the next session.

## What the notebook does

`lab3_cleaning.ipynb` follows the assignment steps:

| Section | What it does |
|---|---|
| 0. Setup | Imports, constants, downloads the file if it is missing |
| 1. Look | First 100 rows: `head()`, `dtypes`, columns, shape |
| 2. Profile | Whole file in chunks of 100,000 rows: dtype, nulls (count and %), min, max |
| 3. Find problems | Exact row counts for each data problem |
| 4. Decide | Policy table: problem, decision (drop / fix / flag), reason, rule owner |
| 5. Clean | `clean(df)` returns `(cleaned_df, {rule: rows_removed})`; sample test; full run |
| 6. Verification | 26 assertion-based tests (printed as a table) |

## Cleaning rules

| Problem | Decision | Owner |
|---|---|---|
| Pickup missing/unparseable | drop | data engineering |
| Pickup outside Jan 2021 (`>= 2021-01-01` and `< 2021-02-01` is valid) | drop | data engineering |
| Dropoff before pickup | drop | data engineering |
| `fare_amount <= 0` | drop | business / data owner |
| `trip_distance <= 0` | drop | business / data owner |
| Duplicate rows (first copy kept) | drop | data engineering |
| `passenger_count == 0` | flag (`passenger_count_zero_flag`) | business / data owner |
| `passenger_count` missing | flag (`passenger_count_missing_flag`) | business / data owner |
| `VendorID` missing | flag (`vendor_missing_flag`) | data engineering |

No rule uses `fix`: there is no reliable way to recover a correct value, and we do not invent one.

Rules run in a fixed order, and each row is counted under the **first** rule that
removes it, so per-rule numbers add up exactly to the rows removed.

**Duplicates across chunks:** every row gets a 128-bit fingerprint (two different
64-bit hashes of all columns). Only fingerprints are kept in a set shared by all
chunks, so duplicates are found across chunk boundaries without holding the data in
memory. Limitation: it is a hash comparison, so a collision is theoretically
possible (about 1e-26 for this file size).

## How to run

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook lab3_cleaning.ipynb
```

Then run all cells (Kernel → Restart & Run All).

**Data file.** Put `yellow_tripdata_2021-01.csv.gz` next to the notebook. If it is not
there, section 0 downloads it from the public mirror
<https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz>.
To use a file stored somewhere else, set the environment variable `TAXI_CSV` to its path.

The full run reads the file several times (profile, problems, cleaning, verification)
and takes a few minutes. Memory stays small because only 100,000 rows are loaded at a time.

Tested with Python 3 and pandas 3.0.

## Repository contents

```
lab3_cleaning.ipynb   notebook with all code and printed results
README.md
requirements.txt
.gitignore            excludes .venv, data files, results/, secrets
Lab3_Report.pdf       the report
```

The dataset, `.venv/` and any keys are intentionally **not** in the repository.
