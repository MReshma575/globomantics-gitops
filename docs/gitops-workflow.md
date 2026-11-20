# GitOps workflow — how this repository works

This repository is the **source of truth** for the workloads it manages. Nobody runs
`kubectl apply` by hand: you change Git, and **Argo CD** continuously reconciles the
cluster to match. If someone changes the cluster manually, Argo CD's `selfHeal`
policy reverts it — drift is treated as a bug.

## Layout

```
manifests/
  apps/
    web/
      base/                 # environment-agnostic manifests
        deployment.yaml     # web Deployment (probes, resources, ConfigMap env)
        service.yaml        # ClusterIP Service
        ingress.yaml        # NGINX Ingress
        hpa.yaml            # autoscaling 2–6 replicas at 70% CPU
        configmap.yaml      # application configuration
        kustomization.yaml
      overlays/
        dev/                # namespace dev, 1 replica, debug logging, alpine image
        prod/               # namespace prod, 3 replicas, higher CPU/memory limits
argocd/
  application.yaml          # Argo CD Application for web → overlays/prod
docs/
  gitops-workflow.md        # this file
```

## Why Kustomize overlays

The **base** holds what never changes between environments. Each **overlay** patches
only what differs — replica count, image tag, namespace, resource limits, log level —
so dev and prod can never silently drift apart in structure. Preview any environment
before it ships:

```bash
kubectl kustomize manifests/apps/web/overlays/prod
```

## How a change ships

1. Edit a manifest (or bump an image tag in an overlay) and open a pull request.
2. Review the rendered diff — `kubectl kustomize` output shows exactly what the
   cluster will become.
3. Merge to `main`. Argo CD polls the repo, detects the change, and syncs:
   `prune` deletes resources removed from Git, `selfHeal` corrects manual drift,
   and failed syncs retry with backoff.
4. Watch it land: `argocd app get web-prod` or the Argo CD UI shows sync/health status.

## Rollback

A rollback is a `git revert` of the bad commit — Argo CD syncs the cluster back to
the previous known-good state. No manual cluster surgery, and the audit trail is
the Git history itself.

## Design choices

- **Probes on every workload** — readiness gates traffic until the app responds;
  liveness restarts wedged containers.
- **Resource requests/limits everywhere** — the scheduler can place pods reliably
  and the HPA has the metrics it needs.
- **Config via ConfigMap, not baked into images** — the same image promotes from
  dev to prod unchanged; only configuration differs.
