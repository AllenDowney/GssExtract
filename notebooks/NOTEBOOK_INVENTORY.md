# Notebook Inventory

**Last Updated:** 2025-01-27  
**Total Notebooks:** 9

## Summary

| Notebook | Purpose | Input Files | Output Files | Uses Utils |
|----------|---------|-------------|--------------|------------|
| `00_make_gss_religion_extract.ipynb` | Extract religion data | `gss7221_r2.dta` | `gss_religion.hdf` | No |
| `01_make_pacs_extract.ipynb` | Extract PACS data | `gss7222_r4.dta.gz` | `gss_pacs_2022.hdf` | No |
| `02_make_eds_extract-2022.ipynb` | Extract EDS data (2022) | `gss7222_r1.dta` | `gss_eds_2022.hdf` | No |
| `02_make_eds_extract.ipynb` | Extract EDS data (2021) | `gss7221_r2.dta` | `gss_eds.hdf` | No |
| `02_make_extract-2022_3a.ipynb` | Extract GSS 2022 r3a | `gss7222_r3a.dta.gz` | `gss_extract_2022_3a.hdf` | Yes |
| `02_make_extract-2022_4.ipynb` | Extract GSS 2022 r4 | `gss7222_r4.dta.gz` | `gss_extract_2022_4.hdf` | Yes |
| `02_make_extract-2024_1.ipynb` | Extract GSS 2024 r1 | `gss7224_r1.dta.gz` | `gss_extract_2024_1.hdf` | Yes |
| `03_make_gun_control_extract.ipynb` | Extract gun control data | `gss7221_r2.dta` | `gss_gun_control.hdf` | No |
| `pacs_workshop_extract.ipynb` | Extract PACS workshop data | `gss7222_r4.dta.gz` | `gss_extract_pacs_workshop.hdf` | Yes |

## Detailed Analysis

### 00_make_gss_religion_extract.ipynb
**Purpose:** Extract data for the GssReligion project  
**Input Files:**
- `../data/raw/gss7221_r2.dta` (GSS 2021 Release 2)

**Output Files:**
- `../data/interim/gss_religion.hdf`

**Columns Extracted:** year, god, conclerg, bible, cohort, age, relig, wtssall, wtssps

**Notes:**
- Removes `wtssps` column after merging with `wtssall`
- Uses complevel=7 for HDF compression

---

### 01_make_pacs_extract.ipynb
**Purpose:** Extract data for the Political Alignment Case Study (PACS)  
**Input Files:**
- `../data/raw/gss7222_r4.dta.gz` (GSS 2022 Release 4)

**Output Files:**
- `../data/interim/gss_pacs_2022.hdf`

**Columns Extracted:** 207 columns (comprehensive PACS dataset including political views, social attitudes, demographics, etc.)

**Notes:**
- Combines `reliten`, `relitenv`, and `relitennv` into single `reliten` variable
- Removes `relitenv` and `relitennv` after combining
- Uses complevel=7 for HDF compression
- Includes git commit/push commands

---

### 02_make_eds_extract-2022.ipynb
**Purpose:** Extract data for Elements of Data Science (2022 version)  
**Input Files:**
- `../data/raw/gss7222_r1.dta` (GSS 2022 Release 1)

**Output Files:**
- `../data/interim/gss_eds_2022.hdf`

**Columns Extracted:** 242 columns (similar to PACS but for EDS)

**Notes:**
- Removes `wtssps` column after merging with `wtssall`
- Uses complevel=6 for HDF compression
- Includes git commit/push commands

---

### 02_make_eds_extract.ipynb
**Purpose:** Extract data for Elements of Data Science (2021 version)  
**Input Files:**
- `../data/raw/gss7221_r2.dta` (GSS 2021 Release 2)

**Output Files:**
- `../data/interim/gss_eds.hdf`

**Columns Extracted:** 242 columns (similar to other EDS extracts)

**Notes:**
- Removes `wtssps` column after merging with `wtssall`
- Uses complevel=6 for HDF compression
- Includes git commit/push commands

---

### 02_make_extract-2022_3a.ipynb
**Purpose:** Extract GSS 2022 Release 3a data  
**Input Files:**
- `../data/raw/gss7222_r3a.dta.gz` (GSS 2022 Release 3a)
- `utils.py` (imports `resample_by_year` function)

**Output Files:**
- `../data/interim/gss_extract_2022_3a.hdf`

**Columns Extracted:** 58 columns (focused subset including demographics, attitudes, climate variables)

**Notes:**
- Uses `resample_by_year` from utils.py to create resampled dataset
- Sets random seed to 17
- Removes `wtssps` column after merging with `wtssall`
- Uses complevel=6 for HDF compression
- Includes git commit/push commands

---

### 02_make_extract-2022_4.ipynb
**Purpose:** Extract GSS 2022 Release 4 data  
**Input Files:**
- `../data/raw/gss7222_r4.dta.gz` (GSS 2022 Release 4)
- `utils.py` (imports `resample_by_year` function)

**Output Files:**
- `../data/interim/gss_extract_2022_4.hdf`

**Columns Extracted:** 60 columns (includes pres16, pres20, sexbirth1, sexnow1 in addition to 2022_3a columns)

**Notes:**
- Uses `resample_by_year` from utils.py to create resampled dataset
- Sets random seed to 17
- Removes `wtssps` column after merging with `wtssall`
- Uses complevel=6 for HDF compression
- Includes git commit/push commands

---

### 02_make_extract-2024_1.ipynb
**Purpose:** Extract GSS 2024 Release 1 data  
**Input Files:**
- `../data/raw/gss7224_r1.dta.gz` (GSS 2024 Release 1)
- `utils.py` (imports `resample_by_year` function)

**Output Files:**
- `../data/interim/gss_extract_2024_1.hdf`

**Columns Extracted:** 60 columns (similar to 2022_4, includes pres16, pres20, sexbirth1, sexnow1)

**Notes:**
- Converts `year` column from int16 to float64 to avoid regression issues
- Uses `resample_by_year` from utils.py to create resampled dataset
- Sets random seed to 17
- Removes `wtssps` column after merging with `wtssall`
- Uses complevel=6 for HDF compression
- Includes git commit/push commands

---

### 03_make_gun_control_extract.ipynb
**Purpose:** Extract gun control related data  
**Input Files:**
- `../data/raw/gss7221_r2.dta` (GSS 2021 Release 2)

**Output Files:**
- `../data/interim/gss_gun_control.hdf`

**Columns Extracted:** 17 columns (gunlaw, owngun, gun, natcrime, plus demographics)

**Notes:**
- Removes `wtssps` column after merging with `wtssall`
- Uses complevel=6 for HDF compression
- Includes git commit/push commands

---

### pacs_workshop_extract.ipynb
**Purpose:** Extract GSS data for the PACS Workshop  
**Input Files:**
- `../data/raw/gss7222_r4.dta.gz` (GSS 2022 Release 4)
- `utils.py` (imports `resample_by_year` function)

**Output Files:**
- `../data/interim/gss_extract_pacs_workshop.hdf`

**Columns Extracted:** 30 columns (focused subset for workshop)

**Notes:**
- Uses `resample_by_year` from utils.py to create resampled dataset
- Sets random seed to 17
- Removes `wtssps` column after merging with `wtssall`
- Uses complevel=6 for HDF compression
- Includes git commit/push commands

---

## Dependencies

### utils.py
**Location:** `notebooks/utils.py`  
**Used by:** 4 notebooks
- `02_make_extract-2022_3a.ipynb`
- `02_make_extract-2022_4.ipynb`
- `02_make_extract-2024_1.ipynb`
- `pacs_workshop_extract.ipynb`

**Function:** `resample_by_year(gss, "wtssall")` - Creates resampled dataset using weights

---

## Input File Usage Summary

### Raw Data Files Used:
- `gss7221_r2.dta` - Used by 3 notebooks (religion, EDS 2021, gun control)
- `gss7222_r1.dta` - Used by 1 notebook (EDS 2022)
- `gss7222_r3a.dta.gz` - Used by 1 notebook (extract 2022_3a)
- `gss7222_r4.dta.gz` - Used by 3 notebooks (PACS, extract 2022_4, PACS workshop)
- `gss7224_r1.dta.gz` - Used by 1 notebook (extract 2024_1)

### Unused Raw Data Files:
- `gss7222_r2.dta.gz`
- `gss7222_r3.dta.gz`
- `gss7224_r2.dta.gz`

---

## Output File Summary

All notebooks write to `../data/interim/`:
- `gss_religion.hdf`
- `gss_pacs_2022.hdf`
- `gss_eds_2022.hdf`
- `gss_eds.hdf`
- `gss_extract_2022_3a.hdf`
- `gss_extract_2022_4.hdf`
- `gss_extract_2024_1.hdf`
- `gss_gun_control.hdf`
- `gss_extract_pacs_workshop.hdf`

**Note:** There's also `gss_pacs.hdf` in the interim directory that doesn't appear to be created by any current notebook.

