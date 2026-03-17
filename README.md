# nemoclaw-k8s

Kubernetes Helm chart for [NVIDIA NemoClaw](https://docs.nvidia.com/nemoclaw/latest/index.html) — deploy OpenClaw with enterprise sandboxing on k8s.

> **Note:** NemoClaw is alpha software (released March 2026 at GTC). This is a **community-maintained** chart and is not officially supported by NVIDIA. If NVIDIA releases an official Helm chart, we will link to it here.

## Prerequisites

- Kubernetes 1.26+
- Helm 3.x (optional — plain YAML manifests also provided)
- An NVIDIA API key from [build.nvidia.com](https://build.nvidia.com)

## Install

```bash
helm install nemoclaw oci://ghcr.io/kubespark/nemoclaw \
  -n nemoclaw --create-namespace \
  --set nemoclaw.apiKey=<your-key>
```

Or install from source:

```bash
git clone https://github.com/kubespark/nemoclaw-k8s.git
cd nemoclaw-k8s
helm install nemoclaw . -n nemoclaw --create-namespace --set nemoclaw.apiKey=<your-key>
```

### Without Helm

Apply the plain YAML manifests directly:

```bash
# Edit manifests/install.yaml and replace REPLACE_ME with your NVIDIA API key
kubectl apply -f manifests/install.yaml
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Container image | `ghcr.io/nvidia/openshell-community/sandboxes/openclaw` |
| `image.tag` | Image tag | `latest` |
| `nemoclaw.apiKey` | NVIDIA API key for cloud inference | `""` |
| `nemoclaw.existingSecret` | Use an existing secret for API key | `""` |
| `nemoclaw.inferenceProvider` | Inference provider | `nvidia-nim` |
| `nemoclaw.model` | Model for inference | `nemotron-3-super-120b` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `256Mi` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.size` | PVC size | `5Gi` |
| `persistence.storageClass` | Storage class | `""` |
| `securityContext.privileged` | Run in privileged mode | `true` |
| `networkPolicy.enabled` | Enable network policy | `true` |
| `nodeSelector` | Node selector labels | `{}` |

## Security considerations

NemoClaw uses NVIDIA OpenShell to run sandboxed containers. On Kubernetes, this requires **privileged mode** (`securityContext.privileged: true`) because the sandbox runs containers within the pod. This is a known trade-off:

- The sandbox enforces its own network policies and filesystem restrictions
- The outer container needs elevated privileges to manage the inner sandbox
- Review [NemoClaw's architecture](https://docs.nvidia.com/nemoclaw/latest/reference/architecture.html) to understand the security model

If your cluster enforces PodSecurityStandards or OPA/Gatekeeper policies, you will need to create an exception for the NemoClaw namespace.

## Available models

| Model | Context Window | Max Output |
|-------|---------------|------------|
| Nemotron 3 Super 120B (default) | 131K | 8K |
| Nemotron Ultra 253B | 131K | 4K |
| Nemotron Super 49B v1.5 | 131K | 4K |
| Nemotron 3 Nano 30B | 131K | 4K |

## Uninstall

```bash
# Helm
helm uninstall nemoclaw -n nemoclaw

# Plain manifests
kubectl delete -f manifests/install.yaml
```

## License

Apache 2.0 — see [LICENSE](LICENSE).
