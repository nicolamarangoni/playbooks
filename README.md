# Ansible playbooks
## Update playbooks
Local:
```
cd ~/playbooks
git pull
```
On several hosts with ansible:
```
ansible-playbook ~/playbooks/tools/update-playbooks.yml --extra-vars "host=client"
ansible-playbook ~/playbooks/tools/update-playbooks.yml --extra-vars "host=maxscale"
ansible-playbook ~/playbooks/tools/update-playbooks.yml --extra-vars "host=mariadb"
```
