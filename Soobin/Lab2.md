# Lab 2 : Kub(ernetes Engine: Quik Start

📒 [link](https://www.cloudskillsboost.google/focuses/878?parent=catalog)

<br/>

## **Overview**

- What is Google Kubernetes Engine(GKE)
  - Google Kubernetes Engine (GKE) **provides a managed environment for
    deploying, managing, and scaling your containerized applications**
    using Google infrastructure.
  - The Kubernetes Engine environment **consists of multiple machines**
    (specifically Compute Engine instances) grouped to form a container cluster.
- Benefit of Kubernetes on Google Cloud
  - [Load balancing](https://cloud.google.com/compute/docs/load-balancing-and-autoscaling) for Compute Engine instances
  - [Node pools](https://cloud.google.com/kubernetes-engine/docs/node-pools) to designate subsets of nodes within a cluster for additional flexibility
  - [Automatic scaling](https://cloud.google.com/kubernetes-engine/docs/cluster-autoscaler) of your cluster's node instance count
  - [Automatic upgrades](https://cloud.google.com/kubernetes-engine/docs/node-auto-upgrade) for your cluster's node software
  - [Node auto-repair](https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-repair) to maintain node health and availability
  - [Logging and Monitoring](https://cloud.google.com/kubernetes-engine/docs/how-to/logging) with Cloud Monitoring for visibility into your cluster

<br/>

## **Task 1. Set a default compute zone**

```bash
gcloud config set compute/zone us-central1-a
```

<br/>

## **Task 2. Create a GKE cluster**

```bash
gcloud container clusters create [CLUSTER-NAME]
```

<br/>

## **Task 3. Get authentication credentials for the cluster**

```bash
gcloud container clusters get-credentials [CLUSTER-NAME]
```

<br/>

## **Task 4. Deploy an application to the cluster**

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
  - Provides declarative updates for Pods and ReplicaSets.
- [**Services**](https://kubernetes.io/docs/concepts/services-networking/service/)
  - Abstrac way to expose an application running on a set of Pods as a network service.

<br/>

✔️ Create a new Deployment

```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```

<br/>
✔️ Create a Kubernetes Service

```bash
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```

<br/>

✔️ Inspect the Service

```bash
kubectl get service
```

<br/>

✔️ View the application from your web browser, open a new tab and enter the following address.

```bash
http://[EXTERNAL-IP]:8080
```

<br/>

## **Task 5. Deleting the cluster**

```bash
gcloud container clusters delete [CLUSTER-NAME]
```

<br/>

## Summary

```bash
Get hands-on practice with container creation and application deployment with GKE.
```
