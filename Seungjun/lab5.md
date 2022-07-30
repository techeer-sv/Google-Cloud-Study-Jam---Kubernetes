# Lab 5. Continuous Delivery with Jenkins in Kubernetes Engine

[Google LInk](https://www.cloudskillsboost.google/focuses/1104?parent=catalog)

## Overview

In this lab, you will learn how to set up a continuous delivery pipeline with `Jenkins` on Kubernetes engine. Jenkins is the go-to automation server used by developers who frequently integrate their code in a shared repository. The solution you'll build in this lab will be similar to the following diagram:

![image](https://user-images.githubusercontent.com/42485462/181916199-65189ab5-0e5a-4874-bb11-03895f28dcf6.png)

## What is Kubernetes Engine?

Kubernetes Engine is Google Cloud's hosted version of `Kubernetes` - a powerful cluster manager and orchestration system for containers. Kubernetes is an open source project that can run on many different environments—from laptops to high-availability multi-node clusters; from virtual machines to bare metal. As mentioned before, Kubernetes apps are built on `containers` - these are lightweight applications bundled with all the necessary dependencies and libraries to run them. This underlying structure makes Kubernetes applications highly available, secure, and quick to deploy—an ideal framework for cloud developers.

## What is Jenkins?

[Jenkins](https://www.jenkins.io/) is an open-source automation server that lets you flexibly orchestrate your build, test, and deployment pipelines. Jenkins allows developers to iterate quickly on projects without worrying about overhead issues that can stem from continuous delivery.

## What is Continuous Delivery / Continuous Deployment?

When you need to set up a continuous delivery (CD) pipeline, deploying Jenkins on Kubernetes Engine provides important benefits over a standard VM-based deployment.

When your build process uses containers, one virtual host can run jobs on multiple operating systems. Kubernetes Engine provides `ephemeral build executors`—these are only utilized when builds are actively running, which leaves resources for other cluster tasks such as batch processing jobs. Another benefit of ephemeral build executors is speed—they launch in a matter of seconds.

Kubernetes Engine also comes pre-equipped with Google's global load balancer, which you can use to automate web traffic routing to your instance(s). The load balancer handles SSL termination and utilizes a global IP address that's configured with Google's backbone network—coupled with your web front, this load balancer will always set your users on the fastest possible path to an application instance.

Now that you've learned a little bit about Kubernetes, Jenkins, and how the two interact in a CD pipeline, it's time to go build one.

## Task 1. Download the source code

1. Set the time zone as `us-west1-c`.

   ```bash
   $ gcloud config set compute/zone us-west1-c
   Updated property [compute/zone].
   ```

2. Copy the lab's sample code.

   ```bash
   $ gsutil cp gs://spls/gsp051/continuous-deployment-on-kubernetes.zip .
   Copying gs://spls/gsp051/continuous-deployment-on-kubernetes.zip...
   / [1 files][  3.3 MiB/  3.3 MiB]
   Operation completed over 1 objects/3.3 MiB.
   ```

   ```bash
   $ unzip continuous-deployment-on-kubernetes.zip
   Archive:  continuous-deployment-on-kubernetes.zip
   ...
   ```

3. Change to the correct directory.

   ```bash
   $ cd continuous-deployment-on-kubernetes
   ```

## Task 2. Provisioning Jenkins

1. Now, run the following command to provision a Kubernetes cluster:

   ```bash
   $ gcloud container clusters create jenkins-cd \
   --num-nodes 2 \
   --machine-type n1-standard-2 \
   --scopes "https://www.googleapis.com/auth/source.read_write,cloud-platform"
   Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
   Default change: During creation of nodepools or autoscaling configuration changes for cluster versions greater than 1.24.1-gke.800 a default location policy is applied. For Spot and PVM it defaults to ANY, and for all other VM kinds a BALANCED policy is used. To change the default values use the `--location-policy` flag.
   Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
   Creating cluster jenkins-cd in us-west1-c... Cluster is being health-checked (master is healthy)...done.
   Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-00-29f661737956/zones/us-west1-c/clusters/jenkins-cd].
   To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-c/jenkins-cd?project=qwiklabs-gcp-00-29f661737956
   kubeconfig entry generated for jenkins-cd.
   NAME: jenkins-cd
   LOCATION: us-west1-c
   MASTER_VERSION: 1.22.8-gke.202
   MASTER_IP: 34.127.92.71
   MACHINE_TYPE: n1-standard-2
   NODE_VERSION: 1.22.8-gke.202
   NUM_NODES: 2
   STATUS: RUNNING
   ```

   This step can take up to several minutes to complete. The extra scopes enable Jenkins to access Cloud Source Repositories and Google Container Registry.

2. Before continuing, confirm that your cluster is running by executing the following command:

   ```bash
   $ gcloud container clusters list
   NAME: jenkins-cd
   LOCATION: us-west1-c
   MASTER_VERSION: 1.22.8-gke.202
   MASTER_IP: 34.127.92.71
   MACHINE_TYPE: n1-standard-2
   NODE_VERSION: 1.22.8-gke.202
   NUM_NODES: 2
   STATUS: RUNNING
   ```

3. Now, get the credentials for your cluster:

   ```bash
   $ gcloud container clusters get-credentials jenkins-cd
   Fetching cluster endpoint and auth data.
   kubeconfig entry generated for jenkins-cd.
   ```

4. Kubernetes Engine uses these credentials to access your newly provisioned cluster—confirm that you can connect to it by running the following command:

   ```bash
   $ kubectl cluster-info
   Kubernetes control plane is running at https://34.127.92.71
   GLBCDefaultBackend is running at https://34.127.92.71/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
   KubeDNS is running at https://34.127.92.71/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
   Metrics-server is running at https://34.127.92.71/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

   To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
   ```

## Task 3. Setup Helm

In this lab, you will use `Helm` to install `Jenkins` from the `Charts` repository. Helm is a **package manager** that makes it easy to configure and deploy Kubernetes applications. Once you have Jenkins installed, you'll be able to set up your **CI/CD pipeline**.

1. Add Helm's stable chart repo:

   ```bash
   $ helm repo add jenkins https://charts.jenkins.io
   "jenkins" has been added to your repositories
   ```

2. Ensure the repo is up to date:

   ```bash
   $ helm repo update
   Hang tight while we grab the latest from your chart repositories...
   ...Successfully got an update from the "jenkins" chart repository
   Update Complete. ⎈Happy Helming!⎈
   ```

## Task 4. Configure and Install Jenkins

When installing Jenkins, a **`values` file can be used as a template** to provide values that are necessary for setup.

You will use a custom `values` file to automatically configure your Kubernetes Cloud and add the **following necessary plugins**:

- Kubernetes:1.29.4
- Workflow-multibranch:latest
- Git:4.7.1
- Configuration-as-code:1.51
- Google-oauth-plugin:latest
- Google-source-plugin:latest
- Google-storage-plugin:latest

This will allow Jenkins to connect to your cluster and your GCP project.

1. Download the custom values file:

   ```bash
   $ gsutil cp gs://spls/gsp330/values.yaml jenkins/values.yaml
   Copying gs://spls/gsp330/values.yaml...
   / [1 files][ 35.0 KiB/ 35.0 KiB]
   Operation completed over 1 objects/35.0 KiB.
   ```

2. Use the Helm CLI to **deploy the chart with your configuration settings**:

   ```bash
   $ helm install cd jenkins/jenkins -f jenkins/values.yaml --wait
   NAME: cd
   LAST DEPLOYED: Fri Jul 22 11:51:31 2022
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

   This command may take a couple minutes to complete.

3. Once that command completes ensure the Jenkins pod goes to the Running state and the container is in the READY state:

   ```bash
   $ kubectl get pods
   NAME           READY   STATUS    RESTARTS   AGE
   cd-jenkins-0   2/2     Running   0          110s
   ```

4. Configure the Jenkins service account to be able to deploy to the cluster:

   ```bash
   $ kubectl create clusterrolebinding jenkins-deploy --clusterrole=cluster-admin --serviceaccount=default:cd-jenkins
   clusterrolebinding.rbac.authorization.k8s.io/jenkins-deploy created
   ```

5. Run the following command to setup port forwarding to the Jenkins UI from the Cloud Shell:

   ```bash
   $ export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
   $ kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
   [1] 1014
   ```

6. Now, check that the Jenkins Service was created properly:

   ```bash
   $ kubectl get svc
   NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
   cd-jenkins         ClusterIP   10.56.15.249   <none>        8080/TCP    4m28s
   cd-jenkins-agent   ClusterIP   10.56.15.164   <none>        50000/TCP   4m28s
   kubernetes         ClusterIP   10.56.0.1      <none>        443/TCP     10m
   ```

You are using the [Kubernetes Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Kubernetes+Plugin) from Jenkins so that our builder nodes will be automatically launched as necessary when the Jenkins master requests them. Upon completion of their work, they will automatically be turned down and their resources added back to the clusters resource pool.

Notice that this service exposes ports `8080` and `50000` for any pods that match the `selector`. This will expose the **Jenkins web UI** and builder/agent registration ports within the Kubernetes cluster. Additionally, the `jenkins-ui` services is exposed using a ClusterIP so that it is not accessible from outside the cluster.

## Task 5. Connect to Jenkins

1. The Jenkins chart will automatically create an admin password for you. To retrieve it, run:

   ```bash
   $ printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
   admin-pw-XXXXXXXXXX
   ```

2. To get to the Jenkins user interface, click on the **Web Preview** button in Cloud Shell, then click **Preview on port 8080**:

   ![Web preview button in cloud shell](https://user-images.githubusercontent.com/42485462/181916165-ac80c5ec-7a2d-43b4-944a-aba958a01503.png)

3. You should now be able to log in with username `admin` and your auto-generated password.

You now have **Jenkins set up** in your Kubernetes cluster! Jenkins will drive your automated CI/CD pipelines in the next sections.

![Jenkins main](https://user-images.githubusercontent.com/42485462/181915766-3719c919-996a-4381-8375-a4a5ebe7bf47.png)

## Task 6. Understanding the Application

You'll deploy the sample application, `gceme`, in your continuous deployment pipeline. The application is written in the `Go` language and is located in the repo's sample-app directory. When you run the gceme binary on a Compute Engine instance, the app displays the instance's metadata in an info card.

![image](https://user-images.githubusercontent.com/42485462/181916180-14791464-ce60-4c12-b76d-2e0b1739a57a.png)

The application mimics a microservice by supporting two operation modes.

- In **backend mode**: gceme listens on port 8080 and returns Compute Engine instance metadata in JSON format.

- In **frontend mode**: gceme queries the backend gceme service and renders the resulting JSON in the user interface.

![Microservice with two modes](https://user-images.githubusercontent.com/42485462/181916151-236e1c3e-c9fb-48e1-83a6-a4834ba9b2da.png)

## Task 7. Deploying the Application

You will deploy the application into two different environments:

- **Production**: The live site that your users access.

- **Canary**: A smaller-capacity site that receives only a percentage of your user traffic. Use this environment to validate your software with live traffic before it's released to all of your users.

1. In Google Cloud Shell, navigate to the sample application directory:

   <img width="843" alt="ls result" src="https://user-images.githubusercontent.com/42485462/181915771-17307359-ee64-493c-b385-7436cf49ed34.png">

   ```bash
   $ cd sample-app
   ```

2. Create the Kubernetes namespace to logically isolate the deployment:

   ```bash
   $ kubectl create ns production
   namespace/production created
   ```

3. Create the production and canary deployments, and the services using the `kubectl apply` commands:

   ```bash
   $ kubectl apply -f k8s/production -n production
   deployment.apps/gceme-backend-production created
   deployment.apps/gceme-frontend-production created

   $ kubectl apply -f k8s/canary -n production
   deployment.apps/gceme-backend-canary created
   deployment.apps/gceme-frontend-canary created

   $ kubectl apply -f k8s/services -n production
   service/gceme-backend created
   service/gceme-frontend created
   ```

   By default, only one replica of the frontend is deployed. Use the `kubectl scale` command to ensure that there are **at least 4 replicas running at all times**.

4. Scale up the production environment frontends by running the following command:

   ```bash
   $ kubectl scale deployment gceme-frontend-production -n production --replicas 4
   deployment.apps/gceme-frontend-production scaled
   ```

5. Now confirm that you have **5 pods running for the frontend**: **4 for production traffic** and **1 for canary releases** (changes to the canary release will only affect **1 out of 5 (20%)** of users):

   ```bash
   $ kubectl get pods -n production -l app=gceme -l role=frontend
   NAME                                         READY   STATUS    RESTARTS   AGE
   gceme-frontend-canary-649bd655bd-bjhv4       1/1     Running   0          2m49s
   gceme-frontend-production-78887ccfbf-5ql99   1/1     Running   0          52s
   gceme-frontend-production-78887ccfbf-lfmkc   1/1     Running   0          52s
   gceme-frontend-production-78887ccfbf-mdd82   1/1     Running   0          52s
   gceme-frontend-production-78887ccfbf-mz8rf   1/1     Running   0          2m59s
   ```

6. Also confirm that you have **2 pods for the backend**: **1 for production** and **1 for canary**:

   ```bash
   $ kubectl get pods -n production -l app=gceme -l role=backend
   NAME                                        READY   STATUS    RESTARTS   AGE
   gceme-backend-canary-6fb5c7bb66-dw6fs       1/1     Running   0          3m30s
   gceme-backend-production-586b4b46cc-cnhhv   1/1     Running   0          3m41s
   ```

7. Retrieve the external IP for the production services:

   ```
   $ kubectl get service gceme-frontend -n production
   NAME             TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)        AGE
   gceme-frontend   LoadBalancer   10.56.11.94   35.230.26.54   80:32123/TCP   4m
   ```

   > Note: It can take several minutes before you see the load balancer external IP address.

   Paste External IP into a browser to see the info card displayed on a card—you should get a similar page:

   ![Blue cards](https://user-images.githubusercontent.com/42485462/181916021-72a36ab5-f5ae-46f3-8d47-be118fadf16f.png)

8. Now, store the _frontend service_ load balancer IP in an environment variable for use later:

   ```bash
   $ export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
   ```

9. Confirm that both services are working by opening the frontend external IP address in your browser.

10. Check the version output of the service by running the following command (it should read 1.0.0):

    ```bash
    $ curl http://$FRONTEND_SERVICE_IP/version
    1.0.0
    ```

You have successfully deployed the sample application! Next, you will set up a pipeline for deploying your changes continuously and reliably.

## Task 8. Creating the Jenkins Pipeline

### Creating a repository to host the sample app source code

1. Create a copy of the gceme sample app and push it to a Cloud Source Repository:

   ```bash
   $ gcloud source repos create default
   Created [default].
   WARNING: You may be billed for this repository. See https://cloud.google.com/source-repositories/docs/pricing for details.
   ```

   You can ignore the warning, you will not be billed for this repository.

   ```bash
   $ git init
   ```

2. Initialize the sample-app directory as its own Git repository:

   ```bash
   $ git config credential.helper gcloud.sh
   ```

3. Run the following command:

   ```bash
   $ git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/default
   ```

4. Set the username and email address for your Git commits. Replace [EMAIL_ADDRESS] with your Git email address and [USERNAME] with your Git username:

   ```bash
   $ git config --global user.email "[EMAIL_ADDRESS]"
   $ git config --global user.name "[USERNAME]"
   ```

5. Add, commit, and push the files:

   ```bash
   $ git add .

   $ git commit -m "Initial commit"
   [master (root-commit) 71ecf56] Initial commit
   31 files changed, 2470 insertions(+)
   ...

   $ git push origin master
   Enumerating objects: 48, done.
   Counting objects: 100% (48/48), done.
   Delta compression using up to 2 threads
   Compressing objects: 100% (43/43), done.
   Writing objects: 100% (48/48), 27.19 KiB | 2.27 MiB/s, done.
   Total 48 (delta 11), reused 0 (delta 0), pack-reused 0
   remote: Resolving deltas: 100% (11/11)
   To https://source.developers.google.com/p/qwiklabs-gcp-00-XXXXXXXXXXXX/r/default
   * [new branch]      master -> master
   ```

### Adding your service account credentials

Configure your credentials to allow Jenkins to access the code repository. Jenkins will use your cluster's service account credentials in order to download code from the Cloud Source Repositories.

**Step 1**: In the Jenkins user interface, click Manage Jenkins in the left navigation then click **Manage Credentials**.

**Step 2**: Click **Jenkins**

![Jenkins - Click Jenkins](https://user-images.githubusercontent.com/42485462/181916448-5ad310ea-742c-4c56-af5d-d90d35c2cbc1.png)

**Step 3**: Click **Global credentials (unrestricted)**.

**Step 4**: Click **Add Credentials** in the left navigation.

**Step 5**: Select **Google Service Account from metadata** from the **Kind** drop-down and click **OK**.

The global credentials has been added. The name of the credential is the `Project ID` found in the `CONNECTION DETAILS` section of the lab.

![Jenkins Global credentials](https://user-images.githubusercontent.com/42485462/181916466-56ca731e-462e-4f9e-b5ac-46ff355ea0a1.png)

### Creating the Jenkins job

Navigate to your Jenkins user interface and follow these steps to configure a Pipeline job.

**Step 1**: Click **Dashboard** > **New Item** in the left panel.

**Step 2**: Name the project **sample-app**, then choose the **Multibranch Pipeline** option and click **OK**.

![Jenkins new job](https://user-images.githubusercontent.com/42485462/181915772-6ba91672-c9a3-43ac-a060-b3869e93e732.png)

**Step 3**: On the next page, in the **Branch Sources** section, click **Add Source** and select **git**.

**Step 4**: Paste the **HTTPS clone URL** of your sample-app repo in Cloud Source Repositories into the **Project Repository** field. Replace `[PROJECT_ID]` with your **Project ID**:

```sh
$ https://source.developers.google.com/p/[PROJECT_ID]/r/default
```

**Step 5**: From the **Credentials** drop-down, select the name of the credentials you created when adding your service account in the previous steps.

**Step 6**: Under **Scan Multibranch Pipeline Triggers** section, check the **Periodically if not otherwise run** box and set the **Interval** value to 1 minute.

**Step 7**: Your job configuration should look like this:

![Jenkins job configuration example](https://user-images.githubusercontent.com/42485462/181916377-050f0fe0-291a-4b30-9658-50dccd0feae2.png)

**Step 8**: Click **Save** leaving all other options with their defaults.

After you complete these steps, a job named `Branch indexing` runs. This meta-job identifies the branches in your repository and ensures changes haven't occurred in existing branches. If you click sample-app in the top left, the `master` job should be seen.

> **Note:** The first run of the master job might fail until you make a few code changes in the next step.

You have successfully created a Jenkins pipeline! Next, you'll create the development environment for continuous integration.

## Task 9. Creating the development environment

Development branches are a set of environments your developers use to test their code changes before submitting them for integration into the live site. These environments are scaled-down versions of your application, but need to be deployed using the same mechanisms as the live environment.

### Creating a development branch

To create a development environment from a feature branch, you can push the branch to the Git server and let Jenkins deploy your environment.

- Create a development branch and push it to the Git server:

  ```bash
  $ git checkout -b new-feature
  ```

### Modifying the pipeline definition

The `Jenkinsfile` that defines that pipeline is written using the [Jenkins Pipeline Groovy syntax](https://www.jenkins.io/doc/book/pipeline/syntax/). Using a `Jenkinsfile` allows an entire build pipeline to be expressed in a single file that lives alongside your source code. Pipelines support powerful features like parallelization and require manual user approval.

In order for the pipeline to work as expected, you need to modify the `Jenkinsfile` to set your project ID.

1. Open the Jenkinsfile in your terminal editor, for example `vi`:

   ```bash
   $ vi Jenkinsfile
   ```

   > **Notes from litsynp:** You gotta change the region from `us-east1-d` to `us-west1-c` for it to work.

2. Start the editor:

   ```bash
   i
   ```

3. Add your `PROJECT_ID` to the `REPLACE_WITH_YOUR_PROJECT_ID` value. (Your `PROJECT_ID` is your Project ID found in the `CONNECTION DETAILS` section of the lab. You can also run `gcloud config get-value project` to find it.

4. Change the `CLUSTER_ZONE` to the default `compute/zone` you set at the beginning of this lab. You can get this value by running `gcloud config get compute/zone`.

   ```
   PROJECT = "REPLACE_WITH_YOUR_PROJECT_ID"
   APP_NAME = "gceme"
   FE_SVC_NAME = "${APP_NAME}-frontend"
   CLUSTER = "jenkins-cd"
   CLUSTER_ZONE = ""
   IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
   JENKINS_CRED = "${PROJECT}"
   ```

5. Save the `Jenkinsfile` file: hit **Esc** then (for `vi` users):

   ```bash
   :wq
   ```

### Modify the site

To demonstrate changing the application, you will change the gceme cards from **blue** to **orange**.

1. Open `html.go`:

   ```bash
   $ vi html.go
   ```

2. Start the editor:

   ```bash
   i
   ```

3. Change the two instances of `<div class="card blue">` with following:

   ```html
   <div class="card orange"></div>
   ```

4. Save the `html.go` file: press **Esc** then:

   ```bash
   :wq
   ```

5. Open `main.go`:

6. Start the editor:

   ```bash
   i
   ```

7. The version is defined in this line:

   ```go
   const version string = "1.0.0"
   ```

   ```go
   const version string = "2.0.0"
   ```

8. Save the `main.go` file one more time: **Esc** then:

   ```
   :wq
   ```

## Task 10. Kick off Deployment

1. Commit and push your changes:

   ```bash
   $ git add Jenkinsfile html.go main.go
   ```

   ```bash
   $ git commit -m "Version 2.0.0"
   ```

   ```bash
   $ git push origin new-feature
   ```

   This will kick off a build of your development environment.

   After the change is pushed to the Git repository, navigate to the Jenkins user interface where you can see that your build started for the `new-feature` branch. It can take up to a minute for the changes to be picked up.

2. After the build is running, click the down arrow next to the **build** in the left navigation and select **Console output**:

3. Track the output of the build for a few minutes and watch for the `kubectl --namespace=new-feature apply...` messages to begin. Your new-feature branch will now be deployed to your cluster.

   > **Note:** In a development scenario, you wouldn't use a public-facing load balancer. To help secure your application, you can use [kubectl proxy](https://kubernetes.io/docs/tasks/extend-kubernetes/http-proxy-access-api/). The proxy authenticates itself with the Kubernetes API and proxies requests from your local machine to the service in the cluster without exposing your service to the Internet.

   If you didn't see anything in `Build Executor`, not to worry. Just go to the Jenkins homepage > sample app. Verify that the `new-feature` pipeline has been created.

4. Once that's all taken care of, start the proxy in the background:

   ```bash
   $ kubectl proxy &
   ```

5. If it stalls, press **Ctrl + C** to exit out. Verify that your application is accessible by sending a request to localhost and letting kubectl proxy forward it to your service:

   ```bash
   $ curl \
   http://localhost:8001/api/v1/namespaces/new-feature/services/gceme-frontend:80/proxy/version
   2.0.0
   ```

   You should see it respond with 2.0.0, which is the version that is now running.

   If you receive a similar error:

   ```json
   {
      "kind": "Status",
      "apiVersion": "v1",
      "metadata": {
      },
      "status": "Failure",
      "message": "no endpoints available for service \"gceme-frontend:80\"",
      "reason": "ServiceUnavailable",
      "code": 503
   ```

6. It means your frontend endpoint hasn't propagated yet—wait a little bit and try the curl command again. Move on when you get the following output:
   ```
   2.0.0
   ```
   You have set up the development environment! Next, you will build on what you learned in the previous module by deploying a canary release to test out a new feature.

## Task 11. Deploying a canary release

You have verified that your app is running the latest code in the development environment, so now deploy that code to the canary environment.

1. Create a canary branch and push it to the Git server:

   ```bash
   $ git checkout -b canary
   Switched to a new branch 'canary'
   ```

   ```bash
   $ git push origin canary
   Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
   To https://source.developers.google.com/p/qwiklabs-gcp-00-XXXXXXXXXXXX/r/default
   * [new branch]      canary -> canary
   ```

2. In Jenkins, you should see the **canary** pipeline has kicked off. Once complete, you can check the service URL to ensure that some of the traffic is being served by your new version. You should see about 1 in 5 requests (in no particular order) returning version `2.0.0`.

   ```bash
   $ export FRONTEND_SERVICE_IP=$(kubectl get -o \
   jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
   ```

   ```bash
   $ while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
   1.0.0
   1.0.0
   1.0.0
   1.0.0
   2.0.0
   1.0.0
   2.0.0
   ...
   ```

3. If you keep seeing 1.0.0, try running the above commands again. Once you've verified that the above works, end the command with **Ctrl + C**.

That's it! You have deployed a canary release. Next you will deploy the new version to production.

## Task 12. Deploying to production

Now that our canary release was successful and we haven't heard any customer complaints, deploy to the rest of your production fleet.

1. Create a canary branch and push it to the Git server:

   ```bash
   $ git checkout master
   Switched to branch 'master'
   ```

   ```bash
   $ git merge canary
   Updating 71ecf56..87ff749
   Fast-forward
   Jenkinsfile | 4 ++--
   html.go     | 4 ++--
   main.go     | 2 +-
   3 files changed, 5 insertions(+), 5 deletions(-)
   ```

   ```bash
   $ git push origin master
   Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
   To https://source.developers.google.com/p/qwiklabs-gcp-00-XXXXXXXXXXXX/r/default
     71ecf56..87ff749  master -> master
   ```

   In Jenkins, you should see the master pipeline has kicked off.

2. Once complete (which may take a few minutes), you can check the service URL to ensure that all of the traffic is being served by your new version, **2.0.0**.

   ```bash
   $ export FRONTEND_SERVICE_IP=$(kubectl get -o \
   jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
   ```

   ```bash
   $ while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
   1.0.0
   1.0.0
   2.0.0
   1.0.0
   1.0.0
   2.0.0
   1.0.0
   1.0.0
   2.0.0
   1.0.0
   2.0.0 # Now I'm seeing only 2.0.0
   2.0.0
   2.0.0
   2.0.0
   2.0.0
   ...
   ```

3. Once again, if you see instances of 1.0.0 try running the above commands again. You can stop this command by pressing **Ctrl + C**.

   You can also navigate to site on which the gceme application displays the info cards. The card color changed from blue to orange.

4. Here's the command again to get the external IP address so you can check it out:

   ```bash
   $ kubectl get service gceme-frontend -n production
   NAME             TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)        AGE
   gceme-frontend   LoadBalancer   10.56.11.94   35.230.26.54   80:32123/TCP   33m
   ```

   ![Orange cards](https://user-images.githubusercontent.com/42485462/181916027-f396159b-76f7-45af-ad4f-0b8f0f35f1ac.png)

## Task 13. Test your Understanding

Below are multiple-choice questions to reinforce your understanding of this lab's concepts. Answer them to the best of your abilities.

### Q1. Which are the following Kubernetes namespaces used in the lab?

- [x] production
- [x] kube-system
- [ ] helm
- [ ] jenkins
- [x] default

### Q2. The Helm chart is a collection of files that describe a related set of Kubernetes resources.

True
