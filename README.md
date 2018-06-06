# Ansible playbooks
Get playbooks:
```
cd ~
git clone https://github.com/nicolamarangoni/playbooks.git
```
## Update playbooks
Local:
```
cd ~/playbooks
git pull
```
On several hosts with ansible:
```
ansible-playbook ~/playbooks/tools/update-playbooks.yml --extra-vars "host=all"
```
## Upgrade system
```
ansible-playbook ~/playbooks/tools/centos/upgrade.yml --extra-vars "host=all"
```
