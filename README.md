# A Multi-VM Vagrant environment for Developing and Testing Ansible Roles

[![Documentation Status](https://readthedocs.org/projects/ansible-development/badge/?version=latest)](http://ansible-development.readthedocs.io/en/latest/?badge=latest)

This is a multi node [Vagrant](https://www.vagrantup.com/ "Vagrant")
environment which represents a real life [Ansible](http://docs.ansible.com/ansible/ "Ansible")
scenario with one Ansible management node and different Linux OS nodes (Ansible
clients).

The supported clients are:

* CentOS 6, 
* CentOS 7, 
* Ubuntu 14.04 LTS (Trusty Tahr),
* Ubuntu 16.04 LTS (Xenial Xerus),
* Ubuntu 18.04 LTS (Bionic Beaver),
* Ubuntu 18.10 (Cosmic Cuttlefish),
* Debian 8 (Jessie) and
* Debian 9 (Stretch).


It is desigend for developing and testing Ansible playbooks and roles on
these operating systems. The configuration can be easily changed to support
other Linux distributions as well. As Vagrant provider (Hypervisors)
[VirtualBox](https://www.virtualbox.org/ "Oracle VirtualBox")
or [libvirt](https://libvirt.org/index.html "libvirt Virtualization API") with KVM can be used, default ist VirtualBox.

For detailed documentation how to use this environment look at:
[Vagrant environment for testing Ansible Roles](http://ansible-development.readthedocs.io/en/latest/ "Ansible Development Environment") 


## Requirements

This setup was tested under Windows 10 with the following components: 

* [VirtualBox = 6.0.4](https://www.virtualbox.org/)
* [Vagrant = 2.2.4](https://www.vagrantup.com/)
* [Ansible = 2.2.3](http://docs.ansible.com/ansible/) within [Cygwin 2.10.0](https://www.cygwin.com/), see [Jeff Geerling's](https://www.jeffgeerling.com/) Blog to [Running Ansible within Windows](http://www.jeffgeerling.com/blog/running-ansible-within-windows)

and under Ubuntu 16.04 LTS (Xenial Xerus) and Ubuntu 18.04 LTS (Bionic Beaver) with:

* [VirtualBox = 6.0.4](https://www.virtualbox.org/)
* [libvirt = 4.0.0](https://libvirt.org/index.html)
* [Vagrant = 2.2.4](https://www.vagrantup.com/)
* [Ansible = 2.7.9](http://docs.ansible.com/ansible/)

preinstalled.


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

### Install vagrant plugins

Before using this Vagrant environment, you still need to install the following plugins.

```bash
vagrant plugin install vagrant-vbguest
vagrant plugin install vagrant-hostmanager
```

If you use Vagrant with libvirt, you also need to install these plugins.
```bash
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-mutate
```

## Initial Provisioning

The next step will start all CentOS, Ubuntu and Debian nodes and the Ansible
management node. While starting the first time Vagrant will be run any
configured provisioners against the running managed machines.

```bash
vagrant up
```

If you want to use vagrant with libvirt instead of VirtualBox, use
```bash
VAGRANT_DEFAULT_PROVIDER=libvirt vagrant up
```

The first time this step takes a while. All required Vagrant Boxes will be
downloaded from the [Vagrant Cloud](https://app.vagrantup.com/boxes/search "Vagrant Cloud").
Depending on the speed of the internet connection, this will take a few minutes.
Subsequently, the individual systems are started and provisioned in sequence.
Then the environment is ready for the development and testing of new Ansible
playbooks and roles.


## Version

Release: 1.2.0


## License

GPLv3


## Author Information

This Vagrant environment was created in 2019 by Cogline.v3.
