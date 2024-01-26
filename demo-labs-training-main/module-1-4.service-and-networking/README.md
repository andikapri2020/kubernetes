# Kubernetes Service and Networking

## Service 
### Service ClusterIP
Create service clusterip
```
vim apps-nginx.yml
```

Add this
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: apps-nginx
  name: apps-nginx
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: apps-nginx
  type: ClusterIP
```

Test service
```
kubectl get svc
curl http://cluster_ip
```

### Service NodePort
Create service clusterip
```
vim apps-nginx.yml
```

Add this
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: apps-nginx
  name: apps-nginx
spec:
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30000 #add
  selector:
    app: apps-nginx
  type: NodePort #add
```

Test service
```
kubectl get svc
curl http://cluster_ip
```

Open url on browser
```
http://ip_node:node_port
```

### Service LoadBalancer
Create service clusterip
```
vim apps-nginx.yml
```

Add this
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: apps-nginx
  name: apps-nginx
spec:
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30000
  selector:
    app: apps-nginx
  type: LoadBalancer #add
```

Test service
```
kubectl get svc
curl http://cluster_ip
```

## Ingress
### Ingress single path and single domain
Create ingress
```
vim apps-ingress.yml
```

Add this
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-nginx

spec:
  rules:
    - host: appsku.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  apps-nginx
                port:
                  number: 80
```

Test ingress
```
kubectl get ing
```

### Ingress multi path
Create ingress
```
vim apps-ingress.yml
```

Add this
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-nginx

spec:
  rules:
    - host: appsku.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  apps-nginx
                port:
                  number: 80
          - path: /apache
            pathType: Prefix
            backend:
              service:
                name:  apps-apache
                port:
                  number: 80
```

Test ingress
```
kubectl get ing
```

### Ingress multi domain
Create ingress
```
vim apps-ingress.yml
```

Add this
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-nginx

spec:
  rules:
    - host: appsku.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  apps-nginx
                port:
                  number: 80
    - host: apache.appsku.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  apps-apache
                port:
                  number: 80
```

Test ingress
```
kubectl get ing
```

### TLS
Create certificate ssl
```
openssl req -newkey rsa:2048 -nodes -keyout domainku.key -x509 -days 365 -out domainku.crt
```

Create secret certifcate ssl
```
kubectl create secret tls ssl-domainku --key domainku.key --cert domainku.crt
```

### Ingress SSL
Create ingress
```
vim apps-ingress.yml
```

Add this
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-nginx
spec:
  tls:
  - hosts:
    - appsku.net
    - apache.appsku.net
    secretName: wildcard-appsku.net

  rules:
    - host: appsku.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  apps-nginx
                port:
                  number: 80
    - host: apache.appsku.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  apps-apache
                port:
                  number: 80
```

Test ingress
```
kubectl get ing
```

## Networking
### Networking Policy
Create Policy Block All Access
```
vim deny-all-access.yml
```

Add this
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-access
spec:
  podSelector:
    matchLabels:
      access: isolated
  policyTypes:
  - Ingress
  - Egress
```

Create Policy Allow www only
```
vim www-access.yml
```

Add this
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: www-only-access
spec:
  podSelector:
    matchLabels:
      access: www-only
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
       matchLabels:
          access: www-only
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    ports:
    - protocol: TCP
      port: 80
```

Create Policy Allow db only
```
vim db-access.yml 
```

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-only-access
spec:
  podSelector:
    matchLabels:
      access: db-only
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
       matchLabels:
          access: db-only
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    ports:
    - protocol: TCP
      port: 3306
```

Add network policy to pod
```
kubectl label pod pod-name key=value
```