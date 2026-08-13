# CarDekho ETL Pipeline

## Overview

This project implements an ETL pipeline for processing and loading CarDekho used-car data into PostgreSQL.

The pipeline automatically detects CSV and Excel files from a configured folder, performs data cleaning and standardization using Python and Pandas, handles missing values using business rules and group-based transformations, generates SHA-256 row hashes for duplicate detection, and incrementally loads new records into PostgreSQL.

The pipeline also maintains an import log to track each execution and the number of records processed.

## Technology Stack

- Python
- Pandas
- NumPy
- PostgreSQL
- SQLAlchemy
- psycopg2
- OpenPyXL
- Jupyter Notebook

## ETL Architecture

```text
Source Files
    |
    v
CSV / Excel File Detection
    |
    v
Data Ingestion
    |
    v
Data Cleaning & Standardization
    |
    v
Missing Value Handling
    |
    v
Business Rule Validation
    |
    v
SHA-256 Hash Generation
    |
    v
Duplicate Detection
    |
    v
Incremental Loading
    |
    v
PostgreSQL
    |
    v
Import Logging
```

## Database Design

The pipeline creates a PostgreSQL schema named:

```text
cardekho
```

Two tables are created:

```text
cardekho.car
cardekho.import_log
```

### car

The main table stores the cleaned vehicle data.

Important columns include:

```text
brand
model
vehicle_age
km_driven
seller_type
fuel_type
transmission_type
mileage
engine
max_power
seats
selling_price
actual_price
fixed_profit
row_hash
source_file
ingested_at
```

### import_log

The logging table records:

```text
run_at
source_folder
files_found
files_loaded
rows_new
rows_skipped
rows_inserted
status
```

## 1. Automated File Ingestion

The pipeline scans a configured folder using Python's `glob` and `os` libraries.

It supports:

```text
.csv
.xlsx
.xls
```

For every successfully loaded file, the pipeline adds:

```text
source_file
ingested_at
```

The datasets are then combined into a single Pandas DataFrame for processing.

In the recorded notebook execution, one CSV file containing 1,052 rows was detected and loaded.

## 2. Data Cleaning

The pipeline applies several transformations to improve data consistency.

### Text Standardization

The pipeline cleans fields such as:

- Brand
- Model
- Seller type
- Fuel type
- Transmission type

Regular expressions are used to remove unwanted characters, followed by trimming and capitalization.

### Category Standardization

Custom functions standardize inconsistent categorical values.

For example, seller types are mapped to:

```text
Individual
Dealer
```

Fuel types are standardized to values such as:

```text
Petrol
Diesel
```

Transmission types are standardized to:

```text
Manual
Automatic
```

### Brand Correction

The pipeline also uses model-based mappings to correct or standardize vehicle brands.

For example, models such as Alto, Swift and Baleno are mapped to Maruti, while models such as i20, Creta and Verna are mapped to Hyundai. Similar mappings are applied for other brands present in the dataset.

## 3. Data Type Conversion

Numerical columns are cleaned and converted into appropriate data types.

Examples include:

```text
vehicle_age     → integer
mileage         → float
selling_price   → integer
actual_price    → integer
fixed_profit    → integer
engine          → integer
max_power       → float
```

## 4. Missing Value Handling

Missing values are handled for several numerical fields, including:

```text
engine
max_power
selling_price
actual_price
fixed_profit
```

The pipeline initially replaces missing numerical values with zero where appropriate before applying additional business rules.

## 5. Business Rule-Based Data Recovery

The pipeline uses the relationship between selling price, actual price and fixed profit to recover missing values.

If selling price is missing:

```text
selling_price = actual_price + fixed_profit
```

If actual price is missing:

```text
actual_price = selling_price - fixed_profit
```

If fixed profit is missing:

```text
fixed_profit = selling_price - actual_price
```

## 6. Group-Based Imputation

The pipeline uses related vehicle attributes to fill certain missing values.

For example:

```text
model + engine
        |
        v
   max_power
```

and:

```text
model + max_power
        |
        v
      engine
```

The implementation uses Pandas `groupby()` and `transform('first')` to propagate known values within matching groups.

## 7. Duplicate Detection

The pipeline generates a SHA-256 hash using important vehicle attributes.

The hash is generated from fields including:

```text
brand
model
vehicle_age
km_driven
seller_type
fuel_type
transmission_type
mileage
engine
max_power
selling_price
actual_price
fixed_profit
```

The resulting hash is stored in:

```text
row_hash
```

This is used to identify duplicates within the current batch and compare incoming records against records already present in PostgreSQL.

## 8. Incremental Database Loading

After duplicate detection, the pipeline identifies records that are not already present in the PostgreSQL database.

Only these records are prepared for insertion into:

```text
cardekho.car
```

The data is loaded using Pandas `to_sql()` through SQLAlchemy.

This prevents previously processed records from being inserted repeatedly.

## 9. Pipeline Logging

After processing, the pipeline writes execution information to:

```text
cardekho.import_log
```

The log records:

- Source folder
- Number of files detected
- Number of files loaded
- Number of new records
- Number of skipped records
- Number of inserted records
- Pipeline status

## Example Execution

The notebook contains an execution where:

```text
Files Detected: 1
Loaded: Cardekho_Unclean_Dataset.csv With 1052 Rows
Total Rows Loaded 1052

Batch duplicates removed : 0
Already in DB (skipped) : 1035
New rows to insert : 0
```

This demonstrates the incremental loading logic: records already available in the database are skipped instead of being inserted again.

## Setup

### 1. Install Python Dependencies

```bash
pip install pandas numpy sqlalchemy psycopg2-binary openpyxl jupyter
```

### 2. Create PostgreSQL Database

Create a PostgreSQL database named:

```text
CarDekho
```

The notebook then creates the required `cardekho` schema and tables.

### 3. Configure the Input Directory

Update the input folder path in the notebook according to your local environment.

### 4. Configure Database Credentials

Update the PostgreSQL connection configuration.

For a production implementation, database credentials should be stored using environment variables rather than being hard-coded in the notebook.

### 5. Run the Notebook

```bash
jupyter notebook
```

Open:

```text
CarDekho Pipeline.ipynb
```

and execute the cells sequentially.

## Key Concepts Demonstrated

This project demonstrates practical implementation of:

- ETL pipeline development
- Automated file ingestion
- Data cleaning
- Data standardization
- Data validation
- Missing-value handling
- Business-rule-based transformations
- Group-based imputation
- SHA-256 hashing
- Duplicate detection
- Incremental data loading
- PostgreSQL database loading
- SQLAlchemy database connectivity
- ETL execution logging
- Source-file tracking
- Data ingestion timestamps

## Future Improvements

Potential improvements for a production implementation include:

- Move database credentials to environment variables.
- Add formal data-quality validation rules.
- Add exception handling and structured error logging.
- Add unit tests for transformation functions.
- Separate configuration from transformation logic.
- Add automated pipeline scheduling.
- Add failure notifications.
- Add CI/CD using GitHub Actions.
- Convert the notebook into reusable Python modules.
- Add a downstream Power BI reporting layer.

## Author

Abhay Singh

Data Analyst | Python | SQL | Power BI | PostgreSQL | ETL
