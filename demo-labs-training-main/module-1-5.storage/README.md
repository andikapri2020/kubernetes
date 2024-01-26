# Storage

## NFS Volume
Check mount endpoint
```
showmount -e 10.10.x.x
```

Install nfs client
```
apt install nfs-common
```

Mount nfs volume to host
```
mount -t nfs 10.10.x.x:/path /mnt
df -h /mnt
```

Testing
```
cd /mnt
mkdir folder
touch folder/file{1..3}.txt
```

## Container Volume
Create deployment template
```
vim apps-vol.yml
```

Add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps-vol
spec:
  selector:
    matchLabels:
      apps: apps-vol
  template:
    metadata:
      labels:
        apps: apps-vol
    spec:
      volumes:
        - name: volume-nfs
          nfs:
            server: 10.10.10.90
            path: /var/nfs/vol-b
      containers:
      - name: ct-apps
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: volume-nfs
          mountPath: /usr/share/nginx/html
```

Test Volume
```
kubectl exec -it apps-vol-xxx -- /bin/bash
df -h 
cd /usr/share/nginx/html
```

## Persitent Volume
Create Persistent Volume
```
vim pv-a.yml
```

Add this
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-a
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  storageClassName: pv-a
  nfs:
    path: /path
    server: 10.10.x.x
```

Check Persistent Volume
```
kubectl get pv
```

## Persistent Volume Claim
Create persistent volume claim
```
vim pvc-a
```

Add this
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-a
spec:
  storageClassName: pv-a
  resources:
    requests:
      storage: 1Gi
  accessModes:
    - ReadWriteMany
```

Check persistent volume claim
```
kubectl get pv,pvc
```

## Container use persistent volume claim
Create deployment template
```
vim apps-pvc-vol.yml
```

Add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps-pvc-vol
spec:
  selector:
    matchLabels:
      apps: apps-pvc-vol
  template:
    metadata:
      labels:
        apps: apps-pvc-vol
    spec:
      volumes:
        - name: volume-nfs
          persistentVolumeClaim:
            claimName: pvc-a
      containers:
      - name: ct-apps
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: volume-nfs
          mountPath: /usr/share/nginx/html
```

Test volume
```
kubectl exec -it apps-vol-xxx -- /bin/bash
df -h 
cd /usr/share/nginx/html
```

## Dynamic Provisioner
Install helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Install dynamic nfs provisioner
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=10.10.x.x --set nfs.path=/path
```

Create StorageClass
```
vim nfs-low.yml
```

Add this
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-low
provisioner: cluster.local/nfs-subdir-external-provisioner
parameters:
  server: 10.10.x.x
  path: /path
  readOnly: "false"

```

Create deployment template
```
vim apps-provision.yml
```

Add this
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apps-provision
spec:
  selector:
    matchLabels:
      apps: apps-provision
  template:
    metadata:
      labels:
        apps: apps-provision
    spec:
      volumes:
        - name: volume-nfs
          persistentVolumeClaim:
            claimName: apps-a
      containers:
      - name: ct-apps
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: volume-nfs
          mountPath: /usr/share/nginx/html
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: apps-a
spec:
  storageClassName: nfs-low
  resources:
    requests:
      storage: 1Gi
  accessModes:
    - ReadWriteMany
```

Test volume
```
kubectl exec -it apps-vol-xxx -- /bin/bash
df -h 
cd /usr/share/nginx/html
```