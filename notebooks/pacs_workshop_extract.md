---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.19.1
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

# Extract GSS Data for the PACS Workshop

Allen Downey

[MIT License](https://en.wikipedia.org/wiki/MIT_License)

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

from utils import resample_by_year
```

Download the Stata data from https://gss.norc.org/get-the-data/stata

Move to data/raw and unzip

```python
!ls ../data/raw/
```

```python
filename = "../data/raw/gss7222_r4.dta.gz"
```

```python
columns = sorted(
    [
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
        "wtssps",
    ]
)
```

```python
gss = pd.read_stata(filename, columns=columns, convert_categoricals=False)
```

```python
# Weights are different in the most recent iterations
# We'll combine them into a single column, but we will never
# mix weights from differnet iterations.
gss["wtssall"].fillna(gss["wtssps"], inplace=True)
gss["wtssall"].describe()
```

```python
del gss["wtssps"]
```

```python
print(gss.shape)
gss.head()
```

```python
from utils import resample_by_year

np.random.seed(17)
sample = resample_by_year(gss, "wtssall")
```

```python
!rm -f ../data/interim/gss_extract_pacs_workshop.hdf
```

```python
sample.to_hdf("../data/interim/gss_extract_pacs_workshop.hdf", "gss", complevel=6)
```

```python
!ls -lh ../data/interim/gss_extract_pacs_workshop.hdf
```

```python
!git add -f ../data/interim/gss_extract_pacs_workshop.hdf
!git commit -m "Updating extract for the PACS workshop"
!git push
```

```python

```
