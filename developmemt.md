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
export TAG=2.14.4
```

## Build

Use the below command to build the deployer image.

```bash
docker build -t $REGISTRY/$APP_NAME/deployer:$TAG \
  --build-arg REGISTRY=$REGISTRY/$APP_NAME \
  --build-arg TAG=$TAG \
  -f deployer/Dockerfile .
```

## Tag

You need tag the images with the same tag used above.

1. deployer
```bash
docker tag $REGISTRY/$APP_NAME/deployer:$TAG $REGISTRY/$APP_NAME/deployer:${TAG%.*}
```

2. yugabyte-platform
  - Need to rebuild the image with GCR based centos base image
  - Image - `gcr.io/cloud-marketplace/google/centos7@sha256:011b9adae93109c49bb7bfd23493e6c6bc2bde0277f12ad4f6d126fd476d6065`
```bash
docker tag <REBUILD_IMAGE> $REGISTRY/$APP_NAME:${TAG%.*}
docker tag <REBUILD_IMAGE> $REGISTRY/$APP_NAME:${TAG}
```

3. postgres
```bash
docker tag <IMAGE> $REGISTRY/$APP_NAME/postgres:${TAG%.*}
docker tag <IMAGE> $REGISTRY/$APP_NAME/postgres:${TAG}
```
4. prometheus
```bash
docker tag <IMAGE> $REGISTRY/$APP_NAME/prometheus:${TAG%.*}
docker tag <IMAGE> $REGISTRY/$APP_NAME/prometheus:${TAG}
```

5. nginx
```bash
docker tag <IMAGE> $REGISTRY/$APP_NAME/nginx:${TAG%.*}
docker tag <IMAGE> $REGISTRY/$APP_NAME/nginx:${TAG}
```

## Push

Push all the images tagged above.

## Environment setup

We need GKE cluster in the same google project `yugabyte-mp-public` to test the app related changes.

- Use the following command to get the credentials of running GKE test cluster.

```bash
gcloud container clusters get-credentials <CLUSTER_NAME> --zone <ZONE_NAME>
```

- It has `test-ns` namespace which used for testing purpose. It already have service account with required privileges.
  Use the following command as reference to deploy the application in cluster for testing.

```bash
mpdev install \
	--deployer=$REGISTRY/$APP_NAME/deployer:$TAG \
	--parameters='{"name": "yw-test", "namespace": "test-ns", "yugaware.serviceAccount": "yugabyte-platform-universe-management"}'
```

## Install

Create appropriate service account for yugabyte-platform deployment because `mpdev` doesn't create SA.

```bash
mpdev install \
	--deployer=$REGISTRY/$APP_NAME/deployer:$TAG \
	--parameters='{"name": "<NAME>", "namespace": "<NS>", "yugaware.serviceAccount": "<SA>"}'
```

## Verify

Execute the below command to verify the changes.

```bash
mpdev verify \
	--deployer=$REGISTRY/$APP_NAME/deployer:$TAG
```

## Notes

We have used following images from GCR on the place of Dockerhub and Quay images.

```bash
# postgres:11.5 -> gcr.io/cloud-marketplace/google/postgresql14:14.4
# prom/prometheus:v2.27.1 -> gcr.io/cloud-marketplace/google/prometheus2:2.33
# nginxinc/nginx-unprivileged:1.17.4-amd64 -> gcr.io/cloud-marketplace/google/nginx1:1.20
```

