# Lab 3 : Orchestrating the Cloud with Kubernetes

📒 [link](https://www.cloudskillsboost.google/focuses/557?parent=catalog))

<br/>

## **Overview**

- kubernetes
- Google Kubernetes Engine

**Example Application**

```bash
Note: App is hosted on GitHub and provides an example 12-Factor application.
During this lab you will be working with the following Docker images:

- kelseyhightower/monolith - Monolith includes auth and hello services.
- kelseyhightower/auth - Auth microservice. Generates JWT tokens for authenticated users.
- kelseyhightower/hello - Hello microservice. Greets authenticated users.
- nginx - Frontend to the auth and hello services.
```

<br/>

## **Task 1. Get the sample code**

Clone the GitHub repository from the Cloud Shell command line

```bash
gsutil cp -r gs://spls/gsp021/* .
```

Change into the directory needed for this lab

```bash
cd orchestrate-with-kubernetes/kubernetes
```

List the files to see what you’re working with

```bash
ls
```

<br/>

✨ The sample has the following layout:

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

Use it to launch a single instace of the nginx container:

```bash
kubectl create deployment nginx --image=nginx:1.10.0
```

Use the `kubectl get pods` command to view the running nginx container

```bash
kubectl get pods
```

Once the nginx container has a Running status you can expose it outside of Kubernetes using the `kubectl expose` command

```bash
kubectl expose deployment nginx --port 80 --type LoadBalancer
```

List our services now using the `kubectl get services` command:

```bash
kubectl get services
```

Add the External IP to this command to hit the Nginx container remotely

```bash
curl http://<External IP>:80
```

<br/>

## **Task 3. Pods**

What is **Pods**?

- The smallest deployable units of computing that you can create and manage in Kubernetes.
- Group of one or more containers with shared storage and network resources, and a specification for how to run the containers.

What is **Volumnes**?

- Data disk that live as long as the pods live, and can be used by the containers in that pod.

What is **Namespace** ?

<br/>

## **Task 4. Creating pods**

Pod configuration file

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

✔️ Pod is made up of one container (ths monolith).

✔️ You’re passing a few arguments to our container.

✔️ You’re opening up port 80 for http traffic.

<br/>

Create the monolith pod using `kubectl`

```bash
kubectl create -f pods/monolith.yaml
```

Examine your pods. Use the kubectl get pods command to list all pods running in the default namespace:

```bash
kubectl get pods
```

Once the pod is running, use `kubectl describe` command to get more information about the monolith pod

```bash
kubectl describe pods monolith
```

<br/>

## **Task 5. Interacting with pods**

By default, pods allocated a private IP address and cannot be reached outside of the cluster. Use the kybectl `port-forward` command to map a local port to a port inside the monolith pod

<br/>

✔️ **2nd terminal**

```bash
kubectl port-forward monolith 10080:80
```

<br/>

✔️ **1nd terminal**

“hello” back from your container.

```bash
curl http://127.0.0.1:10080

```

See what heppens when you hit a secure endpoint

```bash
curl http://127.0.0.1:10080/secure
```

Logging in to get an auth token back from the monolith

```bash
curl -u user http://127.0.0.1:10080/login
```

Create an environment variable for the token

```bash
TOKEN=$(curl
http://127.0.0.1:10080/login -u
user|jq -r '.token')
```

Use the token to hit secure endpoint with `curl`

```bash
curl -H "Authorization: Bearer
$TOKEN" http://127.0.0.1:10080/secure
```

Use the kubectl logs command to view the logs for the monolith Pod

```bash
kubectl logs monolith
```

✔️ **3rd terminal**

Get a stream of the logs happening in real-time

```bash
kubectl logs -f monolith
```

✔️ In the **1st teminal** to interact with the monolith, you can see the logs updating(in the 3rd terminal):

```bash
curl http://127.0.0.1:10080
```

Run an interactive shell inside the Monolith Pod.

```bash
kubectl exec monolith --stdin --tty
-c monolith -- /bin/sh
```

Once you have a shell into the monolith container you can test external connectivity using the `ping` command

```bash
ping -c 3 google.com
```

Log out when done you’re done with interactive shell

```bash
exit
```

Summary → kubectl make it easy to interacting with pods. If you need to hit a container remotely.

<br/>

## **Task 6. Services**

What is **Services**?

- An abstract way to expose an application running on a set of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
   as a network service.
- Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.

`Cluster IP (internal)`

- The default type means that this Service is only visible inside of the cluster

`NodePort`

- NodePort gives each node in the cluster an externally accessible IP and

`LoadBalancer`

- Adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

<br/>

## **Task 7. Creating a service**

Create a secure pod that can handle https traffic.

✔️ If you’ve changed directories, make sure you return to the `~/orchestrate-with-kubernetes/kubernetes`
 directory

        ```bash
        cd ~/orchestrate-with-kubernetes/kubernetes
        ```

✔️ Explore the monolith service configuration file

    ```bash
    cat pods/secure-monolith.yaml
    ```

✔️ Creat the secure-monolith pods and their configuration data

```bash
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
```

Expose the secure-monolith Pod externally.

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

```bash
kubectl create -f services/monolith.yaml
```

```bash
service/monolith created
```

Allow traffic to the monolith service on the exposed nodeport

```bash
gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
```

Get an external IP address for one of the nodes

```bash
gcloud compute instances list
```

Hitting the secure-monolith services using `curl`

```bash
curl -k https://<EXTERNAL_IP>:31000
```

<br/>

## **Task 8. Adding labels to pods**

You can see that you have quite a few pods running with the monolith label.

```bash
kubectl get pods -l "app=monolith"
```

But what about “app=monolith” and “secure-enabled”?

```bash
kubectl get pods -l "app=monolith,secure=enabled"
```

Use the kubectl label command to add the missing secure=enavled label to the secure-monolith Pod.

```bash
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
```

View the list of endpoints on the monolith service

```bash
kubectl describe services monolith | grep Endpoints
```

Test this out by hitting one of our nodes again

```bash
gcloud compute instances list
curl -k https://<EXTERNAL_IP>:31000
```

<br/>

## **Task 9. Deploying applications with Kubernetes**

What is Deployments ?

- Declarative way to ensure that the number of Pods running is equal to the desired number of Pods, specified by the user.

<br/>

## **Task 10. Creating a deployments**

Monolith App with three separate pieces

- **auth** : Generates JWT tokens for authenticated users.
- **hello** : Greet authenticated users.
- **frontend** : Routes trafic to the auth and hello services

✔️ Deployment configuration file

```bash
cat deployments/auth.yaml
```

✔️ Create deployment object

```bash
kubectl create -f deployments/auth.yaml
```

✔️ Create a service for your auth deployment

```bash
kubectl create -f services/auth.yaml
```

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

#### Ref

- [node port vs load balancer vs ingress](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)
