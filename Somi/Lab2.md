# 세션 2

Google Kubernetes Engine은 배포, 관리 그리고 scaling your containerized application하는 기능을 제공한다.

GKE 클러스터를 실행 시, benefits

- [Load balancing](https://cloud.google.com/compute/docs/load-balancing-and-autoscaling) for Compute Engine instances
- [Node pools](https://cloud.google.com/kubernetes-engine/docs/node-pools) to designate subsets of nodes within a cluster for additional flexibility
- [Automatic scaling](https://cloud.google.com/kubernetes-engine/docs/cluster-autoscaler) of your cluster's node instance count
- [Automatic upgrades](https://cloud.google.com/kubernetes-engine/docs/node-auto-upgrade) for your cluster's node software
- [Node auto-repair](https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-repair) to maintain node health and availability
- [Logging and Monitoring](https://cloud.google.com/kubernetes-engine/docs/how-to/logging) with Cloud Monitoring for visibility into your cluster

# 1. Set a default compute zone

```bash
gcloud config set compute/zone us-central1-a
```

zone을 지정한다.

# 2. Create a CKE cluster

cluster는 최소 하나의 cluster master machine을 포함하며, ‘노드'라고 불리는 다수의 worker machine을 가지고 있다.

- Nodes : Compute Engine virtual machine (VM) instances that run the Kubernetes processes necessary to make them part of the cluster. 쿠버네티스 프로세스를 실행하는 가상머신 인스턴스들.

```bash
gcloud container clusters create [CLUSTER-NAME]
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/998d5e82-2441-4073-8db7-545b1987f62c/Untitled.png)

# 3. Get authentication credentials for the cluster

클러스터 생성 후 authentication credential을 받아 interact 해야 한다.

```bash
gcloud container clusters get-credentials [CLUSTER-NAME]
```

인증받기 위해서는 위 명령어를 사용한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ff089420-d4f2-4850-b6d4-b1099aa2a885/Untitled.png)

# 4. Deploy an application to the cluster

이제 클러스터에 컨테이너화 된 어플리케이션을 배포해보자.

GKE는 클러스터의 자원을 생성하고 관리하기 위해 Kubernetes objects를 사용한다.

```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```

- 위의 쿠버네티스 명령어는 `hello-server`라는 Deployment object를 생헝한다.
- `--image` 는 배포할 컨테이너 이미지를 명시한다.
- 명령어는 Container Registry 버킷으로부터 이미지를 pull 한다.
- `[gcr.io/google-samples/hello-app:1.0](http://gcr.io/google-samples/hello-app:1.0)` 은 pull 해 올 이미지의 상세 버전등을 명시한다. 만약 버전이 명시되어 있지 않으면 가장 최신 버전을 사용한다.

```bash
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```

쿠버네티스 서비스를 생성하기 위해 위 명령어를 사용한다.

- `--port` 는 컨테이너가 expose 할 포트 번호를 명시한다.
- `type="LoadBalancer"` 는 컨테이너를 위한 Compute Engine load balancer를 생성한다.

```bash
kubectl get service
```

`hello-server` 점검하기 위한 명령어

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5626e4d0-3f20-43ea-99a5-f283470d5eaf/Untitled.png)

```bash
http://[EXTERNAL-IP]:8080
```

위의 external ip 주소를 입력해서 웹 창에 치면 해당 어플리케이션을 웹 브라우저에서 확인할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9e8e22fd-a35f-44c7-a474-2c5e37b0a740/Untitled.png)

# 5. Deleting the cluster