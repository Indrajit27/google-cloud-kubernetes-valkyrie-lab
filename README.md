# Deploy Kubernetes Applications on Google Cloud — GSP318

A hands-on Google Cloud Skills Boost challenge lab completed with **100/100**. This project documents the workflow used to build a Docker image, publish it to Artifact Registry, and deploy and expose the application on Google Kubernetes Engine (GKE).

> **Note:** This repository documents a learning lab, not a production deployment. The lab's temporary project and resources may be deleted when the lab session ends.

## Objectives

- Create a Docker image from the provided Valkyrie application source.
- Run the container and test the application on port 8080.
- Publish the image to Google Cloud Artifact Registry.
- Deploy the image to the `valkyrie-dev` GKE cluster.
- Expose the application using a Kubernetes `LoadBalancer` Service.

## Technologies

- Google Cloud Shell and Google Cloud CLI (`gcloud`)
- Docker
- Artifact Registry
- Google Kubernetes Engine (GKE)
- Kubernetes manifests (`Deployment` and `Service`)

## Lab configuration

| Setting | Value |
|---|---|
| Lab | GSP318 — Deploy Kubernetes Applications on Google Cloud |
| Artifact Registry region | `europe-west1` |
| Artifact Registry repository | `valkyrie-repository` |
| Image | `valkyrie-prod:v0.0.3` |
| GKE cluster | `valkyrie-dev` |
| Cluster zone | `europe-west1-b` |
| Result | **100/100** |

The project ID and external IP are intentionally omitted because lab projects and IPs are temporary.

## Workflow

### 1. Prepare the application source

```bash
gcloud storage cp gs://spls/gsp318/valkyrie-app.tgz .
tar -xzf valkyrie-app.tgz
cd ~/valkyrie-app
```

### 2. Create the Dockerfile

```dockerfile
FROM golang:1.10
WORKDIR /go/src/app
COPY source .
RUN go install -v
ENTRYPOINT ["app","-single=true","-port=8080"]
```

Build and verify the image:

```bash
docker build -t valkyrie-prod:v0.0.3 .
docker images
```

### 3. Test the container

```bash
docker run -d --name valkyrie-test -p 8080:8080 valkyrie-prod:v0.0.3
docker ps
```

In Cloud Shell, use **Web Preview → Preview on port 8080** to test the app.

### 4. Publish the image to Artifact Registry

Set the lab values:

```bash
export PROJECT_ID="$(gcloud config get-value project)"
export REGION=europe-west1
export REPO=valkyrie-repository
export IMAGE=valkyrie-prod
export TAG=v0.0.3
export IMAGE_URI="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${TAG}"
```

Create the Docker repository, configure authentication, tag, and push:

```bash
gcloud artifacts repositories create "$REPO" \
  --repository-format=docker \
  --location="$REGION"

gcloud auth configure-docker europe-west1-docker.pkg.dev --quiet
docker tag "$IMAGE:$TAG" "$IMAGE_URI"
docker push "$IMAGE_URI"
```

If the repository already exists, skip the create command. Verify the published image:

```bash
gcloud artifacts docker images list \
  europe-west1-docker.pkg.dev/"$PROJECT_ID"/"$REPO"
```

### 5. Deploy to GKE and expose the application

Connect to the lab cluster:

```bash
gcloud container clusters get-credentials valkyrie-dev \
  --zone=europe-west1-b \
  --project="$PROJECT_ID"
kubectl get nodes
```

Update the image in `k8s/deployment.yaml` to the full Artifact Registry URI:

```text
europe-west1-docker.pkg.dev/PROJECT_ID/valkyrie-repository/valkyrie-prod:v0.0.3
```

Replace `PROJECT_ID` with the active lab project ID. Then apply the manifests:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get deployments,pods,services
```

Wait for the service's external IP to be assigned:

```bash
kubectl get service valkyrie-dev -w
```

Open the assigned external IP in a browser to test the exposed service. The lab service used port 80.

## Verification checklist

- [x] Docker image built with the required name and tag.
- [x] Container tested through Cloud Shell Web Preview.
- [x] Image pushed to Artifact Registry.
- [x] Kubernetes deployment and LoadBalancer service created.
- [x] Challenge lab score: **100/100**.

## What this demonstrates

Practical familiarity with container image creation, registry authentication and publishing, GKE cluster access, Kubernetes manifest configuration, deployment rollout checks, and exposing an application through a load balancer.

## Security and cleanup

- Never commit temporary lab passwords, access tokens, service-account keys, or other credentials.
- Do not rely on temporary lab project IDs or external IP addresses after the session expires.
- Lab resources are temporary; confirm the lab has ended before assuming they remain available.

## Reference

- [Google Cloud Skills Boost](https://www.cloudskillsboost.google/)
- [Artifact Registry documentation](https://cloud.google.com/artifact-registry/docs)
- [Google Kubernetes Engine documentation](https://cloud.google.com/kubernetes-engine/docs)
