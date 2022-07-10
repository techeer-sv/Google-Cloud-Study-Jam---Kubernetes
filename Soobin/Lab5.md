# Lab 5 : Continuous Delivery with Jenkins in Kubernetes Engine

📒 [link](https://www.cloudskillsboost.google/focuses/1104?parent=catalog)

<br/>

## **Overview**

- Goal
  - Set up a continuous delivery pipeline with Jenkins on Kubernetes engine.

![https://cdn.qwiklabs.com/1b%2B9D20QnfRjAF8c6xlXmexot7TDcOsYzsRwp%2FH4ErE%3D](https://cdn.qwiklabs.com/1b%2B9D20QnfRjAF8c6xlXmexot7TDcOsYzsRwp%2FH4ErE%3D)

- What you’ll do
  - Provision a Jenkins application into a Kubernetes Engine Cluster
  - Set up your Jenkins application using Helm Package Manager
  - Explore the features of a Jenkins application
  - Create and execise a Kenkins pipeline

<br/>

<details open>
<summary>What is Kubernetes Engine?</summary>
<br>
  Kubernetes Engine is Google Cloud's hosted version of `Kubernetes`
   - a powerful cluster manager and orchestration system for containers. Kubernetes is an open source project that can run on many different environments—from laptops to high-availability multi-node clusters; from virtual machines to bare metal. As mentioned before, Kubernetes apps are built on `containers`
   - these are lightweight applications bundled with all the necessary dependencies and libraries to run them. This underlying structure makes Kubernetes applications highly available, secure, and quick to deploy—an ideal framework for cloud developers.
</details>

<br/>

<details open>
<summary>What is Jenkins?</summary>
<br>
  Jenkins is an open-source automation server that lets you flexibly orchestrate your build, test, and deployment pipelines. Jenkins allows developers to iterate quickly on projects without worrying about overhead issues that can stem from continuous delivery. (https://jenkins.io/)
</details>

<br/>

<details open>
<summary>What is Continuous Delivery / Continuous Deployment?</summary>
<br>
  When you need to set up a continuous delivery (CD) pipeline, deploying Jenkins on Kubernetes Engine provides important benefits over a standard VM-based deployment.
  When your build process uses containers, one virtual host can run jobs on multiple operating systems. Kubernetes Engine provides `ephemeral build executors`—these are only utilized when builds are actively running, which leaves resources for other cluster tasks such as batch processing jobs. Another benefit of ephemeral build executors is *speed*—they launch in a matter of seconds.
  Kubernetes Engine also comes pre-equipped with Google's global load balancer, which you can use to automate web traffic routing to your instance(s). The load balancer handles SSL termination and utilizes a global IP address that's configured with Google's backbone network—coupled with your web front, this load balancer will always set your users on the fastest possible path to an application instance.
  Now that you've learned a little bit about Kubernetes, Jenkins, and how the two interact in a CD pipeline, it's time to go build one.
</details>

<br/>

**Ref**

- [Jenkins on Kubernetes Engine](https://cloud.google.com/architecture/jenkins-on-kubernetes-engine)

<br/>

## Download the source code

✔️ Set zone

```bash
gcloud config set compute/zone us-west1-b
```

✔️ Copy the lab’s sample code

```bash
gsutil cp gs://spls/gsp051/continuous-deployment-on-kubernetes.zip .
unzip continuous-deployment-on-kubernetes.zip
```

✔️ Change to the correct directory

```bash
cd continuous-deployment-on-kubernetes
```

<br/>

## Provisioning Jenkins

✔️ Creating a Kubernetes cluster

```bash
gcloud container clusters create jenkins-cd \
--num-nodes 2 \
--machine-type n1-standard-2 \
--scopes "https://www.googleapis.com/auth/source.read_write,cloud-platform"
```

✔️ Confirm that your cluster is running

```bash
gcloud container clusters list
```

✔️ Get the credentials

```bash
gcloud container clusters get-credentials jenkins-cd
```

✔️ Confirm that you can connect to access your newly provisioned cluster

```bash
kubectl cluster-info
```

<br/>

## Setup Helm

- What is Helm?
  - Helm is a package manager that makes it easy to configure and deploy Kubernetes applications.

✔️ Add Helm’s stable chart repo

```bash
helm repo add jenkins https://charts.jenkins.io
```

✔️ Ensure the repo is up to date

```bash
helm repo update
```

<br/>

## Configure and Install Jenkins

When installing Jenkins, a `values` file can be used as a template to provide values that are necesary for setup.

custom values file

```bash
- Kubernetes:1.29.4
- Workflow-multibranch:latest
- Git:4.7.1
- Configuration-as-code:1.51
- Google-oauth-plugin:latest
- Google-source-plugin:latest
- Google-storage-plugin:latest
```

✔️ Download the custom `values` file

```bash
gsutil cp gs://spls/gsp330/values.yaml jenkins/values.yaml
```

✔️ Use the Helm CLI to deploy the chart with your configuration settings

```bash
helm install cd jenkins/jenkins -f jenkins/values.yaml --wait
```

✔️ Ensure the Jenkins pod goes to the `Running` state and the container is in the READT state

```bash
kubectl get pods
```

✔️ Configure the Jenkins service account to be able to deploy to the cluster

```bash
kubectl create clusterrolebinding jenkins-deploy --
clusterrole=cluster-admin --serviceaccount=default:cd-jenkins
```

✔️ Setup port forwarding to the Jenkins UI from the Cloud Shell

```bash
export POD_NAME=$(kubectl get pods --namespace default -l
"app.kubernetes.io/component=jenkins-master" -l
"app.kubernetes.io/instance=cd" -o jsonpath="
{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
```

✔️ Check that the Jenkins Service was created properly

```bash
kubectl get svc
```

<br/>

## Connect to Jenkins

✔️ Jenkins chart will automatically create an admin password for you.

```bash
printf $(kubectl get secret cd-jenkins -o jsonpath="
{.data.jenkins-admin-password}" | base64 --decode);echo
```

✔️ To get to the Jenkins user interface, click on the Web Preview button in cloud shell, then click Preview on port 8080

![https://cdn.qwiklabs.com/uGPD3Uvpk2P7ejcWOKjIeVCCx9qYX0l6HOdd7F7cvuE%3D](https://cdn.qwiklabs.com/uGPD3Uvpk2P7ejcWOKjIeVCCx9qYX0l6HOdd7F7cvuE%3D)

<br/>

## Understanding the Application

Deploy the sample application, gceme, in your continuous deployment pipeline.

![https://cdn.qwiklabs.com/P1T5JBWWprA4iLf%2FB5%2BO6as7otLE25YBde57gzZwSz4%3D](https://cdn.qwiklabs.com/P1T5JBWWprA4iLf%2FB5%2BO6as7otLE25YBde57gzZwSz4%3D)

<br/>

## Deploying the Application

Environments

- **Production** : The live site that your users access
- **Canary** : A smaller-capacity site that receives only a percentage of your user traffic. Use this environment to validate your software with live traffic before it's released to all of your users.

✔️ Navigate to the sample application directory

```bash
cd sample-app
```

✔️ Create the Kubernetes namespace to logically isolate the deployment

```bash
kubectl create ns production
```

✔️ Create the production and canary deployments, and the services

```bash
kubectl create ns production
```

✔️ Create the production and canary deployments, and the services

```bash
kubectl apply -f k8s/production -n production
```

```bash
kubectl apply -f k8s/canary -n production
```

```bash
kubectl apply -f k8s/services -n production
```

✔️ Scale up the production environment frontends

```bash
kubectl scale deployment gceme-frontend-production
-n production --replicas 4
```

✔️ Now confirm that you have 5 pods running for the frontend, 4 for production traffic and 1 for canary releases (changes to the canary release will only affect 1 out of 5 (20%) of users)

```bash
kubectl get pods -n production -l app=gceme -l role=frontend
```

✔️ Confirm that you have 2 pods for the backend, 1 for production and 1 for canary

```bash
kubectl get pods -n production -l app=gceme -l role=backend
```

✔️ Retrieve the external IP for the production services

```bash
kubectl get service gceme-frontend -n production
```

✔️ Store the frontend service load balancer IP in an environment cariable for use later

```bash
export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="
{.status.loadBalancer.ingress[0].ip}" --namespace=production
services gceme-frontend)
```

✔️ Confirm that both services are working by opening the frontend external IP address in your browser. Check the version output of the service by running the following command

```bash
curl http://$FRONTEND_SERVICE_IP/version
```

set up a pipeline for deploying your changes continuously and reliably.

<br/>

## Creating the Jenkins Pipeline

✔️ Creating a repository to host the sample app source code

```bash
gcloud source repos create default
```

✔️ Initialize the sample-app directory as its own Git repository

```bash
git init
```

```bash
git config credential.helper gcloud.sh
```

```bash
git remote add origin
https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/default
```

Set the username and email address. Add, commit, and push the files.

```bash
git config --global user.email "[EMAIL_ADDRESS]"
git config --global user.name "[USERNAME]"

git add .
git commit -m "Initial commit"
git push origin master
```

✔️ Adding yout service account credentials

- **Step 1**: In the Jenkins user interface, click **Manage Jenkins** in the left navigation then click **Manage Credentials**.
- **Step 2**: Click **Jenkins**
- **Step 3**: Click **Global credentials (unrestricted)**.
- **Step 4**: Click **Add Credentials** in the left navigation.
- **Step 5**: Select **Google Service Account from metadata** from the **Kind** drop-down and click **OK**.

✔️ Creating the Jenkins job

- **Step 1**: Click **New Item** in the left navigation:
- **Step 2**: Name the project **sample-app**, then choose the **Multibranch Pipeline** option and click **OK**.
- **Step 3**: On the next page, in the **Branch Sources** section, click **Add Source** and select **git**.
- **Step 4**: Paste the **HTTPS clone URL** of your sample-app repo in Cloud Source Repositories into the **Project Repository** field. Replace `[PROJECT_ID]` with your **Project ID**:
  ```bash
  https://source.developers.google.com/p/[PROJECT_ID]/r/default
  ```
- **Step 5**: From the **Credentials** drop-down, select the name of the credentials you created when adding your service account in the previous steps.
- **Step 6**: Under **Scan Multibranch Pipeline Triggers** section, check the **Periodically if not otherwise run** box and set the **Interval** value to 1 minute.
- **Step 7**: Your job configuration should look like this:
- **Step 8**: Click **Save** leaving all other options with their defaults.

<br/>

## Creating the Development Environment

✔️ Creating a development branch

```bash
git checkout -b new-feature
```

✔️ Modifying the pipeline definition

Open the Jenkinsfile in your terminal editor

```bash
vi Jenkinsfile
```

```bash
PROJECT = "REPLACE_WITH_YOUR_PROJECT_ID"
APP_NAME = "gceme"
FE_SVC_NAME = "${APP_NAME}-frontend"
CLUSTER = "jenkins-cd"
CLUSTER_ZONE = "us-west1-b"
IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
JENKINS_CRED = "${PROJECT}"
```

✔️ Modify the site

Change the gceme card from blue to orange

```bash
vi html.go
```

```html
<div class="card orange"></div>
```

```html
vi main.go
```

```html
const version string = "2.0.0"
```

<br/>

## Kick off Deployment

✔️ Commit and push your changes

```bash
git add Jenkinsfile html.go main.go
git commit -m "Version 2.0.0"
git push origin new-feature
```

✔️ Start the proxy in the background

```bash
kubectl proxy &
```

✔️ Verify that your application is accessible by sending a request yo [localhost](http://localhost) and letting kubectl proxy forward it to your service

```bash
curl \
http://localhost:8001/api/v1/namespaces/new-
feature/services/gceme-frontend:80/proxy/version
```

<br/>

## Deploying a Canary Release

✔️ Create a canary branch and push it to the Git server

```bash
git checkout -b canary
git push origin canary
```

✔️ Check the service URL to ensure that some of the traffic is being served by your new version. Your should see about 1 in 5 requests (in no particular order) returning version `2.0.0` .

```bash
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --
namespace=production services gceme-frontend)
```

```bash
while true; do curl http://$FRONTEND_SERVICE_IP/version;
sleep 1; done
```

<br/>

## Deploying to production

✔️  Create a canary branch and push it to the Git server

```bash
git checkout master
git merge canary
git push origin master
```

✔️ Check the service URL to ensure that all of the traffic is being served by your new version, 2.0.0.

```bash
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --
namespace=production services gceme-frontend)
```

```bash
while true; do curl http://$FRONTEND_SERVICE_IP/version;
sleep 1; done
```

✔️ Navigate to site on which the gceme application displays the info cards.

```bash
kubectl get service gceme-frontend -n production
```
