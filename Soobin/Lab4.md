# Lab 4 : Managing Deployments Using Kubernetes Engine

📒 [link](https://www.cloudskillsboost.google/focuses/639?parent=catalog)

<br/>

## **Overview**

- Goal
  - Practice in scaling and managing containers so you can accomplish these sommon scenarios where multiple heterogeous deployments are being used.
- Deployment scenarios
  - Continuous Deployment
  - Blue-Green Deployments
  - Canary Deployments
- What you’ll do
  - Practice with kybectl tool
  - Create deployment yaml files
  - Lanch, update, and scale deployments
  - Practice with updating deployments and deployment styles

<br/>

## **Introduction to deployments**

- What is **Heterogeneous deployment**

  - Typically involve connecting two or more distinct infrastructure environments or regions to address a specific technical or operarion need.
  - Heterogeneout Deployments are called “hybrid”, “multi-cloud”, or “public-private”, depending upon the specifics of the deployment.
  - Hetetogeneous deployments include those tht span regions within a single cloud environment, multiple public cloud envieonments (multi-cloud), or a combination of on-preises and public cloud environments (hybrid or public-private).

- Single environment or region’s business and technical challenges
  - **Maxed out resources**: In any single environment, particularly in on-premises environments, you might not have the compute, networking, and storage resources to meet your production needs.
  - **Limited geographic reach**: Deployments in a single environment require people who are geographically distant from one another to access one deployment. Their traffic might travel around the world to a central location.
  - **Limited availability**: Web-scale traffic patterns challenge applications to remain fault-tolerant and resilient.
  - **Vendor lock-in**: Vendor-level platform and infrastructure abstractions can prevent you from porting applications.
  - **Inflexible resources**: Your resources might be limited to a particular set of compute, storage, or networking offerings.

H.P can help address these challenges, but they must be architected using programmatic and deterministic processes and procedures. One-off or ad-hoc deployment procedures can cause deployments or processes to be brittle and intolerant of failures

- Three common scenarios for heterogeneous deployment
  - multi-cloud deployments
  - fronting on-premises data
  - continuous

**Ref**

- [https://enoch-kim.github.io/k8s-heterogeneous-deploy/](https://enoch-kim.github.io/k8s-heterogeneous-deploy/)

<br/>

## **Setup**

✔️ Set zone

```bash
gcloud config set compute/zone us-central1-a
```

✔️ Get sample code

```bash
gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
cd orchestrate-with-kubernetes/kubernetes
```

```bash
gcloud container clusters create bootcamp --num-nodes 5 --scopes
"https://www.googleapis.com/auth/projecthosting,storage-rw"
```

## **Learn about the deployment object**

✔️ Take a look at the Deployment object.

```bash
kubectl explain deployment
```

See all of the fields using the `—recursive` option

```bash
kubectl explain deployment --recursive
```

Go through the lab to help you understand the structure of a Deployment object and understatnd what the individual fields do.

```bash
kubectl explain deployment.metadata.name
```

<br/>

## **Create a deployment**

✔️ Update the deployments/auth.yaml configuration file

```bash
vi deployments/auth.yaml
```

Change the image in the containers section of the Deployment to the following

```bash
...
containers:
- name: auth
  image: "kelseyhightower/auth:1.0.0"
...
```

✔️ Create a simple deployment

```bash
cat deployments/auth.yaml
```

(Output)

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
...
```

When you run the `kubectl create` command to create the auth deployment, it will make one pod that conforms to the data in the Deployment manifest. This means we can scale the number of Pods by changing the number specified in the `replicas` field.

Create deployment

```bash
kubectl create -f deployments/auth.yaml
```

Verify that deployment was created

```bash
kubectl get deployments
```

Verify that a ReplicaSet was created for Deployment

```bash
kubectl get replicasets
```

✔️ View the Pods that were created as part of our Deployment.

The single Pod is created by the Kubernetes when the ReplicaSet is created

```bash
kubectl get pods
```

✔️ Create service for our auth deployment.

```bash
kubectl create -f services/auth.yaml
```

✔️ Create and expose the deployment

```bash
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
```

```bash
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```

- What is [\*\*Configmap](https://kubernetes.io/docs/concepts/configuration/configmap/) ?\*\*
  - API object used to store non-confidential data in key-value pairs.
- What is [**Secrets**](https://kubernetes.io/docs/concepts/configuration/secret/) ?
  - Object that contains a small amount of sensitive data such as a password, a token, or a key.

✔️ Interact with the frontend by grabbing its external IP and then curling to it

```bash
kubectl get services frontend
```

```bash
curl -ks https://<EXTERNAL-IP>
```

Use the output templating feature

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath=
"{.status.loadBalancer.ingress[0].ip}"`
```

✔️ Scale a Deployment

```bash
kubectl scale deployment hello --replicas=5
```

Verify that there are now 5 hello Pods running

```bash
kubectl get pods | grep hello- | wc -l
```

Scale back the application

```bash
kubectl scale deployment hello --replicas=3
```

Verify that you have the corrext number of Pods

```bash
kubectl get pods | grep hello- | wc -l
```

<br/>

## **Rolling update**

```bash
Deployments support updating images to a new version through a
rolling update mechanism. When a Deployment is updated with a new version,
it creates a new ReplicaSet and slowly increases the number of replicas
in the new ReplicaSet as it decreases the replicas in the old ReplicaSet.
```

![https://cdn.qwiklabs.com/uc6D9jQ5Blkv8wf%2FccEcT35LyfKDHz7kFpsI4oHUmb0%3D](https://cdn.qwiklabs.com/uc6D9jQ5Blkv8wf%2FccEcT35LyfKDHz7kFpsI4oHUmb0%3D)

✔️ Trigger a rolling update

Update Deployment

```bash
kubectl edit deployment hello
```

Change the image in the containers section of the Deployment to the following

```bash
...
containers:
  image: kelseyhightower/hello:2.0.0
...
```

See the new ReplicaSet

```bash
kubectl get replicaset
```

See a new entry in the rollout history

```bash
kubectl rollout history deployment/hello
```

✔️ Pause a rolling update

If you detect problems with a running rollout, pause it to stop the update.

```bash
kubectl rollout pause deployment/hello
```

Verify the current state of the rollout

```bash
kubectl rollout status deployment/hello
```

Verify this on the Pods dirextly

```bash
kubectl get pods -o jsonpath --template='{range .items[*]}
{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

✔️ Resume a rolling update

```bash
kubectl rollout resume deployment/hello
```

```bash
kubectl rollout status deployment/hello
```

✔️ Rollback an update

Roll back to the previous version

```bash
kubectl rollout undo deployment/hello
```

Verify the rollback in the history

```bash
kubectl rollout history deployment/hello
```

Verify that all the Pods have rolled back to their previous versions

```bash
kubectl get pods -o jsonpath --template='{range .items[*]}
{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

<br/>

## **Canary deployments**

Canary deployments allow you to release a change to a small subset of your users to mitigate risk associated with new releases.

A canary deployment consists of a separate deployment with your new version and a service that targets both your normal, stable deployment as well as your canary deployment.

![https://cdn.qwiklabs.com/qSrgIP5FyWKEbwOk3PMPAALJtQoJoEpgJMVwauZaZow%3D](https://cdn.qwiklabs.com/qSrgIP5FyWKEbwOk3PMPAALJtQoJoEpgJMVwauZaZow%3D)

✔️ Create a canary deployment

```bash
cat deployments/hello-canary.yaml
```

(output)

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: canary
        # Use ver 2.0.0 so it matches version on service selector
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
...
```

```bash
kubectl create -f deployments/hello-canary.yaml
```

```bash
kubectl get deployments
```

✔️ Verify the canary deployment

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath=
"{.status.loadBalancer.ingress[0].ip}"`/version
```

✔️ Canary deployments in production - session affinity

All clients with the same IP address will have their requests sent to the same version of the `hello`
 application.

```bash
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  sessionAffinity: ClientIP
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
```

<br/>

## **Blue-green deployments**

Use your existing `hello` deployment for the "blue" version. The deployments will be accessed via a Service which will act as the router. Once the new "green" version is up and running, you'll switch over to using that version by updating the Service.

![https://cdn.qwiklabs.com/POW8Q247ZKNY%2ByHIartCsoEu8MAih7k4u1twusCx6pw%3D](https://cdn.qwiklabs.com/POW8Q247ZKNY%2ByHIartCsoEu8MAih7k4u1twusCx6pw%3D)

✔️ The service

Update the service

```bash
kubectl apply -f services/hello-blue.yaml
```

✔️ Updating using Blue-Green Deployment

Create a “green” deployment for new version

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
```

Create the green deployment

```bash
kubectl create -f deployments/hello-green.yaml
```

Verify thT the srrent version of 1.0.0 is still being used

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath=
"{.status.loadBalancer.ingress[0].ip}"`/version
```

Update the service to point to the new version

```bash
kubectl apply -f services/hello-green.yaml
```

Verify that the new version is always being used

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath=
"{.status.loadBalancer.ingress[0].ip}"`/version
```

✔️ Blue-Green Rollback

You can rollback to the old version in the same way. While the ‘blue” deployment is still running, just update the service back to the old version.

```bash
kubectl apply -f services/hello-blue.yaml
```

Verify that the right version is now being used

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath=
"{.status.loadBalancer.ingress[0].ip}"`/version
```
