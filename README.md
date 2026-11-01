# property-management-gitops

Desired state of the property-management deployments, synced by Argo CD.

```
charts/property-management/   Helm chart (Deployment, Service, Ingress, ServiceAccount, SecretStore, ExternalSecret)
envs/floci/values.yaml        Values for the local Floci EKS cluster
```

- **Jenkins** (app repo) builds the image, pushes it to ECR and commits the new tag to
  `envs/<env>/values.yaml` (the line marked `# jenkins-image-tag`).
- **Argo CD** (Application `property-management-floci`, created by Terraform) polls this repo and
  syncs the cluster: automated sync, prune and self-heal are on, so manual `kubectl` changes are reverted.
- **Infrastructure** (EKS, RDS, Secrets Manager, IAM, ESO, ingress-nginx, Argo CD) stays in Terraform.

Change config (replicas, resources, probes, ingress) by committing here, not with kubectl/helm.
Roll back by reverting the tag commit.

## Layout

```
apps/<app>/envs/<env>/values.yaml   per-app, per-environment values (Jenkins updates the image tag here)
charts/property-management/         shared app chart (optional Fluent Bit sidecar + ServiceMonitor)
platform/<addon>/values.yaml        cluster add-ons deployed by Argo CD:
                                      kube-prometheus-stack (Prometheus + Grafana), loki
```

Grafana: http://grafana.localhost:8080 (Explore -> Loki: `{app="property-management"} | json`).
