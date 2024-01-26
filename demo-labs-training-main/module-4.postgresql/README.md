# Postgresql

## Setup PostgreSQL : Installation
Install postgresql from PostgreSQL Apt Repository
```
# Create the file repository configuration:
sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-16' or similar instead of 'postgresql':
sudo apt-get -y install postgresql
```

Check packages
```
# Check installed packages
apt list --installed | grep postgresql
```

Access database
```
# login user postgres
su - postgres

# access postgres database
psql

# access postgres database
psql --host=localhost --port=5432 --dbname=postgres --username=postgres --password
```

Check Version
```
# check version postgres
SELECT version();
```

## Setup PostgreSQL : PostgreSQL Administration
### Managing Databases
Check database
```
# list all databases
\l
```

Create database
```
# create database
create database training;
```

Create database with parameter
```
# create database with parameter
create database training2
with
encoding = 'utf8'
owner = jhon
connection limit = 100;
```

Rename database
```
# rename database
ALTER DATABASE training
RENAME TO training1;
```

Copy database
```
# copy database from existing database
create database training with template training1;
```

Change owner database
```
# change owner database
alter database training1 owner to jhon;
```

Check active connection
```
# check active connection on database
SELECT pid, usename, client_addr FROM pg_stat_activity WHERE datname ='training';
```

Terminate active connection database
```
# terminate active connection on database
select pg_terminate_backend (pid) from pg_stat_activity where datname = 'training';
```

Drop database
```
# delete database
drop database training;
```

Using database
```
# use database
\c training1;
```

### Roles & Privileges
Create role jhon
```
# create role
create role jhon login password 'password';
```

Grant privileges on jhon
```
# grant access to role
grant insert,update,delete on accounts to jhon;

# grant access to generate primary key
grant usage, select on sequence accounts_user_id_seq to jhon;
```

Alter role jhon
```
# alter role 
alter role jhon superuser;
```

Check role
```
# check list role
\du
```

### Managing Tables
Create table account
```
# create table accounts
CREATE TABLE accounts (
        user_id serial PRIMARY KEY,
        username VARCHAR ( 50 ) UNIQUE NOT NULL,
        password VARCHAR ( 50 ) NOT NULL,
        email VARCHAR ( 255 ) UNIQUE NOT NULL,
        created_on TIMESTAMP NOT NULL,
        last_login TIMESTAMP 
);
```

Create table roles
```
# create table roles
CREATE TABLE roles(
   role_id serial PRIMARY KEY,
   role_name VARCHAR (255) UNIQUE NOT NULL
);
```

Create table account_roles
```
# create table account_roles
CREATE TABLE account_roles (
  user_id INT NOT NULL,
  role_id INT NOT NULL,
  grant_date TIMESTAMP,
  PRIMARY KEY (user_id, role_id),
  FOREIGN KEY (role_id)
      REFERENCES roles (role_id),
  FOREIGN KEY (user_id)
      REFERENCES accounts (user_id)
);
```

Check tables
```
# check list tables
\dt
```

Insert tables
```
# insert data on table
INSERT INTO accounts(username, password, email, created_on)
VALUES('rafi','password','rafi@example.com','12-28-2023');
```

Query data from tables
```
# query show data on table
select * from accounts;
```

### Managing Schema
Create schema
```
# create schema
CREATE SCHEMA sales;
```

Create table on schema
```
# create table on schema
CREATE TABLE sales.accounts(
   ...
);
```

Alter schema
```
# alter schema
alter schema sales
rename to finance
owner to rafi;
```   

Insert table on schema
```
# insert data on schema table
INSERT INTO finance.accounts(username, password, email, created_on)
VALUES('rafi','password','rafi@example.com','12-28-2023');
```

Check schema
```
# check list schema
\dn
```

Check current schema
```
# check current schema
SELECT current_schema();
```

Show search path
```
# show search schema
SHOW search_path;
```

Add new schema to search path
```
# add schema to search
set search_path to finance, public;
```

### Config postgresql enable remote access
Config main postgresql configuration
```
# main configuration
vim /etc/postgresql/16/main/postgresql.conf
```

Set this
```
# change listen address
listen_addresses = '*' 
```

Config postgresql access policy configuration
```
# access control configuration
vim /etc/postgresql/16/main/pg_hba.conf
```

Set this
```
# add rule
host    all             all             0.0.0.0/0               md5
```

Restart service
```
# restart service
systemctl restart postgresql
```

## Backup & Restore Databases
Logical Backup Database
```
# backup database to tar file
pg_dump -U jhon -W -F t training --host localhost > training-db.tar

# backup database table to sql file
pg_dump -t tb_data_1 -s test -p 5601 > backup_tb_data_1_test.sql

# backup all database
pg_dumpall -U postgres > all-db.sql
```

Physical Backup Database
```
# offline file system backup
tar -cf logical_primary_db.tar logical_primary_db

# online file system backup
pg_basebackup -h localhost -p 5432-D /home/postgres/pgbackup
```

Restore Database
```
# restore sql file
psql -p 5602 -d test < backup_tb_data_1_test.sql

# restore tar file
pg_restore -U jhon --host=localhost  --dbname=training --verbose training-db.tar

# restore physical backup
tar -xf logical_primary_db.tar
```

## High Availability, Load Balancing, and Replication
### Prequisite 
Create directory centralize data (all)
```
# create centralize directory
mkdir -p /var/lib/pgsql/data/

# change owner directory
chown postgres /var/lib/pgsql/data/
```

Set PATH env variable to load postgres command (all)
```
# login user postgres
su postgres

# add postgresql path to PATH
export PATH=$PATH:/usr/lib/postgresql/16/bin
```

### Setup on primary node
Initialize Primary Database
```
# create database cluster
initdb -D /var/lib/pgsql/data/primary_db
```

Config postgresql enable remote access
```
# main configuration
/var/lib/pgsql/data/primary_db/postgresql.conf
```

Change listen address and port
```
# change listen and port
listen_addresses = '*' 
port = 5441
```

Start database
```
# start database cluster
pg_ctl -D /var/lib/pgsql/data/primary_db/ start -l /var/log/postgresql/primary_db.log
```

Access database
```
# access on port 5441
psql --port 5441
```

Create sample data
```
# create database
create database test;

# use database
\c test;

# create table
create table tb_data_1(id int primary key, name varchar);

# insert data on table 
insert into tb_data_1 values(generate_series(1,10),'data'||generate_series(1,10));

# list database
\l

# list table
\dt

# show data on table
select * from tb_data_1;
```

### Streaming Replication - Asynchronous
#### Setup replication on primary node
Config replication
```
# main configuration
vim /var/lib/pgsql/data/primary_db/postgresql.conf
```

Set this
```
# change listen and port
listen_addresses = '*' 
port = 5441

# config wal replica
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
wal_keep_size = 0
max_slot_wal_keep_size = -1 
```

Create user replication
```
# create user
create user replica with login password 'password' replication;
```

Config access control user replication
```
# access control configuration
vim pg_hba.conf
```

Set this
```
# add rule
host    replication             replica             10.10.10.0/24            md5
```

Restart service
```
# restart
pg_ctl -D /var/lib/pgsql/data/primary_db/ restart -l /var/log/postgresql/primary_db.log
```

#### Setup on replica node
Copy the database to a replica server
```
# create replcation database
pg_basebackup -R -h 10.10.10.82 -U replica --port 5441 -D /var/lib/pgsql/data/replica_1_db -P
```

Config replication
```
# main configuration
vim /var/lib/pgsql/data/primary_db/postgresql.conf
```

Set this
```
# change listen and port
listen_addresses = '*' 
port = 5441

# config standby
hot_standby = on
```

Start database
```
# start database
pg_ctl -D /var/lib/pgsql/data/replica_db/ start -l /var/log/postgresql/replica_db.log
```

#### Verify replication
Expanded display
```
# expanded
\x
```

Show status replication on primary
```
# Show connected replicas and their status on the primary
select * from pg_stat_replication;
```

Show status replication on replica
```
# Shows the WAL receiver process status on Replica
select * from pg_stat_wal_receiver;
```

### Streaming Replication - Synchronous
#### Setup replication on primary node
Config replication
```
# main configuration
vim /var/lib/pgsql/data/sync_primary_db/postgresql.conf
```

Set this
```
# change listen and port
listen_addresses = '*' 
port = 5451

# config wal replica
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
wal_keep_size = 0
max_slot_wal_keep_size = -1 

# config sync
synchronous_commit = on
synchronous_standby_names = '*'
```

Create user replication
```
# create user
create user replica with login password 'password' replication;
```

Config access control user replication
```
# access control configuration
vim /var/lib/pgsql/data/sync_primary_db/pg_hba.conf
```

Set this
```
# add rule
host    replication             replica             10.10.10.0/24            md5
```

Restart service
```
# restart
pg_ctl -D /var/lib/pgsql/data/sync_primary_db/ restart -l /var/log/postgresql/sync_primary_db.log
```

#### Setup on replica node
Copy the database to a replica server
```
# create replcation database
pg_basebackup -R -h 10.10.10.82 -U replica --port 5441 -D /var/lib/pgsql/data/sync_replica_db -P
```

Config replication
```
# main configuration
vim /var/lib/pgsql/data/sync_replica_db/postgresql.conf
```

Set this
```
# change listen and port
listen_addresses = '*' 
port = 5441

# config standby
hot_standby = on
```

Start database
```
# start
pg_ctl -D /var/lib/pgsql/data/sync_replica_db/ start -l /var/log/postgresql/sync_replica_db.log
```

#### Verify replication
Expanded display
```
# expanded
\x
```

Show status replication on primary
```
# Show connected replicas and their status on the primary
select * from pg_stat_replication;
```

Show status replication on replica
```
# Shows the WAL receiver process status on Replica
select * from pg_stat_wal_receiver;
```

### Logical Replication
#### Setup replication on primary node
Config replication
```
# main configuration
vim /var/lib/pgsql/data/logical_primary_db/postgresql.conf
```

Set this
```
# change listen and port
listen_addresses = '*'
port = 5601

# config wal replica
wal_level = logical
```

Create user for replication
```
# create user
create user dba with login password 'password' superuser;
```

Config access control user replication
```
# access control configuration
vim /var/lib/pgsql/data/logical_primary_db/pg_hba.conf
```

Set this
```
# add rule
host    all             dba             10.10.10.0/24            md5
```

Restart service
```
# restart
pg_ctl -D /var/lib/pgsql/data/logical_primary_db/ restart -l /var/log/postgresql/logical_primary_db.log
```

Set publication
```
# create publication
CREATE PUBLICATION testpub FOR TABLE tb_data_1;
```

#### Setup on replica node
Config replication
```
# main configuration
vim /var/lib/pgsql/data/logical_replica_db/postgresql.conf
```

Set this
```
# change listen and port
listen_addresses = '*'
port = 5601

# config wal replica
wal_level = logical
```

Start database
```
# start
pg_ctl -D /var/lib/pgsql/data/logical_replica_db/ start -l /var/log/postgresql/logical_replica_db.log
```

Create subscription to connect publication
```
# create subscription
CREATE SUBSCRIPTION testsub CONNECTION ‘host=localhost port=5601 user=dba dbname=postgres’ PUBLICATION testpub;
```

#### Verify replication
Expanded display
```
# expanded
\x
```

Show status replication on primary
```
# Show connected replicas and their status on the primary
select * from pg_stat_replication;
```

Shows the status of subscription when using logical replication
```
# Shows the status of subscription when using logical replication
pg_stat_subscription
```

Show status replication on replica
```
# Shows the WAL receiver process status on Replica
select * from pg_stat_wal_receiver;
```

### Load Balancing
#### Configuring pgpool-2 for load balancing
Install pgpool-2 with APT Repository
```
# Create the file repository configuration
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists
sudo apt-get update

# Install the latest version of Pgpool-II. Specify the specific version of PostgreSQL which you are using, e.g. postgresql-16-pgpool2
sudo apt-get -y install pgpool2 libpgpool2 postgresql-16-pgpool2
```

Create user to monitor health check
```
CREATE ROLE pgpool WITH LOGIN password 'password';
GRANT pg_monitor TO pgpool;
```

Config access control user replication
```
vim pg_hba.conf
```

Set this
```
host    all             pgpool             10.10.10.0/24            md5
```

Restart service
```
pg_ctl -D /var/lib/pgsql/data/primary_db/ restart -l /var/log/postgresql/primary_db.log
```

Config pgpool.conf
```
vim /etc/pgpool2/pgpool.conf
```

Set this
```
backend_clustering_mode = 'streaming_replication'
listen_addresses = '*'
port = 5400

backend_hostname0 = '10.10.10.82'
backend_port0 = 5441
backend_weight0 = 0
backend_data_directory0 = '/var/lib/pgsql/data/primary_db/'
backend_application_name0 = 'server0'

backend_hostname1 = '10.10.10.194'
backend_port1 = 5441
backend_weight1 = 1
backend_data_directory1 = '/var/lib/pgsql/data/replica_db/'
backend_application_name1 = 'server1'

log_statement = on
log_per_node_statement = on

sr_check_user = 'replica'
sr_check_password = 'password'

health_check_period = 10
health_check_user = 'pgpool'
health_check_password = 'password'
```

Start pgpool
```
systemctl start pgpool2.service
```

Check log
```
journalctl -f -u pgpool2.service
```

Verify
```
psql -U pgpool --port 5400 -c "show pool_nodes;" postgres
psql -U pgpool --port 5400 -c "show pool_processes;" postgres
psql -U pgpool --port 5400 -c "show pool_health_check_stats;" postgres
```

```
insert into tb_data_1 values(generate_series(21,30),'data'||generate_series(21,30));
update tb_data_1 set name = 'test2' where id = 11;
delete from tb_data_1 where id < 20;
select * from tb_data_1
```

### High Availability

## Running Postgresql on kubernetes

## Setup Monitoring for PostgreSQL
### Postgresql Exporter
Download postgresql exporter package
Extract postgresql exporter package

