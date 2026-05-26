# Orchestration Infrastructure for a Web Application

This repository contains the Kubernetes infrastructure and CI workflow for deploying a web application. The work here is centered on container orchestration, distributed cloud deployment, and complex traffic routing.

## Project Goal

The goal of this project was to build a reproducible Kubernetes-based infrastructure that can run the application locally for development and reliably in a production-like distributed cloud environment (via AWS EC2). The repository demonstrates how to package an app into containers and deploy it using Kubernetes manifests.

## Production Architecture & DevOps Highlights

Beyond basic containerization, this project features a robust production-grade deployment with several advanced infrastructure solutions:

* **Distributed Kubernetes Cluster (k3s):** Successfully deployed a multi-node cluster across multiple AWS EC2 instances, separating the Control Plane (Master Node) and computational workloads (Worker Node), so it could work on a few separate weak servers, with 1GB memory each.
* **Domain-based Nginx Routing:** Nginx gracefully routes HTTPS traffic based on domains: forwarding one domain's traffic to a local Docker Compose stack (another project) and another domain's traffic to the Kubernetes cluster.
* **Resource Optimization & Limit Bypassing:** Overcame memory limits (1GB RAM EC2 instances) by configuring Swap memory, preventing Linux crashes and allowing heavy Master Node components to run alongside existing apps.
* **SSL Automation:** The project uses Certbot certificates for secure HTTPS connections.
* **Init Containers for Migrations:** Utilized Kubernetes `InitContainers` to ensure database migrations and availability checks are fully completed before the main application starts accepting traffic.

## What I Implemented

* Kubernetes manifests for application and database deployment.
* A Kubernetes `StatefulSet` for PostgreSQL persistence-friendly workloads.
* Kubernetes `Services` (`ClusterIP`, `NodePort`, and Headless) for internal and external access.
* A `Secret` manifest for secure database credential management.
* A Docker image for the web application.
* A GitHub Actions workflow that creates a local Kind cluster and applies/validates the manifests.

## Architecture Structure

The Kubernetes infrastructure is logically split into two main parts:

* **Web application:** Runs as a Kubernetes `Deployment` and is exposed to the host network via a `NodePort` Service.
* **Database:** Runs as a PostgreSQL `StatefulSet` and is accessed securely inside the cluster network via a headless Service.

## Repository Structure

* `Dockerfile` - Container image instructions for the web application.
* `k8s/app-deploy.yml` - Application deployment manifest (includes Init Containers).
* `k8s/app-service.yml` - Application NodePort service manifest.
* `k8s/db-state.yml` - PostgreSQL StatefulSet manifest for data persistence.
* `k8s/db-service.yml` - PostgreSQL headless service manifest.
* `k8s/db-secret.yml` - Base64 encoded database credentials secret.
* `.github/workflows/main.yml` - CI pipeline for automated Kind-based deployment validation.

## Local Run

You can test the infrastructure locally using [Kind](https://kind.sigs.k8s.io/) and `kubectl`:

```bash
# 1. Create a local Kubernetes cluster
kind create cluster --name dev

# 2. Apply all Kubernetes manifests from the k8s directory
kubectl apply -R -f ./k8s

# 3. Wait for all pods to be fully up and running
kubectl wait pods --all --for condition=Ready --timeout=300s

# 4. Forward the service port to your local machine (using 30080 to match prod)
kubectl port-forward service/webapp-service 30080:80 &
```

Then open your browser and navigate to: 
```
http://localhost:30080
```

## CI Pipeline

The GitHub Actions workflow ensures continuous integration by performing the following automated steps on every push:

1. Creates an isolated local Kind cluster in the CI runner.
2. Applies all Kubernetes manifests from the `k8s` folder.
3. Waits for the deployment and database pods to become fully healthy and `Ready`.
4. Validates the architecture by port-forwarding the web service.

## Notes

- The application code is not the main contribution of this repository.
- The focus is on infrastructure, deployment automation, and repeatable execution in Kubernetes.
- The repository is for showcasing practical DevOps and platform engineering skills.
