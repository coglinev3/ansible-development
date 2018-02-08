# Vagrant: Ansible development environment

The Vagrant environment consist of one Ansible management node named *master*, and different Linux OS nodes: CentOS 6, CentOS 7, Ubuntu 16.04.3 LTS (Xenial Xerus) and Debian 8 (Jessie).  It is desigend for developing and testing Ansible roles on these operating systems.

## 1. Requirements

This setup was tested with the following environment:
* [VirtualBox = 5.1.30](https://www.virtualbox.org/)
* [Vagrant = 2.0.0](https://www.vagrantup.com/)
* [Ansible = 2.4](http://docs.ansible.com/ansible/)
* [Running Ansible within Windows](http://www.jeffgeerling.com/blog/running-ansible-within-windows)

## 2. Setup Vagrant Box

Under Windows now open a cygwin bash for the next steps.

### 2.1 Get Vagrant box configuration from GitLAB

```bash
git clone https://gitlab.com/cogline_vagrant/ansible-development.git
cd ansible-development
```

Now install Ansible roles defined under `provisioning/requirements.yml`:

```bash
ansible-galaxy install -r provisioning/requirements.yml -p provisioning/roles
```

### 2.2 Download official Vagrant imagees

The official Vagrant images can be downloaded from Vagrant Cloud.
- [CentOS 6](https://app.vagrantup.com/centos/boxes/6)
- [CentOS 7](https://app.vagrantup.com/centos/boxes/7)
- [Ubuntu/Xenial64](https://app.vagrantup.com/ubuntu/boxes/xenial64)
- [Debian/Jessie64](https://www.debian.org/releases/jessie/)

```bash
vagrant box add centos/6
vagrant box add centos/7
vagrant box add ubuntu/xenial64
vagrant box add debian/jessie64
```

If your already have one of those Vagrant boxes you can upgrade them with:
```bash
vagrant box update
```

The VirtualBox Guest Additions are not preinstalled. If you use VirtualBox you have to install the [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) plugin first.
```bash
vagrant plugin install vagrant-vbguest
```

### 2.3 Provisioning

#### 2.3.1 Initial Provisioning

The next step will start all CentOS 7, CentOS 6, Ubuntu and Debian nodes and the Ansible management (*master*) node. While starting the first time Vagrant will be run any configured provisioners against the running managed machines.
```bash
vagrant up
```

#### 2.3.1 Provisioning with system updates

If you want perform system updates while provisioning use the following lines:

```bash
ANSIBLE_ARGS='--extra-vars "system_update=yes"' vagrant up --provision
vagrant reload
```

### 2.4 Save and restore current state

If the virtual machine starts sucessfully you could now save the virtual machine
state:

```bash
vagrant snapshot save initial-setup
```

and you can restore the saved state at any time with:
```bash
vagrant snapshot restore initial-setup
```

### 2.5 Logon to Ansible mangement node

```bash
vagrant ssh
```

## 3. Working with the Ansible development environment

If you want to test a new Ansible role put the new role in directory
`/vagrant/provisioning/roles` on the Ansible management node and edit the file
`/etc/ansible/ansible.cfg`. Change the variables *inventory* and *roles_path* to:
```bash
inventory     = /vagrant/provisioning/inventory.ini
roles_path    = /vagrant/provisioning/roles:/etc/ansible/roles:/usr/share/ansible/roles
```
In *vagrant's* bashrc file `/home/vagrant/.bashrc` you have to add the
following line:
```bash
export ANSIBLE_VAULT_PASSWORD_FILE=/vagrant/provisioning/.ansible_vault
```
in order to use the Ansible *"Vault"* feature with your playbooks. With the
command
```bash
. ~/.bashrc
```
you can activate the new environment variable.

Then you can create a *playbook.yml* file in directory `/vagrant/provisioning`
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
cd /vagrant/provisioning
./test-playbook playbook.yml
```

The shell script *test-playbook* will execute the following four steps:

1. Testing the role’s syntax.
1. Perform a dry run.
1. Run the role/playbook.
1. Run the role/playbook again, checking to make sure it's idempotent.


## 4. Delete the hole Vagrant environment

If you have done your work then the next command stops the running machines which Vagrant is managing and destroys all resources that were created during the creation process. After running this command, your computer should be left at a clean state, as if you never created the guest machine in the first place.

```
vagrant halt && vagrant destroy -f
```

## 5. Version

Release: x.y

## 6. License

GPLv3

## 7. Author Information

This Vagrant environment was created in 2018 by Cogline.v3.
