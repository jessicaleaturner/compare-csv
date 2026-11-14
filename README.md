# Compare CSVs

A small Python command-line tool that compares two CSV files and reports the differences between them. Each differing row is written to an Excel spreadsheet with its index, the column that differs, and the value from each file. A step-by-step log is written alongside it.

> **Status:** written for Python 3.8 and the pandas of that time. It fails on current pandas and NumPy. See [Known issues](#known-issues).

## How it works

1. Reads both CSVs with pandas (as `cp1252`) and fills empty cells with `-`, because empty cells never compare as equal.
2. Compares the column headers. Columns that only appear in one file are dropped from that file and noted in the log.
3. Splits both dataframes into equal-sized chunks, so large files print progress while they run.
4. Compares the chunks row by row and collects every row that differs.
5. Writes the results to `output/results.xlsx` and the log to `output/log.txt`.

## Requirements

- Python 3.8
- pandas
- NumPy
- OpenPyXL

```bash
python3 -m pip install pandas numpy openpyxl
```

## Usage

```bash
python3 compare_csv.py ./data/actual.csv ./data/expected.csv
```

Both arguments are paths to CSV files. The `output/` folder must exist (it's included in the repo).

### Output

`output/results.xlsx` has one row per differing row, with these columns:

| Column | Meaning |
| --- | --- |
| `Affected Index` | The first column of the differing row, used as its index |
| `Affected Column` | The column that differs |
| `<first file>_data` | The value in the first file you passed |
| `<second file>_data` | The value in the second file you passed |

`output/log.txt` records the header checks, chunking and each differing pair of rows.

### Sample data

`data/expected.csv` and `data/actual.csv` are small four-row files for trying the tool out. The values differ between them in several places.

## Tests

```bash
python3 test_compare_csv.py
```

## Project layout

```
compare_csv.py          Main program: loading, header matching, chunked comparison, output
utilities.py            Log writer and the helper used to choose a chunk count
test_compare_csv.py     Test that runs the comparison on the sample data
data/                   Sample CSV files
output/                 Results spreadsheet and log
```

## Known issues

- **Doesn't run on current pandas and NumPy.** I ran it with pandas 3.0.5 and NumPy 2.5.3 and it stops with an `AttributeError` in `write_chunk_differences_to_df`. It also uses `DataFrame.append`, which was removed in pandas 2.0. `requirements.txt` is empty, so the versions it was written against aren't recorded.
- **Only one differing column is reported per row.** If several columns differ in the same row, only the last one appears in the results.
- **The test doesn't assert anything.** `test_compare_csv.py` runs the comparison and passes as long as it doesn't raise.
- **Argument order differs between the README and the test.** The usage example passes `actual` then `expected`, and the test does the reverse. The order only affects which file's values appear in which output column.
- **Chunk count is based on a prime divisor of the row count.** Files with a prime number of rows are split into one-row chunks, which is slow for large files.
