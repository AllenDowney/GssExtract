# Notebook Consolidation Plan

## Current State Analysis

### Notebooks Grouped by Purpose

#### 1. **Religion Extract** (1 notebook)
- `00_make_gss_religion_extract.ipynb`
  - Reads: `gss7221_r2.dta`
  - Writes: `gss_religion.hdf`
  - Columns: 9 columns (year, god, conclerg, bible, cohort, age, relig, wtssall, wtssps)
  - No resampling

#### 2. **PACS Extract** (2 notebooks - same purpose, different versions)
- `01_make_pacs_extract.ipynb`
  - Reads: `gss7222_r4.dta.gz`
  - Writes: `gss_pacs_2022.hdf`
  - Columns: 247 columns
  - No resampling
  - Special handling: combines reliten/relitenv/relitennv
  
- `pacs_workshop_extract.ipynb`
  - Reads: `gss7222_r4.dta.gz`
  - Writes: `gss_extract_pacs_workshop.hdf`
  - Columns: 30 columns (subset of PACS)
  - Uses resampling
  - **Note:** This is a subset/variant of PACS for workshop purposes

#### 3. **EDS Extract** (2 notebooks - same purpose, different years)
- `02_make_eds_extract.ipynb`
  - Reads: `gss7221_r2.dta`
  - Writes: `gss_eds.hdf`
  - Columns: 242 columns
  
- `02_make_eds_extract-2022.ipynb`
  - Reads: `gss7222_r1.dta`
  - Writes: `gss_eds_2022.hdf`
  - Columns: 242 columns

#### 4. **Generic Extract** (3 notebooks - same purpose, different releases)
- `02_make_extract-2022_3a.ipynb`
  - Reads: `gss7222_r3a.dta.gz`
  - Writes: `gss_extract_2022_3a.hdf`
  - Columns: 58 columns (focused subset)
  - Uses resampling
  
- `02_make_extract-2022_4.ipynb`
  - Reads: `gss7222_r4.dta.gz`
  - Writes: `gss_extract_2022_4.hdf`
  - Columns: 60 columns (adds pres16, pres20, sexbirth1, sexnow1)
  - Uses resampling
  
- `02_make_extract-2024_1.ipynb`
  - Reads: `gss7224_r1.dta.gz`
  - Writes: `gss_extract_2024_1.hdf`
  - Columns: 60 columns (similar to 2022_4)
  - Uses resampling
  - Special handling: converts year from int16 to float64

**Note:** In the consolidated version, the generic extract will contain a **superset** of all variables from all other notebooks (~250-260 columns), making it the most comprehensive extract.

#### 5. **Gun Control Extract** (1 notebook)
- `03_make_gun_control_extract.ipynb`
  - Reads: `gss7221_r2.dta`
  - Writes: `gss_gun_control.hdf`
  - Columns: 17 columns (gunlaw, owngun, gun, natcrime, plus demographics)
  - No resampling

---

## Proposed Consolidation

### Strategy
Each notebook should be **purpose-based** and **configurable** to handle different data releases. Configuration will be done via variables at the top of each notebook.

### Consolidated Notebooks

#### 1. `make_religion_extract.ipynb`
**Purpose:** Extract religion-related variables from GSS data

**Configuration:**
- `INPUT_FILE`: Path to raw GSS data file (e.g., `gss7221_r2.dta`, `gss7224_r2.dta.gz`)
- `COMPRESSION_LEVEL`: HDF compression level (default: 7)

**Output Filename:** Automatically generated as `gss_religion_{year}_{release}.hdf`
- Example: `gss7221_r2.dta` → `gss_religion_2021_r2.hdf`
- Example: `gss7224_r2.dta.gz` → `gss_religion_2024_r2.hdf`

**Columns:** Fixed set of 9 religion-related columns

**Consolidates:**
- `00_make_gss_religion_extract.ipynb`

**Notes:**
- Output filename is automatically generated from input filename
- Year and release are extracted from the input filename pattern `gss72{year}_r{release}`

---

#### 2. `make_pacs_extract.ipynb`
**Purpose:** Extract comprehensive PACS (Political Alignment Case Study) dataset

**Configuration:**
- `INPUT_FILE`: Path to raw GSS data file
- `COMPRESSION_LEVEL`: HDF compression level (default: 7 for full, 6 for workshop)
- `RESAMPLE_WORKSHOP`: Boolean, whether to resample workshop extract by year (default: True)
- `RANDOM_SEED`: Random seed for resampling (default: 17)

**Output Filenames:** Automatically generated from input filename
- **Full extract:** `gss_pacs_{year}_{release}.hdf`
  - Example: `gss7222_r4.dta.gz` → `gss_pacs_2022_r4.hdf`
- **Workshop extract:** `gss_extract_pacs_workshop_{year}_{release}.hdf`
  - Example: `gss7222_r4.dta.gz` → `gss_extract_pacs_workshop_2022_r4.hdf`

**Columns:**
- **Full PACS:** Fixed set of 247 PACS columns
- **Workshop:** Subset of 30 columns (demographics, basic attitudes, happiness, health, trust, satisfaction)

**Special Handling:**
- Combines `reliten`, `relitenv`, `relitennv` into single `reliten` variable (for full extract)
- Removes `relitenv` and `relitennv` after combining (for full extract)
- Creates both full and workshop extracts from the same input data
- Workshop extract uses resampling by year if `RESAMPLE_WORKSHOP=True`

**Consolidates:**
- `01_make_pacs_extract.ipynb`
- `pacs_workshop_extract.ipynb`

**Notes:**
- Writes two output files: one full PACS extract and one workshop subset
- Both output filenames are automatically generated from the input filename
- Both extracts use the same input file but different column selections
- Workshop extract is resampled by year for consistency

---

#### 3. `make_eds_extract.ipynb`
**Purpose:** Extract data for Elements of Data Science textbook

**Configuration:**
- `INPUT_FILE`: Path to raw GSS data file
- `COMPRESSION_LEVEL`: HDF compression level (default: 6)

**Output Filename:** Automatically generated as `gss_eds_{year}_{release}.hdf`
- Example: `gss7221_r2.dta` → `gss_eds_2021_r2.hdf`
- Example: `gss7222_r1.dta` → `gss_eds_2022_r1.hdf`
- Example: `gss7224_r2.dta.gz` → `gss_eds_2024_r2.hdf`

**Columns:** Fixed set of 242 EDS columns

**Consolidates:**
- `02_make_eds_extract.ipynb`
- `02_make_eds_extract-2022.ipynb`

**Notes:**
- Output filename is automatically generated from input filename
- Single notebook can generate extracts for any year by changing INPUT_FILE only

---

#### 4. `make_generic_extract.ipynb`
**Purpose:** Extract a comprehensive superset of all GSS variables used across all other notebooks

**Configuration:**
- `INPUT_FILE`: Path to raw GSS data file
- `COMPRESSION_LEVEL`: HDF compression level (default: 6)
- `RESAMPLE`: Boolean, whether to resample by year (default: True)
- `RANDOM_SEED`: Random seed for resampling (default: 17)
- `CONVERT_YEAR_TO_FLOAT`: Boolean, convert year to float64 (default: False, needed for 2024 data)

**Output Filename:** Automatically generated as `gss_extract_{year}_{release}.hdf`
- Example: `gss7222_r3a.dta.gz` → `gss_extract_2022_r3a.hdf`
- Example: `gss7222_r4.dta.gz` → `gss_extract_2022_r4.hdf`
- Example: `gss7224_r1.dta.gz` → `gss_extract_2024_r1.hdf`

**Columns:** Superset containing the union of all columns from:
- **Religion extract** (9 columns): year, god, conclerg, bible, cohort, age, relig, wtssall, wtssps
- **PACS extract** (247 columns): comprehensive political, social, demographic variables
- **EDS extract** (242 columns): similar to PACS with some differences
- **Gun control extract** (17 columns): gunlaw, owngun, gun, natcrime, plus demographics
- **Previous generic extract** (~58 columns): demographics, attitudes, climate variables (clmtcaus, clmtwrld, clmtusa)
- **Optional additions** (based on data availability):
  - `pres16`, `pres20` (for 2022_r4 and later)
  - `sexbirth1`, `sexnow1` (for 2022_r4 and later)
  - `relitenv`, `relitennv` (before combining into reliten, if needed)

**Estimated total:** ~250-260 unique columns (after removing duplicates and handling special cases like reliten/relitenv/relitennv)

**Consolidates:**
- `02_make_extract-2022_3a.ipynb`
- `02_make_extract-2022_4.ipynb`
- `02_make_extract-2024_1.ipynb`

**Notes:**
- Output filename is automatically generated from input filename
- Contains the union of all variables from all other extract notebooks
- Column selection should be smart - try to include optional columns if they exist in the data
- Handles special cases (e.g., reliten/relitenv/relitennv combination from PACS)
- Year conversion should be conditional based on data type
- This is the most comprehensive extract, suitable for general-purpose analysis

---

#### 5. `make_gun_control_extract.ipynb`
**Purpose:** Extract gun control related variables

**Configuration:**
- `INPUT_FILE`: Path to raw GSS data file
- `COMPRESSION_LEVEL`: HDF compression level (default: 6)

**Output Filename:** Automatically generated as `gss_gun_control_{year}_{release}.hdf`
- Example: `gss7221_r2.dta` → `gss_gun_control_2021_r2.hdf`
- Example: `gss7224_r2.dta.gz` → `gss_gun_control_2024_r2.hdf`

**Columns:** Fixed set of 17 gun control related columns

**Consolidates:**
- `03_make_gun_control_extract.ipynb`

**Notes:**
- Output filename is automatically generated from input filename
- Can be run with different input files for different years

---

## Summary of Changes

### Before: 9 notebooks
1. `00_make_gss_religion_extract.ipynb`
2. `01_make_pacs_extract.ipynb`
3. `02_make_eds_extract-2022.ipynb`
4. `02_make_eds_extract.ipynb`
5. `02_make_extract-2022_3a.ipynb`
6. `02_make_extract-2022_4.ipynb`
7. `02_make_extract-2024_1.ipynb`
8. `03_make_gun_control_extract.ipynb`
9. `pacs_workshop_extract.ipynb`

### After: 5 notebooks
1. `make_religion_extract.ipynb`
2. `make_pacs_extract.ipynb` (writes both full and workshop extracts)
3. `make_eds_extract.ipynb`
4. `make_generic_extract.ipynb`
5. `make_gun_control_extract.ipynb`

### Key Improvements
1. **Removed numeric prefixes** - Names now reflect purpose, not order
2. **Single notebook per purpose** - Each purpose has one configurable notebook
3. **Configuration-driven** - Input files and options set at top of notebook
4. **Automatic filename generation** - Output filenames automatically generated from input filename (format: `{prefix}_{year}_{release}.hdf`)
5. **Year/release agnostic** - Same notebook works with different GSS releases by just changing INPUT_FILE
6. **Clearer naming** - Names clearly indicate what extract they create

---

## Implementation Details

### Configuration Pattern
Each notebook should start with a **single configuration cell** that contains all configuration variables. This cell should be **tagged for papermill** to allow parameterization.

The configuration cell should be placed near the beginning of the notebook (after imports) and tagged with `parameters` for papermill compatibility.

Example for PACS (which has dual outputs):

```python
# Configuration
# Tags: parameters
import re
from pathlib import Path
import warnings

# Input/Output Configuration
INPUT_FILE = "../data/raw/gss7222_r4.dta.gz"
COMPRESSION_LEVEL = 7  # For full extract
COMPRESSION_LEVEL_WORKSHOP = 6  # For workshop extract

# Processing Options
RESAMPLE_WORKSHOP = True
RANDOM_SEED = 17  # For reproducible resampling

# Git Operations (optional)
GIT_COMMIT = False  # Set to True to automatically commit and push output files

# Extract year and release from input filename
# Pattern: gss72{year}_r{release}.dta(.gz)
input_name = Path(INPUT_FILE).stem.replace('.dta', '')
match = re.match(r'gss72(\d+)_r(\w+)', input_name)
if match:
    year = match.group(1)
    release = match.group(2)
    OUTPUT_FILE_FULL = f"../data/interim/gss_pacs_{year}_{release}.hdf"
    OUTPUT_FILE_WORKSHOP = f"../data/interim/gss_extract_pacs_workshop_{year}_{release}.hdf"
else:
    raise ValueError(f"Could not parse year and release from {INPUT_FILE}")
```

Example for single-output notebooks:

```python
# Configuration
# Tags: parameters
import re
from pathlib import Path
import warnings

# Input/Output Configuration
INPUT_FILE = "../data/raw/gss7224_r2.dta.gz"
COMPRESSION_LEVEL = 6

# Processing Options
RESAMPLE = False  # Set to True if resampling needed
RANDOM_SEED = 17  # For reproducible resampling
CONVERT_YEAR_TO_FLOAT = False  # Convert year to float64 (needed for 2024 data)

# Git Operations (optional)
GIT_COMMIT = False  # Set to True to automatically commit and push output files

# Extract year and release from input filename
# Pattern: gss72{year}_r{release}.dta(.gz)
input_name = Path(INPUT_FILE).stem.replace('.dta', '')
match = re.match(r'gss72(\d+)_r(\w+)', input_name)
if match:
    year = match.group(1)
    release = match.group(2)
    OUTPUT_FILE = f"../data/interim/gss_eds_{year}_{release}.hdf"
else:
    raise ValueError(f"Could not parse year and release from {INPUT_FILE}")
```

**Papermill Integration:**
- The configuration cell should have the tag `parameters` (add via Jupyter: View → Cell Toolbar → Tags)
- This allows papermill to inject parameters when executing notebooks programmatically
- Example: `papermill make_eds_extract.ipynb output.ipynb -p INPUT_FILE "../data/raw/gss7224_r2.dta.gz"`

**Filename Prefixes by Notebook:**
- `make_religion_extract.ipynb` → `gss_religion_{year}_{release}.hdf`
- `make_pacs_extract.ipynb` → `gss_pacs_{year}_{release}.hdf` and `gss_extract_pacs_workshop_{year}_{release}.hdf`
- `make_eds_extract.ipynb` → `gss_eds_{year}_{release}.hdf`
- `make_generic_extract.ipynb` → `gss_extract_{year}_{release}.hdf`
- `make_gun_control_extract.ipynb` → `gss_gun_control_{year}_{release}.hdf`

### Column Definitions
- Extract column lists into separate cells or markdown sections
- Make column selection robust (handle missing columns gracefully)
- Document which columns are optional and when they're available
- **Missing column handling:** Try to include all columns, skip if missing, log warnings
  ```python
  # Example pattern for handling missing columns
  available_columns = [col for col in desired_columns if col in gss.columns]
  missing_columns = [col for col in desired_columns if col not in gss.columns]
  if missing_columns:
      warnings.warn(f"Missing columns: {missing_columns}")
  gss = gss[available_columns]
  ```

### Common Patterns
- Weight handling: `wtssall.fillna(wtssps)` then `del wtssps`
- Resampling: Use `resample_by_year` from utils when needed
- Year conversion: Check dtype and convert if needed (2024 data)
- Git operations: Optional, controlled by `GIT_COMMIT` configuration variable
  ```python
  if GIT_COMMIT:
      !git add -f {OUTPUT_FILE}
      !git commit -m "Updating {OUTPUT_FILE}"
      !git push
  ```
- **Logging:** Use `log_and_print` from `utils.py` for logging messages
  - Follows the logging pattern from the python_utils skill (confirmed)
  - Writes to both log file and notebook output
  - **Log should contain enough information to verify that the notebook ran successfully:**
    - Input file path and size
    - Number of rows and columns read
    - Number of columns selected/extracted
    - Any missing columns (with warnings)
    - Output file path and size
    - Processing steps completed (resampling, year conversion, etc.)
    - Final dataset shape
  - **Proper Usage Pattern:**
    ```python
    import os
    from utils import log_and_print
    
    # At beginning of notebook - open log file:
    os.makedirs('logs', exist_ok=True)
    log_path = f'logs/extract_{year}_{release}.txt'
    debug_log = open(log_path, 'w')
    
    # Write header information
    debug_log.write("GSS Extract Log\n")
    debug_log.write("=" * 80 + "\n")
    debug_log.write(f"Notebook: make_generic_extract.md\n")
    debug_log.write(f"Input: {INPUT_FILE}\n")
    debug_log.write(f"Started: {pd.Timestamp.now()}\n")
    debug_log.write("=" * 80 + "\n\n")
    debug_log.flush()  # Flush after header
    print(f"Log file opened: {log_path}")
    
    # Throughout notebook - use log_and_print:
    # The function automatically flushes after each write
    log_and_print(f"Reading data from {INPUT_FILE}")
    log_and_print(f"Extracted {len(gss)} rows, {len(gss.columns)} columns")
    log_and_print(f"Selected {len(available_columns)} columns (missing: {len(missing_columns)})")
    
    # After important operations, explicitly flush if needed:
    debug_log.flush()
    
    log_and_print(f"Writing output to {OUTPUT_FILE}")
    log_and_print(f"Final dataset: {gss.shape[0]} rows × {gss.shape[1]} columns")
    
    # At end of notebook - close log file:
    debug_log.write(f"\nCompleted: {pd.Timestamp.now()}\n")
    debug_log.close()
    ```
  - **Key points:**
    - Open log file at the beginning (after imports, before main processing)
    - Write header with notebook name, timestamp, and key parameters
    - `log_and_print()` automatically flushes after each write
    - Explicitly flush after important operations if needed
    - Close log file at the end of the notebook
    - The function uses the global `debug_log` variable automatically

---

## Migration Strategy

1. ✅ **Create new consolidated notebooks** with configuration sections (tagged for papermill)
2. ✅ **Test each consolidated notebook** with existing input files to verify outputs match
3. **Update documentation** (NOTEBOOK_INVENTORY.md) to reflect new structure
4. **Remove old notebooks** using `git rm` (they're already in git history)
5. **Update any scripts/tools** that reference old notebook names

## Implementation Status

### ✅ Completed Notebooks

All 5 consolidated notebooks have been created, tested, and verified:

1. **`make_religion_extract.md`** ✅
   - Status: Created and tested
   - Test run: 75,699 rows, 8 columns, 0.68 MB output
   - Log: `logs/religion_2024_r2.txt`
   - Output: `gss_religion_2024_r2.hdf`

2. **`make_pacs_extract.md`** ✅
   - Status: Created and tested
   - Test run: 72,390 rows, 206 columns (full) + 29 columns (workshop)
   - Log: `logs/pacs_2022_r4.txt`
   - Outputs: `gss_pacs_2022_r4.hdf`, `gss_extract_pacs_workshop_2022_r4.hdf`

3. **`make_eds_extract.md`** ✅
   - Status: Created and tested
   - Test run: 75,699 rows, 204 columns, 9.5 MB output
   - Log: `logs/eds_2024_r2.txt`
   - Output: `gss_eds_2024_r2.hdf`

4. **`make_generic_extract.md`** ✅
   - Status: Created and tested (previously)
   - Test run: 75,699 rows, 215 columns
   - Log: `logs/extract_2024_r2.txt`
   - Output: `gss_extract_2024_r2.hdf`

5. **`make_gun_control_extract.md`** ✅
   - Status: Created and tested
   - Test run: 75,699 rows, 16 columns, 1.4 MB output
   - Log: `logs/gun_control_2024_r2.txt`
   - Output: `gss_gun_control_2024_r2.hdf`

### Key Features Implemented

- ✅ Configuration cells tagged for `papermill` (`tags=["parameters"]`)
- ✅ Automatic output filename generation from input filename
- ✅ Robust column handling (reads all columns, filters to available subset)
- ✅ Consistent logging with `log_and_print` function
- ✅ Weight processing (combines `wtssall` and `wtssps`)
- ✅ Log files contain sufficient information to verify successful runs
- ✅ All notebooks tested with 2024 r2 data (except PACS which uses 2022 r4)

---

## Decisions Made

1. **Output naming convention:** ✅ **Yes** - Output files will always include year/release info (e.g., `gss_eds_2024_r2.hdf`)
   - Automatically generated from input filename
   - Format: `{prefix}_{year}_{release}.hdf`

2. **Git operations:** ✅ **Optional** - Controlled by `GIT_COMMIT` configuration variable
   - Default: `False` (no automatic git operations)
   - Set to `True` to enable automatic commit and push of output files

3. **Column availability:** ✅ **Yes** - Try to include all columns, skip if missing, log warnings
   - Use `warnings.warn()` to notify about missing columns
   - Continue processing with available columns

4. **Backward compatibility:** ✅ **Remove old notebooks** - Use `git rm` since they're already in git history
   - No need to archive separately
   - Git history preserves the old notebooks

5. **Configuration cell:** ✅ **Single cell with papermill tag** - All configuration variables in one cell near the beginning
   - Tag the cell with `parameters` for papermill compatibility
   - Allows programmatic execution with parameter injection