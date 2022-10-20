# Carrier Contracts DAG

## Query for active packages

```SQL
SELECT 
  *
FROM huub-dwh-prod.config.cfg_package
WHERE 
  BOOL_IS_ACTIVE = TRUE
  AND DSC_DATA_SOURCE NOT IN (
    "datawarehouse", 
    "bigquery"
  )
  AND DSC_DATA_SOURCE = @data_source
ORDER BY
  DSC_DATA_SOURCE, DSC_PACKAGE
```

The file bigQueryHelper contains the query that currently gets the active real time packages.


## Operator to set the end timestamp for source dag

```python
def get_events_extraction_end_period(dag_manager, **context):
    try:
        ti = context['ti'] # gets current task instance
        period = context["dag_run"].conf
        ti.xcom_push("extraction_end_date", period['end_date']) # pushes a value into xcom, that indicates the end_period for this dag
    except KeyError:
        # don't allow dag to be run outside manager
        error_desc = f"This dag can't be executed standalone. Please use the Dag: {dag_manager}"
        raise RuntimeError(error_desc)
```

## Operator to execute sequential extraction

## Opeartor to execute daily extraction

## Operator to activate the task again