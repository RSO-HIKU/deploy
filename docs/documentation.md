# Deploy - Technical Documentation

## Overview

The deployment repository contains Kubernetes, Helm and ArgoCD artifacts for deploying the components of our application on a cluster.

## Table of Contents

- [Deploy - Technical Documentation](#deploy---technical-documentation)
  - [Overview](#overview)
  - [Table of Contents](#table-of-contents)
  - [Folder Structure](#folder-structure)
  - [Platform Components](#platform-components)
  - [Deployment](#deployment)
    - [Local Development](#local-development)
      - [Required Tools \& Services](#required-tools--services)
      - [Other requirements](#other-requirements)
    - [Test and Prod](#test-and-prod)
  - [CI/CD Pipeline](#cicd-pipeline)
  - [Environment Variables, Values and Secrets](#environment-variables-values-and-secrets)
  - [Contact](#contact)


---

## Folder Structure
- argocd/  
  ArgoCD application definitions and bootstrap manifests.
  - apps/ — App-of-Apps application YAMLs split by environment.
  - projects/ — ArgoCD Project manifests.
  - root/ — bootstrap/root App manifests.

- charts/  
  Helm charts for platform components and subcharts.
  - platform/ — charts for edge, keycloak, rabbitmq, traefik
- docs/  
  Deployment documentation
- manifests/  
  Cluster-level manifests (e.g., cert-manager ClusterIssuer) organized by environment.
- values/  
  Environment-specific Helm values used by charts/ and ArgoCD.

## Platform Components
This repository is in charge of platform components required for the cluster to work properly. 

List of platform components:
- Keycloak -> Used for user authentication
- Traefik -> Ingress controller
- RabbitMQ -> Message Queue
- Ingress routes -> Ingress controller route setup
- Certification issuance for Traefik

## Deployment

### Local Development
For local development, the platform components need to be deployed manually using the helm charts found in `\charts\`

#### Required Tools & Services
- **Java 17+**
- **Maven 3.9+**
- **PostgreSQL 14+**
- **Docker Desktop**
- **Keycloak**
- **RabbitMQ**
- **Minikube**
- **Skaffold**

#### Other requirements
- Docker Desktop is running,
- Minikube cluster is running on Docker Desktop,
- The database is deployed on your local cluster,
- The Traefik ingress controller is deployed on your local cluster,
- Keycloak is deployed on your local cluster.

Example for deploying Keycloak into your local Minikube cluster:
```powerhshell
# Run this from \charts\keycloak
helm upgrade --install keycloak . -f values-dev.yaml -n keycloak
```

### Test and Prod
To deploy the application into a Kubernetes cluster in the test and prod environment, you must first install ArgoCD into that cluster.

Powershell commands to install ArgoCD into a cluster:
```powershell
kubectl create namespace argocd 

helm repo add argo https://argoproj.github.io/argo-helm 

helm repo update

helm install argocd argo/argo-cd -n argocd 

# For the test environment
kubectl apply -f test-root.yaml
```

Once ArgoCD is installed and running, it will manage the deployments of platform components and microservices.

## CI/CD Pipeline

CI is defined by the workflow files in each microservice repository. This repository manages CD using ArgoCD. ArgoCD watches this repository for changes and when they are detected it updates the deployments so that they match the defined state.

ArgoCD uses the app-of-apps structure.

For some deployments, where the order matters, helm charts use ArgoCD sync waves to ensure the correct order. An example of such a deployment is Keycloak which requires a TLS certificate to be issued before it is deployed.

ArgoCD offers a user interface for checking deployment statuses and for manual changes. To access this UI you must portforward the service, for example using the command below:
```powershell
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

## Environment Variables, Values and Secrets
Non-sensitive configuration values of the test and prod environments for all microservices are found the folder `values`. Dev values are found in their respective repositories.

Secrets for the test and prod environments are read from the Azure Key Vault during deployment. 

## Contact

For questions or issues, contact the development team.

---

**Last Updated**: January 11, 2026  
**Version**: 0.1.0
