# Cleaning Messy Customer Data

## Project Overview

This project focuses on cleaning and preparing a messy customer orders dataset using **Python and Pandas**. The main objective was to identify data-quality issues, standardize inconsistent values, handle duplicates, and impute missing values using appropriate techniques.

The project also includes basic feature engineering and validation checks to make the dataset more consistent and suitable for further analysis.

## Objectives

- Profile and understand the dataset.
- Inspect the dataset structure, data types, and missing values.
- Standardize column names and categorical values.
- Identify and handle duplicate records.
- Impute missing values using suitable approaches.
- Standardize inconsistent formats in country, payment method, status, and return indicators.
- Convert order dates into a consistent datetime format.
- Create additional date-related features.
- Identify potential inconsistencies between quantity, unit price, and total amount.

## Tools and Libraries

- Python
- Pandas
- NumPy
- ydata-profiling
- Jupyter Notebook

## Data Cleaning Process

### 1. Data Profiling

The dataset was initially profiled using `ydata-profiling` to gain an overview of the data and identify potential quality issues.

I also performed standard checks, including:

- Dataset shape
- Data types
- Missing-value counts
- Previewing the first few records
- Column-name consistency

### 2. Standardizing Column Names

Column names were standardized by:

- Removing leading and trailing whitespace
- Replacing spaces with underscores
- Converting column names to lowercase

### 3. Handling Duplicate Records

Duplicate records were identified using Pandas' `duplicated()` method.

After reviewing the duplicate records, I removed identical duplicate rows because the duplicated line items contained the same information.

```python
messy_customer_df_wo_dupes = messy_customer_df.drop_duplicates().copy()
```

### 4. Handling Missing Values

Missing values were assessed on a column-by-column basis. The imputation method was selected according to the type of data and the context of the column.

#### Category

- Standardized category values by converting them to lowercase and removing extra whitespace.
- Replaced inconsistent values such as `home & garden` with `home and garden`.
- Investigated product-to-category relationships using a mapping table.
- Because some products were associated with multiple categories, missing category values were filled with `unknown` rather than making an unreliable assumption.

```python
messy_customer_df_wo_dupes["category"] = (
    messy_customer_df_wo_dupes["category"].fillna("unknown")
)
```

#### Unit Price

- Removed currency symbols such as `$`, `€`, and `£`.
- Removed commas and extra whitespace.
- Converted the column to the `float64` data type.
- Filled missing unit prices using the median unit price within each category.

```python
messy_customer_df_wo_dupes["unit_price"] = (
    messy_customer_df_wo_dupes["unit_price"]
    .fillna(
        messy_customer_df_wo_dupes
        .groupby("category")["unit_price"]
        .transform("median")
    )
)
```

#### Total Amount

- Reviewed the distribution and skewness of the `total_amount` column.
- Compared the mean and median to determine an appropriate imputation approach.
- Used category-level median imputation for missing total amounts.

```python
messy_customer_df_wo_dupes["total_amount"] = (
    messy_customer_df_wo_dupes["total_amount"]
    .fillna(
        messy_customer_df_wo_dupes
        .groupby("category")["total_amount"]
        .transform("median")
    )
)
```

#### Country

- Removed extra whitespace and periods.
- Converted country values to lowercase.
- Standardized abbreviations such as:
  - `usa` and `us` → `united states`
  - `uk` → `united kingdom`
  - `de` → `germany`
  - `aus` → `australia`
- Filled missing country values with `unknown`.

#### Customer ID

Missing customer IDs were filled with `unknown` to preserve the records without inventing an identifier.

### 5. Standardizing Categorical Columns

The `status` and `payment_method` columns were standardized by:

- Removing extra whitespace
- Converting values to lowercase

### 6. Standardizing the Return Indicator

The `is_returned` column contained multiple representations of Boolean values.

A mapping dictionary was used to convert values such as `Y`, `Yes`, `1`, and `TRUE` to `True`, while values such as `N`, `No`, `0`, and `FALSE` were converted to `False`.

Missing or unmapped values were filled with `False` in the notebook.

### 7. Standardizing Order Dates

The `order_date` column was converted to a datetime format using `pd.to_datetime()` with `errors="coerce"`.

This approach allows invalid date values to be converted into missing datetime values, which can then be reviewed separately.

### 8. Feature Engineering

The following features were created from the order date:

- `order_year`
- `order_month`
- `order_day_of_week`

A new total amount column was also calculated:

```python
messy_customer_df_wo_dupes["new_total_amount"] = (
    messy_customer_df_wo_dupes["quantity"]
    * messy_customer_df_wo_dupes["unit_price"]
)
```

This calculation was introduced to investigate inconsistencies where the quantity was negative but the recorded total amount was positive.

## Key Data Quality Considerations

- Missing values should be assessed based on the meaning of each column.
- If a large percentage of values is missing, the underlying data pipeline should be investigated before automatically imputing the values.
- Duplicate records should be reviewed before removal.
- Median imputation can be useful when values are skewed or when the mean may be influenced by outliers.
- Standardizing categorical values improves consistency and supports reliable analysis.
- Negative quantities require additional business context to determine whether they represent returns, corrections, or erroneous records.

## Project Structure

```text
Cleaning-Messy-Customer-Data/
│
├── Cleaning Messy Customer Data.ipynb
├── messy_customer_orders.csv
├── report.html
└── README.md
```

> File names may be adjusted depending on how the project is organized in the GitHub repository.

## Conclusion

This project helped me practice practical data-cleaning techniques using Pandas, including data profiling, duplicate handling, missing-value imputation, data standardization, datetime conversion, and feature engineering.

The main focus was not only to clean the data, but also to understand the reasoning behind selecting an appropriate approach for each data-quality issue.
