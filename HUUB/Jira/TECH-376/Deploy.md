# Deploy

## Requirements
1. Guarantee the scheduler and workers are not up:
```bash
sudo service airflow_scheduler stop
sudo service airflow_worker stop
```
2. Merge the pull request.
3. pull the changes in the production environment. 
```bash
cd ~/airflow/dags
git pull
```

## Airflow Configurations

This section describes what has to be done to create the connections on airflow.


1. Run the following query in BigQuery, and result into a json file.
```sql
SELECT
  DSC_SOURCE, DSC_HOST, DSC_PORT, DSC_PWD, DSC_DB_NAME, DSC_USER
FROM huub-dwh-prod.config.cfg_connection c
RIGHT JOIN (
  SELECT 
    DISTINCT DSC_DATA_SOURCE
  FROM huub-dwh-prod.config.cfg_package
  WHERE 
    BOOL_IS_ACTIVE = TRUE
    AND DSC_DATA_SOURCE NOT IN (
      "datawarehouse", 
      "bigquery"
    )
    AND DSC_DATA_SOURCE NOT LIKE "%api%"
  ORDER BY
    DSC_DATA_SOURCE
) package
  ON package.DSC_DATA_SOURCE = c.DSC_SOURCE
```
2. save the results into as a json file locally.
3. copy the contents of the json file, and create a new file in the production environment `~/airflow/dags/scripts/TECH-339/input.json`
```bash
cat <<< '
CTRL+SHIFT+V
' > "~/airflow/dags/scripts/TECH-339/input.json"
```
4. Run the following command:
```bash
cd ~/airflow/dags/scripts/TECH-339
```
5. Guarantee the `airflow_conn_script.sh` is executable: 
```bash
chmod u+x airflow_conn_script.sh
```
6. Run the script passing as parameters, `~/airflow/dags/input.json` representing the input file, and `~/airflow/dags/scripts/TECH-339/connection_script.sh` where the new script is to be inserted.
```bash
./airflow_conn_script.sh input.json connection_script.sh
```
7. Guarantee the `connection_script.sh` is executable, and run the script:
```bash
chmod u+x connection_script.sh
./connection_script.sh
```

## Bigquery Configurations

```sql
BEGIN TRANSACTION;

INSERT INTO config.cfg_airflow_package (
  SK_PACKAGE,
  DSC_PACKAGE, 
  DSC_DATA_SOURCE, 
  DSC_TABLE_PURPOSE, 
  DSC_TABLE_SOURCE_SCHEMA, 
  DSC_TABLE_SOURCE_NAME, 
  DSC_TABLE_DESTINATION_NAME, 
  DSC_QUERY_SELECT_FILE,
  BOOL_IS_ACTIVE, 
  GROUP_ID, 
  DSC_EXECUTION_TYPE
)
WITH 
  max_sk AS (
    SELECT 
      MAX(SK_PACKAGE) AS sk 
    FROM 
      config.cfg_airflow_package
  )
SELECT 
  max_sk.sk+rn AS SK_PACKAGE, * EXCEPT(rn, sk) 
FROM (
  SELECT 
    ROW_NUMBER() OVER() AS rn,
    DSC_PACKAGE, 
    DSC_DATA_SOURCE, 
    'staging' AS DSC_TABLE_PURPOSE,
    DSC_TABLE_SOURCE_SCHEMA,
    DSC_TABLE_SOURCE_NAME,
    DSC_TABLE_DESTINATION_NAME, 
    DSC_QUERY_SELECT_FILE,
    TRUE AS BOOL_IS_ACTIVE,
    1 AS GROUP_ID,
	DSC_EXECUTION_TYPE
  FROM config.cfg_package
  WHERE 
    BOOL_IS_ACTIVE = TRUE
    AND DSC_DATA_SOURCE NOT IN (
      "datawarehouse", 
      "bigquery"
    )
    AND DSC_DATA_SOURCE NOT LIKE "%api%"
  ORDER BY
    DSC_DATA_SOURCE, DSC_PACKAGE
) 
LEFT JOIN max_sk ON TRUE;

UPDATE config.cfg_package cfg_package
SET cfg_package.BOOL_IS_ACTIVE = FALSE
FROM (
  SELECT 
    SK_PACKAGE,
    DSC_PACKAGE
  FROM config.cfg_package
  WHERE 
    BOOL_IS_ACTIVE = TRUE
    AND DSC_DATA_SOURCE NOT IN (
      "datawarehouse", 
      "bigquery"
    )
    AND DSC_DATA_SOURCE NOT LIKE "%api%"
) AS database_extract_pck
WHERE database_extract_pck.SK_PACKAGE = cfg_package.SK_PACKAGE;

COMMIT TRANSACTION;
```


# Deploy

- [ ] Stop the Jenkins ETL pipeline.
- [ ] Run the following script on BigQuery PROD environment: [**bq_script.zip**](https://github.com/Maersk-Global/dags/files/9176694/bq_script.zip)
- [ ] Run the following script on PROD AWS machine (copy and pasting each command might suffice): [**connections_script.zip**](https://github.com/Maersk-Global/dags/files/9176697/connections_script.zip)
- [ ] git pull in dags directory
- [ ] activate all dags, except source_extractor_database_await_manager
- [ ] manually trigger the dag source_extractor_database_manager and wait for the manager cycle to successfully complete.
- [ ] run following python script to create external tables: **[create_external_tables_gbq.zip](https://github.com/Maersk-Global/dags/files/9179248/create_external_tables_gbq.zip)**

# Definition of Done


- [ ] Unit and integration tests
- [x] ~~code coverage equal or higher~~
- [ ] Build is green
- [ ] Sonarcube validation
- [x] ~~Jenkins pipeline  - Data Consumers.~~
- [ ] Code Review
- [ ] Code meets Standards ([different standards](https://maersk-tools.atlassian.net/wiki/spaces/HUUB/pages/181582544796) for different types of changes: Python/Sql)
- [ ] review logging, exceptions, structure of project and tablees, etc.