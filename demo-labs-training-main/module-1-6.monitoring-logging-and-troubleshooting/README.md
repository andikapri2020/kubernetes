# Monitoring, Logging, and Troubleshooting

## Monitoring
### Use Metric Server API Kubernetes
Resource usage monitoring with kubectl
```
kubectl top pod
kubectl top node
```

### Use Prometheus Server and Grafana
Install Prometheus Server on cluster with helm
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus -n monitoring
```

Expose prometheus server
```
kubectl edit svc prometheus -n monitoring
```

Install Grafana on cluster with helm
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana -o monitoring
```

Expose grafana
```
kubectl edit svc prometheus -n monitoring
```

Kubernetes Dashboard Grafana template id
```
10000
12740
```

## Logging
### Use kubectl
```
kubectl logs -f pod-name
kubectl logs -f -l key=value
kubectl logs -f deployment-name/pod-name
```
### Use k9s
Install k9s
```
curl -LO https://github.com/derailed/k9s/releases/download/v0.28.2/k9s_Linux_amd64.tar.gz
tar -zxvf k9s_Linux_amd64.tar.gz
```

Run k9s
```
./k9s
```

## Troubleshooting
### Troubleshooting Applications
Debug Pods
```
kubectl get pod
kubectl describe pod-name

kubectl get replicaset
kubectl describe replicaset replicaset-name
```

Debug Services
```
kubectl get services
kubectl get endpoints
```

Debug a StatefulSets
```
kubectl get statefulset
kubectl describe statefulset statefulset-name
```

Debug event
```
kubectl get events
kubectl get events --namespace=my-namespace
```

### Troubleshooting Clusters
Listing your cluster 
```
kubectl get node -o wide
kubectl cluster-info dump
```

Debugging a down/unreachable node
```
kubectl get nodes
kubectl describe node node-name
```