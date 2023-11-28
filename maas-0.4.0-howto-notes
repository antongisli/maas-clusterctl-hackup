clusterctl configuration file docs:
https://release-0-3.cluster-api.sigs.k8s.io/clusterctl/configuration



Set ~/.cluster-api/clusterctl.yaml
```
CONTROL_PLANE_MACHINE_MINCPU: 4
CONTROL_PLANE_MACHINE_MINMEMORY: 8192
CALICO_IPV4POOL_IPIP: Always
CALICO_IPV4POOL_VXLAN: Never
MAAS_DNS_DOMAIN: maas
WORKER_MACHINE_MINCPU: 4
WORKER_MACHINE_MINMEMORY: 8192
MAAS_API_KEY: abcd #Anton
MAAS_ENDPOINT: http://x.x.x.x:5240/MAAS
WORKER_MACHINE_IMAGE: custom/u-2004-0-k-12410-0
KUBERNETES_VERSION: v1.24.10
CONTROL_PLANE_MACHINE_IMAGE: custom/u-2004-0-k-12410-0
SSH_AUTHORIZED_KEY: ssh-ed25519 AAAA...
MAAS_SSH_KEY_NAME: default
MAAS_SSH_KEY: ssh-ed25519 AAAA...
```

```
kind create cluster --name=maas-cluster

clusterctl init --infrastructure maas:v0.5.0

clusterctl generate cluster maas-cluster --infrastructure=maas:v0.5.0 --control-plane-machine-count=1 --worker-machine-count=1 --kubernetes-version v1.24.10 > my_cluster.yaml

# Edit the cluster and make sure you change the podcidr range
# vi my_cluster.yaml ...

kubectl apply -f my_cluster.yaml

kubectl get clusters
```

Download kubeconfig: 
  `clusterctl get kubeconfig maas-cluster > kcfg`

Edit it to set location of API server e.g.:
`    server: https://abcd.maas:6443`

List nodes using skip verify (because we changed server parameter):
```
export KUBECONFIG=./kcfg
kubectl get nodes -A --insecure-skip-tls-verify
```

Install a CNI like flannel, make sure to set matching pod CIDR range.

Cleanup:
```
unset -f KUBECONFIG
kubectl delete cluster maas-cluster
```

References:
https://www.youtube.com/watch?v=8yUDUhZ6ako&list=LL&index=1

MAAS tag placement and resource pools are missing from the cluster-template, can be specified like this:
```
      resourcePool: vmo-demo
      tags:
      - demo-am
      - amit-tag-test1
```


