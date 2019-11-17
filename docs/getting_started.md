# Quick Start Guide

This document will show you how to get up and running with this multi node
[Vagrant](https://www.vagrantup.com/ "Vagrant") environment. If you are not
familiar with Vagrant, then you should read the [Vagrant Documentation](https://www.vagrantup.com/docs/index.html "Vagrant Documentation") too.


## Requirements

This setup was tested under Windows 10 with the following components: 

* [VirtualBox = 6.0.4](https://www.virtualbox.org/)
* [Vagrant = 2.2.5](https://www.vagrantup.com/)
* [Ansible = 2.8.2](http://docs.ansible.com/ansible/) within [Cygwin 2.10.0](https://www.cygwin.com/), see [Jeff Geerling's](https://www.jeffgeerling.com/) Blog to [Running Ansible within Windows](http://www.jeffgeerling.com/blog/running-ansible-within-windows)

and under Ubuntu 16.04 LTS (Xenial Xerus) and Ubuntu 18.04 LTS (Bionic Beaver) with:

* [VirtualBox = 6.0.10](https://www.virtualbox.org/)
* [libvirt = 4.0.0](https://libvirt.org/index.html)
* [Vagrant = 2.2.5](https://www.vagrantup.com/)
* [Ansible = 2.8.2](http://docs.ansible.com/ansible/)

preinstalled.


!!! Note
    This document does not explain how to install these components. You have to do it yourself by reading the installation guides of these components.


## Get the Vagrant Environment

For the next steps open a bash (under Windows a cygwin bash) on the virtual host system.

### Download Vagrant box configuration

```bash
git clone https://gitlab.com/cogline_vagrant/ansible-development.git
cd ansible-development
```

### Get Ansible roles for provisioning

Now install Ansible roles defined under `provisioning/requirements.yml`:

```bash
ansible-galaxy install -r provisioning/requirements.yml -p provisioning/roles
```

### Install vagrant plugins

Before using this Vagrant environment, you still need to install the following plugins.

```bash
vagrant plugin install vagrant-vbguest
vagrant plugin install vagrant-hostmanager
```

If you use Vagrant with libvirt under Linux, you also need to install the
following plugins
```bash
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-mutate
```
and the NFS kernel server:
```bash
# on Debian/Ubuntu systems
sudo apt install -y nfs-kernel-server
```

## Initial Provisioning

The next step will start all CentOS, Ubuntu and Debian nodes and the Ansible
management node. While starting the first time Vagrant will be run any
configured provisioners against the running managed machines.

!!! Note "This will take a few minutes"
    The first time this step takes a while. All required Vagrant Boxes will be
    downloaded from the [Vagrant Cloud](https://app.vagrantup.com/boxes/search
    "Vagrant Cloud"). Depending on the speed of your internet connection, this
    will take a few minutes. Subsequently, the individual systems are started
    and provisioned in sequence. Then the environment is ready for the
    development and testing of new Ansible playbooks and roles.


```bash
vagrant up
```

If you want to use vagrant with libvirt instead of VirtualBox, use
```bash
VAGRANT_DEFAULT_PROVIDER=libvirt vagrant up --no-parallel
```

!!! attention "libvirt: Switch off the parallel installation the first time"
    The provider libvirt usually performs the installation of virtual machines
    in parallel. It happens that Ansible Provisioner is running on the master
    node before all Ansible clients are up and running. Therefore, the Ansible
    Provisioner sometimes can not reach all clients through SSH from the master
    node, and Ansible fails for the affected clients. Therefore, when starting
    the environment for the first time, the parallel installation should be
    suppressed using the vagrant option `--no-parallel`.
    
    If you have not turned off the parallel installation and the Ansible
    provisioner fails, then the provisioning on the master node can be
    re-executed with the following command after all clients are up and running.

    ```bash
    VAGRANT_DEFAULT_PROVIDER=libvirt vagrant provision master
    ```
