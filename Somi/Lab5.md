# Lab5

# Lab5 : Continuous Delivery with Jenkins in Kubernetes Engine

### Goal

- Learn how to set up a continuous delivery pipeline with `Jenkins` on Kubernetes engine.
- Jenkins is the go-to automation server used by developers who frequently integrate their code in a shared repository.

- Provision a Jenkins application into a Kubernetes Engine Cluster
- Set up your Jenkins application using Helm Package Manager
- Explore the features of a Jenkins application
- Create and exercise a Jenkins pipeline

In the Cloud Architecture Center, see [Jenkins on Kubernetes Engine](https://cloud.google.com/solutions/jenkins-on-container-engine) to learn more about running Jenkins on Kubernetes.

### What is Kubernetes Engine?

Kubernetes Engine is Google Cloud's hosted version of `Kubernetes` - a powerful cluster manager and orchestration system for containers. Kubernetes is an open source project that can run on many different environments—from laptops to high-availability multi-node clusters; from virtual machines to bare metal. As mentioned before, Kubernetes apps are built on `containers` - these are lightweight applications bundled with all the necessary dependencies and libraries to run them. This underlying structure makes Kubernetes applications highly available, secure, and quick to deploy—an ideal framework for cloud developers.

### What is Jenkins?

[Jenkins](https://jenkins.io/) is an open-source automation server that lets you flexibly orchestrate your build, test, and deployment pipelines. Jenkins allows developers to iterate quickly on projects without worrying about overhead issues that can stem from continuous delivery.

## Task 1. Download the source code

```bash
gcloud config set compute/zone us-west1-c
```

set up the your zone

```bash
gsutil cp gs://spls/gsp051/continuous-deployment-on-kubernetes.zip .
```

```bash
unzip continuous-deployment-on-kubernetes.zip
```

copy the sample code.

## Task 2. Provisioning Jenkins

### Creating a Kubernetes cluster

1. Run following command to provision a Kubernetes cluster:

```bash
gcloud container clusters create jenkins-cd \
--num-nodes 2 \
--machine-type n1-standard-2 \
--scopes "https://www.googleapis.com/auth/source.read_write,cloud-platform"
```

The extra scopes enable Jenkins to access Cloud Source Repositories and Google Container Registry.

```bash
--num-nodes 2 \
--machine-type n1-standard-2 \
--scopes "https://www.googleapis.com/auth/source.read_write,cloud-platform"
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster jenkins-cd in us-west1-c... Cluster is being health-checked (master is healthy)...done.
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-00-64509de78d80/zones/us-west1-c/clusters/jenkins-cd].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-c/jenkins-cd?project=qwiklabs-gcp-00-64509de78d80
kubeconfig entry generated for jenkins-cd.
NAME: jenkins-cd
LOCATION: us-west1-c
MASTER_VERSION: 1.22.8-gke.202
MASTER_IP: 34.83.57.246
MACHINE_TYPE: n1-standard-2
NODE_VERSION: 1.22.8-gke.202
NUM_NODES: 2
STATUS: RUNNING
```

1. Confirm that your cluster is running by executing the following command:

```bash
gcloud container clusters list
```

```bash
NAME: jenkins-cd
LOCATION: us-west1-c
MASTER_VERSION: 1.22.8-gke.202
MASTER_IP: 34.83.57.246
MACHINE_TYPE: n1-standard-2
NODE_VERSION: 1.22.8-gke.202
NUM_NODES: 2
STATUS: RUNNING
```

1. Get the credentials for your cluster

```bash
gcloud container clusters get-credentials jenkins-cd
```

```bash
Fetching cluster endpoint and auth data.
kubeconfig entry generated for jenkins-cd.
```

1. Kubernetes Engine uses these credentials to access your newly provisioned cluster- confirm that you can connect to it by running the following command:

```bash
kubectl cluster-info
```

## Task 3. Setup Helm

Use Helm to install Jenkins from the Charts repository.

- Helm is a package manager that makes it easy to configure and deploy Kubernetes applications. Once you have Jenkins installed, you’ll be able to set up your CI/CD pipeline.

1. Add Helm’s stable chart repo

```bash
helm repo add jenkins https://charts.jenkins.io
```

1. Ensure the repo is up do date

```bash
helm repo update
```

## Task 4. Configure and Install Jenkins

When installing Jenkins, a `values` file can be used as a template to provide values that are necessary for setup.

You will use a custom `values` file to automatically configure your Kubernetes Cloud and add the following necessary plugins:

- Kubernetes:1.29.4
- Workflow-multibranch:latest
- Git:4.7.1
- Configuration-as-code:1.51
- Google-oauth-plugin:latest
- Google-source-plugin:latest
- Google-storage-plugin:latest

This will allow Jenkins to connect to your cluster and your GCP project.

1. Download the custom `values` file

```bash
gsutil cp gs://spls/gsp330/values.yaml jenkins/values.yaml
```

1. Use the Helm CLI to deploy the chart with your configuration settings

```bash
helm install cd jenkins/jenkins -f jenkins/values.yaml --wait
```

```bash
--wait
NAME: cd
LAST DEPLOYED: Thu Jul 21 16:10:20 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get your 'admin' user password by running:
  kubectl exec --namespace default -it svc/cd-jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  echo http://127.0.0.1:8080
  kubectl --namespace default port-forward svc/cd-jenkins 8080:8080

3. Login with the password from step 1 and the username: admin
4. Configure security realm and authorization strategy
5. Use Jenkins Configuration as Code by specifying configScripts in your values.yaml file, see documentation: http:///configuration-as-code and examples: https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos

For more information on running Jenkins on Kubernetes, visit:
https://cloud.google.com/solutions/jenkins-on-container-engine

For more information about Jenkins Configuration as Code, visit:
https://jenkins.io/projects/jcasc/

NOTE: Consider using a custom image with pre-installed plugins
```

1. Ensure the Jenkins pod goes to the `Running` state and the container is in the READY state

```bash
kubectl get pods
```

```bash
NAME           READY   STATUS    RESTARTS   AGE
cd-jenkins-0   2/2     Running   0          119s
```

1. Configure the Jenkins service account to be able to deploy the cluster

```bash
kubectl create clusterrolebinding jenkins-deploy --clusterrole=cluster-admin --serviceaccount=default:cd-jenkins
```

```bash
clusterrolebinding.rbac.authorization.k8s.io/jenkins-deploy created
```

1. Run the following command to setup port forwarding to the Jenkins UI from the Cloud Shell:

```bash
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
```

1. Now check that the Jenkins Service was created properly:

```bash
kubectl get svc
```

```bash
NAME               TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
cd-jenkins         ClusterIP   10.56.5.116   <none>        8080/TCP    4m43s
cd-jenkins-agent   ClusterIP   10.56.2.114   <none>        50000/TCP   4m43s
kubernetes         ClusterIP   10.56.0.1     <none>        443/TCP     10m
```

You are using the [Kubernetes Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Kubernetes+Plugin) so that our builder nodes will be automatically launched as necessary when the Jenkins master requests them. Upon completion of their work, they will automatically be turned down and their resources added back to the clusters resource pool.

Notice that this service exposes ports `8080` and `50000` for any pods that match the `selector`. This will expose the Jenkins web UI and builder/agent registration ports within the Kubernetes cluster. Additionally, the `jenkins-ui` services is exposed using a ClusterIP so that it is not accessible from outside the cluster.

## Task 5. Connect to Jenkins

1. The Jenkins chart will automatically create an admin password for you.

```bash
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

1. Get Jenkins user interface

![Untitled](https://user-images.githubusercontent.com/57824857/181254607-6738e826-2577-4924-8e9f-74ff605d81b1.png)

![Untitled](https://user-images.githubusercontent.com/57824857/181254803-d312c4f2-82be-49db-8cdf-63d31363a5d2.png)

1. Login with the account.

![Untitled](https://user-images.githubusercontent.com/57824857/181254910-25708ebb-771c-4a7c-aa91-868d2175d2bf.png)

Jenkins will drvie your automated CI/CD pipelines in the next sections.

## Task 6. Understanding the Application

![Untitled](https://user-images.githubusercontent.com/57824857/181255042-4ed191b1-88b0-4b23-bead-c6f2466d2c1c.png)

The sample application `gceme`

- In **backend mode**: gceme listens on port 8080 and returns Compute Engine instance metadata in JSON format.
- In **frontend mode**: gceme queries the backend gceme service and renders the resulting JSON in the user interface.

## Task 7. Deploying the Application

Deploy the application into two different enviornments:

- **Production**: The live site that your users access.
- **Canary**: A smaller-capacity site that receives only a percentage of your user traffic. Use this environment to validate your software with live traffic before it's released to all of your users.

1. Navigate to the sample application directory and create the kubernetes namespace to logically isolate the deployment.

```bash
cd sample-app
kubectl create ns production
```

1. Create the production and canary deployments, and the services using the `kubectl apply` commands

```bash
kubectl apply -f k8s/production -n production
```

```bash
kubectl apply -f k8s/canary -n production
```

```bash
kubectl apply -f k8s/services -n production
```

By default, only one replica of the frontend is deployed. Use the `kubectl scale` command to ensure that there are at least 4 replicas running at all times.

1. Scale up the production environment frontends

```bash
kubectl scale deployment gceme-frontend-production -n production --replicas 4
```

```bash
deployment.apps/gceme-frontend-production scaled
```

1. Confirm that you have 5 pods running for the frontend, 4 for the production traffic and 1 for canary releases (changes to the canary release will only affect 1 out of 5 (20%) of users)

```bash
kubectl get pods -n production -l app=gceme -l role=frontend
```

```bash
NAME                                         READY   STATUS    RESTARTS   AGE
gceme-frontend-canary-649bd655bd-8m92m       1/1     Running   0          2m42s
gceme-frontend-production-78887ccfbf-nps4b   1/1     Running   0          54s
gceme-frontend-production-78887ccfbf-nq8r8   1/1     Running   0          54s
gceme-frontend-production-78887ccfbf-ps5s5   1/1     Running   0          3m16s
gceme-frontend-production-78887ccfbf-v54cb   1/1     Running   0          54s
```

1. Also confirm that you have 2 pods for the backend, 1 for production and 1 for canary:

```bash
kubectl get pods -n production -l app=gceme -l role=backend
```

```bash
NAME                                        READY   STATUS    RESTARTS   AGE
gceme-backend-canary-6fb5c7bb66-jhlv8       1/1     Running   0          3m45s
gceme-backend-production-586b4b46cc-gdvjs   1/1     Running   0          4m18s
```

1. Retrieve the external IP for the production services

```bash
kubectl get service gceme-frontend -n production
```

```bash
NAME             TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE
gceme-frontend   LoadBalancer   10.56.8.146   35.233.227.209   80:30882/TCP   4m34s
```

![Untitled](https://user-images.githubusercontent.com/57824857/181255212-1e0e178a-4a7c-4f67-b5c0-3132d79133e4.png)

Paste External IP into a browser and you’ll see this card

1. Store the frontend service load balancer IP in an environment variable for use later

```bash
export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
```

1. Confirm that both services are working by opening the frontend external IP address in your browser.

1. Check the version output of the service by running the following command.

```bash
curl http://$FRONTEND_SERVICE_IP/version
```

```bash
1.0.0
```

You have successfully deployed the sample application!

Now set up a pipeline for deploying your changes continuously and reliably.

## Task 8. Creating the Jenkins Pipeline

### Creating a repository to host the sample app source code.

1. Create a copy of the `gceme` sample app and push it to a Cloud Source Repository (GCR)

```bash
gcloud source repos create default
```

```bash
Created [default].
```

1. Initialize the sample-app directory as its own Git repository

```bash
git init
git config credential.helper gcloud.sh
git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/default
```

```bash
git config --global user.email "[EMAIL_ADDRESS]"
git config --global user.name "[USERNAME]"
```

1. add, commit, and push the files to git

### Adding your service account credentials

Configure your credentials to allow Jenkins to access the code repository. Jenkins will use your cluster's service account credentials in order to download code from the Cloud Source Repositories.

Jenkins UI → Manage Jenkins → Manage Credentials → Jenkins

![Untitled](https://user-images.githubusercontent.com/57824857/181255212-1e0e178a-4a7c-4f67-b5c0-3132d79133e4.png)

Global credentials → Add Credentials → Google Service Account from metadata from Kind drop-down

![Untitled](https://user-images.githubusercontent.com/57824857/181255416-4033a26a-9a71-4330-a616-b37d294d416f.png)

The global credentials has been added.

![Untitled](https://user-images.githubusercontent.com/57824857/181255608-4174ad38-a7dc-4be1-af6d-2add187c76ac.png)

Create new item

Multibranch Pipeline → Branch Sources → git

![Untitled](https://user-images.githubusercontent.com/57824857/181255790-3281e91a-f346-4311-a29f-07ef73a5776f.png)

After you complete these steps, a job named `Branch indexing` runs. This meta-job identifies the branches in your repository and ensures changes haven't occurred in existing branches. If you click sample-app in the top left, the `master` job should be seen.

## Task 9. Creating the development environment

Development branches are a set of environments your developers use to test their code changes before submitting them for integration into the live site. These environments are scaled-down versions of your application, but need to be deployed using the same mechanisms as the live environment.

### Creating a development branch

```bash
git checkout -b new-feature
```

### Modifying the pipeline definition

The `Jenkinsfile` that defines that pipeline is written using the [Jenkins Pipeline Groovy syntax](https://jenkins.io/doc/book/pipeline/syntax/). Using a `Jenkinsfile` allows an entire build pipeline to be expressed in a single file that lives alongside your source code. Pipelines support powerful features like parallelization and require manual user approval.

In order for the pipeline to work as expected, you need to modify the `Jenkinsfile` to set your project ID.

1. Open the Jenkinsfile and edit

```bash
vi Jenkinsfile
```

```bash
PROJECT = "REPLACE_WITH_YOUR_PROJECT_ID"
APP_NAME = "gceme"
FE_SVC_NAME = "${APP_NAME}-frontend"
CLUSTER = "jenkins-cd"
CLUSTER_ZONE = "us-west1-c"
IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
JENKINS_CRED = "${PROJECT}"
```

### Modify the site

To demonstrate changing the application, you wil change the gceme cards from blue to orange.

```bash
vi html.go
```

then change `<div class="card blue">` to

```bash
<div class="card orange">
```

open main.go

```bash
vi main.go
```

change the version `1.0.0` into `2.0.0`

```bash
const version string = "2.0.0"
```

## Task 10. Kick off Deployment

1. Commit and push your changes

```bash
git add Jenkinsfile html.go main.go
git commit -m "Version 2.0.0"
git push origin new-feature
```

After the change is pushed to the Git repository, navigate to the Jenkins user interface where you can see that your build started for the `new-feature` branch. It can take up to a minute for the changes to be picked up.

![Untitled](https://user-images.githubusercontent.com/57824857/181255939-dca9b2ff-9514-43ae-9547-7c052b127545.png)

## Task 11. Deploying a canary release

You have verified that your app is running the latest code in the development environment, so now deploy that code to the canary environment.

1. Create a canary branch and push it to the Git server

```bash
git checkout -b canary
git push origin canary
```

In Jenkins, you should see the **canary** pipeline has kicked off. Once complete, you can check the service URL to ensure that some of the traffic is being served by your new version. You should see about 1 in 5 requests (in no particular order) returning version `2.0.0`.

```bash
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
```

```bash
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
```

## Task 12. Deploying to production

Now that our canary release was successful and we haven't heard any customer complaints, deploy to the rest of your production fleet.

1. Create a canary branch and push it to the Git server

```bash
git checkout master
git merge canary
git push origin master
```

1. *Once complete* (which may take a few minutes), you can check the service URL to ensure that all of the traffic is being served by your new version, 2.0.0.

```bash
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
```

```bash
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
```

1. Once again, if you see instances of `1.0.0` try running the above commands again. You can stop this command by pressing **Ctrl + C**.
