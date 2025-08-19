# Cardano Node Operator for Kubernetes

🚀 **Cardano Node Operator** helps you run and manage Cardano nodes on Kubernetes in a **declarative, Kubernetes-native** way.  
You just need to create a **Custom Resource (CR)**, and the Operator will automatically deploy a **StatefulSet**, **ConfigMap**, **Secret**, **PVC**, and a sidecar **metrics exporter**.

---

## ✨ Features

- Manage Cardano Nodes via **CRD `CardanoNode`**.
- Support for multiple networks: `mainnet`, `preprod`, `preview`, or custom configuration.
- Deploys **StatefulSet** with:
  - PVC for the ledger database.
  - ConfigMap for `node-config.json` and `topology.json`.
  - Secret for `kes/vrf/opcert` (if block-producing node).
- Supports metrics sidecar (Prometheus).
- Automatically roll pods when config/topology changes (**rolling update**).
- Updates node status in `.status`.

---

## 📦 Installation

### Requirements
- Kubernetes >= 1.23
- Kubectl, Kustomize
- Docker/Podman to build images
- (Optional) Helm to deploy the operator

### Build & Deploy
```bash
# Clone repo
git clone https://github.com/<your-org>/cardano-node-operator.git
cd cardano-node-operator

# Generate CRD, RBAC
make manifests

# Build & push image
make docker-build IMG=ghcr.io/<your-org>/cardano-node-operator:0.1.0
make docker-push IMG=ghcr.io/<your-org>/cardano-node-operator:0.1.0

# Deploy Operator into the cluster
make deploy IMG=ghcr.io/<your-org>/cardano-node-operator:0.1.0