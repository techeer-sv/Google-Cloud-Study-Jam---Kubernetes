# Lab 3 : Orchestrating the Cloud with Kubernetes

📒 [link](https://github.com/techeer-sv/Google-Cloud-Study-Jam-Kubernetes/blob/main/Soobin/Lab3.md)

<br/>

## Overview

- Goal
  - Provision a complete **Kubernetes cluster using Kubernetes Engine**
  - Deploy and manage Docker containers using `kubectl`
  - Break an application into microservices using Kubernetes’ **Deployments and Services**
- Example Application -> [github](https://github.com/kelseyhightower/app)
  - [kelseyhightower/monolith](https://hub.docker.com/r/kelseyhightower/monolith) - Monolith includes auth and hello services.
  - [kelseyhightower/auth](https://hub.docker.com/r/kelseyhightower/auth) - Auth microservice. Generates JWT tokens for authenticated users.
  - [kelseyhightower/hello](https://hub.docker.com/r/kelseyhightower/hello) - Hello microservice. Greets authenticated users.
  - [nginx](https://hub.docker.com/_/nginx) - Frontend to the auth and hello services.

<br/>

Kubernetes can run on many different environments, from **laptops** to **high-availability multi-node clusters**, from **public clouds** to **on-premise deployments**, from **virtual machine** to **bare metal**.

<br/>

## **Setup and requirements**

<br/>

✔️ Set the zone

```bash
gcloud config set compute/zone us-central1-b
```

✔️ Start up a cluster

```bash
gcloud container cluster create io
```

<br/>

## **Task 1. Get the sample code**

<br/>

✔️ Clone the GitHub repository from the Cloud Shell command line

```bash
gsutil cp -r gs://spls/gsp021/* .
```

<br/>

✔️ Change into the directory needed for this lab

```bash
cd orchestrate-with-kubernetes/kubernetes
```

<br/>

✔️ List the files to see what you’re working with

```bash
ls
```

```bash
deployments/  /* Deployment manifests */
  ...
nginx/        /* nginx config files */
  ...
pods/         /* Pod manifests */
  ...
services/     /* Services manifests */
  ...
tls/          /* TLS certificates */
  ...
cleanup.sh    /* Cleanup script */
```

<br/>

## **Task 2. Quick Kubernetes Demo**

<br/>

✔️ Launch a single instace of the nginx container

```bash
kubectl create deployment nginx --image=nginx:1.10.0
```

<br/>

✔️ View the running nginx container

```bash
kubectl get pods
```

<br/>

✔️ Expose nginx container outside of Kubernetes

```bash
kubectl expose deployment nginx --port 80 --type LoadBalancer
```

<br/>

✔️ List our services

```bash
kubectl get services
```

<br/>

✔️ Add the External IP to this command to hit the Nginx container remotely

```bash
curl http://<External IP>:80
```

<br/>

## **Task 3. Pods**

- What is[ **Pods**](https://kubernetes.io/docs/concepts/workloads/pods/)?

  - The **smallest deployable units** of computing that you can create and manage in Kubernetes.
  - Group of one or more containers with **shared storage and network resources**, and a **specification** for how to run the containers.

- What is[ **Volumnes**](https://kubernetes.io/docs/concepts/storage/volumes/)?

  - Data disk that live as long as the pods live, and can be used by the containers in that pod.

- What is[ **Namespace**](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) ?
  - Provides a mechanism for isolating groups of resources within a single cluster.

![Untitled](https://cdn.qwiklabs.com/tzvM5wFnfARnONAXX96nz8OgqOa1ihx6kCk%2BelMakfw%3D)

<br/>

## **Task 4. Creating pods**

<br/>

✔️ Pod configuration file

```bash
cat pods/monolith.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```

- Pod is made up of one container (ths monolith).

- You’re passing a few arguments to our container.

- You’re opening up port 80 for http traffic.

<br/>

✔️ Create the monolith

```bash
kubectl create -f pods/monolith.yaml
```

<br/>

✔️ Examine your pods. (Use the kubectl get pods command to list all pods running in the default namespace)

```bash
kubectl get pods
```

<br/>

✔️ Get more information about the monolith pod

```bash
kubectl describe pods monolith
```

<br/>

## **Task 5. Interacting with pods**

By default, pods allocated a private IP address and cannot be reached outside of the cluster. Map a local port to a port inside the monolith pod

<br/>

◾️ **2nd terminal**

✔️ Set up port-forwarding

```bash
kubectl port-forward monolith 10080:80
```

<br/>

◾️ **1nd terminal**

✔️ Start talking to your pod

```bash
curl http://127.0.0.1:10080

```

<br/>

✔️ See what heppens when you hit a secure endpoint

```bash
curl http://127.0.0.1:10080/secure
```

<br/>

✔️ Logging in to get an auth token back from the monolith

```bash
curl -u user http://127.0.0.1:10080/login
```

<br/>

✔️ Create an environment variable for the token (Since Cloud Shell does not handle copying long strings well, create an environment variable for the token.)

```bash
TOKEN=$(curl
http://127.0.0.1:10080/login -u
user|jq -r '.token')
```

<br/>

✔️ Use the token to hit secure endpoint

```bash
curl -H "Authorization: Bearer
$TOKEN" http://127.0.0.1:10080/secure
```

<br/>

✔️ View the logs for the monolith Pod

```bash
kubectl logs monolith
```

<br/>

◾️ **3rd terminal**

✔️ View the logs for the monolith Pod

```bash
kubectl logs monolith
```

Get a stream of the logs happening in real-time

```bash
kubectl logs -f monolith
```

<br/>

✔️ In the **1st teminal** to interact with the monolith, you can see the logs updating(in the 3rd terminal):

```bash
curl http://127.0.0.1:10080
```

<br/>

✔️ Run an interactive shell inside the Monolith Pod. (This can come in handy when you want to troubleshoot from within a container)

```bash
kubectl exec monolith --stdin --tty
-c monolith -- /bin/sh
```

<br/>

✔️ Once you have a shell into the monolith container you can test external connectivity

```bash
ping -c 3 google.com
```

<br/>

Log out when done you’re done with interactive shell

```bash
exit
```

Kubectl make it easy to interacting with pods. If you need to hit a container remotely.

<br/>

## **Task 6. Services**

What is[ **Services**](https://kubernetes.io/docs/concepts/services-networking/service/)?

- An abstract way to expose an application running on a set of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
   as a network service.
- Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.

![services](https://cdn.qwiklabs.com/Jg0T%2F326ASwqeD1vAUPBWH5w1D%2F0oZn6z5mQ5MubwL8%3D)

<br/>

`Cluster IP (internal)`

- The default type means that this Service is only visible inside of the cluster

`NodePort`

- NodePort gives each node in the cluster an externally accessible IP and

`LoadBalancer`

- Adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

#### Ref

- [node port vs load balancer vs ingress](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)

<br/>

## **Task 7. Creating a service**

Create a secure pod that can handle https traffic.

✔️ If you’ve changed directories, make sure you return to the `~/orchestrate-with-kubernetes/kubernetes`
 directory

```bash
cd ~/orchestrate-with-kubernetes/kubernetes
```

<br/>

✔️ Explore the monolith service configuration file

```bash
cat pods/secure-monolith.yaml
```

<br/>

✔️ Creat the secure-monolith pods and their configuration data

```bash
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
```

<br/>

✔️ Expose the secure-monolith Pod externally.

To do that, create a Kubernetes service.

```bash
cat services/monolith.yaml
```

```bash
kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
```

✔️ Create the monolith service rom the monolith service configuration file

```bash
kubectl create -f services/monolith.yaml
```

<br/>

✔️ Allow traffic to the monolith service on the exposed nodeport

```bash
gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
```

<br/>

✔️ Get an external IP address for one of the nodes

```bash
gcloud compute instances list
```

<br/>

✔️ Hitting the secure-monolith services using `curl`

```bash
curl -k https://<EXTERNAL_IP>:31000
```

Uh oh! that timed out.

<br/>

## **Task 8. Adding labels to pods**

<br/>

✔️ See that you have quite a few pods running with the monolith label.

```bash
kubectl get pods -l "app=monolith"
```

<br/>

✔️ But what about “app=monolith” and “secure-enabled”?

```bash
kubectl get pods -l "app=monolith,secure=enabled"
```

<br/>

✔️ Add the missing secure=enavled label to the secure-monolith Pod.

```bash
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
```

<br/>

Now that pods are correctly labeled.

✔️ View the list of endpoints on the monolith service

```bash
kubectl describe services monolith | grep Endpoints
```

<br/>

✔️ Test this out by hitting one of our nodes again

```bash
gcloud compute instances list
curl -k https://<EXTERNAL_IP>:31000
```

<br/>

## **Task 9. Deploying applications with Kubernetes**

- What is [ **Deployments** ](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#what-is-a-deployment) ?

  - Declarative way to ensure that the number of Pods running is equal to the desired number of Pods, specified by the user.

- What is Replica [ **Sets** ](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)?
  - Maintain stable set of replica Pods running at any given time.
  - It often used to guarantee the avaliability of a specified number of identical Pods.

![deployments](https://cdn.qwiklabs.com/1UD7MTP0ZxwecE%2F64MJSNOP8QB7sU9rTI0PSv08OVz0%3D)

<br/>

## **Task 10. Creating a deployments**

Monolith App with three separate pieces

- **auth** : Generates JWT tokens for authenticated users.
- **hello** : Greet authenticated users.
- **frontend** : Routes trafic to the auth and hello services

<br/>

✔️ Deployment configuration file

```bash
cat deployments/auth.yaml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  selector:
    matchlabels:
      app: auth
  replicas: 1
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:2.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
...
```

<br/>

✔️ Create deployment object

```bash
kubectl create -f deployments/auth.yaml
```

<br/>

✔️ Create a service for your auth deployment

```bash
kubectl create -f services/auth.yaml
```

<br/>

✔️ Create and Expose deployment

hello deployment

```bash
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
```

frontend deployment

```bash
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```

<br/>
✔️ Interact with the frontend by grabbing its External IP and then curling to it

```bash
kubectl get services frontend
```

```bash
curl -k https://<EXTERNAL-IP>
```

<br/>

## Summary

- Provision a complete Kubernetes cluster using Kubernetes Engine
- Deploy and manage Docker containers using `kubectl`
- Break an application into microservices using Kubernetes’ Deployments and a Services

<br/>
