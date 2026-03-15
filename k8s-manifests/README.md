# Unleash K8s — Kustomize Structure

```
k8s/
├── base/
│   ├── infra/                    # GCP Config Connector
│   │   ├── namespace.yaml
│   │   ├── network.yaml          # PSA for CloudSQL
│   │   ├── cloudsql.yaml         # PostgreSQL 15
│   │   ├── iam.yaml              # SA + Workload Identity
│   │   └── kustomization.yaml
│   ├── unleash/                  # Unleash Server
│   │   ├── namespace.yaml
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   ├── hpa.yaml
│   │   ├── networkpolicy.yaml
│   │   └── kustomization.yaml
│   ├── unleash-edge/             # Unleash Edge
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   └── kustomization.yaml        # Root base — composes all
├── overlays/
│   ├── dev/                      # 1 replica, debug, db-f1-micro
│   ├── staging/                  # 2 replicas, staging host
│   └── prod/                     # 3 replicas, HA, TLS
└── README.md
```

## Deploy

```bash
# Via deploy script
./scripts/ok_deploy.sh deploy --env=dev
./scripts/ok_deploy.sh deploy --env=prod

# Direct kustomize
kubectl apply -k k8s/overlays/dev
kubectl apply -k k8s/overlays/prod

# Preview
kustomize build k8s/overlays/prod
```

## ArgoCD

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: unleash-prod
spec:
  source:
    repoURL: https://github.com/your-org/unleash-k8s
    path: k8s/overlays/prod
  destination:
    server: https://kubernetes.default.svc
```

## Environments

| | dev | staging | prod |
|---|---|---|---|
| Unleash replicas | 1 | 2 | 3 |
| Edge replicas | 1 | 2 | 3 |
| HPA (unleash) | 1–2 | 2–4 | 3–8 |
| HPA (edge) | 1–3 | 2–6 | 3–15 |
| CloudSQL tier | db-f1-micro | db-custom-1-3840 | db-custom-2-7680 |
| CloudSQL HA | ZONAL | ZONAL | REGIONAL |
| Logging | debug | info | info |
| Ingress | default | staging host | prod + TLS |
