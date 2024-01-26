# Load Balancing

## Setup NGINX Load Balancer : Layer 4
```
vim /etc/nginx/nginx.conf
```

```
######## TCP Load Balancing ###############
stream {

upstream galera_cluster {
    server 10.10.10.186:3306;
    server 10.10.10.56:3306 backup;
  }

server {
    listen 3306;
    proxy_pass galera_cluster;

    access_log  /var/log/nginx/galera.access.log;
    error_log /var/log/nginx/galera.error.log;
  }
}
```

```
nginx -t
```

```
nginx -s reload
```

```
mysql -u remote -p -h 10.10.10.67 -e "show variables where variable_name= 'hostname';"
```

## Setup NGINX Load Balancer : Layer 7
```
vim /etc/nginx/nginx.conf
```

```
http {
######## HTTP Load Balancing ###############
upstream backend {
        server 10.10.10.186;
        server 10.10.10.56;
    }

server {
        location / {
            proxy_pass http://backend;

            access_log  /var/log/nginx/http.access.log  main;
            error_log /var/log/nginx/http.error.log;
        }
    }
}
```

```
nginx -t
```

```
nginx -s reload
```

```
curl http://10.10.10.67 
```

## Setup NGINX Load Balancer : K3S Deployment
Setup NGINX Load Balancer for Internal Access
```
stream {
    upstream control-plane {
       server 10.10.10.x:6443;
       server 10.10.10.x:6443;
       server 10.10.10.x:6443;
    }
    
    server {
        listen 6443;
        proxy_pass control-plane;
    }
}
``` 

Setup NGINX Load Balancer for External Access
```
http {
  ######## HTTP Load Balancing ##############
    upstream worker_servers_http {
        least_conn;
        server 10.10.10.x:30767 max_fails=3 fail_timeout=5s;
        server 10.10.10.x:30767 max_fails=3 fail_timeout=5s;
        server 10.10.10.x:30767 max_fails=3 fail_timeout=5s;
    }
    server {
        listen 80;

        location / {
            proxy_pass http://worker_servers_http;
        }
    }

######## HTTPS Load Balancing ##############
    upstream worker_servers_https {
        least_conn;
        server 10.10.10.x:32624 max_fails=3 fail_timeout=5s;
        server 10.10.10.x:32624 max_fails=3 fail_timeout=5s;
        server 10.10.10.x:32624 max_fails=3 fail_timeout=5s;
    }
    server {
        listen 443 ssl;
        ssl_certificate /path/to/tls.crt;
        ssl_certificate_key /path/to/key.key;
        location / {
            proxy_pass https://worker_servers_https;
            proxy_set_header Host $host;
            proxy_ssl_server_name on;
            proxy_ssl_name $host
        }
    }
}
```

## Setup Monitoring for Load Balancer
### Node Exporter
Download node exporter package
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
```

Extract node exporter package
```
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
```

move the node exporter binary to /usr/local/bin
```
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
```

Create a node_exporter user to run the node exporter service
```
sudo useradd -rs /bin/false node_exporter
```

Create a node_exporter service file under systemd
```
vim /etc/systemd/system/node_exporter.service
```

Set this
```
[Unit]
Description=Node Exporter
After=network.target
 
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
 
[Install]
WantedBy=multi-user.target
```

Run node exporter as a Service
```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

### NGINX Exporter
Download nginx exporter package
```
wget https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v1.0.0/nginx-prometheus-exporter_1.0.0_linux_amd64.tar.gz
```

Extract nginx exporter package
```
tar xvfz nginx-prometheus-exporter_1.0.0_linux_amd64.tar.gz
```

move the nginx exporter binary to /usr/local/bin
```
sudo mv nginx-prometheus-exporter /usr/local/bin/
```

Create a nginx_exporter user to run the node exporter service
```
sudo useradd -rs /bin/false nginx_exporter
```

Create a nginx_exporter service file under systemd
```
vim /etc/systemd/system/nginx_exporter.service
```

Set this
```
[Unit]
Description=NGINX Exporter
After=network.target

[Service]
User=nginx_exporter
Group=nginx_exporter
Type=simple
ExecStart=nginx-prometheus-exporter --nginx.scrape-uri=http://localhost:80/stub_status --web.listen-address=0.0.0.0:9101
[Install]
WantedBy=multi-user.target
```

Run nginx exporter as a Service
```
sudo systemctl daemon-reload
sudo systemctl start nginx_exporter
sudo systemctl enable nginx_exporter
```

### MySQL Exporter
Download mysqld exporter package
```
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.15.0/mysqld_exporter-0.15.0.linux-amd64.tar.gz
```

Extract mysqld exporter package
```
tar xvfz mysqld_exporter-0.15.0.linux-amd64.tar.gz
```

move the mysqld exporter binary to /usr/local/bin
```
sudo mv mysqld_exporter-0.15.0.linux-amd64/mysqld_exporter /usr/local/bin/
```

Create a mysqld_exporter user to run the node exporter service
```
sudo useradd -rs /bin/false mysqld_exporter
```

Create a mysql_exporter service file under systemd
```
vim /etc/systemd/system/mysqld_exporter.service
```

Set this
```
[Unit]
Description=Prometheus MySQL Exporter
After=network.target

[Service]
User=mysql_exporter
Group=mysql_exporter
Type=simple
Restart=always
ExecStart=/usr/local/bin/mysqld_exporter \
--config.my-cnf /usr/local/bin/.my.cnf \
--collect.global_status \
--collect.info_schema.innodb_metrics \
--collect.auto_increment.columns \
--collect.info_schema.processlist \
--collect.binlog_size \
--collect.info_schema.tablestats \
--collect.global_variables \
--collect.info_schema.query_response_time \
--collect.info_schema.userstats \
--collect.info_schema.tables \
--collect.perf_schema.tablelocks \
--collect.perf_schema.file_events \
--collect.perf_schema.eventswaits \
--collect.perf_schema.indexiowaits \
--collect.perf_schema.tableiowaits \
--collect.slave_status \
--web.listen-address=0.0.0.0:9102

[Install]
WantedBy=multi-user.target
```

Run mysql exporter as a Service
```
sudo systemctl daemon-reload
sudo systemctl start mysql_exporter
sudo systemctl enable mysql_exporter
```