# Lab 1 : Intoduction to Docker

📒 [link](https://www.cloudskillsboost.google/focuses/1029?parent=catalog)

<br/>

## **Overview**

- What is Docker?
  - Docker is an **open platform for developing, shipping, and running applications**.
    With Docker, you can **separate your applications from your infrastructure**
    and **treat your infrastructure like a managed application**.
  - Docker helps you **ship code faster, test faster, deploy faster,
    and shorten the cycle** between writing code and running code.

<br/>

## **Hello World**

<br/>

✔️ Run a hello world container

```bash
docker run hello-world
```

<br/>

✔️ Take a look at the container image

```bash
docker images
```

<br/>

✔️ Look at the runnung containers

```bash
docker ps
```

<br/>

If you want to see all containers, including ones that have finished executing, run `docker pa -a`

<br/>

## **Build**

<br/>

✔️ Build a Docker image based on a simple node application.

```bash
# Switch into a folder named test

mkdir test && cd test
```

```docker
# Create a Dockerfile

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

```jsx
// Create the node appllication

const http = require("http");
const hostname = "0.0.0.0";
const port = 80;
const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader("Content-Type", "text/plain");
  res.end("Hello World\n");
});
server.listen(port, hostname, () => {
  console.log("Server running at http://%s:%s/", hostname, port);
});
process.on("SIGINT", function () {
  console.log("Caught interrupt signal and will exit");
  process.exit();
});
```

<br/>
Now build the image.

```jsx
docker build =t node-app:0.1 .
```

<br/>

## **Run**

<br/>

✔️ Run containers based on the image you built

```bash
docker run -p 4000:80 --name my-app:0.1
```

<br/>

✔️ Stop and remove the container

```bash
docker stop my-app && docker rm my-app
```

<br/>

✔️ Start the container in the background

```bash
docker run -p 4000:80 --name my-app -d node-app:0.1
```

<br/>

✔️ Edit app.js

```jsx
....
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Welcome to Cloud\n');
});
....
```

Build this new image and tag it with 0.2

```bash
docker build -t node-app:0.2 .
```

Run another container with the new image version.

```bash
docker run -p 8080:80 --name my-app-2 -d node-app:0.2
```

<br/>

## **Debug**

<br/>
✔️ Look at the logs of a container

```bash
docker logs [container_id]
```

If you want to follow the log’s output as the container is running, use th -f option. `docker logs -f [container_id]`

<br/>
✔️ Start an interactive Bash session inside the running container.

```bash
docker exec -t [container_id] bash
```

<br/>
✔️ Examine a container’s metadata in Docker by using Docker inspect

```bash
docker inspect [container_id]
```

<br/>

## **Publish**

<br/>

✔️ Find your project ID by running

```bash
gcloud config list project
```

<br/>

✔️ Tag node-app:0.2.

```bash
docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2
```

<br/>
✔️ Push image to gcr

```bash
docker push gcr.io/[project-id]/node-app:0.2
```

<br/>
✔️ Check that the image exists in gcr

<br/>
✔️ Stop and remove all container

```bash
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```

<br/>
Remove the child images before you remove the node image

```bash
docker rmi node-app:0.2 gcr.io/[project-id]/node-app node-app:0.1
docker rmi node:lts
docker rmi $(docker images -aq) # remove remaining images
```

<br/>
✔️ Pull the image and run it

```bash
docker pull gcr.io/[project-id]/node-app:0.2
docker run -p 4000:80 -d gcr.io/[project-id]/node-app:0.2

curl http://localhost:4000
```

<br/>

## Summary

```jsx
Build, run, and debug Docker containers.
Pull Docker images from Docker Hub and Google Container Registry.
Push Docker images to Google Container Registry.
```
