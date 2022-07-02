# Lab 2. Kubernetes Engine: Quick Start

## Google Kubernetes Engine (GKE)

[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) (GKE) provides a managed environment for **deploying**, **managing**, and **scaling** your containerized applications using Google infrastructure.

The Kubernetes Engine environment consists of multiple machines (specifically [Compute Engine](https://cloud.google.com/compute)
 instances) grouped to form a [container cluster](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture).

## Commands

The commands to be used are [gcloud CLI](https://cloud.google.com/sdk/gcloud) and [kubectl](https://kubernetes.io/docs/reference/kubectl/).

```bash
# List the active account name
gcloud auth list

# List the project ID
gcloud config list project

# Set the default compute zone to us-central1-a
# - The compute zone is an approximate regional location in which the clusters and their resources live. For example, us-central1-a is a zone in the us-central1 region.
gcloud config set compute/zone us-central1-a
# (Output) Updated property [compute/zone].

# Create a GKE cluster
gcloud container clusters create [CLUSTER-NAME]
# (Output)
# NAME: my-cluster
# LOCATION: us-central1-a
# MASTER_VERSION: 1.22.8-gke.202
# MASTER_IP: XXX.XXX.XXX.XXX
# MACHINE_TYPE: e2-medium
# NODE_VERSION: 1.22.8-gke.202
# NUM_NODES: 3
# STATUS: RUNNING

# Get authentication credentials for the cluster
# - After creating your cluster, you need authentication credentials to interact with it.
gcloud container clusters get-credentials [CLUSTER-NAME]
# (Output)
# Fetching cluster endpoint and auth data.
# kubeconfig entry generated for my-cluster.

# Deploy application to cluster
## Create deployment with kubectl
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
# (Output) deployment.apps/hello-server created

## Expose Kubernetes service
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
# (Output) service/hello-server exposed

## Inspect the hello-server service (List services)
kubectl get service
# (Output) (External IP will show up late)
# NAME             TYPE            CLUSTER-IP      EXTERNAL-IP     PORT(S)           AGE
# hello-server     loadBalancer    XXX.XXX.XXX.XXX    YYY.YYY.YYY.YYY   8080:31991/TCP    65s
# kubernetes       ClusterIP       XXX.XXX.XXX.XXX               433/TCP           5m13s

## Visit the application
curl http://[EXTERNAL-IP]:8080

## Delete the cluster
gcloud container clusters delete [CLUSTER-NAME]
```

![Application Visit Result](https://user-images.githubusercontent.com/42485462/177025808-ac1c5e21-a361-41b6-9ea2-2599f51319d1.png)
