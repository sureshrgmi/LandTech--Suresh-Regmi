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

4. The next step is to design our fact and dimension tables (Based on Tables_Design.jpg)

```
CREATE TABLE date_dim (
  date DATE NOT NULL PRIMARY KEY,
  year INT NOT NULL,
  quarter INT NOT NULL,
  month INT NOT NULL,
  weekday VARCHAR(10) NOT NULL,
  is_weekend BOOLEAN NOT NULL
);

CREATE TABLE IF NOT EXISTS account_dim
(
    account_id character varying(25) NOT NULL,
    account_name character varying(100) ,
    CONSTRAINT account_dim_pkey PRIMARY KEY (account_id)
);

CREATE TABLE IF NOT EXISTS industry_dim
(
    industry_id character varying(50)NOT NULL,
    industry_name character varying(50),
    CONSTRAINT industry_dim_pkey PRIMARY KEY (industry_id)
);

CREATE TABLE IF NOT EXISTS type_dim
(
    type_id integer NOT NULL,
    type_name character varying(25),
    CONSTRAINT type_dim_pkey PRIMARY KEY (type_id)
);


CREATE TABLE IF NOT EXISTS contracts_fact
(
    f_date date NOT NULL,
    account_id character varying(25),
    industry_id character varying(25) ,
    type_id integer,
    arr double precision,
    value double precision,
    no_of_license integer,
    CONSTRAINT contracts_fact_pkey PRIMARY KEY (f_date),
    CONSTRAINT account_id_fk FOREIGN KEY (account_id)
        REFERENCES public.account_dim (account_id),
    CONSTRAINT f_date_fk FOREIGN KEY (f_date)
        REFERENCES public.date_dim (date_col) ,
    CONSTRAINT industry_id_fk FOREIGN KEY (industry_id)
        REFERENCES public.industry_dim (industry_id),
    CONSTRAINT type_id_fk FOREIGN KEY (type_id)
        REFERENCES public.type_dim (type_id)
);
```

4. Inserting data into the date dimension table

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
5. Loading data into other dimension tables from the dim_contracts_raw table



6. Loading data into the fact table from the raw data table
