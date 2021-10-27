# Development

Do the development as per the requirements and follow the below steps to build and verify it.

Set some environment variables which will provide ease throughout the development. 

```shell
export REGISTRY=gcr.io/$(gcloud config get-value project | tr ':' '/')
export APP_NAME=yugabyte-platform
```

## Build

Use the below command to build the deployer image.

```shell

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

```shell
mpdev install \
	--deployer=$REGISTRY/$APP_NAME/deployer:<TAG> \
	--parameters='{"name": "<NAME>", "namespace": "<NS>", "yugaware.serviceAccount": "<SA>"}'
```

## Verify

Execute the below command to verify the changes.

```shell
mpdev verify \
	--deployer=$REGISTRY/$APP_NAME/deployer:<TAG>
```

## Notes

We have used following images from GCR on the place of Dockerhub and Quay images.

```shell
postgres:11.5 -> gcr.io/cloud-marketplace/google/postgresql11:11.7
prom/prometheus:v2.27.1 -> gcr.io/cloud-marketplace/google/prometheus2:2.18
nginxinc/nginx-unprivileged:1.17.4-amd64 -> gcr.io/cloud-marketplace/google/nginx1:1.20
quay.io/yugabyte/yugaware -> quay.io/yugabyte/yugaware-itest:2.9.0.0-b4-gcp-mkt
```

