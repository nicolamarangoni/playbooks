# How to use these playbooks
## Prepare host
CentOS
```
ansible-playbook ~/playbooks/tools/centos/prepare-host.yml --extra-vars "host=all"
```
Ubuntu
```
ansible-playbook ~/playbooks/tools/ubuntu/prepare-host.yml --extra-vars "host=all"
```

## Upgrade packages
CentOS
```
ansible-playbook ~/playbooks/tools/centos/upgrade.yml --extra-vars "host=all"
```
Ubuntu
```
ansible-playbook ~/playbooks/tools/ubuntu/upgrade.yml --extra-vars "host=all"
```
## Install SQLPad
```
ansible-playbook ~/playbooks/tools/install-sqlpad.yml --extra-vars "host=clients port=3010"
```
## Install Zeppelin
```
ansible-playbook ~/playbooks/tools/install-zeppelin.yml --extra-vars "host=clients version=0.7.3"
```
