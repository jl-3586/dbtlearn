# Airbnb Analytics dbt Project

## Overview

This dbt (data build tool) project transforms raw Airbnb data into analytics-ready models for business intelligence and data analysis. The project demonstrates data engineering using dbt, including:

- Source data modeling
- Incremental loading
- Data cleansing and transformation
- Data testing and validation
- Custom macros
- Documentation

The project also invovles an analysis of the correlation between full moon dates and Airbnb review sentiment, exploring the hypothesis that full moons might influence guest experiences and reviews.

![Input Schema](assets/input_schema.png)
*Schema of the input data*

## Project Structure

```
├── analyses/              # Ad-hoc analytical queries
├── assets/                # Project assets and images
├── macros/                # Reusable SQL macros
├── models/                # Data transformation models
│   ├── src/               # Source models (raw data with minimal transformations)
│   ├── dim/               # Dimension models (cleansed entities)
│   ├── fct/               # Fact models (metrics and events)
│   └── mart/              # Mart models (business-specific aggregations)
├── seeds/                 # Static data files (CSV)
├── snapshots/             # Slowly changing dimension snapshots
└── tests/                 # Data quality tests
```

## Data Models

### Source Models

- `src_listings`: Raw Airbnb listing data with basic column selection and renaming
- `src_hosts`: Raw host information
- `src_reviews`: Raw review data including review text and sentiment

### Dimension Models

- `dim_listings_cleansed`: Cleansed listing data with proper data types and business rules applied
- `dim_hosts_cleansed`: Cleansed host data
- `dim_listings_w_hosts`: Denormalized dimension combining listings and hosts

### Fact Models

- `fct_reviews`: Review facts with surrogate keys and incremental loading capabilities

### Mart Models

- `mart_fullmoon_reviews`: Combines review data with full moon dates to analyze the correlation between full moons and review sentiment

### Incremental Loading

The fact reviews model uses incremental loading to efficiently process only new data since the last run, with flexible options for date ranges:

```sql
{% if is_incremental() %}
  {% if var("start_date", False) and var("end_date", False) %}
    {{ log('Loading ' ~ this ~ ' incrementally (start_date: ' ~ var("start_date") ~ ', end_date: ' ~ var("end_date") ~ ')', info=True) }}
    AND review_date >= '{{ var("start_date") }}'
    AND review_date < '{{ var("end_date") }}'
  {% else %}
    AND review_date > (select max(review_date) from {{ this }})
    {{ log('Loading ' ~ this ~ ' incrementally (all missing dates)', info=True)}}
  {% endif %}
{% endif %}
```

### Data Quality Tests

The project includes both generic and custom data tests:

- Column-level tests in source definitions (distinct count, regex pattern matching)
- Custom SQL tests (e.g., minimum nights validation)
- Reusable test macros (e.g., no_nulls_in_columns)

### Full Moon Analysis

The project includes a unique analysis exploring the correlation between full moon dates and review sentiment:

```sql
WITH fullmoon_reviews AS (
    SELECT * FROM {{ ref('mart_fullmoon_reviews') }}
)
SELECT
    is_full_moon,
    review_sentiment,
    COUNT(*) as reviews
FROM
    fullmoon_reviews
GROUP BY
    is_full_moon,
    review_sentiment
ORDER BY
    is_full_moon,
    review_sentiment
```

## Packages

The project uses the following dbt packages:

- **dbt-labs/dbt_utils** (v1.3.0): Provides utility macros for common tasks like surrogate key generation
- **calogica/dbt_expectations** (v0.10.0): Implements Great Expectations-style data quality tests

## Documentation

The project includes comprehensive documentation:

- **Column-level documentation**: Detailed descriptions for each column in the models
- **Model-level documentation**: Purpose and usage of each model
- **Project overview**: High-level documentation with schema visualization
- **Exposures**: Documentation for downstream consumers like dashboards