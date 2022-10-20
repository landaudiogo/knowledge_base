# Overview
1. Determine staging queries that use the ext event_sourcing table.
2. Create a new real_time table with the same structure as the staging tables.
3. Create an insert query into real_time table.
4. Create airflow cfg_package insert.
5. Determine facts and dims that use the staging tables in step 1.
6. Change the INSERT and UPDATE queries for the facts and dims.
7. Update script for delivery ID in fact and dim. 
8. Verify views that reference 3pl and delivery\_id
9. Deploy script


# Findings
## 1. Staging queries that get data from ext event_sourcing.

Run grep recursively in the staging directory to determine the .sql files that contain a regex string containing event_sourcing.ext_events.

`regex = '[\b\.]event_sourcing.ext_events'`

command:
```bash
grep -P -e '[\s\.]event_sourcing\.ext_events' -r
```

output: 
```
SEL_PRE_3PL_ORDERS_COMMUNICATED.sql:                        FROM event_sourcing.ext_events
```

We can conclude that the staging files that use the event_sourcing.ext_events table are:
- `SEL_PRE_3PL_ORDERS_COMMUNICATED.sql`


## 2. Create real_time table with the same structure

1. Create a view with the select query to determine the types of each column
```SQL
CREATE VIEW star_schema_view.temp_3pl_communicated AS     
    WITH ext_events AS (SELECT
                            event_json,
                            event_type
                        FROM event_sourcing.ext_events
                        WHERE event_type = 'ExtDeliveryReadyToFulfilEventV2'
                          AND stream_timestamp_date >= CAST(CAST("2022-03-01" AS TIMESTAMP) AS DATE)
                          AND stream_timestamp_hour >= CAST(FORMAT_TIMESTAMP('%F %H:00:00', CAST("2022-03-01" AS TIMESTAMP)) AS TIMESTAMP)
                          AND CAST(DATETIME(TIMESTAMP_MICROS(CAST(ROUND(stream_timestamp/1000) AS INT64)), 'UTC') AS TIMESTAMP) > "2022-03-01"
                          AND CAST(DATETIME(TIMESTAMP_MICROS(CAST(ROUND(stream_timestamp/1000) AS INT64)), 'UTC') AS TIMESTAMP) <= "2022-03-01"
                        GROUP BY event_json, event_type)

    SELECT
        id,
        delivery_id,
        date_delivery_created_at_3pl,
        date_delivery_communicated,
        carrier_communicated,
        service_type_communicated,
        estimated_delivery_date,
        generate_return_label,
        print_return_label,
        return_carrier,
        return_service_type,
        sales_channel,
        fulfillment_type_allowed,
        customer_reference,
        incoterm,
        SUM(source.qnt_expected) AS qnt_expected,
        created_at,
        updated_at
    FROM (SELECT
              CONCAT(JSON_VALUE(event_json, '$.order_reference'), '-', JSON_VALUE(event_json, '$.timestamp')) AS id,
              CAST(JSON_VALUE(event_json, '$.timestamp') AS INT64)                                            AS timestamp,
              CAST(DATETIME(TIMESTAMP_MICROS(CAST(ROUND(CAST(JSON_VALUE(event_json, '$.timestamp') AS INT64) / 1000) AS INT64)),
                            'UTC') AS TIMESTAMP)                                                              AS date_delivery_created_at_3pl,
              CAST(DATETIME(TIMESTAMP_MICROS(CAST(ROUND(CAST(JSON_VALUE(event_json, '$.timestamp') AS INT64) / 1000) AS INT64)),
                            'UTC') AS TIMESTAMP)                                                              AS date_delivery_communicated,
              JSON_VALUE(event_json, '$.order_reference')                                                     AS delivery_id,
              JSON_VALUE(event_json, '$.shipping.shipping_carrier')                                           AS carrier_communicated,
              JSON_VALUE(event_json, '$.shipping.shipping_service_type')                                      AS service_type_communicated,
              CAST(JSON_VALUE(event_json, '$.estimated_delivery_date') AS TIMESTAMP)                          AS estimated_delivery_date,
              CAST(JSON_VALUE(event_json, '$.shipping.generate_return_label') AS BOOLEAN)                     AS generate_return_label,
              CAST(JSON_VALUE(event_json, '$.shipping.print_return_label') AS BOOLEAN)                        AS print_return_label,
              JSON_VALUE(event_json, '$.shipping.return_carrier')                                             AS return_carrier,
              JSON_VALUE(event_json, '$.shipping.return_service_type')                                        AS return_service_type,
              JSON_VALUE(event_json, '$.sales_channel')                                                       AS sales_channel,
              JSON_VALUE(event_json, '$.fulfillment_type_allowed')                                            AS fulfillment_type_allowed,
              JSON_VALUE(event_json, '$.customer_reference')                                                  AS customer_reference,
              JSON_VALUE(event_json, '$.shipping.incoterm')                                                   AS incoterm,
              JSON_QUERY(event_json, '$.packing_list')                                                        AS packing_list,
              CAST(JSON_VALUE(items, '$.qnt_expected') AS INT64)                                              AS qnt_expected,
              CAST(JSON_VALUE(items, '$.variant_id') AS INT64)                                                AS variant_id,
              CAST(DATETIME(TIMESTAMP_MICROS(CAST(ROUND(CAST(JSON_VALUE(event_json, '$.timestamp') AS INT64) / 1000) AS INT64)),
                            'UTC') AS TIMESTAMP)                                                              AS created_at,
              CAST(DATETIME(TIMESTAMP_MICROS(CAST(ROUND(CAST(JSON_VALUE(event_json, '$.timestamp') AS INT64) / 1000) AS INT64)),
                            'UTC') AS TIMESTAMP)                                                              AS updated_at
          FROM ext_events
          LEFT JOIN UNNEST(JSON_QUERY_ARRAY(event_json, '$.packing_list')) AS items
    ) source
    GROUP BY id, date_delivery_created_at_3pl, date_delivery_communicated, delivery_id, carrier_communicated,
        service_type_communicated,estimated_delivery_date, generate_return_label, print_return_label,
        return_carrier, return_service_type, sales_channel, fulfillment_type_allowed, customer_reference,
        incoterm, created_at, updated_at
```

2. Get the column_name, and type of all columns from the created temporary view
```SQL
SELECT column_name || " " ||  data_type
FROM star_schema_view.INFORMATION_SCHEMA.COLUMNS
WHERE table_name = "temp_3pl_communicated"
```

```
id STRING,
delivery_id STRING,
date_delivery_created_at_3pl TIMESTAMP,
date_delivery_communicated TIMESTAMP,
carrier_communicated STRING,
service_type_communicated STRING,
estimated_delivery_date TIMESTAMP,
generate_return_label BOOL,
print_return_label BOOL,
return_carrier STRING,
return_service_type STRING,
sales_channel STRING,
fulfillment_type_allowed STRING,
customer_reference STRING,
incoterm STRING,
qnt_expected INT64,
created_at TIMESTAMP,
updated_at TIMESTAMP
```

3. Remove the temporary view created in step 1.
```SQL
DROP VIEW star_schema_view.temp_3pl_communicated
```

4. Create real time table ddl
```SQL
CREATE TABLE real_time.ext ( 
	event_id STRING, 
	event_type STRING,
	id STRING,
	delivery_id STRING,
	date_delivery_created_at_3pl TIMESTAMP,
	date_delivery_communicated TIMESTAMP,
	carrier_communicated STRING,
	service_type_communicated STRING,
	estimated_delivery_date TIMESTAMP,
	generate_return_label BOOL,
	print_return_label BOOL,
	return_carrier STRING,
	return_service_type STRING,
	sales_channel STRING,
	fulfillment_type_allowed STRING,
	customer_reference STRING,
	incoterm STRING,
	qnt_expected INT64,
	created_at TIMESTAMP,
	updated_at TIMESTAMP,
	real_time_inserted_at TIMESTAMP
)
PARTITION BY
    TIMESTAMP_TRUNC(real_time_inserted_at, DAY)
CLUSTER BY
    created_at;
```
Added: 
	event_id, 
	event_type, 
	real_time_inserted_at
Removed: 
	id, 

## 3. Create insert query into real_time table

1. Create the file INS_EXT.sql in dags/extraction_queries/event_sourcing/INS_EXT.sql
2. Create delivery_qnt CTE
3. Create ext_struct CTE
4. Join both CTE's based on the event_id

## 4. Create airflow cfg_package insert
```SQL

INSERT INTO config.cfg_airflow_package (
  SK_PACKAGE,
  DSC_PACKAGE,
  DSC_DATA_SOURCE,
  DSC_TABLE_PURPOSE,
  DSC_TABLE_SOURCE_SCHEMA,
  DSC_TABLE_SOURCE_NAME,
  DSC_TABLE_DESTINATION_NAME,
  DSC_QUERY_INSERT_FILE,
  BOOL_IS_ACTIVE,
  GROUP_ID,
  BOOL_HAS_KEY_UPDATE,
  BOOL_HAS_KEY_RESERVATION,
  DSC_DAG_ID
)
WITH
  sk_struct AS (
    SELECT MAX(SK_PACKAGE) max_sk
    FROM config.cfg_airflow_package
  )
SELECT 
  max_sk + 1 AS SK_PACKAGE,
  'ext' AS DSC_PACKAGE,
  'bigquery' AS DSC_DATA_SOURCE,
  'real_time' AS DSC_TABLE_PURPOSE,
  'event_sourcing' AS DSC_TABLE_SOURCE_SCHEMA,
  'ext_events' AS DSC_TABLE_SOURCE_NAME,
  'ext' AS DSC_TABLE_DESTINATION_NAME,
  'INS_EXT.sql' AS DSC_QUERY_INSERT_FILE,
  TRUE AS BOOL_IS_ACTIVE,
  1 AS GROUP_ID,
  FALSE AS BOOL_HAS_KEY_UPDATE,
  FALSE AS BOOL_HAS_KEY_RESERVATION,
  'source_extractor_events_ext' DSC_DAG_ID
FROM sk_struct
```

```SQL
UPDATE config.cfg_package 
SET BOOL_IS_ACTIVE = FALSE
WHERE DSC_PACKAGE = 'pre_3pl_orders_communicated'
```

## 5. Determine facts and dims that use the staging tables in 1

The staging select query created a table in the staging schema named `staging.pre_3pl_orders_communicated`.

Therefore we search for files containing this string in the star_schema directory.
```bash
grep -P -e '[\s\.]staging\.pre_3pl_orders_communicated' -r
```

Output: 
```
INS_DIM_3PL_ORDERS_COMMUNICATED.sql:      FROM staging.pre_3pl_orders_communicated AS pre
INS_FACT_3PL_ORDERS_COMMUNICATED.sql:FROM staging.pre_3pl_orders_communicated AS pre
```

These are the files that need to be redirected to the real_time schema:
- INS_DIM_3PL_ORDERS_COMMUNICATED.sql
- INS_FACT_3PL_ORDERS_COMMUNICATED.sql

## 6.  Change the queries from 5.

1. point pre table into real_time.ext
2. add the real_time_inserted_at query

**INS_FACT_3PL_ORDERS_COMMUNICATED**
The delivery id contained an extra \# character which was complicating the joins with the remaining tables related to a delivery.
We change the delivery\_id to have the same structure, but this implies updating the current fact and dim 3pl, and verifying whether there are any views that contain references to 3pl which use the delivery join.


## 7. Create update script for delivery_id
**dim_3pl_orders_communicated**
columns: 
- COD_3PL_ORDERS_COMMUNICATED
```sql
UPDATE star_schema.dim_3pl_orders_communicated d3pl
SET COD_3PL_ORDERS_COMMUNICATED = modified.COD_3PL_ORDERS_COMMUNICATED
FROM (
  SELECT 
    SK_3PL_ORDERS_COMMUNICATED, 
    REPLACE(COD_3PL_ORDERS_COMMUNICATED, '-#', '-') AS COD_3PL_ORDERS_COMMUNICATED
  FROM star_schema.dim_3pl_orders_communicated 
) as modified
WHERE d3pl.SK_3PL_ORDERS_COMMUNICATED = modified.SK_3PL_ORDERS_COMMUNICATED
```

**fact_3pl_orders_communicated**
columns:
- id
- dd_cod_3pl_orders_communicated
```SQL
UPDATE star_schema.fact_3pl_orders_communicated f3pl
SET 
  id = modified.id,
  DD_COD_3PL_ORDERS_COMMUNICATED = modified.DD_COD_3PL_ORDERS_COMMUNICATED
FROM (
  SELECT 
    SK_3PL_ORDERS_COMMUNICATED, 
    REPLACE(id, '-#', '-') AS id,
    REPLACE(DD_COD_3PL_ORDERS_COMMUNICATED, '-#', '-') AS DD_COD_3PL_ORDERS_COMMUNICATED
  FROM star_schema.fact_3pl_orders_communicated 
) as modified
WHERE f3pl.SK_3PL_ORDERS_COMMUNICATED = modified.SK_3PL_ORDERS_COMMUNICATED
```


## 8. Views that reference 3pl with delivery_id

**fact_3pl_orders_communicated**
```bash
grep -P -e '[\s\.]star_schema\.fact_3pl_orders_communicated' -r
```

Views: 
- v_report_non_shipped_items.sql 
- v_report_operational_fulfillment.sql
- v_fact_shipment_outbound.sql

v_report_non_shipped_items.sql
Nothing to change. Does not reference any of the columns that are to be updated.

v_report_operational_fulfillment.sql
Nothing to change. Does not reference any of the columns that are to be updated. 

v_fact_shipment_outbound.sql
Nothing to change. Does not reference any of the columns that are to be updated.


**dim_3pl_orders_communicated**
```bash
grep -P -e '[\s\.]star_schema\.dim_3pl_orders_communicated' -r
```

There are no views that reference this table.

## 9. Deploy Script

