# GSP304 — Build and Deploy a Docker Image to a Kubernetes Cluster

This write-up documents the Google Cloud Skills Boost challenge lab **GSP304**. The lab packages a sample Go web application into a Docker image, pushes it to Google Container Registry, and deploys it to Google Kubernetes Engine (GKE).

> This is a learning-lab implementation. Temporary project IDs, external IP addresses, and lab credentials are intentionally omitted.

## What this lab demonstrates

- Building a Docker image from a supplied application archive
- Tagging and pushing a container image to Google Container Registry
- Creating and connecting to a GKE cluster
- Deploying an application with Kubernetes
- Exposing a Deployment through a LoadBalancer Service
- Testing an application through its external endpoint

## Environment used

| Resource | Configuration |
|---|---|
| Lab | GSP304 — Build and Deploy a Docker Image to a Kubernetes Cluster |
| GKE cluster | `echo-cluster` |
| Zone | `us-central1-a` |
| Cluster nodes | 2 |
| Machine type | `e2-standard-2` |
| Container image | `gcr.io/<PROJECT_ID>/echo-app:v1` |
| Kubernetes Deployment | `echo-web` |
| Service type | `LoadBalancer` |
| Service port | `80` |
| Container target port | `8000` |

## Workflow

### 1. Create the GKE cluster

```bash
export PROJECT_ID="$(gcloud config get-value project)"
export ZONE=us-central1-a

gcloud container clusters create echo-cluster \
  --zone="$ZONE" \
  --num-nodes=2 \
  --machine-type=e2-standard-2
```

### 2. Download and extract the application

```bash
mkdir -p ~/echo-web
cd ~/echo-web

gsutil cp "gs://$PROJECT_ID/echo-web.tar.gz" .
tar -xzf echo-web.tar.gz
```

Change into the extracted application directory containing the Dockerfile before building the image.

### 3. Build and push the Docker image

```bash
docker build -t echo-app:v1 .

docker tag echo-app:v1 "gcr.io/$PROJECT_ID/echo-app:v1"

docker push "gcr.io/$PROJECT_ID/echo-app:v1"
```

### 4. Connect to the cluster and deploy the application

```bash
gcloud container clusters get-credentials echo-cluster \
  --zone="$ZONE" \
  --project="$PROJECT_ID"

kubectl create deployment echo-web \
  --image="gcr.io/$PROJECT_ID/echo-app:v1"
```

Check the Deployment and Pods:

```bash
kubectl get deployments
kubectl get pods
```

### 5. Expose the application

```bash
kubectl expose deployment echo-web \
  --type=LoadBalancer \
  --port=80 \
  --target-port=8000

kubectl get service echo-web
```

Wait until the LoadBalancer Service receives an external IP, then test it:

```bash
curl http://EXTERNAL_IP
```

**Observed application response:**

```text
Echo Test
Version: 1.0.0
Hostname: <POD_HOSTNAME>
Host ip-address(es): <POD_IP>
```

The hostname and pod IP are dynamic Kubernetes values and will differ between deployments.

## Verification

- [x] GKE cluster `echo-cluster` created with two nodes.
- [x] Docker image `echo-app:v1` built and pushed to `gcr.io`.
- [x] Kubernetes Deployment `echo-web` created.
- [x] LoadBalancer Service exposed port 80 to container port 8000.
- [x] Application responded with the expected Echo Test output.

## Skills demonstrated

Docker, Google Container Registry, Google Kubernetes Engine, Kubernetes Deployments, Services, LoadBalancer networking, and container troubleshooting.

## Security and cleanup

- Never commit service-account keys, access tokens, passwords, or lab credentials.
- Treat the project ID and LoadBalancer external IP as temporary lab values.
- Delete lab resources only after confirming the Skills Boost session has ended and cleanup is appropriate.
