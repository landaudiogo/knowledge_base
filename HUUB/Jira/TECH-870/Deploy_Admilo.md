### DO NOT MERGE AND PULL THIS PR BEFORE STEP 9
This PR introduces a new package(dbt) that needs to be installed before the merge.
The work done here was:

- [x] Import billing shipping prices files from sharepoint to google cloud storage
- [x] Dynamic data models creation/update with dbt on top of the last files imported
- [x] Data validation with dbt_expectations on top of the the last files imported
- [x] Move of validated files to a different cloud storage bucket(the same as mentioned on this [PR)](https://github.com/Maersk-Global/huub-etl-orchestrator/pull/32
- [x] Slack Alert when validation fails(To be improved and sent via email)

### Deploy

**PRE REQUIREMENTS**

**New dataset on BigQuery**
1 - Create dataset 'dbt_models'

**Dbt Installation**

1. Activate you virtual env inside airflow folder
2. `pip install dbt-core`
3. `pip install dbt-bigquery`
4. Run `dbt --version` to confirm that you installed the version 1.1.0(*critical*)
5. Move to the airflow/dags folder and run `dbt init` and follow the steps as exemplified bellow:
```bash
$ dbt init
17:06:42  Running with dbt=1.1.0
Enter a name for your project (letters, digits, underscore): dbt
Which database would you like to use?
[1] bigquery
[2] postgres
[3] redshift
[4] snowflake

(Don't see the one you want? https://docs.getdbt.com/docs/available-adapters)

Enter a number: 1
[1] oauth
[2] service_account
Desired authentication method option (enter a number): 2
keyfile (/path/to/bigquery/keyfile.json): C:\\Users\\AEM072\\Documents\\Workspace\\dags\\auth\\huub-dwh-1f56dc412a38-AdmiloRibeiro.json
project (GCP project id): <PROJECT>
dataset (the name of your dbt dataset): dbt_models
threads (1 or more): 2
job_execution_timeout_seconds [300]:
[1] US
[2] EU
Desired location option (enter a number): 2
17:16:03  Profile dbt written to C:\Users\AEM072\.dbt\profiles.yml using target's profile_template.yml and your supplied values. Run 'dbt debug' to validate the connection.
17:16:03
Your new dbt project "dbt" was created!
````
6. Open the file at `$HOME/.dbt/profiles.yml` and make sure it has the following structure:

```yaml
default: # this is the profile name that will be referenced on dbt_project.yml file
  outputs:
    prod:
      type: bigquery
      method: service-account
      project: <GCP PROJECT>
      dataset: dbt_models # You can also use "schema" here
      threads: 2
      keyfile: '<GCP KEY PATH>'
      timeout_seconds: 300
      location: EU # Optional, one of US or EU
      priority: interactive
      retries: 1
```
7. Change to the new directory `dbt` and run `dbt debug` . You should have a response similar to bellow:

```bash
17:22:59  Running with dbt=1.1.0
dbt version: 1.1.0
python version: 3.X.X
...
Using profiles.yml file at <PATH>\.dbt\profiles.yml
Using dbt_project.yml file at <PATH>\dbt\dbt_project.yml

Configuration:
  profiles.yml file [OK found and valid]
  dbt_project.yml file [OK found and valid]

Required dependencies:
 - git [OK found]

Connection:
  method: service-account
  database: huub-dwh
  schema: dbt_models
  location: EU
  priority: interactive
  timeout_seconds: 300
  maximum_bytes_billed: None
  execution_project: huub-dwh
  job_retry_deadline_seconds: None
  job_retries: 1
  job_creation_timeout_seconds: None
  job_execution_timeout_seconds: 300
  Connection test: [OK connection ok]

All checks passed!

```
8. Install `pip install airflow-dbt`
9. Merge this PR(**only at this stage**) and pull. 
You should see the new dbt_project.yml file inside the dbt folder.

10.Run `dbt deps`. This will install the packages specified on the file `packages.yml`

11. All done

### BigQuery Data Models
For dbt to be able to read the data imported into cloud storage, those files needs to be part of an external table in bigQuery.
Then it's possible to filter by import_epoch and get the last import of each one.

To create the external tables, you'll need to:
1. Create the bucket where the files will be sent initially(no validations at this step)
2. Upload a sample(.csv) of each file['extra_items', 'fuel', 'outbound', 'returns', 'stock'] only with headers. 
The files needs to be stored in:
`<bucket_name>/imported/client=<client_name>/warehouse=<warehouse_name>/import_epoch=<epoch>/<filename>.csv`
3. Run the following script on your local machine:

```python
from google.cloud import bigquery
from google.oauth2 import service_account

bq_credentials = service_account.Credentials.from_service_account_file(
    '<PATH_YOUR_GCP_KEY>'
)
GCP_PROJECT = 'huub-dwh-prod'
BUCKET_NAME = '<BUCKET_NAME>'
client = bigquery.Client(project=f"{GCP_PROJECT}", credentials=bq_credentials)

tables = ['extra_items', 'fuel', 'outbound', 'returns', 'stock']

for t in tables:

    sql = f"""CREATE OR REPLACE EXTERNAL TABLE dbt_models.mw_billing_{t}
    WITH PARTITION COLUMNS (
        client STRING,
        warehouse STRING,
        import_epoch INT64
    )
    OPTIONS (
        FORMAT='CSV',
        URIS=['gs://{BUCKET_NAME}/imported/*/{t}.csv'],
        SKIP_LEADING_ROWS=1,
        hive_partition_uri_prefix='gs://{BUCKET_NAME}/imported'
    )"""
    
    client.query(sql).result()
    print(f"mw_billing_{t} created with success")

```
4. You should now have the tables mw_billing_<file_name> inside the dbt_models dataset.