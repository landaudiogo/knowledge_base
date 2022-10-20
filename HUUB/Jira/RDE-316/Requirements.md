The generic-consumer repository is a template codebase that allows a consumer to be created to consume from a single topic in a Kafka environment, and to write to a single BigQuery table in a specified environment.


# Modifiable Parameters 

1. We need to be able to trigger a new image build
2. Define the deployment name
4. Define the namespace
5. specify hosts
6. image version

## Env

CONSUME_ENV
WRITE_ENV
IMPORT_PATH
IMPORT_OBJECT
IGNORE_EVENTS
BQ_TABLE
GROUP_ID


