# Lab 1. Introduction to Docker

## gcloud CLI

[gcloud CLI overview | Google Cloud CLI Documentation](https://cloud.google.com/sdk/gcloud)

You can list the active account name with this command:

```bash
gcloud auth list
```

(Output)

```bash
Credentialed accounts:
- <myaccount>@<mydomain>.com (active)
```

You can list the project ID with this command:

```bash
gcloud config list project
```

(Output)

```bash
[core]
project = <project_ID>
```

## Commands

```bash
# Build
docker build -t node-app:0.2 .

# Run
docker run -p 8080:80 --name my-app-2 -d node-app:0.2

# Check running containers (and their container IDs)
docker ps

# Test
curl http://localhost:8080

# Debug
## Check logs
docker logs -f [container_id]

## Start interactive Bash session (allocate a pseudo-tty and keep stdin open)
## - Runs in the WORKDIR from Dockerfile의
docker exec -it [container_id] bash

## Docker inspect to check container metadata
docker inspect [container_id]

## --format to inspect specific fields of JSON
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' [container_id]

# Publish
# Push image to Google Container Registry (GCR).
# - Format: [hostname]/[project-id]/[image]:[tag]
# - For GCR: [hostname] = gcr.io

## Check out the project ID
gcloud config list project

## Tag
docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2

## Push to GCR
docker push gcr.io/[project-id]/node-app:0.2

## Check that image exists in GCR by visiting the image registry in the web browser
## The ways:
## 1. Navigate via the console to Navigation Menu > Container Registry > node-app
## 2. Visit http://gcr.io/[project-id]/node-app
```
