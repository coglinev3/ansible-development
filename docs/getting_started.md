# Quick Start Guide

This document shows you how to get this multi-node [Vagrant](https://www.vagrantup.com/ "Vagrant") environment up and running. If you are not familiar with Vagrant, you should also read the [Vagrant Documentation](https://www.vagrantup.com/docs/index.html "Vagrant Documentation").


## Requirements

This setup was tested under Ubuntu 22.04 LTS (Jammy Jellyfish) with

* [VirtualBox = 7.0.6](https://www.virtualbox.org/)
* [libvirt = 8.0.0](https://libvirt.org/index.html)
* [Vagrant = 2.3.4](https://www.vagrantup.com/)
* [Ansible = 2.14.2](http://docs.ansible.com/ansible/)

and under Windows 10 with the following components: 

* [VirtualBox = 6.0.14](https://www.virtualbox.org/)
* [Vagrant = 2.2.6](https://www.vagrantup.com/)
* [Ansible = 2.8.2](http://docs.ansible.com/ansible/) within [Cygwin 2.10.0](https://www.cygwin.com/), see [Jeff Geerling's](https://www.jeffgeerling.com/) Blog to [Running Ansible within Windows](http://www.jeffgeerling.com/blog/running-ansible-within-windows)

preinstalled.


!!! Note
    This document does not explain how to install these components. To do this, follow the installation instructions for these components.


## Get the Vagrant Environment

For the next steps open a bash (under Windows a WSL2 bash) on the virtual host system.
Under Windows you can open bash with [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/ "WSL Documentation") (WSL).

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
vagrant plugin install vagrant-timezone
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
# on Enterprise Linux (CentOS/RedHat)
sudo yum -y install nfs-utils
```

## Define Ansible Client(s)

For your Ansible development environment, you need at least one virtual machine
(Vagrant box) with a [supported operating system](/#supported-operating-systems "Supported Operating Systemѕ").
Open the `config.yml` file and define or comment out your Ansible clients here.
Please note that you do not comment out the management node when doing this.

!!! Note "config.yml"
    ```yaml
    ---
    vagrant_boxes:
      # Ansible management node
      master:
        image: generic/alpine317
        start: true
        cpus: 1
        vbox_name: 'Management Node'
      
      # Ansible clients
      clients:
        - image: debian/bullseye64
          start: true
          hostname: debian11-node
          memory: '1024'
          vbox_name: 'Debian 11 (Bullseye) - Node'
          nodes: 3
        - image: generic/ubuntu2204
          start: true
          hostname: ubuntu2204-node
          memory: '1024'
          vbox_name: 'Ubuntu 22.04 (Jammy Jellyfish) - Node'
          nodes: 1
    ```
In the sample configuration, three nodes with Debian 11 and one node with
Ubuntu 22.04 LTS are started in addition to the management node. See section
"[Define Vagrant Boxes](/vagrantfile/#define-vagrant-boxes)" for more details
on how to use the `config.yml` file.

## Initial Provisioning

The next step will start all Ansible Clients and the Ansible
management node. While starting the first time Vagrant will be run any
configured provisioners against the running managed machines.

!!! Note "This will take a few minutes"
    The first time this step takes a while. All required Vagrant Boxes will be
    downloaded from the [Vagrant Cloud](https://app.vagrantup.com/boxes/search
    "Vagrant Cloud"). Depending on the speed of your internet connection, this
    will take a few minutes. Subsequently, the individual systems are started
    and provisioned in sequence. Then the environment is ready for the
    development and testing of new Ansible playbooks and roles.

### Provisioning with provider VirtualBox

The Vagrant environment can be started with the provider VirtualBox as follows.

```bash
export VAGRANT_DEFAULT_PROVIDER=virtualbox
vagrant up
```

### Provisioning with provider libvirt

If you want to use vagrant with libvirt instead of VirtualBox, use

```bash
export VAGRANT_DEFAULT_PROVIDER=libvirt
vagrant up --no-parallel
```


!!! attention "libvirt: Switch off the parallel installation the first time"
    The provider libvirt usually performs the installation of virtual machines
    in parallel. It happens that Ansible Provisioner is running on the master
    node before all Ansible clients are up and running. Therefore, the Ansible
    Provisioner sometimes can not reach all clients through SSH from the master
    node, and Ansible fails for the affected clients. That's why when starting
    the environment for the first time, the parallel installation should be
    suppressed using the vagrant option `--no-parallel`.
    
    If you have not turned off the parallel installation and the Ansible
    provisioner fails, then the provisioning on the master node can be
    re-executed with the following command after all clients are up and running.

    ```bash
    VAGRANT_DEFAULT_PROVIDER=libvirt vagrant provision master
    ```
