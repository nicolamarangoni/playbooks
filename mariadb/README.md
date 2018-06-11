# Installation and configuration
## Install MariaDB client
Install Client on a group of hosts
```
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-client.yml --extra-vars "host=clients"
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-client.yml --extra-vars "host=maxscale"
```
## Install MariaDB server
Install Server on a group of hosts for a galera cluster
```
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-server.yml --extra-vars "host=galera-cluster"
```
Create default users on the first host:
```
ansible-playbook ~/playbooks/mariadb/setup-mariadb-default-users-server.yml --extra-vars "host=mariadb00 password=mariadb create_cnf=yes"
```
## Configure a cluster
Configure galera:
```
ansible-playbook ~/playbooks/mariadb/configure-galera-cluster.yml \
  --extra-vars "host=galera-cluster cluster_name=galera00 domain_id=0 cluster_address=mariadb00,mariadb01,mariadb02 password=mariadb"
```
Start galera:
```
ansible-playbook ~/playbooks/mariadb/start-galera-cluster.yml --extra-vars "host_boot=galera-boot host_join=galera-join"
```
Stop galera:
```
ansible-playbook ~/playbooks/mariadb/stop-galera-cluster.yml --extra-vars "host_boot=galera-boot host_join=galera-join"
```
## Configure a read-only slave for the galera cluster
Install Server on the slave hosts:
```
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-server.yml --extra-vars "host=mariadb-slaves"
```
Create default users on the slave hosts:
```
ansible-playbook ~/playbooks/mariadb/setup-mariadb-default-users-server.yml --extra-vars "host=mariadb-slaves password=mariadb create_cnf=yes"
```
Galera nodes (masters):
```
ansible-playbook ~/playbooks/mariadb/configure-replication-master.yml --extra-vars "host=mariadb00 server_id=0 binlog_name=galera00"
ansible-playbook ~/playbooks/mariadb/configure-replication-master.yml --extra-vars "host=mariadb01 server_id=1 binlog_name=galera00"
ansible-playbook ~/playbooks/mariadb/configure-replication-master.yml --extra-vars "host=mariadb02 server_id=2 binlog_name=galera00"
```
Stop galera:
```
ansible-playbook ~/playbooks/mariadb/stop-galera-cluster.yml --extra-vars "host_boot=galera-boot host_join=galera-join"
```
Start galera:
```
ansible-playbook ~/playbooks/mariadb/start-galera-cluster.yml --extra-vars "host_boot=galera-boot host_join=galera-join"
```
Slave:
```
ansible-playbook ~/playbooks/mariadb/configure-replication-slave.yml --extra-vars "host=mariadb03 server_id=3 binlog_name=galera00 domain_id=0"
```
## Start replication on the slave
Restart slave (mariadb03)
```
ansible mariadb03 -s -m shell -a 'systemctl restart mariadb'
```
or on the slave (mariadb03):
```
systemctl restart mariadb
```
On the master (login to the database):
```
mysql -h mariadb01 -u root -p
```
In the DB:
```
SHOW MASTER STATUS;
```
Take the values in the field *Files* and *Position*

On the slave (login to the database):
```
mysql -h mariadb03 -u root -p
```
In the DB:
```
CHANGE MASTER TO
  MASTER_USE_GTID = current_pos,
  MASTER_HOST='mariadb01',
  MASTER_USER='replication',
  MASTER_PASSWORD='mariadb',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='galera00.000001', -- File read on the master
  MASTER_LOG_POS=000, -- Position read on the master
  MASTER_CONNECT_RETRY=10;

START SLAVE;
```
## Install and configure maxscale
Install MaxScale:
```
ansible-playbook ~/playbooks/mariadb/centos/install-maxscale.yml --extra-vars "host=maxscale version=2.2.9"
```
Create user:
```
ansible-playbook ~/playbooks/mariadb/create-maxscale-user.yml --extra-vars "host=mariadb00 user=maxscale password=maxscale"
```
Configure servers:
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-server.yml \
  --extra-vars "host=maxscale server_host=mariadb00 server_address=mariadb00 server_port=3306"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-server.yml \
  --extra-vars "host=maxscale server_host=mariadb01 server_address=mariadb01 server_port=3306"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-server.yml \
  --extra-vars "host=maxscale server_host=mariadb02 server_address=mariadb02 server_port=3306"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-server.yml \
  --extra-vars "host=maxscale server_host=mariadb03 server_address=mariadb03 server_port=3306"
```
Configure MaxScale CLI service:
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service-cli.yml \
  --extra-vars "host=maxscale"
```
Configure galera monitor:
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-galeramon.yml \
  --extra-vars "host=maxscale user=maxscale password=maxscale service=service-galeramon servers=mariadb00,mariadb01,mariadb02"
```
Configure router services
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service.yml \
  --extra-vars "host=maxscale user=maxscale password=maxscale service=service-rwsplit router=readwritesplit servers=mariadb00,mariadb01,mariadb02,mariadb03"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service.yml \
  --extra-vars "host=maxscale user=maxscale password=maxscale service=service-readonly router=readconnroute servers=mariadb03"
```
Configure avro service:
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service.yml \
  --extra-vars "host=maxscale user=maxscale password=maxscale service=service-replication router=binlogrouter server_id=4000 master_id=3000 router_options=binlogdir=/var/lib/maxscale/binlog,mariadb10-compatibility=1"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service.yml \
  --extra-vars "host=maxscale user=maxscale password=maxscale service=service-avro router=avrorouter source=service-replication router_options=avrodir=/var/lib/maxscale/avro,filestem=galera00"
```
Configure listeners:
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale protocol=maxscaled service=service-maxadmin listener_name=listener-maxadmin socket=default"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale protocol=MariaDBClient service=service-rwsplit listener_name=listener-rwsplit listener_port=4006"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale protocol=MariaDBClient service=service-readonly listener_name=listener-readonly listener_port=4008"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale protocol=MariaDBClient service=service-replication listener_name=listener-replication listener_port=3306"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale protocol=CDC service=service-avro listener_name=listener-avro listener_port=4001"
```
On maxscale hosts:
```
systemctl restart maxscale
```
## Start MaxScale Replication
Connect to the master for the maxscale replication service:
```
mysql -h mariadb03 -u root -p
```
In the DB:
```
SHOW MASTER STATUS;
```
Take the values in the field *Files* and *Position*

Connect to the maxscale replication service:
```
mysql -h maxscale00 -u maxscale -p
```
Configure and start slave:
```
CHANGE MASTER TO
  MASTER_HOST='mariadb03',
  MASTER_PORT=3306,
  MASTER_USER='replication',
  MASTER_PASSWORD='mariadb',
  MASTER_LOG_FILE='galera00.000003',
  MASTER_LOG_POS=1497;

START SLAVE;
```
## Install ColumnStore
Install dependencies on all nodes:
```
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-cs-dependencies.yml --extra-vars "host=mariadb-ax"
```
Install on the first PM:
```
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-cs.yml --extra-vars "host=mariadb12 version=1.1.4 maxscale_version=2.2.9"
```
Run initial configuration:
```
/usr/local/mariadb/columnstore/bin/postConfigure
```
Leave defaults for the single node installation
### Install Alias and create setup users
Install mysql alias for mcsmysql on the UMs:
```
ansible-playbook ~/playbooks/mariadb/configure-cs-alias.yml --extra-vars "host=ax-um"
```
Setup root with password and demo user:
```
ansible-playbook ~/playbooks/mariadb/setup-mariadb-default-users-cs.yml --extra-vars "host=mariadb10 password=mariadb create_cnf=yes"
```
### Install ColumnStore Adapters
Install on the UMs:
```
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-cs-adapters.yml --extra-vars "host=ax-um version=1.1.4 maxscale_version=2.2.9"
```
### Common commands
After rebooting, use the following commands to start/restart/stop the ColumnStore:
```
mcsadmin startSystem
mcsadmin restartSystem
mcsadmin shutdownSystem
```
Monitor status:
```
mcsadmin getSystemNetworkConfig
mcsadmin getModuleConfig
mcsadmin getProcessStatus
```
# Demos
## Install demo databases
MariaDB Server demo
```
ansible-playbook ~/playbooks/mariadb/create-mariadb-demo-dbs-server.yml --extra-vars "host=mariadb00"
```
MariaDB CS demo
```
ansible-playbook ~/playbooks/mariadb/create-mariadb-demo-dbs-cs.yml --extra-vars "host=mariadb10"
```
## CDC Demo
### Setup a CDC streaming demo in MaxScale
Create cdc user:
```
maxadmin call command cdc add_user service-avro cdcuser cdcpasswd
```
Create a table and a procedure that fills it with random text in the demo schema to test the avro service:
```
mysql -h mariadb02 -u demo -p <<EOF
DROP TABLE IF EXISTS demo.tbl_demo_cdc;

CREATE TABLE IF NOT EXISTS demo.tbl_demo_cdc (
  col0 bigint PRIMARY KEY,
  col1 text
);

DROP PROCEDURE IF EXISTS demo.prc_demo_insert_cdc;

DELIMITER //

CREATE PROCEDURE IF NOT EXISTS demo.prc_demo_insert_cdc()   
BEGIN
	DECLARE i INT DEFAULT 0;
	WHILE (i <= 9999) DO
		  DO SLEEP(2);

      INSERT INTO tbl_demo_cdc (
	        col0,
	        col1
	    ) VALUES (
	        CURRENT_TIMESTAMP(),
	        SUBSTRING(MD5(RAND()) FROM 1 FOR RAND()*20 +10)
	    );

	    SET i = i+1;
	END WHILE;
END;
//
EOF
```
Execute the procedure to generate random texts in the demo table:
```
mysql -h mariadb02 -u demo -p <<EOF
CALL demo.prc_demo_insert_cdc();'
EOF
```
Check cdc content:
```
cdc.py -u cdcuser -p cdcpasswd -h maxscale00 -P 4001 demo.tbl_demo_cdc
```
### CDC Demo on ColumnStore
Start CDC ingestion on one UM of choice
```
mxs_adapter -u cdcuser -p cdcpasswd -h maxscale00 -P 4001 demo tbl_demo
```
### Purge replication files
On the maxscale host:
```
systemctl stop maxscale
rm -rf /var/lib/maxscale/avro*
rm -rf /var/lib/maxscale/data*
rm -rf /var/lib/maxscale/binlog*
systemctl start maxscale
```
