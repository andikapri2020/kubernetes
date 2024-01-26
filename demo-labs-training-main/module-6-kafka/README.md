# Kafka

## Setup Kafka
### Install Java
Install java 11 jdk
```
apt install openjdk-11-jre openjdk-11-jre-headless
```

Verify
```
java --version
```

### Install Kafka
Download Kafka release package
```
curl -LO https://dlcdn.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz
```

extract kafka release package
```
tar -zxvf kafka_2.13-3.6.1.tgz
```

move kafka package to home dir
```
mv -v kafka_2.13-3.6.1 /usr/local/kafka
```

### Running Kafka Service
create systemd zookeeper
```
vim /etc/systemd/system/zookeeper.service
```

zookeeper.service
```
[Unit]
Description=Apache Zookeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

create systemd kafka
```
vim /etc/systemd/system/kafka.service
```

kafka.service
```
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64"
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```

### Verify
Check status service
```
systemctl status {kafka, zookeeper}
```

Check log service
```
journalctl -fu kafka
journalctl -fu zookeeper
```

### Connect to Kafka
Set PATH Environtment to load kafka command
```
export PATH=/usr/local/kafka/bin:$PATH 
```
