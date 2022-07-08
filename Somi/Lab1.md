# 세션 1

### 세션 목표

- Docker Container build, run, debug
- Docker Hub로부터 Docker images pull 하기
- Google Container Registry 에 Docker image push 하기

# Google Cloud

Cloud Shell 은 development tools 와 로드되는 가상 머신이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69a2e7fd-a930-46a8-b745-516f395d5405/Untitled.png)

`gcloud` is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.

### Auth list 확인

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/63d694c4-a4a7-43ab-acbe-2835f806ab33/Untitled.png)

```bash
docker run hello-world
```

도커 데몬은 hello-world 이미지를 로컬에서 찾지 못하면 Docker Hub에서 이미지를 pull하여 컨테이너를 생성하고 실행한다.

이미지를 풀 한 후 다시 해당 명령어를 실행하면 이제 docker daemon은 local registry에서 이미지를 찾고, 그 이미지로부터 컨테이너를 실행한다.

# Build

```bash
cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:lts
# Set the working directory in the container to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app
# Make the container's port 80 available to the outside world
EXPOSE 80
# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
```

- The initial line specifies the base parent image, which in this case is the official Docker image for node version long term support (lts).
- In the second, you set the working (current) directory of the container.
- In the third, you add the current directory's contents (indicated by the `"."` ) into the container.
- Then expose the container's port so it can accept connections on that port and finally run the node command to start the application.

### node application

```bash
cat > app.js <<EOF
const http = require('http');
const hostname = '0.0.0.0';
const port = 80;
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
});
server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});
process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
```

위 파일 작성 후 image를 build한다.

```bash
docker build -t node-app:0.1 .
```

- -t 명령어는 name: tag 로 태그를 지정하기 위한 명령어.
- 따라서 이미지의 이름은 node-app 이고, tag는 0.1이 된다.
- 태그를 지정하지 않으면 `latest` 로 저장된다. 이렇게 되면 새로운 이미지가 생겼을 때 헷갈리기 때문에 태그를 반드시 저장하는 것이 중요하다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/89a033df-cbdc-4ed7-942c-2735c303d7a2/Untitled.png)

이미지 빌드 후 도커 이미지 목록을 확인한다.

# Run

위에서 빌드한 이미지 기반의 컨테이너를 실행하기 위해서는 다음 명령어를 실행한다.

```bash
docker run -p 4000:80 --name my-app node-app:0.1
```

- `—name` 은 컨테이너 이름을 지정할 수 있도록 하는 명령어이다.
- `-p` 는 도커가 호스트의 포트 4000번을 컨테이너의 80번 포트로 매핑하도록한다.
- 따라서 호스트는 [http://localhost:4000](http://localhost:4000) 에서 서버를 확인할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2e6a4eb6-f5d4-4f2d-adde-7fee8a32f073/Untitled.png)

도커 컨테이너를 띄운 후 다른 터미널에서 아래 명령어를 확인해본다.

```bash
curl http://localhost:4000
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f3ddd3c-198e-4abf-99c4-2feab0711d0c/Untitled.png)

다음과 같은 명령어를 확인할 수 있다.

만약 컨테이너가 백그라운드에서 데몬으로 돌아가는지 확인하기 위해서는 `-d` flag를 붙인다.

```bash
docker run -p 4000:80 --name my-app -d node-app:0.1
docker ps
```

도커 내렸다가 background에서 run 하도록 하는 명령어

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4c059eca-cfdb-4fea-a067-62880c2c537c/Untitled.png)

컨테이너 아이디를 통해 로그를 확인한다.

```bash
docker build -t node-app:0.2 .
```

app.js 파일 변경 후 다시 이미지를 빌드하고 새로운 태그를 붙인다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/24cb271f-7dc5-4cda-90ba-f4e0303f900f/Untitled.png)

step2 에서는 이미 존재하는 cache layer를 사용한다.

Step3 부터는 app.js에서 변경사항이 있었기 때문에 layer들도 수정된다.

```bash
docker run -p 8080:80 --name my-app-2 -d node-app:0.2
docker ps
```

새로운 이미지를 이용한 컨테이너를 실행한다.

이미 호스트의 4000번 포트를 사용중이기 때문에 다른 포트번호를 사용한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/727b269b-4855-4e24-aa69-6641a7caa665/Untitled.png)

# Debug

컨테이너 아이디를 통해 도커 로그를 확인한다.

```bash
docker logs -f [container_id]
```

```bash
docker exec -it [container_id] bash
```

컨테이너안에서 interactive Bash session 실행

- `-it` flag는 pseudo-tty를 할당함으로써 컨테이너와 상호작용할 수 있도록 한다.
- bash는 `WORKDIR` 인 /app 에서 실행된다는 것을 기억하자. (`Dockerfile` 에서 작성함)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49c1ba91-f5d2-47ce-80b2-06c1a1ea274d/Untitled.png)

```bash
docker inspect [container_id]
```

# Publish

이제 이미지를 Google Container Registry (gcr)에 푸시해보자!

그 후 컨테이너와 이미지들을 삭제할 것이고, 그 다음 풀하고 컨테이너를 실행할 것이다.

gcr에 이미지를 푸시하기 위해서는 이미지 이름을 registry name으로 태그해야한다.

형식은 `[hostname]/[project-id]/[image]:[tag]` 이다.

For gcr:

- `[hostname]`= gcr.io
- `[project-id]`= your project's ID
- `[image]`= your image name
- `[tag]`= any string tag of your choice. If unspecified, it defaults to "latest".

project ID는 아래 명령어를 실행해 확인한다.

```bash
gcloud config list project
```

node-app:0.2 를 태깅해보자.

```bash
docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cb0936c4-5593-4e02-8640-807d6889c6e9/Untitled.png)

이제 이 이미지를 gcr에 푸시해보자.

```bash
docker push gcr.io/[project-id]/node-app:0.2
```

이제 Google Cloud에서 이미지가 잘 푸시되었는지 확인한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/618fae04-fd5c-490b-a12d-917f3f6d713e/Untitled.png)

### 이미지 테스트

다양한 방법으로 시도 할 수 있지만 간단히 존재하던 기존의 컨테이너들을 삭제한다.

```bash
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```

node image를 지우기 전에 `node:lts` 의 child images 들도 삭제해야한다.

다음 명령어를 사용한다.

```bash
docker rmi node-app:0.2 gcr.io/[project-id]/node-app node-app:0.1
docker rmi node:lts
docker rmi $(docker images -aq) # remove remaining images
docker images
```

이제 이미지를 pull 하고 실행해보자

```bash
docker pull gcr.io/[project-id]/node-app:0.2
docker run -p 4000:80 -d gcr.io/[project-id]/node-app:0.2
curl http://localhost:4000
```
