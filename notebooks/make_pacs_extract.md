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

# Extract GSS PACS Dataset

Allen Downey

[MIT License](https://en.wikipedia.org/wiki/MIT_License)

This notebook extracts PACS (Political Alignment Case Study) variables from GSS data.
It produces two outputs: a full PACS extract and a workshop extract.

```python
import pandas as pd
import numpy as np
import warnings
import re
import os
from pathlib import Path

from utils import log_and_print, resample_by_year
```

```python tags=["parameters"]
# Configuration

# Input/Output Configuration
INPUT_FILE = "../data/raw/gss7222_r4.dta.gz"
COMPRESSION_LEVEL = 7  # For full extract
WORKSHOP_COMPRESSION_LEVEL = 6  # For workshop extract

# Processing Options
RESAMPLE_WORKSHOP = True  # Whether to resample workshop extract by year
RANDOM_SEED = 17  # For reproducible resampling

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
    OUTPUT_FILE_FULL = f"../data/interim/gss_pacs_{year}_r{release}.hdf"
    OUTPUT_FILE_WORKSHOP = f"../data/interim/gss_extract_pacs_workshop_{year}_r{release}.hdf"
else:
    raise ValueError(f"Could not parse year and release from {INPUT_FILE}")

# Open log file now that we know year and release
os.makedirs('logs', exist_ok=True)
log_path = f'logs/pacs_{year}_r{release}.txt'
debug_log = open(log_path, 'w')

# Write header information
debug_log.write("GSS PACS Extract Log\n")
debug_log.write("=" * 80 + "\n")
debug_log.write(f"Notebook: make_pacs_extract.md\n")
debug_log.write(f"Input: {INPUT_FILE}\n")
debug_log.write(f"Output (full): {OUTPUT_FILE_FULL}\n")
debug_log.write(f"Output (workshop): {OUTPUT_FILE_WORKSHOP}\n")
debug_log.write(f"Started: {pd.Timestamp.now()}\n")
debug_log.write("=" * 80 + "\n\n")
debug_log.flush()
print(f"Log file opened: {log_path}")

log_and_print(f"Configuration:")
log_and_print(f"  Input file: {INPUT_FILE}")
log_and_print(f"  Output file (full): {OUTPUT_FILE_FULL}")
log_and_print(f"  Output file (workshop): {OUTPUT_FILE_WORKSHOP}")
log_and_print(f"  Resample workshop: {RESAMPLE_WORKSHOP}")
log_and_print(f"  Random seed: {RANDOM_SEED}")
```

## Column Definitions

Full PACS columns (comprehensive set for political alignment analysis).

```python
pacs_columns = sorted([
    "year",
    "id",
    "divorce",
    "sibs",
    "childs",
    "age",
    "educ",
    "degree",
    "sex",
    "race",
    "res16",
    "reg16",
    "adults",
    "income",
    "rincome",
    "region",
    "srcbelt",
    "partyid",
    "pres96",
    "pres00",
    "pres04",
    "pres08",
    "pres12",
    "polviews",
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
    "cappun",
    "gunlaw",
    "grass",
    "relig",
    "fund",
    "attend",
    "reliten",
    "relitenv",
    "relitennv",
    "postlife",
    "pray",
    "popespks",
    "relig16",
    "prayer",
    "bible",
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
    "happy",
    "hapmar",
    "health",
    "life",
    "helpful",
    "fair",
    "trust",
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
    "satjob",
    "class",
    "satfin",
    "finrela",
    "union",
    "fehome",
    "fework",
    "fepres",
    "fepol",
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
    "fear",
    "owngun",
    "pistol",
    "rowngun",
    "hunt",
    "phone",
    "fechld",
    "fehelp",
    "fepresch",
    "fefam",
    "racdif1",
    "racdif2",
    "racdif3",
    "racdif4",
    "god",
    "reborn",
    "savesoul",
    "racwork",
    "fejobaff",
    "discaffm",
    "discaffw",
    "fehire",
    "relpersn",
    "sprtprsn",
    "relexp",
    "spklang",
    "compuse",
    "hrsrelax",
    "trdunion",
    "wkracism",
    "wksexism",
    "wkharsex",
    "databank",
    "goodlife",
    "meovrwrk",
    "miracles",
    "relexper",
    "relactiv",
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
    "hhrace",
    "cohort",
    "ballot",
    "wtssall",
    "wtssps",
    "sexbirth",
    "sexnow",
    "eqwlth",
    "realinc",
    "realrinc",
    "coninc",
    "conrinc",
    "commun",
])

# Workshop extract columns (subset for workshop use)
# Note: wtssps is not included because it gets removed during weight processing
workshop_columns = sorted([
    "year",
    "id",
    "age",
    "cohort",
    "educ",
    "degree",
    "sex",
    "race",
    "rincome",
    "region",
    "srcbelt",
    "class",
    "partyid",
    "polviews",
    "relig",
    "attend",
    "happy",
    "hapmar",
    "health",
    "life",
    "fear",
    "helpful",
    "fair",
    "trust",
    "satjob",
    "satfin",
    "goodlife",
    "ballot",
    "wtssall",
])

log_and_print(f"\nColumn selection:")
log_and_print(f"  Full PACS columns: {len(pacs_columns)}")
log_and_print(f"  Workshop columns: {len(workshop_columns)}")
```

## Read Data

```python
# Read all available columns first, then filter to desired subset
log_and_print(f"\nReading data from {INPUT_FILE}...")
gss_all = pd.read_stata(INPUT_FILE, convert_categoricals=False)

log_and_print(f"  Read {len(gss_all)} rows, {len(gss_all.columns)} total columns available")

# Filter to available PACS columns
available_pacs_columns = [col for col in pacs_columns if col in gss_all.columns]
missing_pacs_columns = [col for col in pacs_columns if col not in gss_all.columns]

if missing_pacs_columns:
    log_and_print(f"  Warning: {len(missing_pacs_columns)} PACS columns not found: {missing_pacs_columns}")

gss = gss_all[available_pacs_columns].copy()
log_and_print(f"  Using {len(gss.columns)} PACS columns")
```

## Data Processing

```python
# Handle weights: combine wtssall and wtssps
log_and_print(f"\nProcessing weights...")
if 'wtssall' in gss.columns and 'wtssps' in gss.columns:
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
# Handle reliten: combine reliten, relitenv, relitennv into single reliten variable
log_and_print(f"\nProcessing reliten variables...")
if 'reliten' in gss.columns and 'relitenv' in gss.columns:
    gss["reliten"] = gss["reliten"].fillna(gss["relitenv"])
    if 'relitennv' in gss.columns:
        gss["reliten"] = gss["reliten"].fillna(gss["relitennv"])
    log_and_print(f"  Combined reliten, relitenv, and relitennv into reliten")
    
    # Remove the split variables
    if 'relitenv' in gss.columns:
        del gss["relitenv"]
    if 'relitennv' in gss.columns:
        del gss["relitennv"]
    log_and_print(f"  Removed relitenv and relitennv")
elif 'reliten' in gss.columns:
    log_and_print(f"  Using reliten only")
else:
    log_and_print(f"  No reliten variables found")
```

```python
# Display data summary
log_and_print(f"\nFull PACS extract summary:")
log_and_print(f"  Shape: {gss.shape[0]} rows × {gss.shape[1]} columns")
log_and_print(f"  Years: {gss['year'].min():.0f} to {gss['year'].max():.0f}")
print(gss.shape)
gss.head()
```

## Write Full PACS Output

```python
# Remove existing output file if it exists
if os.path.exists(OUTPUT_FILE_FULL):
    os.remove(OUTPUT_FILE_FULL)
    log_and_print(f"\nRemoved existing output file: {OUTPUT_FILE_FULL}")

# Write to HDF
log_and_print(f"\nWriting full PACS output to {OUTPUT_FILE_FULL}...")
gss.to_hdf(OUTPUT_FILE_FULL, "gss", complevel=COMPRESSION_LEVEL)

# Check file size
file_size = os.path.getsize(OUTPUT_FILE_FULL) / (1024 * 1024)  # MB
log_and_print(f"  Output file size: {file_size:.2f} MB")
log_and_print(f"  Final dataset: {gss.shape[0]} rows × {gss.shape[1]} columns")

!ls -lh {OUTPUT_FILE_FULL}
```

## Create Workshop Extract

```python
# Filter to workshop columns
log_and_print(f"\nCreating workshop extract...")
available_workshop_columns = [col for col in workshop_columns if col in gss.columns]
missing_workshop_columns = [col for col in workshop_columns if col not in gss.columns]

if missing_workshop_columns:
    log_and_print(f"  Warning: {len(missing_workshop_columns)} workshop columns not found: {missing_workshop_columns}")

gss_workshop = gss[available_workshop_columns].copy()
log_and_print(f"  Workshop extract: {gss_workshop.shape[0]} rows × {gss_workshop.shape[1]} columns")
```

```python
# Resample by year if requested
if RESAMPLE_WORKSHOP:
    log_and_print(f"\nResampling workshop extract by year (seed={RANDOM_SEED})...")
    np.random.seed(RANDOM_SEED)
    gss_workshop = resample_by_year(gss_workshop, "wtssall")
    log_and_print(f"  After resampling: {gss_workshop.shape[0]} rows × {gss_workshop.shape[1]} columns")
else:
    log_and_print(f"\nSkipping resampling (RESAMPLE_WORKSHOP=False)")
```

```python
# Remove existing output file if it exists
if os.path.exists(OUTPUT_FILE_WORKSHOP):
    os.remove(OUTPUT_FILE_WORKSHOP)
    log_and_print(f"\nRemoved existing output file: {OUTPUT_FILE_WORKSHOP}")

# Write to HDF
log_and_print(f"\nWriting workshop output to {OUTPUT_FILE_WORKSHOP}...")
gss_workshop.to_hdf(OUTPUT_FILE_WORKSHOP, "gss", complevel=WORKSHOP_COMPRESSION_LEVEL)

# Check file size
file_size = os.path.getsize(OUTPUT_FILE_WORKSHOP) / (1024 * 1024)  # MB
log_and_print(f"  Output file size: {file_size:.2f} MB")
log_and_print(f"  Final dataset: {gss_workshop.shape[0]} rows × {gss_workshop.shape[1]} columns")

!ls -lh {OUTPUT_FILE_WORKSHOP}
```

## Git Operations (if enabled)

```python
if GIT_COMMIT:
    log_and_print(f"\nGit operations (GIT_COMMIT=True)...")
    !git add -f {OUTPUT_FILE_FULL}
    !git add -f {OUTPUT_FILE_WORKSHOP}
    !git commit -m "Updating GSS PACS extracts: {OUTPUT_FILE_FULL} and {OUTPUT_FILE_WORKSHOP}"
    !git push
    log_and_print(f"  Git commit and push complete")
else:
    log_and_print(f"\nGit operations disabled (GIT_COMMIT=False)")

log_and_print(f"\n✓ Extract complete:")
log_and_print(f"  Full PACS: {OUTPUT_FILE_FULL}")
log_and_print(f"  Workshop: {OUTPUT_FILE_WORKSHOP}")

# Close log file
debug_log.write(f"\nCompleted: {pd.Timestamp.now()}\n")
debug_log.close()
print(f"Log file closed: {log_path}")
```

