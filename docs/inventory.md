## Ansible Inventory

[Home](../README.md)

---

### Statisches Inventory
Wenn man ohne Inventory Datei arbeitet, bekommt man bei den Ad-hoc Kommandos eine Warnmeldung angezeigt
```bash
ansible all -m ping
# [WARNING]: Unable to parse /home/ansible/ansible/inventories/dev/inventory as an inventory source
# [WARNING]: No inventory was parsed, only implicit localhost is available
# [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
```
Wie bereits in der Grundkonfiguration beschrieben und auch schon eingerichtet, wurde eine Inventory Datei angelegt.

##### Einfache Inventory Datei
```bash
vi inventories/dev/inventory

[test_hosts]
debian
centos
suse ansible_user=root
ubuntu

[test_hosts:vars]
ansible_python_interpreter=/usr/bin/python3 
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

```bash
ansible all -m ping

# debian | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
# ...
```

##### Aliases in der Inventory Datei definieren 
```bash
s1 ansible_host=192.168.xxx.60
s2 ansible_host=192.168.xxx.70
s3 ansible_host=192.168.xxx.80 ansible_user=root
s4 ansible_host=192.168.xxx.90

# Bei diesen Beispiel sollte zwingend die Namensauflösung im Netzwerk funktionieren
# ggf. die /etc/hosts Datei anpassen.
s1 ansible_host=debian.domain.local
s2 ansible_host=centos.domain.local
s3 ansible_host=suse.domain.local ansible_user=root
s4 ansible_host=ubuntu.domain.local

## mit Portangabe
##
s1 ansible_host=debian.domain.local ansible_port=2201
```

##### Beispiel Ad-hoc Kommando um die /etc/shadow Datei abzufragen
```bash
ansible all -a "head -1 /etc/shadow"

# debian | FAILED | rc=1 >>
# head: cannot open '/etc/shadow' for reading: Permission deniednon-zero return code
# centos | FAILED | rc=1 >>
# head: cannot open '/etc/shadow' for reading: Permission deniednon-zero return code
# ubuntu | FAILED | rc=1 >>
# head: cannot open '/etc/shadow' for reading: Permission deniednon-zero return code
# suse | CHANGED | rc=0 >>
# bin:!:18430::::::
```

##### Anpassung an der Inventory Datei
```bash
[test_hosts]
debian ansible_host=192.168.xxx.60
centos ansible_host=192.168.xxx.70
suse ansible_host=192.168.xxx.80
ubuntu ansible_host=192.168.xxx.90

[debian_hosts]
debian ansible_host=192.168.xxx.60

[centos_hosts]
centos ansible_host=192.168.xxx.70

[suse_hosts]
suse ansible_host=192.168.xxx.80

[ubuntu_hosts]
ubuntu ansible_host=192.168.xxx.90

[test_hosts:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

[debian_hosts:vars]
ansible_user=ansible
ansible_become=yes
ansible_become_pass=ansible

[centos_hosts:vars]
ansible_user=ansible
ansible_become=yes
ansible_become_method=su
ansible_become_pass=ansible

[suse_hosts:vars]
ansible_user=root

[ubuntu_hosts:vars]
ansible_user=ansible
ansible_become=yes
```

```bash
ansible all -a "head -1 /etc/shadow"

# debian | CHANGED | rc=0 >>
# root:!:18559:0:99999:7:::
# centos | CHANGED | rc=0 >>
# root:$6$aCj9Ai2QdQ$ph7vus6TQTCmLfkTzcqWIC.1c21eOKvlsj.27Y4EvIdWxWbWtnU26GePLtUVkAbXuZambQQmP/gpGC.k0veLd0:18635:0:99999:7:::
# suse | CHANGED | rc=0 >>
# bin:!:18430::::::
# ubuntu | CHANGED | rc=0 >>
# root:!:18559:0:99999:7:::
```

##### Folgende Methoden werden auf den jeweiligen Betriebssystemen angewandt
```bash
======================================
| Host   | Methode                   |
======================================
| debian | sudo                      |
--------------------------------------
| centos | su                        |
--------------------------------------
| suse   | direktes Root-SSH möglich |
--------------------------------------
| ubuntu | sudo (NOPASSWD)           |
--------------------------------------

# ansible_become = Der Hinweis das auf dem Zielsystem eine Rechteerhöhung stattfinden muss. Default ist "no"
# ansible_become_method = Die dazu zu verwendende Methode. Default ist "sudo"
# ansible_become_user = Wenn ein Benutzerwechsel stattfinden muss. Default ist "root"
# ansible_become_pass = Das zur Rechteerhöhung benötigte Passwort.
```