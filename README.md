# CMS-DE10-Inpatient-R-SPSS

Clean and prepare CMS DE-SynPUF inpatient claims (Sample 2, 2008–2010) in R for downstream analysis in SPSS.

> **Note**: This project uses *synthetic Medicare data* (DE-SynPUF Sample 2); it contains no real patient records.

---

## 📦 Overview

This project provides a reproducible R pipeline for processing the **CMS 2008–2010 Data Entrepreneurs’ Synthetic Public Use File (DE-SynPUF)**, specifically the **Inpatient Claims, Sample 2** dataset.

The goals are to:

- Import raw synthetic Medicare inpatient claims data
- Clean, recode, and label variables for clarity
- Filter claims to ensure dates fall within **2008–2010**  
- Retain **only inpatient claims** (`tob_first == 1`) 
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
├── data/                                                  # All input & output files live here
│   ├── inpatient_claims_raw.csv                           # Raw input
│   ├── inpatient_claims_clean.csv                         # Cleaned output (CSV)
│   └── inpatient_claims_clean.sav                         # Cleaned output for SPSS (with labels)
│
├── R/
│   └── inpatient_claims.Rmd                               # Main processing script
├── README.Rmd                                             # Project overview

```

---

## 🧹 Cleaning steps performed

The `inpatient_claims.Rmd` script performs the following steps:

### 1. **Read and parse data**  
   - Loads `inpatient_claims_raw.csv` using `readr::read_csv()`  
   - Sets `CLM_ID` and `DESYNPUF_ID` as `character` to prevent rounding or truncation  

### 2. **Drop uninformative columns**  
   - Removes any constant columns (e.g. 100% NA) using `janitor::remove_constant()`  

### 3a. **Standardize names and basic types**
- Converts column names to `snake_case` using `janitor::clean_names()`  
- Parses date columns into `Date` format:  
  - `admit_date`, `from_date`, `thru_date`, `discharge_date`  
- Converts `clm_drg_cd` into a factor as `drg_code`

### 3b. **Validate date ranges (2008–2010)**
- Defines a valid service window: **2008-01-01 to 2010-12-31**  
- Filters out any claims where:
  - `admit_date`, `from_date`, or `thru_date` is missing or outside the range  
- If `discharge_date` is present but out of bounds, it is replaced with `NA`  
- Logs how many rows were removed due to invalid or missing date values
 

### 4a. **Assign bill-type logic**  
   - Identifies a bill-type field (`bill_type`, `clm_type_cd`, or `tob`)
   - Extracts first digit as `tob_first` (used to identify inpatient claims)
   - Extracts final digit as `tob_last` (used to infer claim status: final, interim, replacement, etc.)  
   - Assigns `tob_rank` to prioritize claims:
     - `7` → replacement (rank 1)  
     - `1` or `4` → final (rank 2)  
     - others → interim/void/no-pay (rank 3)
   - Filters to inpatient claims only: keeps rows where `tob_first == 1`  
   - If bill type is missing (e.g., DE-SynPUF), sets:
     - `tob_first = NA`
     - `tob_last = NA`
     - `tob_rank = 2L` (assumes final-action claim)

### 4b. **De-duplicate claims**  
   - For each `clm_id`, retains the highest-priority row by:
     - `tob_rank` (lowest = best)
     - Latest of `discharge_date` or `thru_date`

### 5. **Calculate Length-of-Stay (LOS)**  
   - For claims with `tob_rank ≤ 2`, calculates:
     - `discharge_date - admit_date`, if available  
     - Otherwise `thru_date - admit_date`

### 6. **Calculate billing span**  
   - Computes the number of days between `from_date` and `thru_date`  
   - Stored in new variable `bill_span`  

### 7. **Add value labels for SPSS export (requires integer conversion)** 
   - Converts `tob_first`, `tob_last`, and `tob_rank` to integers
   - Applies SPSS-style value labels using `labelled::val_labels()`
  
### 8. **Label variables for SPSS compatibility**  
   - Uses `labelled::var_label()` to apply variable descriptions
   - Includes labels for LOS, billing span, stay dates, DRGs, bill-type variables, ICD-9 codes, etc.

### 9. **Export cleaned data**  
   - Writes both `.csv` and `.sav` files to the `data/` folder  

---

## 💾 Outputs

- `data/inpatient_claims_clean.csv`: clean analysis-ready data for general use  
- `data/inpatient_claims_clean.sav`: SPSS-compatible dataset with full labeling


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
