# CMS-DE10-Inpatient-R-SPSS

Clean and prepare CMS DE-SynPUF inpatient claims (Sample 2, 2008–2010) in R for downstream analysis in SPSS.

> **Note**: This project uses *synthetic Medicare data* (DE-SynPUF Sample 2); it contains no real patient records.

---

## 📦 Overview

This project provides a reproducible R pipeline for processing the **CMS 2008–2010 Data Entrepreneurs’ Synthetic Public Use File (DE-SynPUF)** — specifically the **Inpatient Claims, Sample 2** dataset.

The goals are to:

- Import raw synthetic Medicare inpatient claims data
- Clean, recode, and label variables for clarity
- Export tidy datasets as:
  - `.csv` — for easy inspection or use in SPSS
  - `.sav` — for SPSS with full variable/value labels preserved

The cleaned output is ready for descriptive or inferential analysis in SPSS.

---

## 📁 Data source

- **Dataset**: [CMS DE-SynPUF 2008–2010, Sample 2 – Inpatient Claims](https://www.cms.gov/data-research/statistics-trends-and-reports/medicare-claims-synthetic-public-use-files/cms-2008-2010-data-entrepreneurs-synthetic-public-use-file-de-synpuf/de10-sample-2)
- **Type**: Synthetic 5% Medicare claims sample
- **Years covered**: 2008–2010
- **Files used**: `DE1_0_2008_to_2010_Inpatient_Claims_Sample_2.csv`

This data is fully synthetic and publicly available, suitable for method development, teaching, or pipeline testing.

---

## 🛠️ Project structure

```text
.
├── data/                              # All input & output files live here
│   ├── inpatient_claims_raw.csv                           # raw input
│   ├── inpatient_claims_clean.csv                         # cleaned output
│   └── inpatient_claims_clean.sav                         # SPSS output
│
├── R/
│   └── inpatient_claims.Rmd                               # Main processing script
├── README.Rmd                                             <- This file

```

---

## 🧹 Cleaning steps performed

The `inpatient_claims.Rmd` script performs the following steps:

1. **Read and parse data**  
   - Loads `inpatient_claims_raw.csv` using `readr::read_csv()`  
   - Sets `CLM_ID` and `DESYNPUF_ID` as `character` to prevent rounding or truncation  

2. **Drop uninformative columns**  
   - Removes any constant columns (e.g. 100% NA) using `janitor::remove_constant()`  

3. **Standardize names and parse dates**  
   - Renames columns to `snake_case`  
   - Parses key date fields: `admit_date`, `thru_date`, `discharge_date`  
   - Converts `drg_code` to factor  

4. **Assign bill-type logic**  
   - Identifies a bill-type field (`bill_type`, `clm_type_cd`, or `tob`)  
   - Extracts final digit as `tob_last`  
   - Assigns `tob_rank` to prioritize claims:
     - `7` → replacement (rank 1)  
     - `1` or `4` → final (rank 2)  
     - others → interim/void/no-pay (rank 3)  
   - If bill type is missing (e.g., DE-SynPUF), sets:
     - `tob_last = NA`
     - `tob_rank = 2L` (assumes final-action claim)

5. **De-duplicate claims**  
   - For each `clm_id`, retains the highest-priority row by:
     - `tob_rank` (lowest = best)
     - Latest of `discharge_date` or `thru_date`

6. **Calculate Length-of-Stay (LOS)**  
   - For claims with `tob_rank ≤ 2`, calculates:
     - `discharge_date - admit_date`, if available  
     - Otherwise `thru_date - admit_date`  

7. **Label variables for SPSS compatibility**  
   - Uses `labelled::var_label()` to apply variable descriptions  
   - Includes labels for stay dates, DRG, and bill-type variables  

8. **Export cleaned data**  
   - Writes both `.csv` and `.sav` files to the `data/` folder  

---

## 💾 Outputs

- `data/inpatient_claims_clean.csv` — clean, analysis-ready data  
- `data/inpatient_claims_clean.sav` — SPSS-compatible file with labels

---

## 📦 R packages used

| Package    | Purpose                                                                 |
|------------|-------------------------------------------------------------------------|
| `tidyverse` | Core data wrangling (`read_csv()`, `mutate()`, `arrange()`, etc.)       |
| `janitor`   | Remove constant columns and clean variable names                        |
| `lubridate` | Parse and manipulate date fields (e.g., `ymd()`)                        |
| `glue`      | Construct readable warnings and messages                                |
| `labelled`  | Add SPSS-style variable labels                                          |
| `haven`     | Export `.sav` files for SPSS with metadata                              |

To install all required packages:

```r
install.packages(c("tidyverse", "janitor", "lubridate", "glue", "labelled", "haven"))
```
