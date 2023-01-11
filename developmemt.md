# Development

## Update YB Platform Helm chart

1. Update the YB Platform helm chart as per the requirements.
2. Set the RBAC creation to `false` because we already added the RBAC and SA creation part in `schema.yaml` as per the GKE app creation docs.

```bash
# Set rbac.create to false in chart/yugaware/values.yaml
rbac:
  create: false

# Set pdbPolicyVersionOverride to v1 in chart/yugaware/values.yaml
pdbPolicyVersionOverride: "v1"
```

**Note** - We need to update the charts according to following [PR](https://github.com/yugabyte/charts/pull/101). [We can remove the following point once the PR merged]

## Build deployer

- Set the following environment variables which will provide ease throughout the development.

  ```bash
  export REGISTRY=gcr.io/$(gcloud config get-value project | tr ':' '/')
  export APP_NAME=yugabyte-platform
  export TAG=2.14.6
  ```

- Use the below command to build the deployer image.

  ```bash
  docker build -t $REGISTRY/$APP_NAME/deployer:$TAG \
    --build-arg REGISTRY=$REGISTRY/$APP_NAME \
    --build-arg TAG=$TAG \
    -f deployer/Dockerfile .
  ```

## Pull components images

We have used following images from GCR on the place of Dockerhub and Quay images.

```bash
# postgres:11.5 -> gcr.io/cloud-marketplace/google/postgresql14@sha256:83f4dc4192f0d9c19cbfdde7788903ed367182b1a8b8305fe8e103b5ab3fbe16
# prom/prometheus:v2.27.1 -> gcr.io/cloud-marketplace/google/prometheus2@sha256:dbb59cab276e492ec0f05e87b6fe216531fabcf35af4b120b383ce5ed75ef3b9
# nginxinc/nginx-unprivileged:1.17.4-amd64 -> gcr.io/cloud-marketplace/google/nginx1@sha256:30e8694c19e0d680edfca5b9e5f8ee5219df2e644d8387e00c61c9ce45623297
```

## Tag and push images to GCR

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

**Note** - Used quay image directly for 2.14.6

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

## Environment setup to test the changes

- We need GKE cluster in the same google project `yugabyte-mp-public` to test the app related changes.

- Set project

  ```bash
  gcloud config set project yugabyte-mp-public
  ```
- Deploy a GKE cluster.

- Use the following command to get the credentials of running GKE test cluster.

  ```bash
  gcloud config set project yugabyte-mp-public
  gcloud container clusters get-credentials <CLUSTER_NAME> --zone <ZONE_NAME>
  ```

- Setup following environment variables.

  ```bash
  export YBA_NAMESPACE="yb-platform"
  export REGISTRY=gcr.io/$(gcloud config get-value project | tr ':' '/')
  export APP_NAME=yugabyte-platform
  export TAG=2.14.6
  ```

- Setup namespace, service account, and clusterrolebinding to test the gke app. We have created the service account for app because in `installation` mode `mpdev` won't create the service account.

  ```bash
  kubectl create ns ${YBA_NAMESPACE}
  kubectl apply -f https://raw.githubusercontent.com/yugabyte/charts/master/rbac/yugabyte-platform-universe-management-sa.yaml -n ${YBA_NAMESPACE}
  curl -s https://raw.githubusercontent.com/yugabyte/charts/master/rbac/platform-global.yaml \
  | sed "s/namespace: <SA_NAMESPACE>/namespace: ${YBA_NAMESPACE}"/g \
  | kubectl apply -n ${YBA_NAMESPACE} -f -
  ```

- Install the GKE app in the test cluster.

  ```bash
  mpdev install \
  	--deployer=$REGISTRY/$APP_NAME/deployer:$TAG \
  	--parameters='{"name": "yw-test", "namespace": "yb-platform", "yugaware.serviceAccount": "yugabyte-platform-universe-management"}'
  ```

- Delete the GKE app.

  ```bash
  kubectl delete application yw-test -n ${YBA_NAMESPACE}
  ```

## Verify

Execute the below command to verify the changes.

```bash
mpdev verify --deployer=$REGISTRY/$APP_NAME/deployer:$TAG
```

