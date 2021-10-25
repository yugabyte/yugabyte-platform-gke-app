# Overview

Yugabyte Platform gives you the simplicity and support to deliver a private
database-as-a-service (DBaaS) at scale. Use Yugabyte Platform to deploy
YugabyteDB across any cloud anywhere in the world with a few clicks, simplify
day 2 operations through automation, and get the services needed to realize
business outcomes with the database.

## About Google Click to Deploy

Popular open stacks on Kubernetes packaged by Google.

## Overview

https://docs.yugabyte.com/latest/yugabyte-platform/overview/

# Installation

## Quick install with Google Cloud Marketplace

Get up and running with a few clicks! Install this Yugabyte Platform app to a
Google Kubernetes Engine cluster using Google Cloud Marketplace. Follow the
[on-screen instructions]().

## Command line instructions

You can use [Google Cloud Shell](https://cloud.google.com/shell/) or a local
workstation to complete these steps.

[![Open in Cloud Shell](http://gstatic.com/cloudssh/images/open-btn.svg)]()

### Prerequisites

#### Set up command-line tools

You'll need the following tools in your development environment. If you are
using Cloud Shell, `gcloud`, `kubectl`, Docker, and Git are installed in your
environment by default.

-   [gcloud](https://cloud.google.com/sdk/gcloud/)
-   [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
-   [docker](https://docs.docker.com/install/)
-   [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
-   [helm](https://helm.sh/)

Configure `gcloud` as a Docker credential helper:

```shell
gcloud auth configure-docker
```

#### Create a Google Kubernetes Engine cluster

Create a new cluster from the command line:

```shell
export CLUSTER=yugabyte-cluster
export ZONE=us-east1-a

gcloud container clusters create "$CLUSTER" --zone "$ZONE"
```

Configure `kubectl` to connect to the new cluster:

```shell
gcloud container clusters get-credentials "$CLUSTER" --zone "$ZONE"
```

#### Clone this repo

Clone this repo and the associated tools repo.

```shell
git clone --recursive https://github.com/GoogleCloudPlatform/click-to-deploy.git
```

#### Install the Application resource definition

An Application resource is a collection of individual Kubernetes components,
such as Services, Deployments, and so on, that you can manage as a group.

To set up your cluster to understand Application resources, run the following
command:

```shell
kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
```

You need to run this command once.

The Application resource is defined by the [Kubernetes
SIG-apps](https://github.com/kubernetes/community/tree/master/sig-apps)
community. The source code can be found on
[github.com/kubernetes-sigs/application](https://github.com/kubernetes-sigs/application).

### Install the Application

Navigate to the `yugabyte-platform-gke-app` directory.

```shell
cd click-to-deploy/k8s/yugabyte-platform
```

#### Configure the app with environment variables

Choose an instance name and namespace for the app.

```shell
export APP_INSTANCE_NAME=yugabyte-platform
export NAMESPACE=test-ns
```

Configure the container images.

```shell
export TAG=0.1.6-20211025
export IMAGE_YUGABYTE_PLATFORM_REPO="gcr.io/yugabyte-pcf/yugabyte-platform"
```

#### Create namespace in your Kubernetes cluster

If you use a different namespace than `test-ns`, run the command below to create
a new namespace:

```shell
kubectl create namespace "$NAMESPACE"
```

#### Create the Service Account

Follow the [link](https://docs.yugabyte.com/latest/yugabyte-platform/configure-yugabyte-platform/set-up-cloud-provider/kubernetes/) to know more.

#### Expand the manifest template

Use `helm template` to expand the template. We recommend that you save the
expanded manifest file for future updates to the application.

```shell
helm template "${APP_INSTANCE_NAME}" chart/yugaware 
  --namespace "${NAMESPACE}" \
  --set image.commonRegistry="${IMAGE_YUGABYTE_PLATFORM_REPO}" \
  --set image.tag="${TAG}" \
  --set image.postgres.tag="${TAG}" \
  --set image.prometheus.name="prometheus" \
  --set image.prometheus-tag="${TAG}" \
  --set image.nginx.name="nginx" \
  --set image.nginx.tag="${TAG}" \
  --set yugaware.serviceAccount="${SERVICE_ACCOUNT}" \
  > "${APP_INSTANCE_NAME}_manifest.yaml"
```

#### Apply the manifest to your Kubernetes cluster
Use kubectl to apply the manifest to your Kubernetes cluster. This installation creates:

- An Application resource, which collects all the deployment resources into one logical entity.
- A PersistentVolume and PersistentVolumeClaim. If an installation is deleted, the volume is not deleted with it. If you delete an installation, then recreate it with the same name, the new installation will use the same PersistentVolume. As a result, there will be no app initialization, and the old configuration will be used.
- A StatefulSet.
- A Services, which expose Yugabyte Platform UI on 80 for non-TLS and 443 for TLS).

```shell
kubectl apply -f "${APP_INSTANCE_NAME}_manifest.yaml" --namespace "${NAMESPACE}"
```

#### View the app in the Google Cloud Console

To get the Console URL for your app, run the following command:

```shell
echo "https://console.cloud.google.com/kubernetes/application/${ZONE}/${CLUSTER}/${NAMESPACE}/${APP_INSTANCE_NAME}"
```

To view your app, open the URL in your browser.

# Access the Yugabyte Platform UI

Get the external endpoint from application page in the Google Cloud Console.

# Application metrics

# Scaling

# Backup and restore

## Restore the database

# Upgrading the app

# Uninstalling the Application

## Using the Google Cloud Platform Console

## Using the command line

### Delete the Yugabyte Platform App

```
kubectl delete -f ${APP_INSTANCE_NAME} --namespace $NAMESPACE
```

### Delete the GKE cluster

Optionally, if you don't need the deployed application or the GKE cluster,
delete the cluster using this command:

```shell
gcloud container clusters delete "${CLUSTER}" --zone "${ZONE}"
```
