# Lab3

# Lab3 : **Orchestrating the Cloud with Kubernetes**

Lab 목표

- Kubernetes Engine을 활용한 Kubernetes cluster provision
- `kubect1` 이용한 도커 컨테이너 배포 및 관리
- Kubernetes의 Deployments와 Services를 활용해 application을 microservice 로 쪼개기

```bash
gcloud config set compute/zone us-central1-b
```

zone 설정

```bash
gcloud container clusters create io
```

클러스터 생성

### 1. Get the sample code

```bash
gsutil cp -r gs://spls/gsp021/* .
```

GitHub repository를 클론한다.

필요한 디렉토리로 이동한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/827c6fa8-7fbb-40e4-8a5f-a0fbe1e242ef/Untitled.png)

### 2. Quick Kubernetes Demo

쿠버네티스를 시작하는 가장 쉬운 방법은 `kbect1 create` 커맨드를 사용하는 것이다.

```bash
kubectl create deployment nginx --image=nginx:1.10.0
```

위 명령어 사용하여 쿠버네티스 배포를 생성한다.

실행중인 노드에 장애가 발생하더라도 배포는 계속 실행된다.

쿠버네티스에서는 모든 컨테이너들이 pod에서 실행된다.

```bash
kubectl get pods
```

실행중인 nginx 컨테이너를 확인한다.

```bash
kubectl expose deployment nginx --port 80 --type LoadBalancer
```

nginx 컨테이너가 실행중이면 위 명령어를 통해 쿠버네티스 외부에 expose 할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/43f1a5cb-cd0f-4ecd-b4e7-1f2f41a0fa8d/Untitled.png)

쿠버네티스가 공용 IP 주소를 통해 외부 로드밸런서를 생성했다.

해당 공용 IP 주소를 조회하는 모든 클라이언트는 해당 pod로 라우팅된다.

위 경우에는 nginx pod로 라우팅된다.

```bash
kubectl get services
```

현재 사용중인 서비스 리스트를 조회한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cdeaf2dc-27af-4443-838f-0768a86da172/Untitled.png)

```bash
curl http://<External IP>:80
```

Nginx 컨테이너를 원격으로 hit 하기 위해 위 명령어를 사용한다.

### 3. Pods

쿠버네티스의 핵심에는 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) 가 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6132673d-10b1-42c1-b11c-38ab74b6c73e/Untitled.png)

Pods는 하나 이상의 컨테이너 집합을 나타낸다. 서로에 대한 의존도가 높은 여러개의 컨테이너가 있는 경우 컨테이너를 단일 pod 내에 패키징한다.

Pods에는 [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)가 있다.

볼륨은 pods가 살아있는 동안 지속되는 데이터 디스크로, 해당 pod의 컨테이너에서 사용 가능하다. Pods는 가지고 있는 내용에 대한 shared namespace를 제공하여  pod 내부의 컨테이너들이 서로 통신할 수 있고, 연결되어 있는 볼륨도 공유할 수 있도록 한다.

Pods는 network namespace도 공유하여 하나의 pod당 하나의 IP Address를 가지고 있다.

### 4. Creating pods

Pod는 pod configuration file을 통해 생성된다.

```bash
cat pods/monolith.yaml
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/30ce5a00-0177-4ff6-b445-e03214ab548c/Untitled.png)

- 현재 pod는 하나의 컨테이너 (monolith)로 이루어져있다.
- 컨테이너가 시작할 때 몇가지 argument들을 전달한다.
- http traffic 을 위해 80번 포트를 연다.

```bash
kubectl create -f pods/monolith.yaml
```

위 명령어를 사용해 monolith pod를 생성한다.

```bash
kubectl get pods
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a8ead290-b6e1-445f-aa9e-fc2d37e61fb7/Untitled.png)

default namespace에서 실행중인 모든 pod 리스트를 확인한다.

```bash
kubectl describe pods monolith
```

monolith pod 에 대한 정보들 출력

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d862c19b-9179-47ab-9467-1e22c64ffcb5/Untitled.png)

- Pod IP 주소와 event log를 포함한 정보들을 확인 가능하다.
    - troubleshooting 시 용이하게 사용가능
    

### 5. Interacting with pods

기본적으로 pod들은 할당된 사설 IP 주소이고 클러스터 외부에서 접근이 불가능하다. 따라서 `kubecgt1 port-forward` 명령어를 통해 로컬 포트를 monolith pod 내부로 매핑해야한다.

새로운 터미널을 열어 하나는 `kubect1 port-forward` 명령어를 실행하고 다른 터미널은 `curl` 명령어를 실행한다.

```bash
kubectl port-forward monolith 10080:80
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9617182f-e572-4f31-901a-f5d2b3eb3bf3/Untitled.png)

```bash
curl http://127.0.0.1:10080
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5c254b5f-d499-4770-a0f4-363ba312456e/Untitled.png)

`curl` 명령어를 이용해 pod와 소통하기

컨테이너에서 “hello” 메시지를 수신한 것을 확인할 수 있다.

```bash
curl http://127.0.0.1:10080/secure
```

authroization fail 응답을 수신한다.

```bash
curl -u user http://127.0.0.1:10080/login
```

password를 입력하라고 해서 입력하면 로그인이 성공하고, JWT token을 수신받는다.

```bash
TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
```

토큰 정보 변수 생성

```bash
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
```

Bearer Token을 담아 다시 보내면 제대로된 응답을 받게 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f8ee82b7-d14a-42cc-b5ee-69cb3be7d469/Untitled.png)

```bash
kubectl logs monolith
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49f15c05-cb1d-466a-81fd-e7997807adb9/Untitled.png)

monolith Pod의 로그를 보기 위해 위 명령어를 사용한다.

```bash
kubectl logs -f monolith
```

새로운 터미널에서 -f 옵션을 붙여 실행하면 실시간으로 발생하는 로그 스트림을 받을 수 있다.

```bash
kubectl exec monolith --stdin --tty -c monolith -- /bin/sh
```

Monolith Pod 내부에서 interactive shell을 실행하기 위해 위 명령어를 사용한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7b0fbedb-3632-486e-826a-e1e7f7bdb307/Untitled.png)

이런식으로 외부에 `ping` 명령어 등을 이용해 외부 연결성을 테스트할 수 있다.

```bash
exit
```

interactive shell을 종료할 때는 `exit` 명령어를 사용한다.

### 6. Services

Pods들은 영원하지않다. 만약 재시작되면 해당 pod들은 다른 IP 주소를 가지게 될 것이다. 이럴 때 [Services](https://kubernetes.io/docs/concepts/services-networking/service/)를 사용한다.

Services는 pods들의 고정 endpoint를 제공한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6d09ea05-82d9-44ad-bf65-50effd1a5f65/Untitled.png)

서비스는 라벨을 사용하여 어떤 pod에서 작동하는지 판별한다.

서비스 타입에 따른 service 단계

- ClusterIP (internal) : 해당 서비스가 클러스터 내부에서만 보인다
- NodePort : 클러스터의 각 노드에 외부에서 액세스할 수 있는 Ip를 제공
- LoadBalancer : 클라우드 제공자의 로드밸런서를 사용하여 서비스에서 노드로 트래픽을 전달한다.

이제 서비스를 생성하고 label selectors를 이용하여 외부에 노출하자.

### 7. Creating a service

서비스 생성 전 https traffic를 다루는 안전한 pod를 생성한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b7e6b93-b904-4c28-aa40-3ef23ff72d33/Untitled.png)

```bash
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
```

secure-monolith pods 와 configuration data를 생성한다.

```bash
cat services/monolith.yaml
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/43f4906c-83fa-425c-a2c0-e38a124bcaa4/Untitled.png)

monolith service configuration file

- 라벨 `app: monolith` 와 `secure: enbaled` 가 있는 pod들을 자동으로 찾고 expose 하는 selector가 있다.
- 여기에 nodeport를 expose해서 31000번 포트에서 ngins(port 443)으로 외부 트래픽을 전송해야한다.

```bash
kubectl create -f services/monolith.yaml
```

위 명령어를 사용해 monolith service configuration file로부터 monolith service를 생성한다.

이제 서비스를 expose 하기 위해 31000번 포트를 사용하고 있다.

다른 앱이 해당 포트에 바인딩하려고 하면 충돌이 생긴다.

일반적으로 쿠버네티스는 이러한 포트 assignment를 조정한다.

```bash
gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
```

위 명령어를 이용해 exposed node port의 monolith service로의 트래픽을 허용한다.

이제 클러스터 외부에서 port forwarding 없이도 secure-monolith service에 hit 할 수 있다.

```bash
gcloud compute instances list
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ff1bb267-724a-4255-aef8-6e3bda20fe28/Untitled.png)

위 명령어를 통해 노드 중 하나의 외부 IP 주소를 확인한다.

```bash
curl -k https://<EXTERNAL_IP>:31000
```

이러면 문제가 발생한다. 이제 해결해보자

### 8. Adding labels to pods

지금 monolith service는 endpoint를 가지고 있지 않다.

```bash
kubectl get pods -l "app=monolith"
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4937ffc3-2171-4e39-aed6-a68c46c74f57/Untitled.png)

monolith label을 가지고 있는 실행중인 pods들을 확인할 수 있다.

```bash
kubectl get pods -l "app=monolith,secure=enabled"
```

구체적인 라벨들을 확인하면?

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c76a0ebb-1c6c-441d-b5ec-703b6abcf21c/Untitled.png)

아무런 리소스도 없는 것을 확인할 수 있다.

그렇기 때문에 위에서 오류가 났던 것이고, `secure=enabled` 라벨을 추가해주어야한다.

```bash
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8772c22b-ba4a-456f-bc35-bdf21d2b9588/Untitled.png)

라벨을 붙이고 라벨이 업데이트 되었는지 확인한다.

```bash
kubectl describe services monolith | grep Endpoints
```

정상적으로 잘 붙었다면 monolith 서비스의 엔드포인트 리스트를 확인한다.

```bash
gcloud compute instances list
curl -k https://<EXTERNAL_IP>:31000
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e35ef7c-bf96-4dab-8d4b-fc142ebe9d1d/Untitled.png)

잘 돌아간다!

### 9. Deploying applications with Kubernetes

이제 컨테이너들을 scaling하고 매니징해보자.

[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#what-is-a-deployment)는 실행 중인 pods의 수가 사용자가 지정한 원하는 pods의 수와 같도록 하는 방법이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/34b82e34-2c5f-4cc7-a292-8c5a63bfb841/Untitled.png)

The main benefit of Deployments is in abstracting away the low level details of managing Pods. Behind the scenes Deployments use [Replica Sets](http://kubernetes.io/docs/user-guide/replicasets/)
 to manage starting and stopping the Pods. If Pods need to be updated or scaled, the Deployment will handle that. Deployment also handles restarting Pods if they happen to go down for some reason.

### 10. Creating deployments

현재 진행하던 앱을 세가지 파트로 나누면

- auth : 인증된 사용자들에게 JWT token 생성
- hello : 인증된 사용자들에게 인사
- frontend : auth와 hello 서비스들에 트래픽을 라우팅함.

auth와 hello deployments를 위한 내부적인 서비스들을 정의하고 frontend deployment를 위한 외부 서비스를 정의한다.

```bash
cat deployments/auth.yaml
```

auth deployment configuration file 생성

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4aec4ff3-183f-474d-aa25-085843c6e5d7/Untitled.png)

- deployment는 하나의 replica를 생성하는 것이다.
- auth 컨테이너의 버전 2.0.0 을 사용한다.
- When you run the `kubectl create`
 command to create the auth deployment it will make one pod that conforms to the data in the Deployment manifest. This means you can scale the number of Pods by changing the number specified in the Replicas field.

```bash
kubectl create -f deployments/auth.yaml
```

위 명령어로 deployment object를 생성한다.

```bash
kubectl create -f services/auth.yaml
```

auth deployment를 위한 서비스를 생성한다.

```bash
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
```

```bash
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```

동일하게 hello deployment와 frontend Deployment에도 적용한다.

```bash
kubectl get services frontend
curl -k https://<EXTERNAL-IP>
```

frontend의 외부 IP 주소를 통해 curling해서 interact 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5c3b5faa-5c27-40b7-8f4a-a3f7da1a00af/Untitled.png)

### Study More

- Pod가 무엇인지?
- LoadBalancer
- Scaling 이 무엇이냐악