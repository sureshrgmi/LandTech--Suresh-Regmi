Let's start with understanding the current table structure

The table data appears to be a list of contracts(assuming of one of the LandTech products) bought by different accounts with the contract dates (start date, end date, cancellation date if applicable), contract types, industry of the contract stakeholders, value of those contracts and the annual recurring revenue.




1. Create a table for the raw data with the name dim_contracts_raw and load the data
```
CREATE TABLE IF NOT EXISTS dim_contracts_raw
(
    id character varying(32) ,
    arr real,
    value real,
    account_id character varying(32) ,
    source character varying(16) ,
    closed_date date,
    start_date date,
    end_date date,
    cancellation_date date,
    original_end_date date,
    number_of_licenses integer,
    type character varying(16),
    industry character varying(32)
)

```

2. Add a new column in the table to store the new type field based on the logic provided

```	
ALTER TABLE dim_contracts_raw
ADD COLUMN new_type VARCHAR(25);
```

3. Now, update the field with a new type based on the logic provided

```

UPDATE dim_contracts_raw
SET new_type = (
    CASE
        WHEN NOT EXISTS (
            SELECT 1
            FROM dim_contracts_raw
            WHERE account_id = dim_contracts_raw.account_id
            AND end_date > dim_contracts_raw.start_date
        )
        THEN 'New Business'
        WHEN end_date <= (closed_date + INTERVAL '90 days')
        THEN 'Renewal 90d Grace'
        WHEN end_date > (closed_date + INTERVAL '90 days')
        THEN 'Re-Engagement'
        ELSE 'Mid-Cycle Expansion'
    END
);

```

4. The next step is to design our fact and dimension tables




```
CREATE TABLE date_dim (
  date DATE NOT NULL PRIMARY KEY,
  year INT NOT NULL,
  quarter INT NOT NULL,
  month INT NOT NULL,
  weekday VARCHAR(10) NOT NULL,
  is_weekend BOOLEAN NOT NULL
);

```

Inserting data into the date dimension table

```
INSERT INTO date_dim (date_col, year, quarter, month, weekday, is_weekend)
SELECT
  date_col,
  EXTRACT(YEAR FROM date_col) AS year,
  EXTRACT(QUARTER FROM date_col) AS quarter,
  EXTRACT(MONTH FROM date_col) AS month,
  to_char(date_col, 'DAY') AS weekday,
  CASE WHEN TO_CHAR(date_col, 'DAY') IN ('SATURDAY', 'SUNDAY') THEN TRUE ELSE FALSE END AS is_weekend
FROM (
  SELECT
    generate_series('2010-01-01', '2030-12-31', INTERVAL '1 DAY') AS date_col
) AS dates;


```


Creating the table for 
