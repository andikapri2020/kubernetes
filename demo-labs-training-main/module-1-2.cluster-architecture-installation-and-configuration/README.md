# Kubernetes Cluster

## Setup Cluster
### Setup Cluster on master node with k3s
Initialize cluster on master node
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.26.6+k3s1" K3S_KUBECONFIG_MODE="644" K3S_DATASTORE_ENDPOINT="etcd" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=10.100.0.0/16 --disable-network-policy" sh -
```

Check token to join cluster
```
cat /var/lib/rancher/k3s/server/token
```

### Join Cluster on worker node
Join worker node to cluster kubernetes
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.26.6+k3s1" INSTALL_K3S_EXEC="agent --server https://10.10.10.11:6443 --token xxxxx" sh -s -
```

### Check node on cluster
Check a node in a cluster
```
kubectl get node
```

### Opsional Reset Cluster
on server
```
/usr/local/bin/k3s-uninstall.sh
```

on agent
```
/usr/local/bin/k3s-agent-uninstall.sh
```

## Install CNI Plugin
Install calico operator
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/tigera-operator.yaml
```

download config calico crd
```
curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/custom-resources.yaml

vim custom-resources.yaml
```

edit range ip
```
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
      cidr: 10.100.0.0/16
```

Install calico crd
```
kubectl apply -f custom-resources.yaml
```

## Metric Server
Check resource usage on node
```
kubectl top node
```

Check resource usage on pod
```
kubectl top pod
```

## Cluster Management
### Labeling role worker
Check a node in a cluster
```
kubectl get node
```

Change role on worker node
```
kubectl label node training-k3s-worker1 node-role.kubernetes.io/worker=worker
```

### Management node
Check a node in a cluster
```
kubectl get node
```

Pause node
```
kubectl cordon node node-name
```

Drain node
```
kubectl drain node node-name
```

Unpause node
```
kubectl uncordon node node-name
```

### Change cluster name
check cluster
```
kubectl config view
kubectl config get-contexts
```

Change cluster name
```
vim /etc/rancher/k3s/k3s.yaml
```

edit
```
apiVersion: v1
clusters:
- cluster:
    server: https://10.10.10.11:6443 #set server ip
  name: cluster-rafi #set cluster name
contexts:
- context:
    cluster: cluster-rafi #set cluster name
    user: admin #set user
  name: admin-cluster #set context name
current-context: admin-cluster #set context name
users:
- name: admin #set user
```

## Manage RBAC
### User Account
Generate PKI Private Key and CSR
```
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr -subj "/CN=client/O=developer"
```

Create a CertificateSigningRequest
```
cat client.csr | base64 | tr -d "\n"

cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: client
spec:
  groups:
  - system:authenticated
  request: $(cat client.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

Approve the CertificateSigningRequest 
```
kubectl get csr
kubectl certificate approve client
```

Get the certificate
```
kubectl get csr/client -o yaml
kubectl get csr client -o jsonpath='{.status.certificate}'| base64 -d > client.crt
```

Create Role
```
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods --namespace dev
```

Create Role Binding
```
kubectl create rolebinding developer-binding --role=developer --user=client --namespace dev
```

Add to kubeconfig
```
kubectl config set-credentials client --client-key=client.key --client-certificate=client.crt --embed-certs=true
kubectl config set-context client-cluster --cluster=cluster-rafi --user=client --namespace=dev
kubectl config get-contexts
kubectl config use-context client-cluster
```

### Service Account
Create Service Account
```
kubectl get serviceaccount
kubectl create sa pod-discovery
```

Attach Role Binding to Service Account
```
kubectl edit rolebinding dev-binding
```

add
```
subjects:
- kind: ServiceAccount
  name: pod-discovery
  namespace: default
```

Create pod use Service Account
```
vim pod-sa.yml
```

edit
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: pod-sa
spec:
  serviceAccount: pod-discovery
  containers:
  - image: nginx
    name: pod-nginx
```

apply pod
```
kubectl apply -f pod-sa.yml
```

Test pod with Service Account
```
kubectl exec -it pod-sa -- /bin/bash
```

execute command on pod
```
APISERVER=https://kubernetes.default.svc
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
TOKEN=$(cat ${SERVICEACCOUNT}/token)
CACERT=${SERVICEACCOUNT}/ca.crt

curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods
```

## Kubernetes Dashboard
Deploy Dashboard UI
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Expose Dashboard UI
```
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```

Create Service Account
```
kubectl create sa admin-user --namespace kubernetes-dashboard
```

Create ClusterRoleBinding
```
kubectl create clusterrolebinding admin-access --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-user
```

Getting a Bearer Token
```
kubectl -n kubernetes-dashboard create token admin-user
```

## Backup and Restore etcd
Install etcd client
```
apt install etcd-client
```

Set Environtment Variable etcd
```
export ETCDCTL_API=3
export ENDPOINT=127.0.0.1:2379
export ETCDCTL_CACERT=/var/lib/rancher/k3s/server/tls/etcd/client.crt
export ETCDCTL_CERT=/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt
export ETCDCTL_KEY=/var/lib/rancher/k3s/server/tls/etcd/server-ca.key
```

Manage etcd check healthy and member
```
etcdctl --endpoints $ENDPOINT -w table endpoint status
etcdctl --endpoints $ENDPOINT -w table endpoint health
etcdctl --endpoints $ENDPOINT -w table member list
```

Manage etcd kubernetes config
```
etcdctl --endpoints $ENDPOINT get /registry/ --prefix --keys-only
etcdctl --endpoints $ENDPOINT get /registry/ --prefix --keys-only | grep pods/default
etcdctl --endpoints $ENDPOINT get /registry/pods/dev/pod-dev
```

Backup etcd
```
etcdctl --endpoints $ENDPOINT snapshot save backup-cluster-1.26
etcdctl --endpoints $ENDPOINT snapshot status backup-cluster-1.26
```

Restore etcd (opsional)
```
etcdctl --endpoints $ENDPOINT snapshot restore backup-cluster-1.26
```

## Upgrade Cluster
Install the system-upgrade-controller
```
kubectl apply -f https://github.com/rancher/system-upgrade-controller/releases/latest/download/system-upgrade-controller.yaml
```

Configure plans
```
vim k3s-upgrade.yml
```

edit
```
# Server plan
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: server-plan
  namespace: system-upgrade
spec:
  concurrency: 1
  cordon: true
  nodeSelector:
    matchExpressions:
    - key: node-role.kubernetes.io/control-plane
      operator: In
      values:
      - "true"
  serviceAccountName: system-upgrade
  upgrade:
    image: rancher/k3s-upgrade
  version: v1.27.3+k3s1
---
# Agent plan
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: agent-plan
  namespace: system-upgrade
spec:
  concurrency: 1
  cordon: true
  nodeSelector:
    matchExpressions:
    - key: node-role.kubernetes.io/control-plane
      operator: DoesNotExist
  prepare:
    args:
    - prepare
    - server-plan
    image: rancher/k3s-upgrade
  serviceAccountName: system-upgrade
  upgrade:
    image: rancher/k3s-upgrade
  version: v1.27.3+k3s1
```

Apply plans
```
kubectl apply -f k3s-upgrade.yml
```

Check a node in a cluster
```
kubectl get node
```