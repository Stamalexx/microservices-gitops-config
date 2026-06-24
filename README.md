# Microservices GitOps Configuration & CI/CD Pipeline

**Author:** Alexios Stamelos

[![Academic Thesis](https://img.shields.io/badge/Read-Full_Academic_Thesis-blue.svg)](https://github.com/Stamalexx/MSc-Thesis-GitOps-Microservices.git)

## Overview
This repository serves as the central continuous integration and declarative continuous delivery (GitOps) hub for the Google Online Boutique microservices deployment. It contains the essential infrastructure-as-code (IaC) configurations and the automated Jenkins pipeline logic required to execute the "Shift-Left" testing and deployment architecture.

By decoupling this configuration from the primary application source code, the architecture enforces a strict separation of concerns, restricting the credential blast radius and ensuring secure, zero-touch deployments to the live Kubernetes cluster.

## Repository Contents
* **Jenkinsfile (Continuous Integration):** The declarative Groovy script that orchestrates the CI pipeline. It defines the logic for dynamic service compilation, ephemeral environment provisioning, automated Playwright test execution, and post-execution infrastructural cleanup.
* **Helm Configurations (Continuous Delivery):** The Kubernetes deployment manifests and `values.yaml` files. This repository acts as the absolute single source of truth for the live cluster state, actively monitored by the Argo CD controller.

## CI/CD Workflow Execution
This repository drives the following automated sequence:

### Phase 1: Continuous Integration (Jenkins)
1. **Smart Checkout & Detection:** The pipeline clones the application repository and identifies exactly which microservices have been modified using a dynamic `git diff`.
2. **Build & Tag:** Docker images are compiled locally exclusively for the altered services and assigned unique build tags.
3. **Ephemeral Environment:** A complete, localized replica of the application is synthesized on the CI execution node using `docker-compose`.
4. **Shift-Left Quality Gate:** The pipeline triggers headless End-to-End (E2E) Playwright tests against the ephemeral environment. Any functional failures or inter-service dependency crashes immediately terminate the pipeline.
5. **Artifact Registry Push:** Upon successfully passing the automated quality gate, Jenkins pushes the validated Docker images to Docker Hub.
6. **GitOps Synchronization Update:** Jenkins automatically updates the Helm `values.yaml` file within this repository to reflect the newly tagged images.
7. **Resource Janitor:** A mandatory post-execution block aggressively prunes dangling images and tears down the ephemeral containers to prevent CI server resource exhaustion.

### Phase 2: Continuous Delivery (Argo CD)
* **Autonomous Reconciliation:** Argo CD, operating securely within the localized Ubuntu Server Kubernetes cluster, continuously monitors this repository. Upon detecting the automated update made by Jenkins in Step 6, it autonomously pulls the new images and applies the declarative changes to the live environment, achieving zero-downtime synchronization.
* **Self-Healing:** If manual, out-of-band changes occur directly in the Kubernetes cluster (state drift), Argo CD utilizes this repository to autonomously restore the infrastructure to its declared state.

## Related Repositories
* [**Academic Thesis & Empirical Evaluation**](https://github.com/Stamalexx/MSc-Thesis-GitOps-Microservices.git)
* [**Application Source Code**](https://github.com/Stamalexx/microservices-demo-DevOps-apply.git)
* [**E2E Playwright Testing Suite**](https://github.com/Stamalexx/microservices-playwright-testing.git)
* [**Docker Registry Artifacts**](https://hub.docker.com/repositories/stamalexx)
