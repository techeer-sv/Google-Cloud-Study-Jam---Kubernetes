# 주제

**Kubernetes를 통한 클라우드 조정**

- [Kubernetes Engine](https://cloud.google.com/container-engine)을 사용하여 완전한 [Kubernetes](http://kubernetes.io/) 클러스터를 프로비저닝합니다.
- `kubectl`을 사용하여 Docker 컨테이너를 배포하고 관리합니다.
- Kubernetes의 디플로이먼트 및 서비스를 사용하여 애플리케이션을 마이크로서비스로 분할합니다.

**<사용하는 image 목록>**

- [kelseyhightower/monolith](https://hub.docker.com/r/kelseyhightower/monolith) - 모놀리식에 auth 및 hello 서비스 포함
- [kelseyhightower/auth](https://hub.docker.com/r/kelseyhightower/auth) - auth 마이크로서비스로, 인증된 사용자를 위한 JWT 토큰 생성
- [kelseyhightower/hello](https://hub.docker.com/r/kelseyhightower/hello) - hello 마이크로서비스로, 인증된 사용자를 안내
- [ngnix](https://hub.docker.com/_/nginx) - auth 및 hello 서비스의 프런트엔드

# shell&description

```bash
Use “gcloud config set project [PROJECT_ID]” to change to a different project.

# us-centrall-b 로 영역 설정
student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ gcloud config set compute/zone us-central1-b
Updated property [compute/zone].

# 클러스터 시작
student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ gcloud container clusters create io
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters,please pass the --no-enable-ip-alias flag
Note: Your Pod address range (--cluster-ipv4-cidr) can accommodate at most 1008 node(s).
Creating cluster io in us-central1-b... Cluster is being configured...working     ...                        Creating cluster io in us-central1-b... Cluster is being configured...working
...            Creating cluster io in us-central1-b... Cluster is being configured...working
.
.
.
Creating cluster io in us-central1-b... Cluster is being health-checked (mast
er is healthy)...done.
Created [[https://container.googleapis.com/v1/projects/qwiklabs-gcp-00-afc8564ed978/zones/us-central1-b/clusters/io](https://container.googleapis.com/v1/projects/qwiklabs-gcp-00-afc8564ed978/zones/us-central1-b/clusters/io)].
To inspect the contents of your cluster, go to: [https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-b/io?project=qwiklabs-gcp-00-afc8564ed978](https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-b/io?project=qwiklabs-gcp-00-afc8564ed978)
kubeconfig entry generated for io.
NAME: io
LOCATION: us-central1-b
MASTER_VERSION: 1.22.8-gke.202
MASTER_IP: 34.68.240.174
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.22.8-gke.202
NUM_NODES: 3
STATUS: RUNNING
```

## 클러스터?

pods를 실행할 수 있는 일종의 환경이라고 이해하면 될 것 같다.

도화지

## gutil 실행 시 뜨는 메시지

```bash
==> NOTE: You are performing a sequence of gsutil operations that may
run significantly faster if you instead use gsutil -m cp ... Please
see the -m section under "gsutil help options" for further information
about when gsutil -m can be advantageous.
```

- -m

지원되는 작업(acl ch, acl set, cp, mv, rm, rsync, setmeta)이 동시에 실행됩니다. 이렇게 하면 비교적 빠른 네트워크 연결을 통해 다수의 파일에 작업을 수행할 때 성능이 크게 향상될 수 있습니다.
gsutil은 멀티 스레드 및 다중 처리를 조합하여 지정된 작업을 수행합니다. 스레드 및 프로세서의 수는 각각 `parallel_thread_count` 및 `parallel_process_count`에 따라 결정됩니다. 이러한 값은 .boto 구성 파일에서 설정되거나 `-o` 최상위 플래그를 사용하여 개별 요청에 지정됩니다. gsutil에는 제한 요청에 대한 기본 지원 기능이 없으므로 이러한 값을 사용해 실험해야 합니다. 최적 값은 네트워크 속도, CPU 수, 사용 가능한 메모리를 포함한 여러 요소에 따라 달라질 수 있습니다.
-m 옵션을 사용하면 상당한 네트워크 대역폭을 소비할 수 있으며 네트워크 속도가 느린 경우 문제가 생기거나 성능이 저하될 수 있습니다. 예를 들어 다수의 다른 중요한 작업에서도 사용하는 네트워크 링크를 통해 대규모 rsync 작업을 시작하면 해당 작업의 성능이 저하될 수 있습니다. 마찬가지로 -m 옵션을 사용하면 성능이 저하될 수 있습니다. 로컬 디스크를 '스래싱'할 수 있으므로 로컬에서 모든 작업을 수행하는 경우에 특히 그렇습니다.
이러한 문제를 방지하려면 `parallel_thread_count` 및 `parallel_process_count`의 값을 줄이거나 -m 옵션 사용을 완전히 중지합니다. gsutil에서 사용하는 I/O 용량의 양을 제한하고 로컬 디스크를 독점하지 못하도록 하기 위해 사용할 수 있는 도구 중 하나는 여러 Linux 시스템에 내장되어 있는 [ionice](http://www.tutorialspoint.com/unix_commands/ionice.htm)입니다. 예를 들어 다음 명령어는 로컬 디스크를 독점하지 않도록 gsutil의 I/O 우선순위를 줄입니다.

`ionice -c 2 -n 7 gsutil -m rsync -r ./dir gs://some bucket`

전체 전송이 완료되기 전에 동시 전송을 사용하는 다운로드나 업로드 작업이 실패하면(예: 파일 1,000개 중 300개가 전송된 후 실패) 전체 전송을 다시 시작해야 합니다.
또한 -m 플래그를 중지하면 오류 발생 시 일반적으로 대부분의 명령어가 실패합니다. 하지만 멀티 스레드 또는 멀티 프로세스로 -m을 사용 설정하면 모든 명령어가 계속 모든 작업을 시도하며 명령어 실행이 끝날 때 실패한 작업 횟수(있는 경우)가 예외로 보고됩니다.

### 파일 구조

```
deployments/  /* 디플로이먼트 매니페스트  */
  ...
nginx/        /* nginx 구성 파일 */
  ...
pods/         /* 포드 매니페스트 */
  ...
services/     /* 서비스 매니페스트 */
  ...
tls/          /* TLS 인증서 */
  ...
cleanup.sh    /* 정리 스크립트 */content_copy
```

```bash

student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create deployment nginx --image=nginx:1.10.0
error: failed to create deployment: Post "[http://localhost:8080/apis/apps/v1/namespaces/default/deployments?fieldManager=kubectl-create&fieldValidation=Strict](http://localhost:8080/apis/apps/v1/namespaces/default/deployments?fieldManager=kubectl-create&fieldValidation=Strict)": dial tcp 127.0.0.1:8080: connect: connection refused
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create deployment nginx --image=nginx:1.10.0kubectl create deployment nginx --image=nginx:1.10.0
error: exactly one NAME is required, got 4
See 'kubectl create deployment -h' for help and examples
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create deployment nginx--image=nginx:1.10.0kubectl create deployment nginx --image=nginx:1.10.0
error: exactly one NAME is required, got 4
See 'kubectl create deployment -h' for help and examples
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create deployment nginx --image=nginx:1.10.0
W0715 06:41:39.597186     851 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
deployment.apps/nginx created
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl get pods
W0715 06:41:48.689177     863 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
NAME                    READY   STATUS    RESTARTS   AGE
nginx-56cd7f6b6-lkmb9   1/1     Running   0          9s
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl expose deployment nginx --port 80 --type LoadBalancer
W0715 06:42:00.879048     870 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
service/nginx exposed
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl get services
W0715 06:42:08.481243     876 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S) AGE
kubernetes   ClusterIP      10.72.0.1     <none>        443/TCP 4m14s
nginx        LoadBalancer   10.72.0.248   <pending>     80:30508/TCP 7s
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ curl [http://10.72.0.248:80](http://10.72.0.248/)
curl: (28) Failed to connect to 10.72.0.248 port 80: Connection timedout
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl get services
W0715 06:45:26.546914     891 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)   AGE
kubernetes   ClusterIP      10.72.0.1     <none>          443/TCP   7m33s
nginx        LoadBalancer   10.72.0.248   35.224.188.12   80:30508/TCP   3m26s
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ curl [http://35.224.188.12:80](http://35.224.188.12/)
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
```

```bash
<p>For online documentation and support please refer to
<a href="[http://nginx.org/](http://nginx.org/)">nginx.org</a>.<br/>
Commercial support is available at
<a href="[http://nginx.com/](http://nginx.com/)">nginx.com</a>.</p>
```

```bash
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ cat pods/monolith.yaml
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
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create -f pods/monolith.yaml
W0715 06:47:08.812625     905 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
pod/monolith created
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl get pods
W0715 06:47:34.921021     914 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
NAME                    READY   STATUS    RESTARTS   AGE
monolith                1/1     Running   0          24s
nginx-56cd7f6b6-lkmb9   1/1     Running   0          5m55s
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl describe pods monolith
W0715 06:48:54.704203     922 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
Name:         monolith
Namespace:    default
Priority:     0
Node:         gke-io-default-pool-c5640bba-lfg7/10.128.0.4
Start Time:   Fri, 15 Jul 2022 06:47:11 +0000
Labels:       app=monolith
Annotations:  <none>
Status:       Running
IP:           10.68.2.8
IPs:
IP:  10.68.2.8
Containers:
monolith:
Container ID:  containerd://0ecc472f0946d9972b017f0caecbe8872b5ab41d93b326c145cc294e65c10412
Image:         kelseyhightower/monolith:1.0.0
Image ID:      sha256:980e09dd5c76f726e7369ac2c3aa9528fe3a8c92382b78e97aa54a4a32d3b187
Ports:         80/TCP, 81/TCP
Host Ports:    0/TCP, 0/TCP
Args:
-http=0.0.0.0:80
-health=0.0.0.0:81
-secret=secret
State:          Running
Started:      Fri, 15 Jul 2022 06:47:15 +0000
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
/var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-hr5d6 (ro)
Conditions:
Type              Status
Initialized       True
Ready             True
ContainersReady   True
PodScheduled      True
Volumes:
kube-api-access-hr5d6:
Type:                    Projected (a volume that contains injected data from mltiple sources)
TokenExpirationSeconds:  3607
ConfigMapName:           kube-root-ca.crt
ConfigMapOptional:       <nil>
DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 [node.kubernetes.io/not-ready:NoExecute](http://node.kubernetes.io/not-ready:NoExecute) op=Exists for300s
[node.kubernetes.io/unreachable:NoExecute](http://node.kubernetes.io/unreachable:NoExecute) op=Exists for 300s
Events:
Type    Reason     Age   From               Message
```

```bash
Normal  Scheduled  104s  default-scheduler  Successfully assigned default/monolith to gke-io-default-pool-c5640bba-lfg7
Normal  Pulling    101s  kubelet            Pulling image "kelseyhightower/monolith:1.0.0"
Normal  Pulled     100s  kubelet            Successfully pulled image "kelseyhightower/monolith:1.0.0" in 1.634438249s
Normal  Created    100s  kubelet            Created container monolith
Normal  Started    100s  kubelet            Started container monolith
student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kuberstudent_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwikl
abs-gcp-00-afc8564ed978)$ kubectl port-forward monolith 10
080:80
W0715 06:50:02.965391     930 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
Forwarding from 127.0.0.1:10080 -> 80
Handling connection for 10080
Handling connection for 10080
Handling connection for 10080
Handling connection for 10080
Handling connection for 10080
```

```
deployments/  /* 디플로이먼트 매니페스트  */
  ...
nginx/        /* nginx 구성 파일 */
  ...
pods/         /* 포드 매니페스트 */
  ...
services/     /* 서비스 매니페스트 */
  ...
tls/          /* TLS 인증서 */
  ...
cleanup.sh    /* 정리 스크립트 */content_copy
```

- 3
  ```bash
  Your Cloud Platform project in this session is set to qwiklabs-gcp-00-afc8564ed978.Use “gcloud config set project [PROJECT_ID]” to change to a different project.
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ curl [http://127.0.0.1:10080/secure](http://127.0.0.1:10080/secure)
  authorization failed
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ curl -u user [http://127.0.0.1:10080/login](http://127.0.0.1:10080/login)
  Enter host password for user 'user':
  {"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJleHAiOjE2NTgxMjczODUsImlhdCI6MTY1Nzg2ODE4NSwiaXNzIjoiYXV0aC5zZXJ2aWNlIiwic3ViIjoidXNlciJ9.hdTrxRgWyAS4wLdLTnJxyCgqF7ki4gxAuV8-0HlgqIs"}
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ TOKEN=$(curl [http://127.0.0.1:10080/login](http://127.0.0.1:10080/login) -u user|jq -r '.token')
  Enter host password for user 'user':
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
  Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:100   222  100   222    0     0    229      0 --:--:-- --:--:-- --:--:100   222  100   222    0     0    229      0 --:--:-- --:--:-- --:--:--   228
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ curl -H "Authorization: Bearer $TOKEN" [http://127.0.0.1:10080/secure](http://127.0.0.1:10080/secure)
  {"message":"Hello"}
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ kubectl logs monolith
  W0715 06:56:49.389474    1035 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  2022/07/15 06:47:15 Starting server...
  2022/07/15 06:47:15 Health service listening on 0.0.0.0:81
  2022/07/15 06:47:15 HTTP service listening on 0.0.0.0:80
  127.0.0.1:41512 - - [Fri, 15 Jul 2022 06:56:15 UTC] "GET /secure HTTP/1.1" curl/7.74.0
  127.0.0.1:41520 - - [Fri, 15 Jul 2022 06:56:25 UTC] "GET /login HTTP/1.1" curl/7.74.0
  127.0.0.1:41528 - - [Fri, 15 Jul 2022 06:56:33 UTC] "GET /login HTTP/1.1" curl/7.74.0
  127.0.0.1:41538 - - [Fri, 15 Jul 2022 06:56:41 UTC] "GET /secure HTTP/1.1" curl/7.74.0
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ kubectl logs -f monolith
  W0715 06:56:58.855530    1044 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  2022/07/15 06:47:15 Starting server...
  2022/07/15 06:47:15 Health service listening on 0.0.0.0:81
  2022/07/15 06:47:15 HTTP service listening on 0.0.0.0:80
  127.0.0.1:41512 - - [Fri, 15 Jul 2022 06:56:15 UTC] "GET /secure HTTP/1.1" curl/7.74.0
  127.0.0.1:41520 - - [Fri, 15 Jul 2022 06:56:25 UTC] "GET /login HTTP/1.1" curl/7.74.0
  127.0.0.1:41528 - - [Fri, 15 Jul 2022 06:56:33 UTC] "GET /login HTTP/1.1" curl/7.74.0
  127.0.0.1:41538 - - [Fri, 15 Jul 2022 06:56:41 UTC] "GET /secure HTTP/1.1" curl/7.74.0
  127.0.0.1:41566 - - [Fri, 15 Jul 2022 06:57:12 UTC] "GET / HTTP/1.1" curl/7.74.0
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$
  ```
- 4
  ```bash
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ curl [http://127.0.0.1:10080](http://127.0.0.1:10080/)
  {"message":"Hello"}
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ kubectl exec monolith --stdin --tty -c monolith /bin/sh
  kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
  W0715 06:57:31.463941    1052 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  / # ping -c 3 [google.com](http://google.com/)
  PING [google.com](http://google.com/) (142.250.136.102): 56 data bytes
  64 bytes from 142.250.136.102: seq=0 ttl=114 time=0.900 ms
  64 bytes from 142.250.136.102: seq=1 ttl=114 time=0.859 ms
  64 bytes from 142.250.136.102: seq=2 ttl=114 time=1.994 ms
  ```
  ```bash
  -- [google.com](http://google.com/) ping statistics ---
  3 packets transmitted, 3 packets received, 0% packet loss
  round-trip min/avg/max = 0.859/1.251/1.994 ms
  / # exit
  student_04_1f54aedf35da@cloudshell:~ (qwiklabs-gcp-00-afc8564ed978)$ cd ~/orchestrate-with-kubernetes/kubernetes
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ cat pods/secure-monolith.yaml
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
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create secret generic tls-certs --from-file tls/
  kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
  kubectl create -f pods/secure-monolith.yaml
  W0715 06:58:52.291766 1070 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  secret/tls-certs created
  W0715 06:58:52.991298 1076 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  configmap/nginx-proxy-conf created
  W0715 06:58:53.556370 1082 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  pod/secure-monolith created
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ cat services/monolith.yaml
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
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create -f services/monolith.yaml
  W0715 06:59:11.112715 1091 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  service/monolith created
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ service/monolith created
  -bash: service/monolith: No such file or directory
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
  Creating firewall...working..Created [[https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-afc8564ed978/global/firewalls/allow-monolith-nodeport](https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-afc8564ed978/global/firewalls/allow-monolith-nodeport)].
  Creating firewall...done.
  NAME: allow-monolith-nodeport
  NETWORK: default
  DIRECTION: INGRESS
  PRIORITY: 1000
  ALLOW: tcp:31000
  DENY:
  DISABLED: False
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
  Creating firewall...failed.
  ERROR: (gcloud.compute.firewall-rules.create) Could not fetch resource:
  ```
  ```bash
  The resource 'projects/qwiklabs-gcp-00-afc8564ed978/global/firewalls/allow-monolith-nodeport' already exists
  ```
  ```bash
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ gcloud compute instances list
  NAME: gke-io-default-pool-c5640bba-87rt
  ZONE: us-central1-b
  MACHINE_TYPE: e2-medium
  PREEMPTIBLE:
  INTERNAL_IP: 10.128.0.2
  EXTERNAL_IP: 34.170.244.157
  STATUS: RUNNING
  ```
  ```bash
  NAME: gke-io-default-pool-c5640bba-lfg7
  ZONE: us-central1-b
  MACHINE_TYPE: e2-medium
  PREEMPTIBLE:
  INTERNAL_IP: 10.128.0.4
  EXTERNAL_IP: 104.198.218.233
  STATUS: RUNNING
  ```
  ```bash
  NAME: gke-io-default-pool-c5640bba-qmlr
  ZONE: us-central1-b
  MACHINE_TYPE: e2-medium
  PREEMPTIBLE:
  INTERNAL_IP: 10.128.0.3
  EXTERNAL_IP: 34.173.9.16
  STATUS: RUNNING
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ curl -k https//34.173.9.16:31000curl: (6) Could not resolve host: https
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ curl -k [https://34.173.9.16:31000](https://34.173.9.16:31000/)
  curl: (7) Failed to connect to 34.173.9.16 port 31000: Connection refused
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl get pods -l "app=monolith"
  W0715 07:01:44.127716    1146 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  NAME              READY   STATUS    RESTARTS   AGE
  monolith          1/1     Running   0          14m
  secure-monolith   2/2     Running   0          2m50s
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl get pods -l "app=monolith,secure=enabled"
  W0715 07:01:59.288121    1152 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  No resources found in default namespace.
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl label pods secure-monolith 'secure=enabled'
  kubectl get pods secure-monolith --show-labels
  W0715 07:02:10.031309    1159 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  pod/secure-monolith labeled
  W0715 07:02:10.846482    1165 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  NAME              READY   STATUS    RESTARTS   AGE     LABELS
  secure-monolith   2/2     Running   0          3m17s   app=monolith,secure=enabled
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl describe services monolith | grep Endpoints
  W0715 07:02:16.648138    1171 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  Endpoints:                10.68.0.4:443
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ gcloud compute instances list
  curl -k https://10.68.0.4:443:31000
  NAME: gke-io-default-pool-c5640bba-87rt
  ZONE: us-central1-b
  MACHINE_TYPE: e2-medium
  PREEMPTIBLE:
  INTERNAL_IP: 10.128.0.2
  EXTERNAL_IP: 34.170.244.157
  STATUS: RUNNING
  ```
  ```bash
  NAME: gke-io-default-pool-c5640bba-lfg7
  ZONE: us-central1-b
  MACHINE_TYPE: e2-medium
  PREEMPTIBLE:
  INTERNAL_IP: 10.128.0.4
  EXTERNAL_IP: 104.198.218.233
  STATUS: RUNNING
  ```
  ```bash
  NAME: gke-io-default-pool-c5640bba-qmlr
  ZONE: us-central1-b
  MACHINE_TYPE: e2-medium
  PREEMPTIBLE:
  INTERNAL_IP: 10.128.0.3
  EXTERNAL_IP: 34.173.9.16
  STATUS: RUNNING
  curl: (3) URL using bad/illegal format or missing URL
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ cat deployments/auth.yaml
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
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create -f deployments/auth.yaml
  W0715 07:03:03.813185    1197 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  deployment.apps/auth created
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create -f services/auth.yaml
  W0715 07:03:07.739139    1204 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  service/auth created
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create -f deployments/hello.yaml
  kubectl create -f services/hello.yaml
  W0715 07:03:12.258043    1211 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  deployment.apps/hello created
  W0715 07:03:13.108629    1219 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  service/hello created
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
  kubectl create -f deployments/frontend.yaml
  kubectl create -f services/frontend.yaml
  W0715 07:03:17.201963    1225 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  configmap/nginx-frontend-conf created
  W0715 07:03:17.783584    1231 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  deployment.apps/frontend created
  W0715 07:03:18.654145    1238 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  service/frontend created
  student_04_1f54aedf35da@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-00-afc8564ed978)$ kubectl get services frontend
  W0715 07:03:22.336246    1245 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
  To learn more, consult [https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)
  NAME       TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
  frontend   LoadBalancer   10.72.3.49   <pending>     443:31507/TCP   3s
  ```

# summary
