# Globomantics GitOps Repository

This repository contains Kubernetes manifests for deploying the Globomantics application using GitOps principles with Argo CD.

## Contents

- `namespace.yaml` - Kubernetes namespace definition for the Globomantics application
- `.gitattributes` - Git attributes configuration

## Purpose

This repository serves as the source of truth for deploying and managing the Globomantics application on Kubernetes clusters through Argo CD. Changes to application configuration are made by updating the manifests in this repository and committing them to trigger automated deployments.

## Usage

This repository is designed to work with Argo CD for continuous deployment. The application manifests define the desired state of the Globomantics application in the Kubernetes cluster.

## GitOps Workflow

1. Make changes to Kubernetes manifests in this repository
2. Commit and push changes to trigger Argo CD sync
3. Argo CD automatically applies changes to the target Kubernetes cluster
4. Monitor deployment status through Argo CD UI or CLI

## Prerequisites

- Kubernetes cluster with Argo CD installed
- Argo CD configured to watch this repository
- Proper RBAC permissions for the application namespace

## Repository structure

- `manifests/apps/web/base/` — environment-agnostic Kubernetes manifests (Deployment, Service, Ingress, HPA, ConfigMap)
- `manifests/apps/web/overlays/dev|prod/` — Kustomize overlays: dev runs 1 replica with debug logging; prod runs 3 replicas with higher resource limits
- `argocd/application.yaml` — Argo CD Application syncing the prod overlay with prune + selfHeal
- Deep dive: [docs/gitops-workflow.md](docs/gitops-workflow.md) — how changes ship, rollback, and design choices
