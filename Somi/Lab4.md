# Lab4 : Managing Deployments Using Kubernetes Engine

Goal

- provide practice in sacling and managing containers so accomplish common scenarios where multiple heterogeneous deployments are being used.

<br>

# **What you'll do**

- Practice with kubectl tool
- Create deployment yaml files
- Launch, update, and scale deployments
- Practice with updating deployments and deployment styles

heterogeneous deployments include those that span regions within a single cloud environment, multiple public cloud environments (multi-cloud), or a combination of on-premises and public cloud environments (hybrid or public-private).

Heterogeneous deployments can help address these challenges, but they must be architected using programmatic and deterministic processes and procedures. One-off or ad-hoc deployment procedures can cause deployments or processes to be brittle and intolerant of failures. Ad-hoc processes can lose data or drop traffic. Good deployment processes must be repeatable and use proven approaches for managing provisioning, configuration, and maintenance.

Sample code 얻기

```bash
gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
cd orchestrate-with-kubernetes/kubernetes
```

```bash
gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
```

Create a cluster with five `n1-standard-1` nodes

### Learn about the deployment object

Let's get started with Deployments. First let's take a look at the Deployment object. The `explain`
 command in `kubectl` can tell us about the Deployment object.

```bash
kubectl explain deployment
```

```bash
kubectl explain deployment --recursive
```

`--recursive` option shows all of the fields.

```bash
kubectl explain deployment.metadata.name
```

You can use the explain command as you go through the lab to help you understand the structure of a Deployment object and understand what the individual fields do.

### Create a deployment

```bash
vi deployments/auth.yaml
```

Update the `deployments/auth.yaml` configuration file

Change the image file into version 1.0.0

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e781c13f-0b36-4011-92fb-a2125251746a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174059Z&X-Amz-Expires=86400&X-Amz-Signature=46aefe53af569c9f69d5d431d9d08c5ce24effb69d660f45daa1fb46b383a1ff&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

When you run the `kubectl create`
 command to create the auth deployment, it will make one pod that conforms to the data in the Deployment manifest. This means we can scale the number of Pods by changing the number specified in the `replicas` field.

```bash
kubectl create -f deployments/auth.yaml
```

create your deployment object using `kubectl create`

```bash
kubectl get deployments
```

Once you have created the Deployment, you can verify that it was created.

```bash
kubectl get replicasets
```

Once the deployment is created, Kubernetes will create a ReplicaSet for the Deployment. We can verify that a ReplicaSet was created for our Deployment:

```bash
kubectl get pods
```

Finally, we can view the Pods that were created as part of our Deployment. The single Pod is created by the Kubernetes when the ReplicaSet is created.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0fb6aff4-40e1-47bf-b7f3-ec7ee017caa1/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174158Z&X-Amz-Expires=86400&X-Amz-Signature=9e3eecee02053d31a67a832fed95ed7b30af3f11700697785823009819cee0b3&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

```bash
kubectl create -f services/auth.yaml
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
```

create a service for our auth deployment. Use the `kubectl create` command to create the auth service.

Now, do the same thing to create and expose the hello Deployment.

```bash
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```

And one more time to create and expose the frontend Deployment.

You created a ConfigMap for the frontend.

```bash
kubectl get services frontend
```

Interact with the frontend by grabbing its external IP and then curling to it.

```bash
curl -ks https://<EXTERNAL-IP>
```

And you get the hello response back.

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
```

You can also use the output templating feature of `kubectl` to use curl as a one-liner:

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3d99de52-a062-4754-86ed-5e70940064ab/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174218Z&X-Amz-Expires=86400&X-Amz-Signature=55097cf69573529956c05b63add6e4024f4fa767f70aac0b77c7bc2be0915fa4&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

### Scale a Deployment

```bash
kubectl explain deployment.spec.replicas
```

Now that we have a Deployment created, we can scale it. Do this by updating the `spec.replicas`
 field. You can look at an explanation of this field using the `kubectl explain`command again.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/28619aab-cfac-4c10-838c-b6cd5daade04/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174237Z&X-Amz-Expires=86400&X-Amz-Signature=f6c31803734dae6a5819515ee8f7776db221c96ba78068436fc9561f254c7e40&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

The replicas field can be most easily updated using the `kubectl scale` command:

```bash
kubectl scale deployment hello --replicas=5
```

After the Deployment is updated, Kubernetes will automatically update the associated ReplicaSet and start new Pods to make the total number of Pods equal 5.

```bash
kubectl get pods | grep hello- | wc -l
```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/c645dccb-7928-40c2-90d6-61482c14429c/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174313Z&X-Amz-Expires=86400&X-Amz-Signature=f6b960f2667de0bd0728990561bc192166d839b0773a61eb3658ad6cccb14c25&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

Now scale back the application:

```bash
kubectl scale deployment hello --replicas=3
```

### Rolling update

Deployments support updating images to a new version through a rolling update mechanism. When a Deployment is updated with a new version, it creates a new ReplicaSet and slowly increases the number of replicas in the new ReplicaSet as it decreases the replicas in the old ReplicaSet.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/edff1dd6-391a-46ca-b054-e68cdffedd32/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174350Z&X-Amz-Expires=86400&X-Amz-Signature=6dd92d4dfc5879bb48fab3b89709b6217a7d1d8ab4637915f7b312db7971621e&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

the updated Deployment will be saved to your cluster and Kubernetes will begin a rolling update.

```bash
kubectl rollout history deployment/hello
```

You can also see a new entry in the rollout history

### Pause a rolling update

If you detect problems with a running rollout, pause it to stop the update.

```bash
kubectl rollout pause deployment/hello
```

```bash
kubectl rollout status deployment/hello
```

Verify the current state of the rollout

You can also verify this on the Pods directly:

```bash
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

### Resume a rolling update

The rollout is paused which means that some pods are at the new version and some pods are at the older version. We can continue the rollout using the `resume` command.

```bash
kubectl rollout resume deployment/hello
```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/43636eab-fc32-4e89-a32e-bf45aeb81ac1/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174408Z&X-Amz-Expires=86400&X-Amz-Signature=471403bcd378d44c25276dd3a04154effa5fb7b7b2623989244042dbeb0b4eac&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

```bash
kubectl rollout status deployment/hello
```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/df6ce8eb-6c06-4815-9018-8e2d042d3ffa/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174421Z&X-Amz-Expires=86400&X-Amz-Signature=b20a398921f595ad423e93b94054c58e475e2ca20af7d97cfecced9d5869b9be&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

### Rollback an update

Assume that a bug was detected in your new version. Since the new version is presumed to have problems, any users connected to the new Pods will experience those issues.

You will want to roll back to the previous version so you can investigate and then release a version that is fixed properly.

Use the `rollout` command to roll back to the previous version:

```bash
kubectl rollout undo deployment/hello
```

Verify the roll back in the history:

```bash
kubectl rollout history deployment/hello
```

```bash
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/89822944-2160-4173-90bb-cf17d99fbaa6/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174436Z&X-Amz-Expires=86400&X-Amz-Signature=198ffe1da80cee7f65f6f7515b103ad69cc48157acd775526206d383a3b5f25a&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

## Canary deployments

When you want to test a new deployment in production with a subset of your users, use a canary deployment. Canary deployments allow you to release a change to a small subset of your users to mitigate risk associated with new releases.

### Create a canary deployment

A canary deployment consists of a separate deployment with your new version and a service that targets both your normal, stable deployment as well as your canary deployment

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0f895c5f-95f2-47f2-95dd-e01a5f6fb5e0/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174508Z&X-Amz-Expires=86400&X-Amz-Signature=aeb55bc07f8d4e1d6df3c0b2c1d41a2f5963da16c0d09d8b15a801207dc84b95&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

Create the canary deployment :

```bash
kubectl create -f deployments/hello-canary.yaml
```

After the canary deployment is created, you should have two deployments, `hello` and `hello-canary`. Verify it with this `kubectl` command:

```bash
kubectl get deployments
```

### Verify the canary deployment

You can verify the `hello` version being served by the request:

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

Run this several times and you should see that some of the requests are served by hello 1.0.0 and a small subset (1/4 = 25%) are served by 2.0.0.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/cc9dad9e-101e-4954-bb25-e1250b8a900a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174525Z&X-Amz-Expires=86400&X-Amz-Signature=5704f792085f51ecdef57cc5787493dcdb51ffd8adecb6d5931dff51a297aa0c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

### **Canary deployments in production - session affinity**

In this lab, each request sent to the Nginx service had a chance to be served by the canary deployment. But what if you wanted to ensure that a user didn't get served by the Canary deployment? A use case could be that the UI for an application changed, and you don't want to confuse the user. In a case like this, you want the user to "stick" to one deployment or the other.

You can do this by creating a service with session affinity. This way the same user will always be served from the same version. In the example below the service is the same as before, but a new `sessionAffinity` field has been added, and set to ClientIP. All clients with the same IP address will have their requests sent to the same version of the `hello` application.

### Blue-green deployments

Rolling updates are ideal because they allow you to deploy an application slowly with minimal overhead, minimal performance impact, and minimal downtime. There are instances where it is beneficial to modify the load balancers to point to that new version only after it has been fully deployed. In this case, blue-green deployments are the way to go.

Kubernetes achieves this by creating two separate deployments; one for the old "blue" version and one for the new "green" version. Use your existing `hello` deployment for the "blue" version. The deployments will be accessed via a Service which will act as the router. Once the new "green" version is up and running, you'll switch over to using that version by updating the Service.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/24728c13-3f63-4db8-8ea4-1457389beefb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174552Z&X-Amz-Expires=86400&X-Amz-Signature=776ad0d45629280394a5558b3cf723d5db546c0d961ae7c4ee1ad2a977ed0c39&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

### The service

Use the existing hello service, but update it so that it has a selector `app:hello`
, `version: 1.0.0`
. The selector will match the existing "blue" deployment. But it will not match the "green" deployment because it will use a different version.

```bash
kubectl apply -f services/hello-blue.yaml
```

### Updating using Blue-Green Deployment

In order to support a blue-green deployment style, we will create a new "green" deployment for our new version. The green deployment updates the version label and the image path.

Create the green deployment:

```bash
kubectl create -f deployments/hello-green.yaml
```

Once you have a green deployment and it has started up properly, verify that the current version of 1.0.0 is still being used:

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

Now, update the service to point to the new version:

```bash
kubectl apply -f services/hello-green.yaml
```

When the service is updated, the "green" deployment will be used immediately. You can now verify that the new version is always being used.

```bash
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/54303ede-6474-4d77-914d-f929e9bc3307/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220715%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220715T174606Z&X-Amz-Expires=86400&X-Amz-Signature=adf6f4ecfdd5971bf412e2b6bf066ebb834067222c415a8178fc9c417d20d9da&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

### Blue-Green Rollback

If necessary, you can roll back to the old version in the same way. While the "blue" deployment is still running, just update the service back to the old version.

```bash
kubectl apply -f services/hello-blue.yaml
```

Once you have updated the service, your rollback will have been successful. Again, verify that the right version is now being used: