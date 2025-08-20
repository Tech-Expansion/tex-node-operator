# Cardano Node Operator for Kubernetes

🚀 **Cardano Node Operator** helps you run and manage Cardano nodes on Kubernetes in a **declarative, Kubernetes-native** way.
You just need to create a **Custom Resource (CR)**, and the Operator will automatically deploy a **StatefulSet**, **ConfigMap**, **Secret**, **PVC**, and a sidecar **metrics exporter**.

---

## ✨ Features

- Manage Cardano Nodes via **CRD `CardanoNode`**.
- Support for Cardano Node, PostgreSQL, Ogmios, and DB-Sync deployments.
- Declarative configuration for all components.
- Persistent storage for Cardano Node and PostgreSQL data.

---

## 🛠 Prerequisites

Before you begin, ensure you have the following:

*   A running Kubernetes cluster (e.g., Minikube, Kind, GKE, EKS, AKS).
*   `kubectl` installed and configured to connect to your cluster.

---

## 🚀 Deployment Steps

Follow these steps to deploy the Cardano ecosystem on your Kubernetes cluster.

### 1. Clone the Repository

If you haven't already, clone this repository to your local machine:

```bash
git clone <repository-url>
cd <repository-name>
```

### 2. Create the Namespace

First, create the dedicated namespace for Cardano components:

```bash
kubectl apply -f k8s/namespace.yaml
```

This will create a `cardano` namespace:

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cardano
```

### 3. Create Secrets

If you are using PostgreSQL, it's recommended to create a secret for database credentials. Review and apply `k8s/secrets/postgres-secret.yaml`:

```bash
kubectl apply -f k8s/secrets/postgres-secret.yaml
```

### 4. Configure Persistent Storage

The Cardano Node and PostgreSQL require persistent storage.

**Persistent Volume (PV)**:
If your cluster doesn't dynamically provision Persistent Volumes, you might need to create a `PersistentVolume` manually. Review `k8s/cardano-node/pv.yaml` and apply if necessary:

```bash
kubectl apply -f k8s/cardano-node/pv.yaml
```

**Persistent Volume Claim (PVC)**:
The `PersistentVolumeClaim` will request storage for the Cardano Node. The `cardano-pvc` requests 10Gi of storage.

```bash
kubectl apply -f k8s/pvc.yaml
```

This will create the following PVC:

```yaml
# k8s/pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cardano-pvc
  namespace: cardano
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### 5. Configure Cardano Node

Cardano Node requires several configuration files and genesis files. These are typically managed via `ConfigMap` objects.

Apply the general ConfigMap:

```bash
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/config/configmap.yaml
```

Then, apply the Cardano Node specific configuration files as ConfigMaps. These files include the genesis files and node configuration.

```bash
kubectl apply -f k8s/cardano-node/alonzo-genesis.json
kubectl apply -f k8s/cardano-node/byron-genesis.json
kubectl apply -f k8s/cardano-node/config.json
kubectl apply -f k8s/cardano-node/shelley-genesis.json
kubectl apply -f k8s/cardano-node/topology.json
```

An example snippet from `k8s/cardano-node/config.json`:

```json
{
  // ...
  "rotation": {
    "rpKeepFilesNum": 10,
    "rpLogLimitBytes": 5000000,
    "rpMaxAgeHours": 24
  },
  "setupBackends": [
    "KatipBK"
  ],
  "setupScribes": [
    {
      "scFormat": "ScText",
      "scKind": "StdoutSK",
      "scName": "stdout",
      "scRotation": null
    }
  ]
  // ...
}
```

### 6. Deploy Cardano Node

The Cardano Node is deployed as a StatefulSet to ensure stable network identities and persistent storage.

```bash
kubectl apply -f k8s/cardano-node/cardano-node.yaml
```

This will create a StatefulSet (defined in `k8s/cardano-node/cardano-node.yaml`) for the Cardano Node.

### 7. Deploy Supporting Services

The project also includes deployments for PostgreSQL, Ogmios, and Cardano DB-Sync. Deploy them in the following order:

#### PostgreSQL

Deploy the PostgreSQL StatefulSet and Service:

```bash
kubectl apply -f k8s/postgres/postgres-statefulset.yaml
kubectl apply -f k8s/postgres/postgres-service.yaml
```

#### Ogmios

Deploy the Ogmios application and service, which provides a WebSocket interface to the Cardano Node:

```bash
kubectl apply -f k8s/ogmios/ogmios-deployment.yaml
kubectl apply -f k8s/ogmios/ogmios-service.yaml
```

#### Cardano DB-Sync

Deploy the Cardano DB-Sync application and service, which syncs the blockchain data to the PostgreSQL database:

```bash
kubectl apply -f k8s/db-sync/db-sync-deployment.yaml
kubectl apply -f k8s/db-sync/db-sync-service.yaml
```

### 8. Expose Cardano Node Service

The `cardano-service` exposes the Cardano Node to other services within the cluster.

```bash
kubectl apply -f k8s/service.yaml
```

This will create a ClusterIP service:

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: cardano-service
  namespace: cardano
spec:
  selector:
    app: cardano-node
  ports:
    - protocol: TCP
      port: 3001
      targetPort: 3001
  type: ClusterIP
```

### 9. Verify Deployment

After applying all the configurations, verify that all pods are running and services are accessible:

```bash
kubectl get pods -n cardano
kubectl get svc -n cardano
kubectl get pvc -n cardano
```

You should see output similar to this (pod names and statuses may vary):

```
NAME                                READY   STATUS    RESTARTS   AGE
cardano-node-0                      1/1     Running   0          5m
postgres-0                          1/1     Running   0          4m
ogmios-deployment-xxxxxxxxx-xxxxx   1/1     Running   0          3m
db-sync-deployment-xxxxxxxxx-xxxxx  1/1     Running   0          3m

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
cardano-service       ClusterIP   10.xx.xx.xx     <none>        3001/TCP         5m
postgres-service      ClusterIP   10.xx.xx.xx     <none>        5432/TCP         4m
ogmios-service        ClusterIP   10.xx.xx.xx     <none>        1337/TCP         3m
db-sync-service       ClusterIP   10.xx.xx.xx     <none>        5432/TCP         3m

NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
cardano-pvc         Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   10Gi       RWO            standard       5m
```

### 10. Accessing the Cardano Node (Optional)

If you need to access the Cardano Node from outside the cluster, you might consider changing the `type` of `cardano-service` to `NodePort` or `LoadBalancer`, or setting up an Ingress.

---

## ⚙ Configuration

You can customize the deployment by editing the `.yaml` files in the `k8s/` directory.

*   **Resource Requests/Limits**: Adjust CPU and memory requests/limits in the deployment files (`cardano-node.yaml`, `postgres-statefulset.yaml`, etc.) according to your needs.
*   **Storage Size**: Modify the `storage` request in `k8s/pvc.yaml` to allocate more or less disk space for the Cardano Node.
*   **Cardano Node Configuration**: Update `k8s/cardano-node/config.json` and `k8s/cardano-node/topology.json` to change the node's behavior (e.g., logging, network peers).
*   **PostgreSQL Credentials**: Modify `k8s/secrets/postgres-secret.yaml` to change PostgreSQL credentials. Remember to re-apply the secret and restart the PostgreSQL pod if you change it after initial deployment.

---

## 🗑 Cleanup

To remove all deployed Cardano components from your Kubernetes cluster, run the following commands in reverse order of deployment:

```bash
kubectl delete -f k8s/db-sync/db-sync-service.yaml
kubectl delete -f k8s/db-sync/db-sync-deployment.yaml
kubectl delete -f k8s/ogmios/ogmios-service.yaml
kubectl delete -f k8s/ogmios/ogmios-deployment.yaml
kubectl delete -f k8s/postgres/postgres-service.yaml
kubectl delete -f k8s/postgres/postgres-statefulset.yaml
kubectl delete -f k8s/service.yaml
kubectl delete -f k8s/cardano-node/cardano-node.yaml
kubectl delete -f k8s/cardano-node/topology.json
kubectl delete -f k8s/cardano-node/shelley-genesis.json
kubectl delete -f k8s/cardano-node/config.json
kubectl delete -f k8s/cardano-node/byron-genesis.json
kubectl delete -f k8s/cardano-node/alonzo-genesis.json
kubectl delete -f k8s/config/configmap.yaml
kubectl delete -f k8s/configmap.yaml
kubectl delete -f k8s/pvc.yaml
kubectl delete -f k8s/cardano-node/pv.yaml # If you manually created a PV
kubectl delete -f k8s/secrets/postgres-secret.yaml
kubectl delete -f k8s/namespace.yaml
```

---

## ⚠️ Important Notes

*   **Initial Sync**: The Cardano Node will take a significant amount of time (potentially days) to sync the entire blockchain history, depending on your network speed and node resources.
*   **Resource Requirements**: Running a full Cardano Node and its supporting services can be resource-intensive. Ensure your Kubernetes cluster has sufficient CPU, memory, and disk space.
*   **Security**: Always review and understand the contents of all YAML files before applying them to your production cluster. Consider implementing additional security measures like network policies and stricter RBAC.
*   **Updates**: Keep your Cardano Node and associated components up-to-date with the latest versions. Monitor official Cardano announcements for new releases and security patches.
