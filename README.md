![logo_ironhack_blue 7](https://user-images.githubusercontent.com/23629340/40541063-a07a0a8a-601a-11e8-91b5-2f13e4e6b441.png)

# Lab | Data Formats and Execution Engines

## Overview

This lab gives you hands-on experience with the modern local analytics stack: Parquet as a columnar file format, Apache Arrow as an in-memory standard, pandas for eager exploration, and DuckDB for optimized analytical queries. You will create Parquet files, inspect their metadata, observe predicate pushdown in action, compare eager vs lazy execution, and run identical analyses in both pandas and DuckDB. The emphasis is on understanding *why* things are fast or slow, not just getting the right answer.

## Learning Goals

By the end of this lab, you should be able to:

- Create, inspect, and query Parquet files with awareness of their internal structure
- Use Apache Arrow to move data between formats efficiently
- Observe and explain predicate pushdown and column pruning in practice
- Compare pandas (eager) and DuckDB (lazy/optimized) execution on the same queries
- Choose the right tool for different stages of an analytical workflow

## Setup and Context

You will work in a Jupyter Notebook. All data is generated in memory or saved to local Parquet files. No cloud services or external databases are required.

## Requirements

- Fork this repository to your own GitHub account.
- Clone your fork to your machine.
- Install the required packages:

```bash
pip install pandas numpy pyarrow duckdb
```

## Getting Started

Create a new notebook and name it `m2-03-data-formats-execution-engines-lab.ipynb`. Complete all tasks in this notebook. Before you submit, restart your kernel and run the notebook top to bottom.

## Tasks

### Task 1: Parquet Internals and Metadata Inspection

**Step 1:** Create a dataset with 500,000 rows and at least 8 columns: a mix of integers, floats, strings, booleans, and timestamps. Use `np.random.seed(42)` for reproducibility.

**Step 2:** Save the dataset as a Parquet file. Then use `pyarrow.parquet.ParquetFile` to inspect:
- The number of row groups
- The number of rows and columns
- The schema (column names and types)
- For each column in the first row group: physical type, compressed size, and any available statistics (min, max, null count)

**Step 3:** Save the same dataset as CSV. Compare file sizes. Print both sizes in KB and the compression ratio.

**Step 4:** Write a markdown cell explaining what information the Parquet metadata provides that CSV does not, and why that matters for query performance.

### Task 2: Column Projection and Selective Reads

**Step 1:** Read the full Parquet file from Task 1 and time it.

**Step 2:** Read only 2 columns from the same Parquet file and time it.

**Step 3:** Read only 2 columns from the CSV file and time it (you will need to read the entire CSV and select columns after, which is what pandas does).

**Step 4:** Write a markdown cell explaining why Parquet selective reads are faster. Connect your explanation to the column-chunk layout you observed in Task 1.

### Task 3: Predicate Pushdown in Practice

**Step 1:** Using PyArrow, read the Parquet file with a filter (e.g., `age > 50`). Time the read.

**Step 2:** Read the full Parquet file without a filter and apply the same filter in pandas after loading. Time this approach.

**Step 3:** Compare the number of rows returned and the execution times. Write a markdown cell explaining what predicate pushdown does and why it is faster.

**Step 4:** Run the same filtered query using DuckDB's SQL interface directly on the Parquet file:

```python
import duckdb
result = duckdb.sql("SELECT * FROM 'your_file.parquet' WHERE age > 50").df()
```

Time it and compare with both PyArrow and pandas approaches.

### Task 4: pandas vs DuckDB — Identical Queries

Run the same five analytical queries in both pandas and DuckDB on the dataset from Task 1. For each query, record the code and the execution time.

**Query 1:** Count records per city.

**Query 2:** Average score by city, ordered by average score descending.

**Query 3:** For each city, find the percentage of active users whose score is above 75.

**Query 4:** Find the top 10 users by score for each city (window function / rank).

**Query 5:** Compute a running total of scores ordered by user_id, partitioned by city.

For DuckDB, write the query in SQL using `duckdb.sql()`. For pandas, use native pandas operations (groupby, transform, rank, etc.).

**Step 5:** Write a markdown cell comparing:
- Which tool was easier to express each query in?
- Which was faster?
- For which queries did the difference matter most?

### Task 5: Arrow as the Bridge

**Step 1:** Create a pandas DataFrame and convert it to an Arrow Table using `pyarrow.Table.from_pandas()`.

**Step 2:** Inspect the Arrow Table schema and compare it to the pandas dtypes. Note any differences.

**Step 3:** Write the Arrow Table to Parquet using `pyarrow.parquet.write_table()`. Read it back into a new Arrow Table.

**Step 4:** Convert the Arrow Table back to a pandas DataFrame using `.to_pandas()`. Verify the data matches the original.

**Step 5:** Demonstrate the pandas `dtype_backend="pyarrow"` option by reading the Parquet file with Arrow-backed dtypes. Print the dtypes and compare with traditional pandas dtypes.

**Step 6:** Write a markdown cell explaining the role of Arrow in the modern analytics stack. How does it connect Parquet (disk) to pandas (analysis) to DuckDB (SQL)?

## Submission

### What to submit

Submit the following file:

- A notebook file `m2-03-data-formats-execution-engines-lab.ipynb` containing all five tasks with code and markdown explanations

### Definition of done (checklist)

Before you submit, make sure:

- [ ] The notebook runs **top to bottom** without errors after a kernel restart.
- [ ] Task 1 includes Parquet metadata inspection and CSV size comparison.
- [ ] Task 2 includes timed selective reads from both Parquet and CSV.
- [ ] Task 3 demonstrates predicate pushdown with timing comparisons across PyArrow, pandas, and DuckDB.
- [ ] Task 4 includes all five queries in both pandas and DuckDB with timing comparisons.
- [ ] Task 5 demonstrates Arrow conversions and explains Arrow's role in the analytics stack.
- [ ] Markdown cells provide clear explanations connecting observations to concepts.

### How to submit (Git workflow)

When you are done, make sure all changes are saved, then run:

```bash
git add .
git commit -m "Solved m2-03 lab"
git push -u origin HEAD
```

- Make a pull request.
- Paste the link to your pull request in the Student Portal.

## Evaluation Criteria

Your work will be evaluated on technical correctness, performance analysis, and conceptual understanding. Each query should produce correct results in both pandas and DuckDB. Timing comparisons should use consistent methodology. Your markdown explanations should connect observed performance differences to the underlying format and execution engine concepts from the lesson, demonstrating that you understand *why* things are fast or slow, not just *that* they are.
