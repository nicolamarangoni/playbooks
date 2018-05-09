# How to use these playbooks
## Prepare host
CentOS
```
ansible-playbook ~/playbooks/tools/centos/prepare-host.yml --extra-vars "host=client"
ansible-playbook ~/playbooks/tools/centos/prepare-host.yml --extra-vars "host=maxscale"
ansible-playbook ~/playbooks/tools/centos/prepare-host.yml --extra-vars "host=mariadb"
```
Ubuntu
```
ansible-playbook ~/playbooks/tools/ubuntu/prepare-host.yml --extra-vars "host=client"
ansible-playbook ~/playbooks/tools/ubuntu/prepare-host.yml --extra-vars "host=maxscale"
ansible-playbook ~/playbooks/tools/ubuntu/prepare-host.yml --extra-vars "host=mariadb"
```

## Upgrade packages
CentOS
```
ansible-playbook ~/playbooks/tools/centos/upgrade.yml --extra-vars "host=client"
ansible-playbook ~/playbooks/tools/centos/upgrade.yml --extra-vars "host=maxscale"
ansible-playbook ~/playbooks/tools/centos/upgrade.yml --extra-vars "host=mariadb"
```
Ubuntu
```
ansible-playbook ~/playbooks/tools/ubuntu/upgrade.yml --extra-vars "host=client"
ansible-playbook ~/playbooks/tools/ubuntu/upgrade.yml --extra-vars "host=maxscale"
ansible-playbook ~/playbooks/tools/ubuntu/upgrade.yml --extra-vars "host=mariadb"
```
