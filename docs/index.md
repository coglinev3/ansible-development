# A Multi-VM Vagrant environment for Developing and Testing Ansible Roles

This is a multi node [Vagrant](https://www.vagrantup.com/ "Vagrant")
environment which represents a real life [Ansible](http://docs.ansible.com/ansible/ "Ansible")
scenario with one Ansible management node and different Linux OS nodes (Ansible
clients):
![Ansible figure](ansible_figure.svg)
The supported clients are:

* CentOS 6, 
* CentOS 7, 
* Ubuntu 16.04 LTS (Xenial Xerus),
* Ubuntu 18.04 LTS (Bionic Beaver),
* Ubuntu 18.10 (Cosmic Cuttlefish),
* Debian 8 (Jessie),
* Debian 9 (Stretch) and
* Debian 10 (Buster).


It is desigend for developing and testing Ansible playbooks and roles on
these operating systems. The configuration can be easily changed to support
other Linux distributions as well. As Vagrant provider (Hypervisors) [VirtualBox](provider/virtualbox.md "VirtualBox")
or [libvirt](provider/libvirt.md) with KVM can be used, default ist VirtualBox.

Of course, you can use this environment to develop and test other things like
Java applications, but that's not the focus of this documentation.

* [Motivation](/motivation/)
* [Getting Started](/getting_started/)
* [Test Ansible Roles](/test_ansible_roles/)

