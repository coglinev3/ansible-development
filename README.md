# Development environment for Ansible

[![Documentation Status](https://readthedocs.org/projects/ansible-development/badge/?version=latest)](http://ansible-development.readthedocs.io/en/latest/?badge=latest)

This is a multi node [Vagrant](https://www.vagrantup.com/ "Vagrant")/[VirtualBox](https://www.virtualbox.org/ "VirtualBox")
environment which represents a real life [Ansible](http://docs.ansible.com/ansible/ "Ansible")
scenario with one Ansible management node and different Linux OS nodes (Ansible
clients):

* CentOS 6, 
* CentOS 7, 
* Ubuntu 14.04 LTS (Trusty Tahr),
* Ubuntu 16.04 LTS (Xenial Xerus),
* Ubuntu 17.10 (Artful Aardvark),
* Debian 8 (Jessie) and
* Debian 9 (Stretch).

It is desigend for developing Ansible playbooks and roles and testing them on
these operating systems. The configuration can be easily changed to support
other Linux distributions as well.

For detailed documentation how to use this environment look at:
[Ansible Development Environment](http://ansible-development.readthedocs.io/en/latest/ "Ansible Development Environment") 

## Requirements

This setup was tested under Windows 10 with the following environment:
* [VirtualBox = 5.2.8](https://www.virtualbox.org/)
* [Vagrant >= 2.0.2](https://www.vagrantup.com/)
* [Ansible = 2.4](http://docs.ansible.com/ansible/)
* [Cygwin 2.10.0](https://www.cygwin.com/), see [Jeff Geerling's](https://www.jeffgeerling.com/) information to [Running Ansible within Windows](http://www.jeffgeerling.com/blog/running-ansible-within-windows)

Of course, this Vagrant environment should also work on nativ Linux if *Vagrant*, *VirtualBox* and *Ansible* are installed.


## Setup Vagrant Box Environment

For the next steps open a bash (under Windows a cygwin bash) on the host system.

### Get Vagrant box configuration from GitLAB

```bash
git clone https://gitlab.com/cogline_vagrant/ansible-development.git
cd ansible-development
```

Now install Ansible roles defined under `provisioning/requirements.yml`:

```bash
ansible-galaxy install -r provisioning/requirements.yml -p provisioning/roles
```

### Install VirtualBox Guest Plugin

On some Vagrant boxes the VirtualBox Guest Additions are not preinstalled. Therefore you have to install the [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) plugin first.
```bash
vagrant plugin install vagrant-vbguest
```

### Start the environment

The next step will start all CentOS, Ubuntu and Debian nodes and the Ansible management (*master*) node. While starting the first time Vagrant will be run any configured provisioners against the running managed machines.

```bash
vagrant up
```

## Version

Release: 1.1.0

## License

GPLv3

## Author Information

This Vagrant environment was created in 2018 by Cogline.v3.
