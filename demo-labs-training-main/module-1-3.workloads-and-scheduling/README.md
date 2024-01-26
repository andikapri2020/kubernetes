# Kubernetes Workloads and Scheduling

## Workloads
### Pod
you can create pod directly
```
kubectl run pod-test --image nginx
```

or you can create template for reusable create pod
```
kubectl run pod-test --image nginx --dry-run=client -o yaml > pod.yml
```

config template
```
vim pod.yml
```

edit config
```
apiVersion: v1
kind: Pod
metadata:
 name: pod-nginx
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

and to create pod from a template like this
```
kubectl apply -f pod.yml
```

### Pod label
add label directly
```
kubectl label pod pod-name key=value
```

overwrite label
```
kubectl label pod pod-name key=value --overwrite
```

delete label
```
kubectl label pod pod-name key-
```

add label on template
```
cp pod.yml pod-label.yml
vim pod-label.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
# add this
  labels:   
    apps-type: frontend
  name: pod-label
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

deploy pod label
```
kubectl apply -f pod-label.yml
```

filter pod by label
```
kubectl get pod -l key=value
kubectl get pod -l key
```

### Pod Annotations
add annotate directly
```
kubectl annotate pod pod-name key=value
```

copy template
```
cp pod.yml pod-annotation.yml
vim pod-annotation.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    apps-type: frontend
# add this
  annotations:
    description: ini belajar membuat pod
  name: pod-annotate
spec:
  containers:
  - image: nginx
    name: ct-nginx
```

deploy pod annotations
```
kubectl apply -f pod-annotation.yml
```

### Pod Liveness probes
copy template
```
cp pod.yml pod-liveness.yml
vim pod-liveness.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    apps-type: frontend
# add this
  annotations:
    description: ini belajar membuat pod
  name: pod-liveness
spec:
  containers:
  - image: nginx
    name: ct-nginx
# add this
  livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
```

deploy pod liveness
```
kubectl apply -f pod-liveness.yml
```

### Pod Readiness probes
copy template
```
cp pod.yml pod-readiness.yml
vim pod-readiness.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    apps-type: frontend
# add this
  annotations:
    description: ini belajar membuat pod
  name: pod-readiness
spec:
  containers:
  - image: nginx
    name: ct-nginx
# add this
  readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
```

deploy pod readiness
```
kubectl apply -f pod-readiness.yml
```

### Pod Startup probes
copy template
```
cp pod.yml pod-startup.yml
vim pod-startup.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    apps-type: frontend
# add this
  annotations:
    description: ini belajar membuat pod
  name: pod-startup
spec:
  containers:
  - image: nginx
    name: ct-nginx
# add this
    startupProbe:
      httpGet:
        path: /health
        port: 80
      failureThreshold: 30
      periodSeconds: 1
```

deploy pod startup
```
kubectl apply -f pod-startup.yml
```

### Pod Resource Limit
copy template
```
cp pod.yml pod-limit.yml
vim pod-limit.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    apps-type: frontend
# add this
  annotations:
    description: ini belajar membuat pod
  name: pod-limit
spec:
  containers:
  - image: nginx
    name: ct-nginx
# add this
    resources:
      requests:
        memory: "1024Mi"
        cpu: "100m"
      limits:
        memory: "1024Mi"
        cpu: "200m"
    ports:
      - containerPort: 80
```

deploy pod limit
```
kubectl apply -f pod-limit.yml
```

### Namespace
create namespace dev
```
kubectl create ns dev
```

create pod specific on namespace
```
kubectl run pod-dev -n dev --image nginx
```

list pod on specific namespace
```
kubectl get pod -n dev
```

create template pod on specific namespace
```
cp pod.yml pod-ns-dev.yml
vim pod-ns-dev.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  label:
    env: dev
    apps-type: frontend-dev
  name: pod-nginx-dev
# add this
  namespace: dev
spec:
  containers:
  - name: ct-nginx
    image: nginx
    ports:
      - containerPort: 80
```

deploy pod namespace
```
kubectl apply -f pod-ns-dev.yml
```

### Pod env variable
copy template
```
cp pod.yml pod-env.yml
vim pod-env-db.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-env-db
  labels:
    apps-type: database
  annotations:
    deskripsi: ini aplikasi dibuat untuk testing
spec:
  containers:
  - name: ct-database
    image: mariadb
    ports:
      - containerPort: 3306
# add this
    env:
      - name: MARIADB_ROOT_PASSWORD
        value: admin123
```

deploy pod env
```
kubectl apply -f pod-env-db.yml
```

### Configmaps
#### Environtment Variable
create configmaps env variable
```
vim config-db.yml
```

add this
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-db
data:
  MARIADB_ROOT_PASSWORD: admin123
  MARIADB_DATABASE: training
  MARIADB_USER: userku
  MARIADB_PASSWORD: userku123
```

deploy configmaps
```
kubectl apply -f config-db
```

create pod to use env variable configmaps
```
vim pod-config-db.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
 name: pod-config-db
spec:
  containers:
  - image: mariadb
    name: ct-db
    envFrom:
    - configMapRef:
        name: config-db
```

deploy pod use env variable configmaps
```
kubectl apply -f pod-config-db.yml
```

#### Config file
create configmaps config file
```
vim config-nginx.yml
```

add this
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events {
    }
    http {
      server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # Set nginx to serve files from the shared volume!
        root /usr/share/nginx/html;
        server_name _;
        location / {
          try_files $uri $uri/ =404;
        }
        location /detik {
          proxy_pass http://detik.com;
        }
      }
    }
```

deploy configmaps
```
kubectl apply -f config-nginx.yml
```

create pod to use config on configmaps
```
vim pod-config-nginx.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx-config
  labels:
    apps-type: backend
  annotations:
    deskripsi: ini aplikasi dibuat untuk testing
spec:
# add this
  volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
      items:
        - key: nginx.conf
          path: nginx.conf
  containers:
  - name: ct-nginx
    image: nginx
    ports:
      - containerPort: 80
# add this
    volumeMounts:
      - name: nginx-config-volume
        mountPath: /etc/nginx/
        readOnly: true
```

deploy pod use config on configmaps
```
kubectl apply -f pod-config-nginx.yml
```

### Secret
#### Environtment Variable
create .env
```
vim .env
```

add this
```
APP_PORT=3000
```

create secret env variable
```
kubectl create secret generic env-file --from-file .env
```

create pod to use secret
```
vim pod-apps.yml
```

add this
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-apps
  labels:
    apps-type: backend
  annotations:
    deskripsi: ini aplikasi dibuat untuk testing
spec:
  volumes:
  - name: secret-env
    secret:
      secretName: env-file
      items:
        - key: .env
          path: .env
  containers:
  - name: ct-apps
    image: rydrafi/simple-apps
    ports:
      - containerPort: 80
    volumeMounts:
      - name: secret-env
        mountPath: /app/.env
        subPath: .env
        readOnly: true
```

deploy pod use secret
```
kubectl apply -f pod-apps.yml
```

### ReplicaSet
create replicaset template
```
vim replica-nginx.yml
```

add this
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      apps-type: nginx-replicaset
  template:
    metadata:
      labels:
        apps-type: nginx-replicaset
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

deploy replicaset
```
kubectl apply -f replica-nginx.yml
```

### Deployment
create deployment template
```
vim deploy-nginx.yml
```

add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      apps-type: nginx-deployment
  template:
    metadata:
      labels:
        apps-type: nginx-deployment
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
```

deploy deployment
```
kubectl apply -f deploy-nginx.yml
```

### HPA
#### Deployment
create deployment template
```
vim deployment-hpa.yml
```

add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-hpa
spec:
  selector:
    matchLabels:
      type: testing-hpa
  template:
    metadata:
      labels:
        type: testing-hpa
    spec:
      containers:
      - name: ct-php-apache
        image: rydrafi/hpa-example
        resources:
          limits:
            memory: "100Mi"
            cpu: "100m"
        ports:
        - containerPort: 80
```

deploy deployment
```
kubectl apply -f deployment-hpa.yml
```

#### Service
create service
```
vim service-hpa.yml
```

add this
```
apiVersion: v1
kind: Service
metadata:
 name: svc-hpa
spec:
 ports:
 - port: 80
 selector:
  type: testing-hpa
```

deploy service
```
kubectl apply -f service-hpa.yml
```

#### HPA
create deployment HPA template
```
vim hpa.yml
```

add this
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-testing
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: deployment-hpa
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

deploy deployment hpa
```
kubectl apply -f hpa.yml
```

testing hpa
```
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://svc-hpa; done"
```

### Statefulset
create statefulset template
```
vim db-sts.yml
```

add this
```
apiVersion: v1
kind: Service
metadata:
  name: svc-db
  labels:
    app: db
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: db
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-db-a
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: db
  hostPath:
    path: /data/db-a
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-db-b
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: db
  hostPath:
    path: /data/db-b
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  selector:
    matchLabels:
      app: db
  replicas: 2
  template:
    metadata:
      labels:
        app: db
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: ct-db
        image: mariadb
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: database
          mountPath: /var/lib/mysql/
        envFrom:
        - configMapRef:
            name: config-db
  volumeClaimTemplates:
  - metadata:
      name: database
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: db
      resources:
        requests:
          storage: 1Gi
```

deploy statefulset
```
kubectl apply -f db-sts.yml
```

## Scheduling
### NodeSelector
Create label to node
```
kubectl label node node-name key=value
```

Creare workload with selector
```
vim apps-arm.yml
```

Add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps-arm
spec:
  replicas: 2
  selector:
    matchLabels:
      apps-type: apps-arm
  template:
    metadata:
      labels:
        apps-type: apps-arm
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
      nodeSelector:
        cpu: arm
```

Deploy workload to specific selector on node
```
kubectl apply -f apps-arm.yml
```

### Affinity
Create workload with affinity
```
vim apps-affinity.yml
```

Add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps-affinity
spec:
  selector:
    matchLabels:
      apps-type: apps-affinity
  replicas: 2
  template:
    metadata:
      labels:
        apps-type: apps-affinity
    spec:
      containers:
      - name: ct-nginx
        image: nginx
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: apps-type
                operator: In
                values:
                - apps-affinity
            topologyKey: "kubernetes.io/hostname"
```

Deploy workload to specific selector on node
```
kubectl apply -f apps-affinity.yml
```

### Anti Affinity
Create workload with affinity
```
vim apps-anti-affinity.yml
```

Add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps-anti-affinity
spec:
  selector:
    matchLabels:
      apps-type: apps-anti-affinity
  replicas: 2
  template:
    metadata:
      labels:
        apps-type: apps-anti-affinity
    spec:
      containers:
      - name: ct-nginx
        image: nginx
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: apps-type
                operator: In
                values:
                - apps-anti-affinity
            topologyKey: "kubernetes.io/hostname"
```

Deploy workload to specific selector on node
```
kubectl apply -f apps-anti-affinity.yml
```

### Taints
Add taint label to node
```
kubectl taint node node key=value:NoSchedule
kubectl taint node node key=value:NoExecute
```

Test deploy pod 
```
kubectl run pod-test --image nginx
```

### Tolerations
Create workloads with toleration
```
vim apps-toleration.yml
```

Add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps-toleration
spec:
  replicas: 2
  selector:
    matchLabels:
      apps-type: apps-toleration
  template:
    metadata:
      labels:
        apps-type: apps-toleration
    spec:
      containers:
      - name: ct-nginx
        image: nginx
        ports:
        - containerPort: 80
      tolerations:
      - key: "ready"
        operator: "Equal"
        value: "yes"
        effect: "NoSchedule"
      #- key: "ready"
      #  operator: "Equal"
      #  value: "no"
      #  effect: "NoExecute"
      #  tolerationSeconds: 60
```

Deploy workloads with toleration
```
kubectl apply -f apps-toleration.yml
```