# How to use these playbooks
## Install MariaDB server
Install Server on a group of hosts for a galera cluster
```
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-server.yml --extra-vars "host=galera-cluster"
```
Create default users on the first host:
```
ansible-playbook ~/playbooks/mariadb/create-mariadb-default-users.yml --extra-vars "host=mariadb00 password=mariadb"
```
## Configure a cluster
Configure galera:
```
ansible-playbook ~/playbooks/mariadb/configure-galera-cluster.yml \
  --extra-vars "host=galera-cluster cluster_name=galera00 domain_id=0 cluster_address=192.168.56.100,192.168.56.101,192.168.56.102 password=mariadb"
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
Install Server on the slave host
```
ansible-playbook ~/playbooks/mariadb/centos/install-mariadb-server.yml --extra-vars "host=mariadb03"
```
Create default users on the first host:
```
ansible-playbook ~/playbooks/mariadb/create-mariadb-default-users.yml --extra-vars "host=mariadb03 password=mariadb"
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
## Configure maxscale
Install MaxScale:
```
ansible-playbook ~/playbooks/mariadb/centos/install-maxscale.yml --extra-vars "host=maxscale version=2.2.5"
```
Create user:
```
ansible-playbook ~/playbooks/mariadb/create-maxscale-user.yml --extra-vars "host=mariadb00 password=maxscale"
ansible-playbook ~/playbooks/mariadb/create-maxscale-user.yml --extra-vars "host=mariadb03 password=maxscale"
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
  --extra-vars "host=maxscale service=service-galeramon password=maxscale servers=mariadb00,mariadb01,mariadb02"
```
Configure router services
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service.yml \
  --extra-vars "host=maxscale service=service-rwsplit router=readwritesplit password=maxscale servers=mariadb00,mariadb01,mariadb02,mariadb03"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service.yml \
  --extra-vars "host=maxscale service=service-readonly router=readconnroute password=maxscale servers=mariadb03"
```
Configure avro service:
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service.yml \
  --extra-vars "host=maxscale service=service-replication router=binlogrouter password=maxscale server_id=4000 server_id=3000 binlogdir=/var/lib/maxscale"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-service.yml \
  --extra-vars "host=maxscale service=service-avro router=avrorouter password=maxscale source=service-replication"
```
Configure listeners:
```
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener-cli.yml \
  --extra-vars "host=maxscale"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale service=service-rwsplit listener_name=listener-rwsplit listener_port=4006"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale service=service-readonly listener_name=listener-readonly listener_port=4008"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale service=service-replication listener_name=listener-replication listener_port=3306"
ansible-playbook ~/playbooks/mariadb/configure-maxscale-listener.yml \
  --extra-vars "host=maxscale service=service-avro listener_name=listener-avro listener_port=4001"
```
