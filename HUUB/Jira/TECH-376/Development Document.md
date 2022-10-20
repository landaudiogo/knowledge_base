# Development
#airflow #extraction #database #orchestrator

> This task refers to the following design document: 
> https://maersk-tools.atlassian.net/wiki/spaces/HUUB/pages/edit-v2/181652719288?draftShareId=d2fcb393-625a-46c1-93b2-a18fdfa60c8e&inEditorTemplatesPanel=auto_closed

1. Determine the active extraction packages that use a database as a source.
2. Develop template dag.
3. Determine how to get start and end date to pass as parameters to query. 
4. Determine how to use query parameters.
5. How to set a connection for the PostgresToGCSOperator

# 1. Active extraction packages that use a database

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
  AND DSC_DATA_SOURCE NOT LIKE "%api%"
ORDER BY
  DSC_DATA_SOURCE, DSC_PACKAGE
```

Result: 
![[HUUB/Jira/TECH-376/active_database_extraction.csv]]


# 2. Develop template Dag
We will be following the design of one dag per database source.

![[db_source_dag.svg]]

## C1
Contains a single task responsible for setting the end_timestamp for the tasks that are going to extract data from a database source.

## C2 
Contains as many tasks as packages that are active that fetch data from a database source.

These tasks have to be configured with: 
- credentials to access database
- start_timestamp
- end_timestamp

## C3 
Single task that awaits for all tasks in C2 to terminate so it can trigger execution of the next extraction cycle.

## S 

Indicates the tasks executing within it are of type sequential

## D 

Group of tasks that are executed on a daily process.
These have a conditional execution.
If the task has already executed with success for that day, then it is skipped, otherwise it is executed. 

> Airflow documentation skip task: 
> https://airflow.apache.org/docs/apache-airflow/stable/concepts/tasks.html#special-exceptions


# 3. Get Start and End Timestamp

## CDC Execution Period
### Start Timestamp
Depending on whether we want to guarantee redundancy between executions, there are 2 alternatives to define the lower timestamp.

For redundancy, the following diagram illustrates how 2 tasks fetched data for any given time instant.
![[start-end 1.svg]]
To implement this solution, if task $n$ is starting, we want it to use task's $n-2$ end period as task $n$'s start period.

Therefore, using xcoms, on task completion we should upload a the end_timestamp used by the task. 

For any given task, we only need 2 end periods in the xcom table. 

When a new task starts, it queries the xcom's table and requests for the second record. If there is only one record, we should use a default value.

### End Timestamp

When a dag starts, we have to guarantee the end timestamp for all tasks to be executed within.

For this reason, the first task in the dag is responsible for setting this value in the xcoms environment. 

Using the extraction from event sourcing as an example, there is a dag controller which is responsible for determining until when the data is to be fetched from the source with the value `extraction_end_date`.

> The following function demonstrates how the extraction_manager sets the extraction_end_date for all subtasks.
> https://github.com/Maersk-Global/dags/blob/26b2c42d80aea1b4138cae33f516d9f141405d09/helpers/dagExecutionHelper.py#L48

In our case we will use the variable in the xcom `db_extraction_end_date`

# 4. Determine how to use query parameters

> PostgresToGCSOperator: 
> https://airflow.apache.org/docs/apache-airflow-providers-google/stable/_modules/airflow/providers/google/cloud/transfers/postgres_to_gcs.html#PostgresToGCSOperator

> BaseSQLToGCSOperator: 
> https://airflow.apache.org/docs/apache-airflow-providers-google/stable/_modules/airflow/providers/google/cloud/transfers/sql_to_gcs.html#BaseSQLToGCSOperator


We would like to use a dynamic query parameter in our queries, that indicate the query the cdc time period for which we want to get our data.

For a class that inherits the BaseOperator class, the class attributed, `template_fields` indicates which instance attributes are "templatable". 

Since the PostgresToGCSOperator inherits from the BaseSQLToGCSOperator, and this class specifies that the attribute parameters is templatable, we would like to pass our `end_timestamp` and `start_timestamp` as query parameters that will be rendered through this attribute. 

# 5. Set a Postgres connection

Currently, we hold all the connection data on a datawarehouse table `config.cfg_connection`. 

## Set a conn for each entry in the table

 > airflow documentation to manage conns: 
 > https://airflow.apache.org/docs/apache-airflow/stable/howto/connection.html

> postgres connection documentation
> https://airflow.apache.org/docs/apache-airflow-providers-postgres/stable/connections/postgres.html

If this is possible, we could dynamically set the postgres connections.



