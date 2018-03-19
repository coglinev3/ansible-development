# Vagrant: Ansible development environment

This is a multi node Vagrant/VirtualBox environment which should represent a real life Ansible scenario with one Ansible management node named *master* and different Linux OS nodes (Ansible clients):
* CentOS 6, 
* CentOS 7, 
* Ubuntu 14.04 LTS (Trusty Tahr),
* Ubuntu 16.04 LTS (Xenial Xerus),
* Ubuntu 17.10 (Artful Aardvark),
* Debian 8 (Jessie) and
* Debian 9 (Stretch).

It is desigend for developing and testing Ansible roles for these operating systems. The configuration can be easily changed to support other Linux distributions as well.


## 1. Requirements

This setup was tested under Windows 10 with the following environment:
* [VirtualBox = 5.2.8](https://www.virtualbox.org/)
* [Vagrant = 2.0.2](https://www.vagrantup.com/)
* [Ansible = 2.4](http://docs.ansible.com/ansible/)
* [Cygwin 2.10.0](https://www.cygwin.com/), see [Jeff Geerling's](https://www.jeffgeerling.com/) information to [Running Ansible within Windows](http://www.jeffgeerling.com/blog/running-ansible-within-windows)


## 2. Setup Vagrant Box

Under Windows open a cygwin bash for the next steps.


### 2.1 Get Vagrant box configuration from GitLAB

```bash
git clone https://gitlab.com/cogline_vagrant/ansible-development.git
cd ansible-development
```

Now install Ansible roles defined under `provisioning/requirements.yml`:

```bash
ansible-galaxy install -r provisioning/requirements.yml -p provisioning/roles
```


### 2.2 Download official Vagrant Boxes

The official Vagrant boxes can be downloaded from [Vagrant Cloud](https://app.vagrantup.com/).
- [CentOS 6](https://app.vagrantup.com/centos/boxes/6)
- [CentOS 7](https://app.vagrantup.com/centos/boxes/7)
- [Ubuntu/Trusty64](https://app.vagrantup.com/ubuntu/boxes/trusty64)
- [Ubuntu/Xenial64](https://app.vagrantup.com/ubuntu/boxes/xenial64)
- [Ubuntu/Artfule64](https://app.vagrantup.com/ubuntu/boxes/artful64)
- [Debian/Jessie64](https://app.vagrantup.com/debian/boxes/jessie64)
- [Debian/Stretch64](https://app.vagrantup.com/debian/boxes/stretch64)

```bash
vagrant box add centos/6
vagrant box add centos/7
vagrant box add ubuntu/trusty64
vagrant box add ubuntu/xenial64
vagrant box add ubuntu/artful64
vagrant box add debian/jessie64
vagrant box add debian/stretch64
```

If your already have installed the Vagrant boxes you can upgrade them with:
```bash
vagrant box update
```

On some Vagrant boxes the VirtualBox Guest Additions are not preinstalled. Therefore you have to install the [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) plugin first.
```bash
vagrant plugin install vagrant-vbguest
```


### 2.3 Edit the Vagrantfile to fit your needs

#### 2.3.1 Configure Linux operation systems

The `Vagrantfile` contains a variable named boxes, which defines all vargrant box/images that are needed.

```bash
boxes = [
  { # Enterprise Linux 6 (RHEL6/CentOS6)
    :image => 'centos/6',
    :start => true,
    :ip_offset => 240,
    :nodes => 1,
    :hostname => 'el6-node',
    :vbox_name => 'EL6 - Node'
  },
  {
    # Official Ubuntu Server 16.04 LTS (Xenial Xerus)
    :image => 'ubuntu/xenial64',
    :start => true,
    :ip_offset => 220, 
    :nodes => 1,
    :hostname => 'ubuntu-xenial-node', 
    :vbox_name => 'Ubuntu (Xenial) - Node'
  },

  ...
]
```

You can add your own Vagrant boxes or delete other boxes here if you need.

Attribute description:
- **image**: A official Vagrant Box/Image, see  [Vagrant Cloud](https://app.vagrantup.com/)
- **start**: Should the virtual maschien be started on `vagrant up`
- **ip_offset**: The offset for the last 3 ip address digits, e.g. 240 will be convertet to *240* + *node number*. PLease pay attention to the ip_offset attribute to avoid ip address conflicts.
- **nodes**: The number of nodes [1-9] for this os.
- **hostname**: The hostname prefix. The real hostname will be *hostname* + *node number*
- **vbox_name**: Then name prefix in Oracles Virtualbox GUI

**Hint:**
If you define new machines, you must also defiune them in the Ansible inventory file `/vagrant/provisioning/inventory.ini` in order to use the machine with Ansible.

#### 2.3.2 Set correct timezone

The Vagrant timezone configuration doesn't work correctly. That's why I use the solution from [Frédéric Henri](https://stackoverflow.com/users/4296747/fr%c3%a9d%c3%a9ric-henri), see discussion: [How to correct system clock in vagrant automatically](https://stackoverflow.com/questions/33939834/how-to-correct-system-clock-in-vagrant-automatically).

Seach for the following entry in Vagrantfile:
```bash
subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
```

You have to replace 'Europe/Berlin' with the timezone you want to set.


### 2.4 Provisioning

#### 2.4.1 Initial Provisioning

The next step will start all CentOS, Ubuntu and Debian nodes and the Ansible management (*master*) node. While starting the first time Vagrant will be run any configured provisioners against the running managed machines.

```bash
vagrant up
```

**Hint:**
There is a know bug with Vagrant and Ubuntu 17.10 (Artful Aardvark) on creating a private network, see [issue #9134](https://github.com/hashicorp/vagrant/issues/9134). Vagrant tries to use the `/sbin/ifup` and `/sbin/ifdown` commands from the *ifupdown* package, which isn't installed anymore, because Ubuntu 17.10+ uses *netplan* for network configuration instead of the legacy */etc/network/interfaces* method.

That's why `vagrant up` will crash while starting a ubuntu/artful64 box the first time.
If this happens, you must log in to the virtual machine as user *vagrant* with password *vagrant* using the VirtualBox GUI. Install the ifupdown package and switch off the maschine.

```bash
sudo apt-get -y install ifupdown
sudo poweroff
```

Afterwards you can execute `vagrant up` again.

#### 2.4.2 Provisioning with extra Ansible variables

If you want to perform provisioning with special Ansible variables you can use:

```bash
ANSIBLE_ARGS='--extra-vars "variable=value"' vagrant up --provision
```


### 2.5 Save and restore current state

Sometimes it makes sense to safe the current state before testing a new Ansible Role. Thats why you can save the virtual machine state at any time, for example with:

```bash
vagrant snapshot save initial-setup
```
Here *initial-setup* is the name for the snapshot. You can restore the saved state with:

```bash
vagrant snapshot restore initial-setup
```


## 3. Working with the Ansible development environment

### 3.1 Logon to Ansible mangement node

```bash
vagrant ssh
```


### 3.2 Environment preparation

On the management node open the file `/etc/ansible/ansible.cfg` and check the variables *inventory* and *roles_path*:

```bash
inventory     = /vagrant/provisioning/inventory.ini
roles_path    = /vagrant/provisioning/roles:/etc/ansible/roles:/usr/share/ansible/roles
```

If you want to use ŧhe [Ansible Vault](http://docs.ansible.com/ansible/2.4/vault.html) feature with your roles you have to create a Ansible vault password file `/vagrant/provisioning/.ansible_vault` and
* add the following line in *vagrant's* bashrc file: 
```bash
export ANSIBLE_VAULT_PASSWORD_FILE=/vagrant/provisioning/.ansible_vault
```
* and uncomment in the `Vagrantfile` the line:
```bash
# ansible.vault_password_file = "provisioning/.ansible_vault"
```

You can activate the new environment variable with:

```bash
. ~/.bashrc
```


### 3.3 Test if the environment is ready for Ansible

You can cotrol if your ansible environment is working with:
```bash
ansible all -m ping
```

All nodes listed in the `/vagrant/provisioning/inventory.ini` file should respond to this command.


### 3.4 Create and test a new Ansible role

If you want to create a new Ansible role put the new role in directory `/vagrant/provisioning/roles` on the Ansible management node. Remember thœ folder `/vagrant` is a shared folder between the virtual host and the virtual guest system.

```bash
cd /vagrant/provisioning/roles
ansible-galaxy init new-role-name
```
Then you can edit the new role. After finishing the new role you can create a *playbook.yml* file in directory `/vagrant/provisioning`

```bash
cd /vagrant/provisioning
vi playbook.yml
```

and put your new role in the playbook.

```yaml
---
# file: playbook.yml

- hosts: all
  become: yes
  roles:
    - { role: new-role-name }
```

After that you can test your new role/playbook with:

```bash
./test-playbook playbook.yml
```

The shell script *test-playbook* will execute the following four steps:

1. Testing the role’s syntax.
1. Perform a dry run.
1. Run the role/playbook.
1. Run the role/playbook again, checking to make sure it's idempotent.

Thanks to [Jeff Geerling](https://www.jeffgeerling.com/) for his articles to Ansible test strategies. Hє is also the author of [Ansible for DevOps](https://www.jeffgeerling.com/project/ansible-devops), a great book about Ansible.


## 4. Delete the hole Vagrant environment

If you have done your work then the next command stops the running machines which Vagrant is managing and destroys all resources that were created during the creation process. After running this command, your computer should be left at a clean state, as if you never created the guest machines in the first place.

```
vagrant halt && vagrant destroy -f
```

## 5. Version

Release: 1.0.0

## 6. License

GPLv3

## 7. Author Information

This Vagrant environment was created in 2018 by Cogline.v3.
