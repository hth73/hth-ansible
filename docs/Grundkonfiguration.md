## Ansible/Vagrant Grundkonfiguration

[[Home | README]]

---

### Testumgebung in Windows 10 vorbereiten
* Arbeite mit Windows 10, Cygwin (Vagrant) und einem Raspberry Pi als Ansible Contol Host.
* Vagrant wurde unter Windows 10 in das Verzeichnis C:\DevEnv\Vagrant installiert.
* VirtualBox ist ebenfalls unter Windows installiert.
* In Cygwin wurden die Pfade angepasst, damit über Vagrant Maschinen installiert werden können.

##### Cygwin anpassen für Vagrant
```bash
## Lokale cygwin Umgebung anpassen, damit die vagrant.exe unter Windows gefunden wird
vi ~/.bashrc
PATH=$HOME/bin:/cygdrive/c/DevEnv/Vagrant/bin:$PATH
```

##### Linux Maschinen für Vagrant konfigurieren 
```bash
## Wenn man unter Vagrant komplett neu anfängt muss eine Vagrantfile angelegt werden
mkdir ~/vagrant && cd $_
vagrant init

vi ~/vagrant/Vagrantfile
```

##### Folgenden Inhalt in das Vagrantfile kopieren
```bash
$script = <<-'SCRIPT'

# direct:        Direktes root-Login moeglich
# sudo:          root via sudo
# sudo-nopasswd: root via sudo ohne Passworteingabe
# su:            root via su

ANSIBLE_USERPASS=ansible:ansible
ROOT_USERPASS=root:ansible

useradd -m -s /bin/bash ansible
echo $ANSIBLE_USERPASS | chpasswd

if [ "$1" = "sudo" ]; then
   echo 'ansible ALL=(ALL) ALL' >/etc/sudoers.d/ansible
   sed -ri 's/^(PermitRootLogin).*/\1 no/' /etc/ssh/sshd_config
elif [ "$1" = "sudo-nopasswd" ]; then
   echo 'ansible ALL=(ALL) NOPASSWD: ALL' >/etc/sudoers.d/ansible
   sed -ri 's/^(PermitRootLogin).*/\1 no/' /etc/ssh/sshd_config
elif [ "$1" = "su" ]; then
   echo $ROOT_USERPASS | chpasswd
   sed -ri 's/^(PermitRootLogin).*/\1 no/' /etc/ssh/sshd_config
elif [ "$1" = "direct" ]; then
   echo $ROOT_USERPASS | chpasswd
   sed -ri 's/^(PermitRootLogin).*/\1 yes/' /etc/ssh/sshd_config
fi

systemctl restart sshd.service
SCRIPT

Vagrant.configure("2") do |config|

  config.vm.define "debian" do |debian|
      debian.vm.box = "bento/debian-10"
      debian.vm.hostname = "debian"
      debian.vm.network "public_network", ip: "192.168.xxx.60"
      debian.vm.provider "virtualbox" do |p|
        p.customize ["modifyvm", :id, "--memory", "1024"]
        p.customize ["modifyvm", :id, "--cpus", 1]
        p.customize ["modifyvm", :id, "--ioapic", "on"]
        p.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        p.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
        p.auto_nat_dns_proxy = false
        p.name = "debian"
      end
      debian.vm.boot_timeout = 300
      debian.vm.synced_folder ".", "/vagrant", disabled: true
      debian.vm.provision :shell, inline: $script, :args => "sudo"
    end

  config.vm.define "centos" do |centos|
      centos.vm.box = "bento/centos-8"
      centos.vm.hostname = "centos"
      centos.vm.network "public_network", ip: "192.168.xxx.70"
      centos.vm.provider "virtualbox" do |p|
        p.customize ["modifyvm", :id, "--memory", "1024"]
        p.customize ["modifyvm", :id, "--cpus", 1]
        p.customize ["modifyvm", :id, "--ioapic", "on"]
        p.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        p.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
        p.auto_nat_dns_proxy = false
        p.name = "centos"
      end
      centos.vm.boot_timeout = 300
      centos.vm.synced_folder ".", "/vagrant", disabled: true
      centos.vm.provision :shell, inline: $script, :args => "su"
    end

  config.vm.define "suse" do |suse|
      suse.vm.box = "bento/opensuse-leap-15.1"
      suse.vm.hostname = "suse"
      suse.vm.network "public_network", ip: "192.168.xxx.80"
      suse.vm.provider "virtualbox" do |p|
        p.customize ["modifyvm", :id, "--memory", "1024"]
        p.customize ["modifyvm", :id, "--cpus", 1]
        p.customize ["modifyvm", :id, "--ioapic", "on"]
        p.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        p.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
        p.auto_nat_dns_proxy = false
        p.name = "suse"
      end
      suse.vm.boot_timeout = 300
      suse.vm.synced_folder ".", "/vagrant", disabled: true
      suse.vm.provision :shell, inline: $script, :args => "direct"
    end

  config.vm.define "ubuntu" do |ubuntu|
      ubuntu.vm.box = "bento/ubuntu-20.04"
      ubuntu.vm.hostname = "ubuntu"
      ubuntu.vm.network "public_network", ip: "192.168.xxx.90"

      ubuntu.vm.provider "virtualbox" do |p|
        p.customize ["modifyvm", :id, "--memory", "1024"]
        p.customize ["modifyvm", :id, "--cpus", 1]
        p.customize ["modifyvm", :id, "--ioapic", "on"]
        p.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        p.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
        p.auto_nat_dns_proxy = false
        p.name = "ubuntu"
      end
      ubuntu.vm.boot_timeout = 300
      ubuntu.vm.synced_folder ".", "/vagrant", disabled: true
      ubuntu.vm.provision :shell, inline: $script, :args => "sudo-nopasswd"
    end
end
```
##### Linux Maschinen über Vagrant installieren lassen
```bash
## Dauer beim ersten mal Ausführen ca. ~ 10 Minuten
cd ~/vagrant
vagrant up
```

##### Auf die Linux Maschinen per ssh aufschalten
```bash
## SSH Verbindung zu den vagrant Maschinen aufbauen
cd ~/vagrant
vagrant ssh (debian/centos/suse/ubuntu)
```

##### Folgende Packete müssen in den einzelnen Maschinen installiert werden
```bash
## Ansible auf Debian installieren
##
sudo -i
echo "deb http://deb.debian.org/debian buster-backports main contrib non-free" > /etc/apt/sources.list.d/backports.list
apt update && apt -t buster-backports install ansible -y
ansible --version

## Ansible auf CentOS installieren
##
su -
yum install epel-release -y && yum install ansible python3 -y
# oder
yum install centos-release-ansible-29 ansible -y
ansible --version

## Ansible auf OpenSUSE 15 installieren
##
su -
zypper install ansible -y
ansible --version

## Ansible auf Ubuntu installieren
##
sudo -i
apt update && apt install ansible -y
ansible --version
```

### Ansible Control Host (Raspberry Pi)
```bash
## User 'ansible' anlegen und in die sudo Gruppe aufnehmen
sudo /usr/sbin/useradd -s /bin/bash -m -d /home/ansible -c "ansible" ansible
sudo passwd ansible
sudo /usr/sbin/usermod -a -G sudo ansible

## Benutzer auf der Console wechseln
su - ansible

## Ordnerstruktur für den Ansible Control Host anlegen
mkdir ~/ansible-examlpe && cd $_
mkdir -p inventories/dev/{host,group}_vars fact_cache logs playbooks/{templates,vars} retry_files roles ssh_keys

# ansible-example/
# ├── fact_cache
# ├── inventories
# │   └── dev
# │       ├── group_vars
# │       └── host_vars
# ├── logs
# ├── playbooks
# │   ├── templates
# │   └── vars
# ├── retry_files
# ├── roles
# └── ssh_keys

## Ansible Inventory anlegen
cd ~/ansible-example
vi inventories/dev/inventory

# Ansible Test Environment
[ansible_hosts]
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

[ansible_hosts:vars]
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

##### SSH Key anlegen und auf allen Maschinen verteilen
```bash
## Dieser SSH Key wird nur von Ansible ohne Passphrase benötigt.
ssh-keygen -t ed25519 -f ~/ansible-example/ssh_keys/ansible -C'ansible@ansible'

chmod 700 ~/ansible-example/ssh_keys
chmod 600 ~/ansible-example/ssh_keys/ansible
chmod 644 ~/ansible-example/ssh_keys/ansible.pub

## SSH Key verteilen
ssh-copy-id -i ~/ansible-example/ssh_keys/ansible.pub ansible@debian -f
ssh-copy-id -i ~/ansible-example/ssh_keys/ansible.pub ansible@centos -f
ssh-copy-id -i ~/ansible-example/ssh_keys/ansible.pub ansible@ubuntu -f
ssh-copy-id -i ~/ansible-example/ssh_keys/ansible.pub root@suse -f
```

##### Ansible Konfigurationsdatei anlegen und in der Umgebung als Default definieren
```bash
vi ~/ansible-example/ansible.cfg

[defaults]
## default number of parallel processes
forks = 10

## default location of the inventory file
inventory = ~/ansible-example/inventories/dev/inventory

## Ansible Roles Path
roles_path = ~/ansible-example/roles

## Ansible Default LogPath
log_path = ~/ansible-example/logs/ansible.log

## Ansible Default Path for the SSH Keys
private_key_file = ~/ansible-example/ssh_keys/ansible

## Ansible Retry File Log Path
retry_files_enabled = yes
retry_files_save_path = ~/ansible-example/retry_files

## Facts Caching
gathering = smart
fact_caching = jsonfile
fact_caching_connection = ~/ansible-example/fact_cache
fact_caching_timeout = 86400

ansible --version
# ansible 2.9.6
#   config file = /home/ansible/ansible-example/ansible.cfg
#   ...
```

##### SSH Agent/SSH Key beim Aufruf einer Console automatisch starten/einlesen
```bash
vi ~/.bashrc

## load ssh key to the environment
if [ -z "$SSH_AUTH_SOCK" ] ; then
    eval `ssh-agent`
    ssh-add
    ssh-add ~/ansible-example/ssh_keys/ansible
fi

## defining ansible config file
export ANSIBLE_CONFIG=~/ansible-example/ansible.cfg

## working directory
if [ -d ~/ansible-exampel ] ; then
   cd ~/ansible-example
else
   cd ~
fi
# ...
```

##### Verbindungen zu den VMs mit Ansible überprüfen
```bash
cd ~/ansible-example

ansible all --key-file ~/ansible-example/ssh_keys/ansible -i inventories/dev/inventory -m ping

ansible all -m ping
ansible all -m gather_facts

ansible all -u ansible -m command -a "cat /home/ansible/.ssh/authorized_keys"
ansible 'all:!suse' -u ansible -m command -a "cat /home/ansible/.ssh/authorized_keys" 
ansible suse -u root -m command -a "cat /root/.ssh/authorized_keys" 

## Updates auf allen Maschinen installieren
ansible all -m apt -a update_cache=true --become --ask-become-pass
ansible all -m apt -a name=vim --become --ask-become-pass

## Da es unterschiedliche Distribution sind, verfügen nicht alle über den gleichen Paket Manager (apt), jedee Distribution muss unterschiedlich behandelt werden.
ansible debian,ubuntu, -m apt -a upgrade=dist --become --ask-become-pass
# oder
ansible 'all:!suse:!centos' -m {{ansible_pkg_mgr}} -a upgrade=dist --become --ask-become-pass

ansible centos -m yum -a "name=* state=latest update_cache=true" --become --ask-become-pass
ansible centos -m {{ansible_pkg_mgr}} -a "name=* state=latest update_cache=true" --become --ask-become-pass
ansible centos -m {{ansible_pkg_mgr}} -a "autoremove=yes" --become --ask-become-pass

ansible suse -u root -m zypper -a "name=vim state=latest update_cache=true" --become --ask-become-pass # Bei Suse muss immer ein Paket mit Namen angegeben werden
ansible suse -u root -m {{ansible_pkg_mgr}} -a "name=vim state=latest update_cache=true" --become --ask-become-pass 
ansible suse -u root -m {{ansible_pkg_mgr}} -a "name=* state=latest update_cache=true" --become --ask-become-pass ## Update in Suse Linux
# ...
```
