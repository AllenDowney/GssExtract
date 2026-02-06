---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.19.1
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

# Extract GSS Generic Dataset

Allen Downey

[MIT License](https://en.wikipedia.org/wiki/MIT_License)

This notebook extracts a comprehensive superset of GSS variables suitable for general-purpose analysis.

```python
import pandas as pd
import numpy as np
import warnings
import re
import os
from pathlib import Path

from utils import resample_by_year, log_and_print
```

```python tags=["parameters"]
# Configuration

# Input/Output Configuration
INPUT_FILE = "../data/raw/gss7224_r2.dta.gz"
COMPRESSION_LEVEL = 6

# Processing Options
RESAMPLE = False  # Whether to resample by year
RANDOM_SEED = 17  # For reproducible resampling
CONVERT_YEAR_TO_FLOAT = True  # Convert year to float64 (needed for 2024 data)

# Git Operations (optional)
GIT_COMMIT = False  # Set to True to automatically commit and push output files

# Extract year and release from input filename
# Pattern: gss72{YY}_r{release}.dta(.gz) where YY is last 2 digits of year
# Convert YY to full 4-digit year (e.g., 24 -> 2024, 22 -> 2022, 21 -> 2021)
input_name = Path(INPUT_FILE).stem.replace('.dta', '')
match = re.match(r'gss72(\d{2})_r(\w+)', input_name)
if match:
    year_2digit = match.group(1)
    release = match.group(2)
    # Convert 2-digit year to 4-digit (assumes 2000s)
    year = f"20{year_2digit}"
    OUTPUT_FILE = f"../data/interim/gss_extract_{year}_r{release}.hdf"
else:
    raise ValueError(f"Could not parse year and release from {INPUT_FILE}")
```

```python
# Open log file now that we know year and release
os.makedirs('logs', exist_ok=True)
log_path = f'logs/extract_{year}_r{release}.txt'
debug_log = open(log_path, 'w')

# Write header information
debug_log.write("GSS Extract Log\n")
debug_log.write("=" * 80 + "\n")
debug_log.write(f"Notebook: make_generic_extract.md\n")
debug_log.write(f"Input: {INPUT_FILE}\n")
debug_log.write(f"Output: {OUTPUT_FILE}\n")
debug_log.write(f"Started: {pd.Timestamp.now()}\n")
debug_log.write("=" * 80 + "\n\n")
debug_log.flush()
print(f"Log file opened: {log_path}")

log_and_print(f"Configuration:")
log_and_print(f"  Input file: {INPUT_FILE}")
log_and_print(f"  Output file: {OUTPUT_FILE}")
log_and_print(f"  Resample: {RESAMPLE}")
log_and_print(f"  Random seed: {RANDOM_SEED}")
```

## Column Definitions

Base set of columns plus optional additions based on data availability.

```python
# Superset of all columns from all extract notebooks
# This includes columns from: religion, PACS, EDS, gun control, and previous generic extracts
base_columns = sorted([
    # Demographics (from all notebooks)
    "year",
    "id",
    "age",
    "cohort",
    "educ",
    "degree",
    "sex",
    "race",
    "hispanic",
    "res16",
    "reg16",
    "adults",
    "rincome",
    "income",
    "region",
    "srcbelt",
    "divorce",
    "sibs",
    "childs",
    # Political variables
    "partyid",
    "polviews",
    "pres96",
    "pres00",
    "pres04",
    "pres08",
    "pres12",
    # Religion variables
    "relig",
    "relig16",
    "fund",
    "attend",
    "reliten",
    "relitenv",
    "relitennv",
    "god",
    "bible",
    "postlife",
    "pray",
    "popespks",
    "prayer",
    "reborn",
    "savesoul",
    "relpersn",
    "sprtprsn",
    "relexp",
    "miracles",
    "relexper",
    "relactiv",
    # Attitudes and values
    "happy",
    "hapmar",
    "health",
    "life",
    "fear",
    "helpful",
    "fair",
    "trust",
    "goodlife",
    # Confidence in institutions
    "confinan",
    "conbus",
    "conclerg",
    "coneduc",
    "confed",
    "conlabor",
    "conpress",
    "conmedic",
    "contv",
    "conjudge",
    "consci",
    "conlegis",
    "conarmy",
    # Satisfaction
    "satjob",
    "satfin",
    "class",
    "finrela",
    # National spending priorities
    "natspac",
    "natenvir",
    "natheal",
    "natcity",
    "natcrime",
    "natdrug",
    "nateduc",
    "natrace",
    "natarms",
    "nataid",
    "natfare",
    "natroad",
    "natsoc",
    "natmass",
    "natpark",
    "natchld",
    "natsci",
    "natenrgy",
    # Free speech variables
    "spkath",
    "colath",
    "libath",
    "spksoc",
    "colsoc",
    "libsoc",
    "spkrac",
    "colrac",
    "librac",
    "spkcom",
    "colcom",
    "libcom",
    "spkmil",
    "colmil",
    "libmil",
    "spkhomo",
    "colhomo",
    "libhomo",
    "spkmslm",
    "colmslm",
    "libmslm",
    # Gun control (from gun control extract)
    "gunlaw",
    "owngun",
    "gun",
    "pistol",
    "rowngun",
    "hunt",
    "cappun",
    # Social issues
    "grass",
    "abdefect",
    "abnomore",
    "abhlth",
    "abpoor",
    "abrape",
    "absingle",
    "abany",
    "chldidel",
    "sexeduc",
    "divlaw",
    "premarsx",
    "teensex",
    "xmarsex",
    "homosex",
    "marhomo",
    "marhomo1",
    "pornlaw",
    "spanking",
    "letdie1",
    "polhitok",
    "polabuse",
    "polmurdr",
    "polescap",
    "polattak",
    # Race relations
    "racmar",
    "racdin",
    "racpush",
    "racseg",
    "racopen",
    "raclive",
    "racclos",
    "racdis",
    "racinteg",
    "rachome",
    "racschol",
    "racfew",
    "rachaf",
    "racmost",
    "racpres",
    "racchurh",
    "affrmact",
    "racwork",
    "racdif1",
    "racdif2",
    "racdif3",
    "racdif4",
    # Gender roles
    "fehome",
    "fework",
    "fepres",
    "fepol",
    "fechld",
    "fehelp",
    "fepresch",
    "fefam",
    "fejobaff",
    "discaffm",
    "discaffw",
    "fehire",
    # Work and economy
    "union",
    "trdunion",
    "wkracism",
    "wksexism",
    "wkharsex",
    "databank",
    "meovrwrk",
    # Income and wealth
    "eqwlth",
    "realinc",
    "realrinc",
    "coninc",
    "conrinc",
    # Other
    "commun",
    "phone",
    "compuse",
    "hrsrelax",
    "spklang",
    # Sexual behavior (from PACS/EDS)
    "matesex",
    "frndsex",
    "acqntsex",
    "pikupsex",
    "paidsex",
    "othersex",
    "sexsex",
    "sexfreq",
    "sexsex5",
    "sexornt",
    # Demographics continued
    "sexbirth",
    "sexnow",
    "hhrace",
    "ballot",
    "wtssall",
    "wtssps",
    # Climate variables (from generic extract)
    "clmtcaus",
    "clmtwrld",
    "clmtusa",
])

# Optional columns (included if available in the data)
optional_columns = sorted([
    "pres16",
    "pres20",
    "sexbirth1",
    "sexnow1",
])

# Combine all desired columns and remove duplicates
desired_columns = sorted(list(set(base_columns + optional_columns)))

log_and_print(f"\nColumn selection:")
log_and_print(f"  Base columns: {len(base_columns)}")
log_and_print(f"  Optional columns: {len(optional_columns)}")
log_and_print(f"  Total desired (after removing duplicates): {len(desired_columns)}")
```

## Read Data

```python
# Read the Stata file
log_and_print(f"\nReading data from {INPUT_FILE}...")
# Read all columns first, then filter to desired ones that exist
gss = pd.read_stata(INPUT_FILE, convert_categoricals=False)

# Check which columns are available
available_columns = [col for col in desired_columns if col in gss.columns]
missing_columns = [col for col in desired_columns if col not in gss.columns]

if missing_columns:
    warnings.warn(f"Missing columns: {missing_columns}")
    log_and_print(f"  Warning: Missing {len(missing_columns)} columns: {missing_columns}")

log_and_print(f"  Read {len(gss)} rows, {len(gss.columns)} total columns in file")
log_and_print(f"  Selected {len(available_columns)} columns from desired set")

# Select only available columns
gss = gss[available_columns]
```

## Data Processing

```python
# Convert year to float64 if needed (for 2024 data)
if CONVERT_YEAR_TO_FLOAT or gss['year'].dtype != 'float64':
    if gss['year'].dtype != 'float64':
        log_and_print(f"\nConverting year from {gss['year'].dtype} to float64...")
        gss['year'] = gss['year'].astype(np.float64)
        log_and_print(f"  Year conversion complete")
    elif CONVERT_YEAR_TO_FLOAT:
        log_and_print(f"\nConverting year to float64 (explicitly requested)...")
        gss['year'] = gss['year'].astype(np.float64)
        log_and_print(f"  Year conversion complete")
```

```python
# Handle weights: combine wtssall and wtssps
log_and_print(f"\nProcessing weights...")
if 'wtssall' in gss.columns and 'wtssps' in gss.columns:
    # weights are different in 2021 and 2022 so mixing them in might seem like a bad idea,
    # but we only use them for resampling within one year of the survey,
    # so I think it's ok
    gss["wtssall"].fillna(gss["wtssps"], inplace=True)
    del gss["wtssps"]
    log_and_print(f"  Combined wtssall and wtssps, removed wtssps")
elif 'wtssall' in gss.columns:
    log_and_print(f"  Using wtssall only")
elif 'wtssps' in gss.columns:
    gss["wtssall"] = gss["wtssps"]
    del gss["wtssps"]
    log_and_print(f"  Renamed wtssps to wtssall")
else:
    warnings.warn("No weight columns found")
    log_and_print(f"  Warning: No weight columns found")
```

```python
# Display data summary
log_and_print(f"\nData summary:")
log_and_print(f"  Shape: {gss.shape[0]} rows × {gss.shape[1]} columns")
log_and_print(f"  Years: {gss['year'].min():.0f} to {gss['year'].max():.0f}")
print(gss.shape)
gss.head()
```

## Resampling (if enabled)

```python
if RESAMPLE:
    log_and_print(f"\nResampling by year (seed={RANDOM_SEED})...")
    np.random.seed(RANDOM_SEED)
    gss = resample_by_year(gss, "wtssall")
    log_and_print(f"  Resampling complete: {gss.shape[0]} rows")
else:
    log_and_print(f"\nResampling disabled (RESAMPLE=False)")
```

## Write Output

```python
# Remove existing output file if it exists
import os
if os.path.exists(OUTPUT_FILE):
    os.remove(OUTPUT_FILE)
    log_and_print(f"\nRemoved existing output file: {OUTPUT_FILE}")

# Write to HDF
log_and_print(f"\nWriting output to {OUTPUT_FILE}...")
gss.to_hdf(OUTPUT_FILE, "gss", complevel=COMPRESSION_LEVEL)

# Check file size
file_size = os.path.getsize(OUTPUT_FILE) / (1024 * 1024)  # MB
log_and_print(f"  Output file size: {file_size:.2f} MB")
log_and_print(f"  Final dataset: {gss.shape[0]} rows × {gss.shape[1]} columns")

!ls -lh {OUTPUT_FILE}
```

## Git Operations (if enabled)

```python
if GIT_COMMIT:
    log_and_print(f"\nGit operations (GIT_COMMIT=True)...")
    !git add -f {OUTPUT_FILE}
    !git commit -m "Updating GSS extract: {OUTPUT_FILE}"
    !git push
    log_and_print(f"  Git commit and push complete")
else:
    log_and_print(f"\nGit operations disabled (GIT_COMMIT=False)")

log_and_print(f"\n✓ Extract complete: {OUTPUT_FILE}")

# Close log file
debug_log.write(f"\nCompleted: {pd.Timestamp.now()}\n")
debug_log.close()
print(f"Log file closed: {log_path}")
```

