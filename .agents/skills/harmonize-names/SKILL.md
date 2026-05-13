---
name: harmonize-names
description: Normalize names in a CSV column with user guidance. Use when asked to fix, harmonize, or edit a column of names or similar values.
---

## Inspect the CSV

```bash
head -n 5 input.csv
```

If the CSV has headers, read the header row and a few data rows to identify the column that most likely contains the target names. If more than one column could apply, ask the user which one to work on. Ignore any obvious trailing notes or extra rows.

Record the column name and index.

## Build a crosswalk

Create a crosswalk CSV named `<column name>_crosswalk.csv` with two columns: `raw` and `mapped`. Store it in an `intermediate/` folder next to the source CSV. If the file already exists, ask the user whether they want to continue. If so, you may continue and overwrite the file.

Use Python's `csv` library to extract the unique values from the target column.

```python
import csv

target_column = "name"  # Replace with the detected column name.

with open("input.csv", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    unique_values = sorted({row[target_column] for row in reader if row.get(target_column)})

print(unique_values)
```

## Review values

Review the extracted values for inconsistencies and misspellings.

1. Clear misspellings of one or two letters can be corrected and should be noted to the user.
2. If multiple values appear to refer to the same entity, propose a single replacement value to the user. For example: `USA`, `United States of America`, and `US of A` might combine to `USA`.

For both kinds of changes, append the raw and corrected values to the crosswalk.

## Apply the crosswalk

Use Python's `csv` library to write and run a script that:

1. Reads the crosswalk file
2. Reads the data csv
3. Replaces values in the target column using the crosswalk
4. Saves the corrected CSV in a `processed/` folder at the same level as the source CSV

```python
import csv
from pathlib import Path

source_csv = Path("input.csv")
crosswalk_csv = Path("intermediate/name_crosswalk.csv")
output_csv = Path("processed/input_corrected.csv")
target_column = "name"  # Replace with the detected column name.

with crosswalk_csv.open(newline="", encoding="utf-8") as f:
    mapping = {row["raw"]: row["mapped"] for row in csv.DictReader(f)}

with source_csv.open(newline="", encoding="utf-8") as f:
    rows = list(csv.DictReader(f))
    fieldnames = rows[0].keys()

for row in rows:
    value = row.get(target_column)
    if value in mapping:
        row[target_column] = mapping[value]

output_csv.parent.mkdir(parents=True, exist_ok=True)
with output_csv.open("w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(rows)
```
