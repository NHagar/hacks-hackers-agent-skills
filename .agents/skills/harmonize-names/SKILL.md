---
name: harmonize-names 
description: Takes names in a file and, with user input, changes them as needed. Use when asked to fix/harmonize/edit a column of names or similar in a CSV. 
---

## Understand the structure of the CSV.

```bash
head -n 5 input.csv
```

If headers are present in the first row, read the headers and the first few lines of data and identify the column most likely to contain the data the user looking for. If multiple columns might apply, ask the user which column to work on. There may be extra rows and you may need to ignore them.

Note the index and name of the column.

## Build a crosswalk file

The crosswalk file will be a CSV (`<column name>_crosswalk.csv`) and it will have two columns: `raw` and `mapped`. It should be located in a folder called `intermediate/` in the same folder as the CSV.

Extract the unique values from the data in the column by using the `csv` library in Python.

## Read and compare

Read through the columns and look for inconsistencies and misspellings.

1. Clear misspellings of one or two letters can be corrected and should be noted to the user.
2. For any name that seems like multiple entries might refer to the same thing (e.g., "USA", "United States of America", "US of A"), propose to the user a version to replace them with.

For both of these kinds of changes, add them to the crosswalk by appending the raw and corrected versions in the relevant columns.

## Fix the spreadsheet

Use the csv library in Python to write and execute a code snippet that:

1. Reads the crosswalk file
2. Reads the data csv
3. Makes the replacements from the crosswalk in the relevant data column
4. Saves the corrected data as a csv in a folder called processed/ at the same level/location as the CSV
