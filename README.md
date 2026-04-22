# k8s_reflector

Reflector for microk8s — automatically mirrors Secrets and ConfigMaps across namespaces using annotations.

## Directory layout

```
k8s_reflector/
├── helm/
│   └── Chart.yaml                   # Reflector wrapper chart (emberstack/reflector dependency)
├── helm_values/
│   └── values-micro-seconddns.yaml  # Lightweight config for microk8s
└── argocd-app.yaml
```

## How it works

Add annotations to a Secret or ConfigMap in the source namespace. Reflector automatically creates and maintains copies in the specified target namespaces.

### Annotations

```yaml
metadata:
  annotations:
    reflector.v1.k8s.emberstack.com/reflection-allowed: "true"
    reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces: "web,dns"
    reflector.v1.k8s.emberstack.com/reflection-auto-enabled: "true"
    reflector.v1.k8s.emberstack.com/reflection-auto-namespaces: "web,dns"
```

- `reflection-allowed` — permit this resource to be reflected
- `reflection-allowed-namespaces` — comma-separated list of allowed target namespaces
- `reflection-auto-enabled` — automatically create reflections
- `reflection-auto-namespaces` — comma-separated list of namespaces to auto-reflect into

## Installation

```bash
# 1. Add Emberstack repo
helm repo add emberstack https://emberstack.github.io/helm-charts
helm repo update

# 2. Pull chart dependencies
helm dependency update helm/

# 3. Install
helm upgrade --install reflector helm/ \
  -f helm_values/values-micro-seconddns.yaml \
  -n kube-system
```

## Verify

```bash
kubectl get pods -n kube-system | grep reflector
```

## Usage with ESO

When using External Secrets Operator, add reflector annotations to the ESO target template:

```yaml
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: rmq-credentials
  namespace: rmq
spec:
  target:
    name: rmq-credentials
    creationPolicy: Owner
    template:
      metadata:
        annotations:
          reflector.v1.k8s.emberstack.com/reflection-allowed: "true"
          reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces: "web"
          reflector.v1.k8s.emberstack.com/reflection-auto-enabled: "true"
          reflector.v1.k8s.emberstack.com/reflection-auto-namespaces: "web"
```

This creates the secret in `rmq` namespace and Reflector auto-mirrors it to `web`.
