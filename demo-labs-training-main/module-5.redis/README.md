# Redis
## Labs
On environtment
```
redis-01 (primary) ip_address = 10.10.10.72, port = 6379
redis-02 (replica) ip_address = 10.10.10.180, port = 6379
redis-03 (replica) ip_address = 10.10.10.49, port = 6379
```

## Setup Redis
### Install Redis
Install Redis on Linux Ubuntu/Debian
```
# download gpg keys
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

# add repository redis
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

# update and install redis
sudo apt-get update
sudo apt-get install redis
```

Verify
```
# check service redis
systemctl status redis-server

# check listen port
ss -tulpn
```

### Connect to Redis
Redis CLI
```
# redis connect locally
redis-cli

# redis connect remotely
redis-cli -h 10.10.10.72 -p 6379 --user redis
```

RedisInsight
```
# redis deploy on docker
docker run -dit -v redisinsight:/db -p 5000:5000 --name redisinsight oblakstudio/redisinsight
```

Clients
```
# python client
pip install redis

# script
## redis-py.py

import redis
r = redis.Redis(host='localhost', port=6379, decode_responses=True)
data = r.keys()
print(data)
```

## Redis Administration
### Configure Redis
Config redis 
```
vim /etc/redis/redis.conf
```

Set this
```
bind 0.0.0.0
protected-mode yes
port 6379
```

Restart redis
```
systemctl restart redis-server
```

Config redis from CLI
```
CONFIG GET *
CONFIG SET protected-mode no
CONFIG REWRITE
```

Restart redis
```
systemctl restart redis-server
```

### Manage RBAC Redis
Config redis
```
vim /etc/redis/redis.conf
```

Set this
```
requirepass redis
user rafiryd +@all allkeys on >@dmin123
```

Restart redis
```
systemctl restart redis-server
```

Access redis
```
redis-cli --user rafiryd -h 10.10.10.72 -p 6379
```

Create ACL user from CLI
```
# create user
acl setuser admin on +@all allkeys >admin123

# list acl
acl list

# check user
acl users
```

### Manage Redis Persistent
#### No persistence
Configure redis
```
vim /etc/redis/redis.conf
```

Set this
```
save ""
```

#### RDB
Saving a Snapshot
```
vim /etc/redis/redis.conf
```

Set this
```
dbfilename backup_dump.rdb
save 3600 1
save 300 100
save 60 1000
save 20 3
```

Restart redis
```
systemctl restart redis-server
```

Test verify
```
SET a 1
SET b 2
SET c 3
```

Verify
```
ls /var/lib/redis/
```

#### AOF 
Enable AOF
```
vim /etc/redis/redis.conf
```

Set this
```
appendonly yes
appendfilename "appendonly.aof"
appenddirname "appendonlydir"

# appendfsync always
appendfsync everysec
# appendfsync no
```

Restart redis
```
systemctl restart redis-server
```

Test verify
```
SET a 1
SET b 2
SET c 3
```

Verify
```
ls /var/lib/redis/appendonlydir
```

## Redis High Availability
### Replication
#### Set on primary node
Config replication
```
vim /etc/redis/redis.conf
```

Set this
```
masterauth redis
```

Restart primary redis
```
systemctl restart redis-server
```

#### Set on replica node
Config replication
```
vim /etc/redis/redis.conf
```

Set this
```
bind 0.0.0.0
protected-mode no
port 6379

replicaof 10.10.10.72 6379
masterauth redis

requirepass redis
```

Restart replica redis
```
systemctl restart redis-server
```

#### Verify
Verify access on primary
```
redis-cli -h 10.10.10.72 -p 6379 --user redis
```

Test replication
```
SET a 1
SET b 2
SET c 3
```

Verify on replica
```
redis-cli -h 10.10.10.72 -p 6379 --user redis
```

Verify replication
```
keys *
```

### Sentinels
#### Installation
Install package sentinel
```
apt install redis-sentinel
```

#### Config Sentinel
Config sentinel
```
vim /etc/sentinel/sentinel.conf
```

Set this
```
port 26379
sentinel monitor myprimary 10.10.10.72 6379 3
sentinel down-after-milliseconds myprimary 5000
sentinel failover-timeout myprimary 60000
sentinel auth-pass myprimary redis
```

#### Verify
Verify access on sentinel
```
redis-cli -h 10.10.10.72 -p 26379 --user redis
```

Check info sentinel
```
# Provides information about the Primary
SENTINEL master myprimary

# Gives you information about the replicas connected to the Primary
SENTINEL replicas myprimary

# Provides information on the other Sentinels
SENTINEL sentinels myprimary

# Provides the IP address of the current Primary
SENTINEL get-master-addr-by-name myprimary
```

### Clustering
#### Setup Clustering Redis
Configure Redis Cluster
```
vim redis.conf
```

Set this master config
```
# redis.conf file
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

Set this replica config
```
# redis.conf file
port 7001
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

Start Service Redis
```
redis-server ./redis.conf
```

Start Cluster Redis
```
redis-cli --cluster create 10.10.10.72:7000 10.10.10.180:7000 10.10.10.49:7000 \
10.10.10.72:7001 10.10.10.180:7001 10.10.10.49:7001 \
--cluster-replicas 1
```

Verify Redis Cluster
```
# Access Redis
redis-cli -h 10.10.10.72 -p 7000 --user redis

# Check Cluster slots
redis-cli -p 7000 cluster slots

# Check Cluster node
redis-cli -p 7000 cluster nodes
```

## Observability
Redis Info Command
```
# all info
INFO

# client info
INFO CLIENTS
```

Latency and stats data via redis-cli options
```
# Continuously sample latency
redis-cli --latency

# In order to sample for longer than one second you can use latency-history which has a default interval of 15 seconds but can be specified using the -i param
redis-cli --latency-history -i 60 
```

check stats data
```
# Get rolling stats from the server using the stat flag.
redis-cli -a redis --stat

# Redis includes a MEMORY command that includes a subcommand to get stats.
memory stats
```

Latency monitoring config
```
# If the debugging session is intended to be temporary the threshold can be set via redis-cli.
CONFIG SET latency-monitor-threshold 10

# To disable the latency framework the threshold should be set back to 0.
CONFIG SET latency-monitor-threshold 0
```

Latency monitoring 
```
# For example, you can use the LATENCY LATEST subcommand and you may see some data like this:
latency latest

# Some of the latency commands require a specific event be passed.
latency graph command
```

Install redis datasource on grafana, refer to this [link](https://grafana.com/grafana/plugins/redis-datasource/).

## Running Redis on Kubernetes
### Running High Availability Redis on Kubernetes with Helm
Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Install Helm Repository
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Install Helm Redis
```
helm install my-redis bitnami/redis --version 18.7.1
```

Create custom values
```
vim values.yml
```

Set this
```
auth:
  password: "redis"
replica:
  replicaCount: 3
sentinel:
  enabled: true
```

Create service account 
```
kubectl create sa my-redis
```

Update Helm enable Redis Sentinel
```
helm upgrade my-redis bitnami/redis -f values.yml
```

Create redis pod
```
vim redis-cli.yml
```

Set this
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: redis-cli
  name: redis-cli
spec:
  containers:
  - image: redis
    name: redis
```

Deploy redis pod
```
kubectl apply -f redis-cli.yml
```

Verify
```
# exec to pod redis cli
kubectl exec -it redis-cli -- /bin/bash

# connect to service redis
redis-cli -h my-redis -p 6379

# connect to service sentinel
redis-cli -h my-redis -p 26379
```

### Running High Availability Redis on Kubernetes with Operator
Installation Helm chart
```
helm repo add ot-helm https://ot-container-kit.github.io/helm-charts
```

Deploy redis operator
```
helm upgrade redis-operator ot-helm/redis-operator \
--install --create-namespace --namespace ot-operators
```

Verify deployment
```
helm test redis-operator --namespace ot-operators
```

Creating redis cluster, standalone, replication and sentinel setup.
```
# Create redis standalone setup
helm upgrade redis ot-helm/redis \
--install --namespace ot-operators

# Create redis cluster setup
helm upgrade redis-cluster ot-helm/redis-cluster \
--set redisCluster.clusterSize=3 --install \ 
--namespace ot-operators

# Create redis replication setup
helm upgrade redis-replication ot-helm/replication \
--install --namespace ot-operators

# Create redis sentinel setup
helm upgrade redis-sentinel ot-helm/sentinel \
--install --namespace ot-operators
```