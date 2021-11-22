# Development

## Update YB Platform Helm chart

1. Update the YB Platform helm chart as per the requirements. 
2. Set the RBAC creation to `false` because we already added the RBAC and SA creation part in `schema.yaml` as per the GKE app creation docs. 
   
```bash
# Set rbac.create to false in chart/yugaware/values.yaml
rbac:
  create: false
```

3. Set the following environment variables which will provide ease throughout the development. 

```bash
export REGISTRY=gcr.io/$(gcloud config get-value project | tr ':' '/')
export APP_NAME=yugabyte-platform
```

## Build

Use the below command to build the deployer image.

```bash
docker build -t $REGISTRY/$APP_NAME/deployer:<TAG> \
  --build-arg REGISTRY=$REGISTRY/$APP_NAME \
  --build-arg TAG=<TAG> \
  -f deployer/Dockerfile .
```

## Tag or push other images

You need to push or tag the other images with the same tag used above.
1. yugabyte-platform
2. postgres
3. prometheus
4. nginx

## Install

Create appropriate service account for yugabyte-platform deployment because `mpdev` doesn't create SA.

```bash
mpdev install \
	--deployer=$REGISTRY/$APP_NAME/deployer:<TAG> \
	--parameters='{"name": "<NAME>", "namespace": "<NS>", "yugaware.serviceAccount": "<SA>"}'
```

## Verify

Execute the below command to verify the changes.

```bash
mpdev verify \
	--deployer=$REGISTRY/$APP_NAME/deployer:<TAG>
```

## Notes

We have used following images from GCR on the place of Dockerhub and Quay images.

```bash
# postgres:11.5 -> gcr.io/cloud-marketplace/google/postgresql11:11.7
# prom/prometheus:v2.27.1 -> gcr.io/cloud-marketplace/google/prometheus2:2.18
# nginxinc/nginx-unprivileged:1.17.4-amd64 -> gcr.io/cloud-marketplace/google/nginx1:1.20
```

