# playbooks
## Update playbooks
Local:
```
cd ~/playbooks
git pull
```
On several hosts:
```
ansible-playbook ~/playbooks/common/update-playbooks.yml --extra-vars "host=client"
ansible-playbook ~/playbooks/common/update-playbooks.yml --extra-vars "host=maxscale"
ansible-playbook ~/playbooks/common/update-playbooks.yml --extra-vars "host=mariadb"
```
