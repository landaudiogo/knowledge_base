# Bigquery

Create table audit.etl:
```sql
CREATE OR REPLACE TABLE audit.etl (
	SK_CYCLE INT64,
	START_PERIOD TIMESTAMP, 
	END_PERIOD TIMESTAMP, 
	IS_SUCCESSFUL BOOLEAN
);
```

Insert the first rows into the table: 
```sql
INSERT INTO audit.etl(SK_CYCLE, START_PERIOD, END_PERIOD, IS_SUCCESSFUL)
SELECT
	ROW_NUMBER() OVER(ORDER BY START_PERIOD ASC) as rn, 
	aux.*
FROM(
	SELECT
		LEAD(START_DATE) OVER(ORDER BY START_DATE DESC) AS START_PERIOD,
		START_DATE AS END_PERIOD,
		TRUE
	FROM audit.star_schema_audit
	WHERE 
		dsc_table = 'dim_delivery'
		AND CAST(START_DATE AS DATE) >= current_date()
	ORDER BY start_date DESC
	LIMIT 3
) aux;
LIMIT 2
```

# Prod EC2

Add to config.py the following line:
```python
ETL_FIRST_ORCHESTRATOR = "external_orchestrator"
```

