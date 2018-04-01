# Quick Start Guide

This document will show you how to get up and running with this multi node
[Vagrant](https://www.vagrantup.com/ "Vagrant") environment. If you are not
familiar with Vagrant and  VirtualBox, then you should read thєse documents:

* [Vagrant Documentation](https://www.vagrantup.com/docs/index.html "Vagrant Documentation")
* [VirtualBox User Manual](https://www.virtualbox.org/manual/ "VirtualBox User Manual")


## Requirements

This setup was tested under Windows 10 with the following components installed:

* [VirtualBox = 5.2.8](https://www.virtualbox.org/)
* [Vagrant = 2.0.3](https://www.vagrantup.com/)
* [Ansible = 2.4](http://docs.ansible.com/ansible/)
* [Cygwin 2.10.0](https://www.cygwin.com/), see [Jeff Geerling's](https://www.jeffgeerling.com/) Blog to [Running Ansible within Windows](http://www.jeffgeerling.com/blog/running-ansible-within-windows)

Of course, this Vagrant environment should also work on nativ Linux if
*Vagrant*, *VirtualBox* and *Ansible* are installed on the virtual host system.


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

### Install vagrant-vbguest plugin

On some Vagrant boxes the VirtualBox Guest Additions are not preinstalled.
Therefore you have to install the
[vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest "vagrant-vbguest")
plugin first.

```bash
vagrant plugin install vagrant-vbguest
```

## Initial Provisioning

The next step will start all CentOS, Ubuntu and Debian nodes and the Ansible
management node. While starting the first time Vagrant will be run any
configured provisioners against the running managed machines.

```bash
vagrant up
```

The first time this step takes a while. All required Vagrant Boxes will be
downloaded from the [Vagrant Cloud](https://app.vagrantup.com/boxes/search "Vagrant Cloud").
Depending on the speed of the internet connection, this will take a few minutes.
Subsequently, the individual systems are started and provisioned in sequence.
Then the environment is ready for the development and testing of new Ansible
playbooks and roles.


!!! warning "Known Bug with Ubuntu 17.10:"
    There is a know bug with Vagrant until version 2.0.2 and Ubuntu 17.10
    (Artful Aardvark) on creating a private network, see
    [issue #9134](https://github.com/hashicorp/vagrant/issues/9134). Vagrant
    tries to use the `/sbin/ifup` and `/sbin/ifdown` commands from the
    *ifupdown* package, which isn't installed anymore, because Ubuntu 17.10+
    uses *netplan* for network configuration instead of the legacy
    `/etc/network/interfaces` method.

    **Solution:**

    * Vagrant 2.0.3 solves this issue, that's why the recommended way ist to
      upgrade to version 2.0.3.
    * If you can't upgrade to Vagrant 2.0.3 `vagrant up` will crash while starting a
      ubuntu/artful64 box the first time. If this happens, you must log in to
      the virtual machine as user *vagrant* with password *vagrant* using the
      VirtualBox GUI. Install the *ifupdown* package and switch off the machine
      (see below). Afterwards you can execute `vagrant up` again on the virtual
      host console.

    Login as user *vagrant* via VirtualBox GUI :
    
        # solve issue #9134 on Ubuntu 17.10 (Artful Aardvark) system
        sudo apt-get -y install ifupdown
        sudo poweroff

