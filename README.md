## Ansible Examples

### Testumgebung in Windows 10 vorbereiten
* Arbeite mit Windows 10 und Cygwin.
* Vagrant wurde unter Windows 10 in das Verzeichnis C:\DevEnv\Vagrant installiert.
* VirtualBox ist ebenfalls unter Windows installiert.
* In Cygwin wurde Ansible installiert und die Pfade dementsprechend in der .bashrc angepasst.

##### Cygwin anpassen
```bash
## Lokale cygwin Umgebung anpassen, damit die vagrant.exe unter Windows gefunden wird
vi ~/.bashrc
PATH=$HOME/bin:/cygdrive/c/ChefEnv/Vagrant/bin:$PATH
```

##### Ansible in Cygwin vorbereiten
```bash
## Für ansible wurde die passende Umgebung in Cygwin angelegt
mkdir ~/ansible-example && cd $_
```

##### Linux Maschinen für Vagrant konfigurieren
```bash
## Wenn man unter Vagrant komplett neu anfängt muss eine Vagrantfile angelegt werden
vagrant init
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

  config.vm.define "ansible" do |ansible|
    ansible.vm.box = "bento/debian-10"
    ansible.vm.hostname = "ansible"
    ansible.vm.network :private_network, ip: "192.168.xxx.50"
    ansible.vm.provider "virtualbox" do |p|
      p.customize ["modifyvm", :id, "--memory", "1024"]
      p.customize ["modifyvm", :id, "--cpus", 1]
      p.customize ["modifyvm", :id, "--ioapic", "on"]
      p.customize ["modifyvm", :id, "--nictype1", "virtio" ]
      p.customize ["modifyvm", :id, "--nictype2", "virtio" ]
      p.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      p.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      p.auto_nat_dns_proxy = false
      p.name = "ansible"
    end
    ansible.vm.boot_timeout = 300
    ansible.vm.synced_folder ".", "/vagrant", disabled: true
    ansible.vm.provision :shell, inline: $script, :args => "sudo-nopasswd"
  end

  config.vm.define "debian" do |debian|
    debian.vm.box = "bento/debian-10"
    debian.vm.hostname = "debian"
    debian.vm.network :private_network, ip: "192.168.xxx.60"
    debian.vm.provider "virtualbox" do |p|
      p.customize ["modifyvm", :id, "--memory", "1024"]
      p.customize ["modifyvm", :id, "--cpus", 1]
      p.customize ["modifyvm", :id, "--ioapic", "on"]
      p.customize ["modifyvm", :id, "--nictype1", "virtio" ]
      p.customize ["modifyvm", :id, "--nictype2", "virtio" ]
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
    centos.vm.network :private_network, ip: "192.168.xxx.70"
    centos.vm.provider "virtualbox" do |p|
      p.customize ["modifyvm", :id, "--memory", "1024"]
      p.customize ["modifyvm", :id, "--cpus", 1]
      p.customize ["modifyvm", :id, "--ioapic", "on"]
      p.customize ["modifyvm", :id, "--nictype1", "virtio" ]
      p.customize ["modifyvm", :id, "--nictype2", "virtio" ]
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
    suse.vm.network :private_network, ip: "192.168.xxx.80"
    suse.vm.provider "virtualbox" do |p|
      p.customize ["modifyvm", :id, "--memory", "1024"]
      p.customize ["modifyvm", :id, "--cpus", 1]
      p.customize ["modifyvm", :id, "--ioapic", "on"]
      p.customize ["modifyvm", :id, "--nictype1", "virtio" ]
      p.customize ["modifyvm", :id, "--nictype2", "virtio" ]
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
    ubuntu.vm.network :private_network, ip: "192.168.xxx.90"
    ubuntu.vm.provider "virtualbox" do |p|
      p.customize ["modifyvm", :id, "--memory", "1024"]
      p.customize ["modifyvm", :id, "--cpus", 1]
      p.customize ["modifyvm", :id, "--ioapic", "on"]
      p.customize ["modifyvm", :id, "--nictype1", "virtio" ]
      p.customize ["modifyvm", :id, "--nictype2", "virtio" ]
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
vagrant up
```

##### Auf die Linux Maschinen per ssh aufschalten
```bash
## SSH Verbindung zu den vagrant Maschinen aufbauen
vagrant ssh (ansible/debian/centos/suse/ubuntu)
```

##### Folgende Packete müssen in den einzelnen Maschinen installiert werden
```bash
## Ansible auf Debian installieren
##
echo "deb http://deb.debian.org/debian buster-backports main contrib non-free" > /etc/apt/sources.list.d/backports.list
apt update
apt -t buster-backports install ansible -y
ansible --version
apt update && apt upgrade -y

## Ansible auf CentOS installieren
##
yum install epel-release -y
yum install ansible python3 -y

oder

yum install centos-release-ansible-29 -y
yum install ansible -y
ansible --version
yum update -y

## Ansible auf OpenSUSE 15 installieren
##
zypper install ansible -y
ansible --version
zypper update -y

## Ansible auf Ubuntu installieren
##
apt update
apt install ansible -y
ansible --version
apt update && apt upgrade -y
```

### Unser Ansible Control Host
```bash
vagrant ssh ansible
vagrant@ansible:~$ su - ansible (Password)
```

##### Ordnerstruktur für den Ansible Control Host anlegen
```bash
mkdir ~/ansible && cd $_
mkdir -p inventories/dev/{host,group}_vars fact_cache logs playbooks/{templates,vars} retry_files roles ssh_keys

# ansible/
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
```

##### SSH Key anlegen und auf allen Maschinen verteilen
```bash
## Dieser SSH Key wird nur von Ansible ohne Passphrase benötigt.
ssh-keygen -t ed25519 -f ~/ansible/ssh_keys/ansible -C'ansible@ansible'

chmod 700 ~/ansible/ssh_keys
chmod 600 ~/ansible/ssh_keys/ansible
chmod 644 ~/ansible/ssh_keys/ansible.pub

## SSH Key verteilen
ssh-copy-id -i ~/ansible/ssh_keys/ansible.pub ansible@debian -f
ssh-copy-id -i ~/ansible/ssh_keys/ansible.pub ansible@centos -f
ssh-copy-id -i ~/ansible/ssh_keys/ansible.pub ansible@ubuntu -f
ssh-copy-id -i ~/ansible/ssh_keys/ansible.pub root@suse -f
```

##### Ansible Konfigurationsdatei anlegen und in der Umgebung als Default definieren
```bash
vi ~/ansible/ansible.cfg

## Komplette ansible.cfg Datei
##
# [defaults]
# ## default number of parallel processes
# forks = 10

# ## default location of the inventory file
# inventory = ~/ansible/inventories/dev/inventory

# ## Ansible Roles Path
# roles_path = ~/ansible/roles

# ## Ansible Default LogPath
# log_path = ~/ansible/logs/ansible.log

# ## Ansible Default Path for the SSH Keys
# private_key_file = ~/ansible/ssh_keys/ansible

# ## Ansible Retry File Log Path
# retry_files_enabled = yes
# retry_files_save_path = ~/ansible/retry_files

# ## Facts Caching
# gathering = smart
# fact_caching = jsonfile
# fact_caching_connection = ~/ansible/fact_cache
# fact_caching_timeout = 86400

ansible --version
# ansible 2.9.6
#   config file = /home/ansible/ansible/ansible.cfg
#   ...
```

##### SSH Agent/SSH Key beim Aufruf einer Console automatisch starten/einlesen
```bash
vi ~/.bashrc

## load ssh key to the environment
if [ -z "$SSH_AUTH_SOCK" ] ; then
    eval `ssh-agent`
    ssh-add
    ssh-add ~/ansible/ssh_keys/ansible
fi

## defining ansible config file
export ANSIBLE_CONFIG=~/ansible/ansible.cfg

## working directory
if [ -d ~/ansible ] ; then
   cd ~/ansible
else
   cd ~
fi
# ...
```
