# Lab 3. Orchestrating the Cloud with Kubernetes

[Google LInk](https://www.cloudskillsboost.google/focuses/557)

## Overview

In this lab you will learn how to:

- Provision a complete [Kubernetes](http://kubernetes.io/) cluster using [Kubernetes Engine](https://cloud.google.com/container-engine).
- Deploy and manage Docker containers using `kubectl`.
- Break an application into microservices using Kubernetes' Deployments and Services.

Kubernetes is all about applications. In this part of the lab you will use an example application called "app".

> **Note:** > [App](https://github.com/kelseyhightower/app) is hosted on GitHub and provides an example 12-Factor application. During this lab you will be working with the following Docker images:

- [kelseyhightower/monolith](https://hub.docker.com/r/kelseyhightower/monolith) - Monolith includes auth and hello services.
- [kelseyhightower/auth](https://hub.docker.com/r/kelseyhightower/auth) - Auth microservice. Generates JWT tokens for authenticated users.
- [kelseyhightower/hello](https://hub.docker.com/r/kelseyhightower/hello) - Hello microservice. Greets authenticated users.
- [nginx](https://hub.docker.com/_/nginx) - Frontend to the auth and hello services.
  >

Kubernetes is an open source project (available on [kubernetes.io](http://kubernetes.io/)) which can run on many different environments, from laptops to high-availability multi-node clusters, from public clouds to on-premise deployments, from virtual machines to bare metal.

For this lab, using a managed environment such as Kubernetes Engine allows you to focus on experiencing Kubernetes rather than setting up the underlying infrastructure.

## Set up Google Kubernetes Engine

1. Set the zone:

   ```bash
   $ gcloud config set compute/zone us-central1-b
   Updated property [compute/zone].
   ```

2. Start up a cluster for the lab (This will take a while to provision a few VMs)

   ```bash
   $ gcloud container clusters create io
   Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
   Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
   Creating cluster io in us-central1-b... Cluster is being health-checked (master is healthy)...done.
   Created [https://container.googleapis.com/v1/projects/{project-id}/zones/us-central1-b/clusters/io].
   To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-b/io?project={project-id}
   kubeconfig entry generated for io.
   NAME: io
   LOCATION: us-central1-b
   MASTER_VERSION: 1.22.8-gke.202
   MASTER_IP: {master-ip}
   MACHINE_TYPE: e2-medium
   NODE_VERSION: 1.22.8-gke.202
   NUM_NODES: 3
   STATUS: RUNNING
   ```

## Task 1. Get the sample code

1. Clone the GitHub repository

   ```bash
   $ gsutil cp -r gs://spls/gsp021/* .
   ```

2. Change into the direcotry needed

   ```bash
   $ cd orchestrate-with-kubernetes/kubernetes
   ```

3. List the files inside

   ```bash
   $ ls
   cleanup.sh  deployments  nginx  pods  services  tls
   ```

   The files are following:

   ```
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

## Task 2. Quick Kubernetes Demo

The easiest way to get started with Kubernetes is to use the `kubectl create` command.

1.  Use it to launch a single instance of the nginx container:

    ```bash
    $ kubectl create deployment nginx --image=nginx:1.10.0
    deployment.apps/nginx created

    $ kubectl get deployment
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/1     1            1           31s
    ```

2.  In Kubernetes, all containers run in a pod.
    Use the `kubectl get pods` command to view the running **nginx container**.
    `bash $ kubectl get pods NAME READY STATUS RESTARTS AGE nginx-56cd7f6b6-dvnmv 1/1 Running 0 62s `
3.  Once the **nginx container** has a `Running` status, you can expose it outside of Kubernetes using the `kubectl expose` command:

    ```bash
    $ kubectl expose deployment nginx --port 80 --type LoadBalancer
    service/nginx exposed
    ```

    Behind the scenes, Kubernetes created an external `Load Balancer` with **a public IP address attached to it**. Any client who hits that public IP address will be **routed to the pods behind the service** (the nginx pod in this case).

4.  List our services now using the `kubectl get services` command:

    ```bash
    $ kubectl get services
    NAME         TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP      10.72.0.1    <none>        443/TCP        7m11s
    nginx        LoadBalancer   10.72.7.31   <pending>     80:31685/TCP   28s
    ```

    > **Note:**
    >  It may take a few seconds before the `ExternalIP`
    >  field is populated for your service. This is normal -- just re-run the `kubectl get services`
    >  command every few seconds until the field populates.

    Let’s hit it again.

    ```bash
    $ kubectl get services
    NAME         TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)        AGE
    kubernetes   ClusterIP      10.72.0.1    <none>         443/TCP        7m53s
    nginx        LoadBalancer   10.72.7.31   {external-ip}   80:31685/TCP   70s
    ```

5.  Add the External IP to this command to hit the Nginx container remotely:

    ```bash
    $ curl http://<External IP>:80
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

## Task 3. Pods

At the core of Kubernetes is the [Pod](http://kubernetes.io/docs/user-guide/pods/).

Pods represent and hold a collection of one or more containers. Generally, if you have multiple containers with a hard dependency on each other, you package the containers inside a single pod.

![image](https://user-images.githubusercontent.com/42485462/179185747-53266c04-60ef-40e6-8233-a869be241706.png)

In this example there is a pod that contains the `monolith` and `nginx` containers.

### Volumes

Pods also have [Volumes](http://kubernetes.io/docs/user-guide/volumes/). Volumes are `data disks` that live as long as the pods live, and can be used by the containers in that pod. Pods provide a **shared namespace** for their contents which means that the **two containers inside of our example pod can communicate with each other**, and they **also share the attached volumes**.

Pods also **share a network namespace**. This means that there is **one IP Address per pod**.

## Task 4. Creating pods

Pods can be created using `pod configuration` files.

Let’s explore the monolith pod configuration file.

1. Run the following

   ```bash
   $ cat pods/monolith.yaml
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

   - Your pod is made up of **one container** (the `monolith`).
   - You're passing **a few arguments to our container when it starts up**. (`—args`)
   - You're opening up **port 80 for http traffic**.

2. Create the monolith pod using `kubectl`:

   ```bash
   $ kubectl create -f pods/monolith.yaml
   pod/monolith created
   ```

3. Examine your pods.

   ```bash
   $ kubectl get pods
   NAME                    READY   STATUS    RESTARTS   AGE
   monolith                1/1     Running   0          14s
   nginx-56cd7f6b6-dvnmv   1/1     Running   0          5m9s
   ```

4. Once the pod is running, run `kubectl describe` to get more information about the monolith pod.

   ```bash
   $ kubectl describe pods monolith
   Name:         monolith
   Namespace:    default
   Priority:     0
   Node:         gke-io-default-pool-XXXX-XXXX/XX.XX.XX.XX
   Start Time:   XXXXXXX
   Labels:       app=monolith
   Annotations:  <none>
   Status:       Running
   IP:           10.68.1.4
   IPs:
     IP:  10.68.1.4
   Containers:
     monolith:
       Container ID:  containerd://3929101ab38bdb2672e2e989f0ca8e6f59d641e66ee711970750baa83be7cb4d
       Image:         kelseyhightower/monolith:1.0.0
       Image ID:      sha256:980e09dd5c76f726e7369ac2c3aa9528fe3a8c92382b78e97aa54a4a32d3b187
       Ports:         80/TCP, 81/TCP
       Host Ports:    0/TCP, 0/TCP
       Args:
         -http=0.0.0.0:80
         -health=0.0.0.0:81
         -secret=secret
       State:          Running
         Started:      Fri, 15 Jul 2022 07:29:55 +0000
       Ready:          True
       Restart Count:  0
       Limits:
         cpu:     200m
         memory:  10Mi
       Requests:
         cpu:        200m
         memory:     10Mi
       Environment:  <none>
       Mounts:
         /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bwwct (ro)
   Conditions:
     Type              Status
     Initialized       True
     Ready             True
     ContainersReady   True
     PodScheduled      True
   Volumes:
     kube-api-access-bwwct:
       Type:                    Projected (a volume that contains injected data from multiple sources)
       TokenExpirationSeconds:  3607
       ConfigMapName:           kube-root-ca.crt
       ConfigMapOptional:       <nil>
       DownwardAPI:             true
   QoS Class:                   Guaranteed
   Node-Selectors:              <none>
   Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
   Events:
     Type    Reason     Age   From               Message
     ----    ------     ----  ----               -------
     Normal  Scheduled  52s   default-scheduler  Successfully assigned default/monolith to gke-io-default-pool-ef6d59fe-p2n3
     Normal  Pulling    50s   kubelet            Pulling image "kelseyhightower/monolith:1.0.0"
     Normal  Pulled     49s   kubelet            Successfully pulled image "kelseyhightower/monolith:1.0.0" in 1.6362145s
     Normal  Created    49s   kubelet            Created container monolith
     Normal  Started    48s   kubelet            Started container monolith
   ```

   This information will come in handy when troubleshooting.

Kubernetes makes it easy to:

- `create pods` by describing them in configuration files
- `view information` about them when they are running.

At this point you have the ability to **create all the pods** your deployment requires!

## Task 5. Interacting with pods

**By default**, **pods are allocated a private IP address** and **cannot be reached outside of the cluster**. Use the `kubectl port-forward` command to **map a local port to a port inside the monolith pod**.

1. Open a second Cloud Shell terminal. Now you have two terminals, one to run the `kubectl port-forward` command, and the other to issue `curl` commands.
2. In the **2nd terminal**, run this command to set up port-forwarding:

   ```bash
   $ kubectl port-forward monolith 10080:80
   Forwarding from 127.0.0.1:10080 -> 80
   ...
   ```

3. Now in the **1st terminal** start talking to your pod using `curl`:

   ```bash
   $ curl http://127.0.0.1:10080
   {"message":"Hello"}
   ```

4. Now use the `curl` command to see what happens when you hit a secure endpoint:

   ```bash
   $ curl http://127.0.0.1:10080/secure
   authorization failed
   ```

   `authorization failed` — Uh oh.

5. 1. Try logging in to get an **auth token** back from the monolith:

   ```bash
   $ curl -u user http://127.0.0.1:10080/login
   Enter host password for user 'user':
   ```

6. At the login prompt, use the super-secret password "`password`" to login.

   Logging in caused a JWT token to print out.

7. Since Cloud Shell does not handle copying long strings well, create an environment variable for the token.

   ```bash
   $ TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
   Enter host password for user 'user':
     % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
   100   222  100   222    0     0    247      0 --:--:-- --:--:-- --:--:--   247
   ```

8. Enter the super-secret password "password" again when prompted for the host password.
9. Use this command to copy and then use the token to hit the secure endpoint with `curl`:

   ```bash
   $ curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
   {"message":"Hello"}
   ```

10. Use the `kubectl logs` command to view the logs for the `monolith` Pod.

    ```bash
    $ kubectl logs monolith
    2022/07/15 07:29:55 Starting server...
    2022/07/15 07:29:55 Health service listening on 0.0.0.0:81
    2022/07/15 07:29:55 HTTP service listening on 0.0.0.0:80
    127.0.0.1:46256 - - [Fri, 15 Jul 2022 07:31:57 UTC] "GET / HTTP/1.1" curl/7.74.0
    127.0.0.1:46290 - - [Fri, 15 Jul 2022 07:32:21 UTC] "GET /secure HTTP/1.1" curl/7.74.0
    127.0.0.1:46340 - - [Fri, 15 Jul 2022 07:32:52 UTC] "GET /login HTTP/1.1" curl/7.74.0
    127.0.0.1:46388 - - [Fri, 15 Jul 2022 07:33:21 UTC] "GET /secure HTTP/1.1" curl/7.74.0
    ```

11. **Open a 3rd terminal** and use the `-f` flag to get **a stream of the logs** happening in real-time:

    ```bash
    $ kubectl logs -f monolith
    2022/07/15 07:29:55 Starting server...
    2022/07/15 07:29:55 Health service listening on 0.0.0.0:81
    2022/07/15 07:29:55 HTTP service listening on 0.0.0.0:80
    127.0.0.1:46256 - - [Fri, 15 Jul 2022 07:31:57 UTC] "GET / HTTP/1.1" curl/7.74.0
    127.0.0.1:46290 - - [Fri, 15 Jul 2022 07:32:21 UTC] "GET /secure HTTP/1.1" curl/7.74.0
    127.0.0.1:46340 - - [Fri, 15 Jul 2022 07:32:52 UTC] "GET /login HTTP/1.1" curl/7.74.0
    127.0.0.1:46388 - - [Fri, 15 Jul 2022 07:33:21 UTC] "GET /secure HTTP/1.1" curl/7.74.0
    ```

12. 1. Now if you use `curl` in the **1st terminal** to interact with the monolith, you can see the logs updating (in the **3rd terminal**):

    ```bash
    $ curl http://127.0.0.1:10080
    {"message":"Hello"}
    ```

13. 1. Use the `kubectl exec` command to run an `interactive shell` inside the Monolith Pod. This can come in handy when you want to troubleshoot from within a container:

    ```bash
    $ kubectl exec monolith --stdin --tty -c monolith -- /bin/sh
    #
    ```

14. For example, once you have a shell into the monolith container you can test **external connectivity** using the `ping` command:

    ```bash
    # ping -c 3 google.com
    PING google.com (142.250.148.138): 56 data bytes
    64 bytes from 142.250.148.138: seq=0 ttl=114 time=1.863 ms
    64 bytes from 142.250.148.138: seq=1 ttl=114 time=0.519 ms
    64 bytes from 142.250.148.138: seq=2 ttl=114 time=0.441 ms

    --- google.com ping statistics ---
    3 packets transmitted, 3 packets received, 0% packet loss
    round-trip min/avg/max = 0.441/0.941/1.863 ms
    ```

15. Be sure to log out when you're done with this interactive shell.

    ```bash
    # exit
    $
    ```

As you can see, interacting with pods is as easy as using the `kubectl` command.

If you need to hit a container remotely, or get a login shell, Kubernetes provides everything you need to get up and going.

## Task 6. Services

**Pods aren't meant to be persistent.** They **can be stopped** or **started** for many reasons - like `failed liveness` or `readiness checks` - and this **leads to a problem**:

- What happens if you want to communicate with a set of Pods? **When they get restarted they might have a different IP address.**

That's where [Services](http://kubernetes.io/docs/user-guide/services/) come in. Services provide `stable endpoints` for Pods.

![image](https://user-images.githubusercontent.com/42485462/179185776-a158a792-aea5-4e74-abad-71ce5afcd5a5.png)

Services use `labels` to **determine what Pods they operate on**. If Pods have the correct labels, they are **automatically picked up and exposed by our services**.

The **level of access** a service provides to a set of pods depends on the **Service's type**. Currently there are 3 types:

- `ClusterIP` (**internal**) -- the **default type** means that this Service is **only visible inside** of the cluster,
- `NodePort` gives **each node** in the cluster an **externally accessible IP** and
- `LoadBalancer` adds a **load balancer** from the cloud provider which **forwards traffic from the service to Nodes** within it.

Now you'll learn how to:

- Create a service
- Use label selectors to expose a limited set of Pods externally

## Task 7. Creating a service

Before you can create our services, first create a secure pod that can handle https traffic.

1. If you've changed directories, make sure you return to the `~/orchestrate-with-kubernetes/kubernetes` directory:

   ```bash
   $ cd ~/orchestrate-with-kubernetes/kubernetes
   ```

2. Explore the monolith service configuration file:

   ```bash
   $ cat pods/secure-monolith.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: "secure-monolith"
     labels:
       app: monolith
   spec:
     containers:
       - name: nginx
         image: "nginx:1.9.14"
         lifecycle:
           preStop:
             exec:
               command: ["/usr/sbin/nginx","-s","quit"]
         volumeMounts:
           - name: "nginx-proxy-conf"
             mountPath: "/etc/nginx/conf.d"
           - name: "tls-certs"
             mountPath: "/etc/tls"
       - name: monolith
         image: "kelseyhightower/monolith:1.0.0"
         ports:
           - name: http
             containerPort: 80
           - name: health
             containerPort: 81
         resources:
           limits:
             cpu: 0.2
             memory: "10Mi"
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
     volumes:
       - name: "tls-certs"
         secret:
           secretName: "tls-certs"
       - name: "nginx-proxy-conf"
         configMap:
           name: "nginx-proxy-conf"
           items:
             - key: "proxy.conf"
               path: "proxy.conf"
   ```

3. Create the secure-monolith pods and their configuration data:

   ```bash
   $ kubectl create secret generic tls-certs --from-file tls/
   secret/tls-certs created

   $ kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
   configmap/nginx-proxy-conf created

   $ kubectl create -f pods/secure-monolith.yaml
   pod/secure-monolith created
   ```

   Now that you have a secure pod, it's time to expose the secure-monolith Pod externally.To do that, create a Kubernetes service.

4. Explore the monolith service configuration file:

   ```bash
   $ cat services/monolith.yaml
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

   Things to note:

   - There's a `selector` which is **used to automatically find and expose any pods with the labels** `app: monolith` and `secure: enabled`.
   - Now you have to **expose the nodeport here** because this is how you'll **forward external traffic from port 31000 to nginx** (on port 443).

5. Create the monolith service from the monolith service configuration file:

   ```bash
   $ kubectl create -f services/monolith.yaml
   service/monolith created
   ```

   You're using a port to expose the service. This means that it's possible to have `port collisions` if another app tries to bind to port 31000 on one of your servers.

   **Normally, Kubernetes would handle this `port assignment`**. In this lab you chose a port so that it's easier to configure health checks later on.

6. Use the `gcloud compute firewall-rules` command to **create fireall rule to allow TCP traffic on the 31000 port to the monolith service on the exposed nodeport**:

   ```bash
   $ gcloud compute firewall-rules create allow-monolith-nodeport \
     --allow=tcp:31000
   Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/{project-id}/global/firewalls/allow-monolith-nodeport].
   Creating firewall...done.
   NAME: allow-monolith-nodeport
   NETWORK: default
   DIRECTION: INGRESS
   PRIORITY: 1000
   ALLOW: tcp:31000
   DENY:
   DISABLED: False
   ```

### Test completed task

Now that everything is set up you should be able to hit the secure-monolith service from outside the cluster **without using port forwarding**.

1. First, get an external IP address for one of the nodes.

   ```bash
   $ gcloud compute instances list
   NAME: gke-io-default-pool-ef6d59fe-0gfj
   ZONE: us-central1-b
   MACHINE_TYPE: e2-medium
   PREEMPTIBLE:
   INTERNAL_IP: 10.128.0.4
   EXTERNAL_IP: {external-ip-1}
   STATUS: RUNNING

   NAME: gke-io-default-pool-ef6d59fe-p2n3
   ZONE: us-central1-b
   MACHINE_TYPE: e2-medium
   PREEMPTIBLE:
   INTERNAL_IP: 10.128.0.3
   EXTERNAL_IP: {external-ip-2}
   STATUS: RUNNING

   NAME: gke-io-default-pool-ef6d59fe-sppw
   ZONE: us-central1-b
   MACHINE_TYPE: e2-medium
   PREEMPTIBLE:
   INTERNAL_IP: 10.128.0.2
   EXTERNAL_IP: {external-ip-3}
   STATUS: RUNNING
   ```

2. Now try hitting the secure-monolith service using `curl`:

   ```bash
   $ curl -k https://<EXTERNAL_IP>:31000
   curl: (7) Failed to connect to 34.133.59.254 port 31000: Connection refused
   ```

   Something’s wrong.

   - Why are you **unable to get a response** from the monolith service?
   - **How many endpoints** does the monolith service have?
   - What **labels must a Pod have to be picked up by the monolith service**?

   This has something to do with `labels`.

## \***\*Task 8. Adding labels to pods\*\***

Currently the monolith service does not have endpoints. One way to troubleshoot an issue like this is to use the `kubectl get pods` command with a label query.

1. You can see that you have quite a few pods running with the monolith `label`.

   ```bash
   $ kubectl get pods -l "app=monolith"
   NAME              READY   STATUS    RESTARTS   AGE
   monolith          1/1     Running   0          13m
   secure-monolith   2/2     Running   0          5m39s
   ```

2. But what about "`app=monolith`" and "`secure=enabled`"?

   ```bash
   $ kubectl get pods -l "app=monolith,secure=enabled"
   No resources found in default namespace.
   ```

   - Notice this label query does not print any results. It seems like you need to add the "secure=enabled" label to them.

3. Add the missing `secure=enabled` label to the secure-monolith Pod. Afterwards, you can check and see that your labels have been updated.

   ```bash
   $ kubectl label pods secure-monolith 'secure=enabled'
   pod/secure-monolith labeled

   $ kubectl get pods secure-monolith --show-labels
   NAME              READY   STATUS    RESTARTS   AGE     LABELS
   secure-monolith   2/2     Running   0          6m22s   app=monolith,secure=enabled
   ```

4. Now that your pods are correctly labeled, view the list of endpoints on the monolith service:

   ```bash
   $ kubectl describe services monolith | grep Endpoints
   Endpoints:                10.68.0.5:443
   ```

   **And you have one!**

5. Test this out by hitting one of our nodes again.

   ```bash
   $ gcloud compute instances list
   NAME: gke-io-default-pool-ef6d59fe-0gfj
   ZONE: us-central1-b
   MACHINE_TYPE: e2-medium
   PREEMPTIBLE:
   INTERNAL_IP: 10.128.0.4
   EXTERNAL_IP: {external-ip-1}
   STATUS: RUNNING

   NAME: gke-io-default-pool-ef6d59fe-p2n3
   ZONE: us-central1-b
   MACHINE_TYPE: e2-medium
   PREEMPTIBLE:
   INTERNAL_IP: 10.128.0.3
   EXTERNAL_IP: {external-ip-2}
   STATUS: RUNNING

   NAME: gke-io-default-pool-ef6d59fe-sppw
   ZONE: us-central1-b
   MACHINE_TYPE: e2-medium
   PREEMPTIBLE:
   INTERNAL_IP: 10.128.0.2
   EXTERNAL_IP: {external-ip-3}
   STATUS: RUNNING

   $ curl -k https://<EXTERNAL_IP>:31000
   <html>
   <head><title>400 The plain HTTP request was sent to HTTPS port</title></head>
   <body bgcolor="white">
   <center><h1>400 Bad Request</h1></center>
   <center>The plain HTTP request was sent to HTTPS port</center>
   <hr><center>nginx/1.9.14</center>
   </body>
   </html>
   ```

## \***\*Task 9. Deploying applications with Kubernetes\*\***

The goal of this lab is to get you ready for `**scaling` and `managing` containers\*\* in production.

That's where [Deployments](http://kubernetes.io/docs/user-guide/deployments/#what-is-a-deployment) come in. `Deployments` are a **declarative way** to **ensure that the number of Pods running is equal to the desired number of Pods**, specified by the user.

![image](https://user-images.githubusercontent.com/42485462/179185802-0b7cb10a-1a39-417f-bfb0-8d823b7e00fb.png)

The main benefit of Deployments is in **abstracting away the low level details of managing Pods**.

Behind the scenes Deployments use [Replica Sets](http://kubernetes.io/docs/user-guide/replicasets/) to **manage starting and stopping the Pods**.

If Pods **need to be updated or scaled**, the `Deployment` will handle that. Deployment also handles **restarting Pods** if they happen to go down for some reason.

Look at a quick example:

![image](https://user-images.githubusercontent.com/42485462/179185810-45e4cb78-3c34-44d0-a983-93c2ef05b9cc.png)

`Pods` are **tied to the lifetime of the Node they are created on**. In the example above, **Node3 went down** (**taking a Pod with it**).

Instead of manually creating a new Pod and finding a Node for it, your `Deployment` **created a new Pod and started it on Node2**.

That's pretty cool!

It's time to combine everything you learned about Pods and Services to break up the monolith application into smaller Services using Deployments.

## \***\*Task 10. Creating deployments\*\***

You're going to break the monolith app into 3 separate pieces:

- **auth** - **Generates JWT** tokens for authenticated users.
- **hello** - **Greet authenticated users**.
- **frontend** - **Routes traffic** to the auth and hello services.

You are ready to create `deployments`, **one for each service**.

Afterwards, you'll define **internal services** for the `auth` and `hello` deployments and an **external service** for the `frontend` deployment.

Once finished you'll be able to interact with the microservices just like with Monolith only now each piece will be able to be scaled and deployed, independently!

1. Get started by examining the auth deployment configuration file.

   ```bash
   $ cat deployments/auth.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: auth
   spec:
     selector:
       matchLabels:
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
             resources:
               limits:
                 cpu: 0.2
                 memory: "10Mi"
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

   The deployment is creating 1 replica, and you're using version 2.0.0 of the auth container.

   When you run the `kubectl create` command to **create the auth deployment** it will make **one pod that conforms to the data in the `Deployment` manifest**. This means you can **scale the number of Pods** by **changing the number specified** in the `replicas` field.

2. Create your deployment object:

   ```bash
   $ kubectl create -f deployments/auth.yaml
   deployment.apps/auth created
   ```

3. Create a service for your auth deployment.

   ```bash
   $ kubectl create -f services/auth.yaml
   service/auth created
   ```

4. Now do the same thing to create and expose the hello deployment:

   ```bash
   $ kubectl create -f deployments/hello.yaml
   deployment.apps/hello created

   $ kubectl create -f services/hello.yaml
   service/hello created
   ```

5. And one more time to create and expose the frontend Deployment.

   ```bash
   $ kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
   configmap/nginx-frontend-conf created

   $ kubectl create -f deployments/frontend.yaml
   deployment.apps/frontend created

   $ kubectl create -f services/frontend.yaml
   service/frontend created
   ```

6. 1. Interact with the frontend by grabbing its External IP and then curling to it:

   ```bash
   $ kubectl get services frontend
   NAME       TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)         AGE
   frontend   LoadBalancer   10.72.13.88   34.170.78.128   443:32289/TCP   31s
   ```

7. And you get a hello response back!

   ```bash
   $ curl -k https://<EXTERNAL-IP>
   ```
