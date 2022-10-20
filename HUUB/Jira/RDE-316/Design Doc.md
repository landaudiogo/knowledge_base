# Overview

Currently the data engineering team uses a single consumer image to deploy multiple consumer instances that base themselves on environment variables to understand their context.  

The context specifies: 
- From which environment it consumes information (Kafka)
- To which environment the data is to be written (BigQuery)
- To which table in Bigquery the data is to be inserted.
- Which topic the data is to be consumed from
- Which events are to be ignored
- The consumer-group id

# Context

When the consumer codebase changes currently the team has to manually build and push the consumer image to the container registry. 

Additionally, the team is also responsible for managing the consumer deployments. There is no versioning, and no monitoring being done over the consumer instances.

Provided the DevOps team has automated the CI pipeline and provides with the required tools for CD in a Kubernetes environment, the generic-consumer's is to make use of this solution and approach to simplify code updates, and rollbacks.

# Requirements

1. gcp credentials
2. prod1, prod2, prod3, uat defined in the /etc/hosts of the consumer pods.
3. creating multiple instances from the same image
4. Providing with different configurations to each consumer instance.
5. Using different GCP credentials and write environments based on the current cluster the consumer finds itself in.

# Goals

1. Seperate development and deployment concerns.
2. Automate building and deploying new versions of the consumer. 
3. Migrate consumer instances from GKE to EKS.

# Existing Solution

Currently, when the data-engineering development team wants to either consumer from a new domain, or update a consumer instance, it has to go through the process of rebuilding an image, pushing it to the image repository, creating a new deployment manifest that points to the new image, and redefine the environment variables. 

This process is done manually, and stores no deployment versioning, so if a consumer instance is updated, we lose the information regarding the previous deployment. A new GitOps approach would benefit the team.

# Proposed Solution

## Solution Overview
1. Create a Jenkins pipeline that automates building the image, pushing it to the repository, and updates the values.yaml file in the ArgoCD repository. 
2. Create an ArgoCD application that points to the values.yaml file, that launches the requested consumers specified in the consumers parameter of the file.

## Jenkinsfile_helm

There are 2 expected behaviours we desire when going through the Jenkins pipeline.

The first, has the pipeline building a new consumer image and sending it to the image repository. It is possible that we request to rebuild the image, and there is no new commit. This happens when the huub-messaging package is updated and the codebase doesn't change.

The second behaviour is when a new image is not required, and we simply want to add a new consumer instance. Here the expected behaviour would be to go through the Jenkins pipeline without rebuilding the image.

In both scenarios, the values.yaml file would have to be updated, therefore having the ArgoCD application go outofsync.

## ArgoCD

When the values.yaml file is updated, there are 2 alternate behaviours we wish to control. 

When the consumer codebase changes, we might want to update the whole application and all consumer instances. 

Contrarily, when a new image is created simply because the huub-messaging package has to be updated, we simply want to recreate the consumer instance the needs the updated requirements.

## Consumer Codebase Changes

The consumer process is started as follows: 
```
python3 manage.py <domain>
```
where domain is to be used by the consumer to understand its context in the environment variables.
When the consumer wants its environment variable, it gets the value using the base_name + domain. base_name, is the part of the name that does not change regardless of the domain, and domain is a unique attribute for a consumer group.

As an example of the previous development, the following attributes have to be provided for the consumer's context: BQ_TABLE, IMPORT_PATH, IMPORT_OBJECT, IGNORE_EVENTS, CONSUME_ENV, WRITE_ENV, GROUP_ID.
Since the same configmap is used for all consumers, each consumer has to understand from that configmap which variables refer to it. As such, if we specify `<domain>` to be `DELIVERY`, in the configmap we would have that the .env file would contain: 
```
BQ_TABLE_DELIVERY='delivery_events_temp'
IMPORT_PATH_DELIVERY='contracts.delivery.delivery_events_v10'
IMPORT_OBJECT_DELIVERY='DeliveryEventsV10Topic'
IGNORE_EVENTS_DELIVERY='"[\"ShippingPricesEstimatedEvent\"]"'
CONSUME_ENV_DELIVERY='prod'
GROUP_ID_DELIVERY='test-new-uat-ip'
WRITE_ENV='prod'
```
One thing to notice is that WRITE_ENV is not specified using base_name + domain. Depending on which cluster the configmap is written to, we would have to define this variable for all consumers, so it would be consulted using simply the base_name

Instead of using environment variables to set the consumer's configurations, a .env file is written in the consumer's directory, which should then be loaded, providing with the consumer's context. This .env is mounted using a configmap. (DE)

The template deployment for each consumer should have these entries in the hosts of the deployment. (DevOps)

The consumer should read a single gcp credentials file, which is mounted and specified in the configmap specific to the cluster. (DevOps) - has to specify the directory that will mount the configmap where the pod can access this information.
(DE) - Change the way in which the environment variables are loaded into the consumer, and the gcp credentials file the consumer has to point to. 


# Cross-Team Impact

This development will require coordination with the DevOps teams, as we require a pipeline to create build and push a new consumer image to ECR.

 # Actions
 
 1. Change the consumer codebase: 
	 1. new .env file
	 2. single gcp_credentials file
	 3. new environment variables naming
 2.  request that the required hosts are added to the template consumer deployment.
 3. request a new view in the pipeline that builds and pushes the image to the ECR.
 4. Create a new application in Argo that points to the correct values.yaml