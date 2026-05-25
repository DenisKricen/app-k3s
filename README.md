# Orchestration Infrastructure for a Web Application

This repository contains the Kubernetes infrastructure and CI workflow for deploying a pre-built web application. The application itself is not the focus of this project; the work here is centered on infrastructure-as-code, container orchestration, and deployment automation.

## Project Goal

The goal of this project was to build a reproducible Kubernetes-based infrastructure that can run the application in a local cluster and through automated CI. The repository demonstrates how to package the app into containers, deploy it with Kubernetes manifests, and connect it to a PostgreSQL backend.

## What I Implemented

- Kubernetes manifests for application and database deployment
- A Kubernetes `StatefulSet` for PostgreSQL persistence-friendly workloads
- Kubernetes `Services` for internal and external access
- A `Secret` manifest for database credentials
- A Docker image for the web application
- A GitHub Actions workflow that creates a local Kind cluster and applies the manifests

## Architecture

The infrastructure is split into two main parts:

- **Web application**: runs as a Kubernetes `Deployment` and is exposed through a `NodePort` `Service`
- **Database**: runs as a PostgreSQL `StatefulSet` and is exposed through a headless `Service`

The application uses environment variables and Kubernetes secrets to connect to PostgreSQL. The deployment also includes an init container that runs database migrations before the main application container starts.

## Repository Structure

- `Dockerfile` - container image for the web application
- `k8s/app-deploy.yml` - application deployment manifest
- `k8s/app-service.yml` - application service manifest
- `k8s/db-state.yml` - PostgreSQL StatefulSet manifest
- `k8s/db-service.yml` - PostgreSQL headless service manifest
- `k8s/db-secret.yml` - database credentials secret
- `.github/workflows/main.yml` - CI pipeline for Kind-based deployment and validation

## Local Run

You can test the infrastructure locally with Kind and kubectl:

```bash
kind create cluster --name dev
kubectl apply -R -f ./k8s
kubectl wait pods --all --for condition=Ready
kubectl port-forward service/webapp-service 8080:80 &
```

Then open:

```bash
http://localhost:8080
```

## CI Pipeline

The GitHub Actions workflow performs the following steps:

1. Creates a local Kind cluster
2. Applies all Kubernetes manifests from the `k8s` folder
3. Waits for the pods to become ready
4. Port-forwards the web service
5. Runs Cypress-based smoke tests

## Resume Value

This project is a strong DevOps portfolio piece because it demonstrates:

- containerization with Docker
- Kubernetes deployment design
- service discovery and networking
- secret management
- stateful workload handling with PostgreSQL
- CI-driven infrastructure validation

## Notes

- The application code is not the main contribution of this repository.
- The focus is on infrastructure, deployment automation, and repeatable execution in Kubernetes.
- The repository is suitable for showcasing practical DevOps and platform engineering skills.
