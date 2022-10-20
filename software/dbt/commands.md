---
title: DBT Commands
author: Diogo Landau
date: 2022-09-15
---

# dbt run

Executes compiled sql model files against the current target database. dbt
connects to the target database and runs the relevant SQL required to
materialize all data models. 

dbt run minimizes the amount of time in which a model is unavailable by first
building each model with a tmeporary name, then dropping the existing model,
then renaming the model to its correct name. This drop and renaming happens
within a single database transaction.

## Run Models with Variables

> DBT docs variables: 
> https://docs.getdbt.com/docs/building-a-dbt-project/building-models/using-variables

Variables can be defined globally at the `dbt_project.yml` level, or at runtime. 

If a variable exists in the `dbt_project.yml` file, and it is set on runtime,
the value defined on runtime overrides the project level value. 

To set a variable at runtime: 
```bash
dbt run --vars '{"key": "value", "date": 20180101}'
```

Setting variables in the `dbt_project.yml` file
```yaml
# dbt_project.yml

# ...
vars: 
  # The variable is accessible to hosting project resource and package resources
  start_date: '2022-01-01'
  
  # The `platforms` variable is only acccessible to resources in the
  # my_dbt_project project
  my_dbt_project: 
    platforms: ['web', 'mobile']
    
  # The `app_ids` variable is only accessible to resources in the snowplow
  # package
  snowplow: 
    app_ids: ['marketing', 'app', 'landing-page']
  
# ... 
  
```

# dbt test

runs tests defined  on models, sources, snapshots, and seeds. It expects that
you have already created those resources through the appropriate commands.

## Exit Code

If there is at least one error when running the test command, it is considered
to have a non-zero status code. 

## Storing Failures

> DBT docs store failures:
> https://docs.getdbt.com/reference/resource-configs/store_failures

Using the `store_failures` config implies that the failed rows for a given test
are stored in the database. 

To specify where to store the failures in the data warehouse, in the
dbt_project.yml it is possible to specify whether the test is stored and the
dataset where the data is to be stored


# dbt compile
