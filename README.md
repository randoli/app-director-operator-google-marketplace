# Overview

App Director is a platform engineering solution on Kubernetes for the complex cloud native landscape.

Reduces the burden on DevOps by automating repetitive tasks, while providing self-service to accelerate developers.

For more information, visit
[App Director official website](https://randoli.ca/app-director/).

## About Google Click to Deploy

Popular open stacks on Kubernetes, packaged by Google.

# Installation

## Quick install with Google Cloud Marketplace

Get up and running with a few clicks! To install this App Director Operator app to a
Google Kubernetes Engine cluster via Google Cloud Marketplace, follow the
on-screen instructions(tbt).

## Command-line instructions

### Prerequisites

#### Set up command-line tools

You'll need the following tools in your development environment. If you are
using Cloud Shell, then `gcloud`, `kubectl`, Docker, and Git are installed in
your environment by default.

- [gcloud](https://cloud.google.com/sdk/gcloud/)
- [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)
- [docker](https://docs.docker.com/install/)
- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [helm](https://helm.sh/)
- [envsubst](https://command-not-found.com/envsubst)

Configure `gcloud` as a Docker credential helper:

```shell
gcloud auth configure-docker
```

#### Create a Google Kubernetes Engine (GKE) cluster

Create a new cluster from the command-line:

```shell
export CLUSTER=app-director-operator-cluster
export ZONE=us-west1-a

gcloud container clusters create "${CLUSTER}" --zone "${ZONE}"
```

Configure `kubectl` to connect to the new cluster:

```shell
gcloud container clusters get-credentials "${CLUSTER}" --zone "${ZONE}"
```

#### Clone this repo

Clone this repo, and its associated tools repo:

```shell
git clone --recursive https://github.com/randoli/app-director-operator-helm-chart.git
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

The Application resource is defined by the
[Kubernetes SIG-apps](https://github.com/kubernetes/community/tree/master/sig-apps)
community. You can find the source code at
[github.com/kubernetes-sigs/application](https://github.com/kubernetes-sigs/application).

#### Configure the app with environment variables

Choose an instance name and
[namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
for the app. In most cases, you can use the `default` namespace.

```shell
export APP_INSTANCE_NAME=app-director-operator-1
export NAMESPACE=app-director
```

Set up the image tag:

It is advised to use a stable image reference, which you can find on
[Marketplace Container Registry](https://marketplace.gcr.io).
For example:

```shell
export TAG="1.0.1"
```

Configure the container image:

```shell
export OPERATOR_IMAGE="northamerica-northeast2-docker.pkg.dev/randoli-public/app-director/app-director-operator"
export KUBE_RBAC_PROXY_IMAGE="northamerica-northeast2-docker.pkg.dev/randoli-public/app-director/kube-rbac-proxy:${TAG}"
```

#### Create namespace in your Kubernetes cluster

If you use a different namespace than the `default`, run the following
command to create a new namespace:

```shell
kubectl create namespace "${NAMESPACE}"
```

#### Expand the manifest template

Use `helm template` to expand the template. We recommend that you save the
expanded manifest file for future updates to your app.

```shell
helm template "${APP_INSTANCE_NAME}" chart/app-director-operator \
  --namespace "${NAMESPACE}" \
  --set image.repository="${OPERATOR_IMAGE}" \
  --set image.tag="${TAG}" \
  --set kubeRbacProxy.image="${KUBE_RBAC_PROXY_IMAGE}" \
  > "${APP_INSTANCE_NAME}_manifest.yaml"
```

#### Apply the manifest to your Kubernetes cluster

Use `kubectl` to apply the manifest to your Kubernetes cluster:

```shell
kubectl apply -f "${APP_INSTANCE_NAME}_manifest.yaml" --namespace "${NAMESPACE}"
```

#### View the app in the Google Cloud Console

To get the Cloud Console URL for your app, run the following command:

```shell
echo "https://console.cloud.google.com/kubernetes/application/${ZONE}/${CLUSTER}/${NAMESPACE}/${APP_INSTANCE_NAME}"
```

To view the app, open the URL in your browser.

# Deploy an Agent

To deploy an Agent see (How to deploy an Agent in GKE)[https://docs.appdir.randolicloud.ca/how-to/deploy-agent/gke]

# Upgrading the app

To update your deployment of App Director Operator, refer to the
[official documentation](https://docs.appdir.randolicloud.ca/how-to/update-operator)

# Uninstall the app

## Using the Google Cloud Console

1. In the Cloud Console, open
   [Kubernetes Applications](https://console.cloud.google.com/kubernetes/application).

1. From the list of apps, click **App Director Operator**.

1. On the Application Details page, click **Delete**.

## Using the command-line

### Prepare your environment

Set your installation name and Kubernetes namespace:

```shell
export APP_INSTANCE_NAME=app-director-operator-1
export NAMESPACE=app-director
```

### Delete the resources

> **NOTE:** We recommend that you use a `kubectl` version that is the same
> version as that of your cluster. Using the same versions for `kubectl` and
> the cluster helps to avoid unforeseen issues.

To delete the resources, use the expanded manifest file used for the
installation.

Run `kubectl` on the expanded manifest file:

```shell
kubectl delete -f ${APP_INSTANCE_NAME}_manifest.yaml --namespace ${NAMESPACE}
```

You can also delete the resources by using types and a label:

```shell
kubectl delete application \
  --namespace ${NAMESPACE} \
  --selector app.kubernetes.io/name=${APP_INSTANCE_NAME}
```

### Delete the GKE cluster

If you don't need the deployed app or the GKE cluster, delete the cluster
by using this command:

```shell
gcloud container clusters delete "${CLUSTER}" --zone "${ZONE}"
```
