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

