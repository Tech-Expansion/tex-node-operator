 Prerequisites:
 K8s admin, kubectl

## Output: 
### Cardano Namespace
First, lets create a dedicated namespace for it and make it the default.
```
$ kubectl create namespace cardano
namespace/cardano created

$ kubectl config set-context --current --namespace=cardano
Context "kubernetes-admin@kubernetes" modified.
```

### Attaching Node Labels
Here we get the external IP for our relay node in the cluster
```kubectl get nodes --output wide```
output:
```
NAME                   STATUS     ROLES           AGE   VERSION    INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
standard-fuchsia-owl   NotReady   control-plane   42m   v1.30.14   103.188.83.198   <none>        Ubuntu 22.04.5 LTS   5.15.0-69-generic   containerd://1.7.27
```
