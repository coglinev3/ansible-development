# Release Notes for Version 2.5.0

  * Added Alpine 3.9
  * Added Alpine 3.10
  * Added Alpine 3.11
  * Added Alpine 3.12
  * Added Fedora 32
  * Added Ubuntu 20.04 TLS (Focal Fossa)
  * Removed Ubuntu 19.04 (Disco Dingo) - End of Life
  * Replace pre_task statements for python installation with role ansible_python
  * Set python interpreter discovery for Ansible to 'auto'

# Release Notes for Version 2.4.0

  * Added new feature *dynamic_inventory* to automaticaly create the Ansible
  inventory file

# Release Notes for Version 2.3.0

  * Added Fedora 30
  * Added Fedora 31
  * Added `config.yml` to configure some Vagrant environment options

# Release Notes for Version 2.2.1

  * Set hostmanager.manage_host back to false by default to avoid problems on
  Windows systems (see https://ansible-development.readthedocs.io/en/master/vagrantfile/#configure-the-hostmanager-plugin).

# Release Notes for Version 2.2.0

  * Added CentOS 8

# Release Notes for Version 2.1.0

  * Added Ubuntu 19.04 (Disco Dingo)
  * Added Debian 10 (Buster)
  * Removed Ubuntu 18.10 (Cosmic Cuttlefish) - End of Life

# Release Notes for Version 2.0.1

* Add a hint to documentation how to switch off the parallel installation
  the first time when using provider libvirt, to avoid provisioning on the
  master node before all clients are up and running (see [initial provisioning](https://ansible-development.readthedocs.io/en/latest/getting_started/#initial-provisioning)).


# Release Notes for Version 2.0.0
  * Individual options for the count of nodes have been completely removed.
  * The Vagrant boxes are now configured using the YAML file `boxes.yml`.
  * Two Vagrant providers are now supported: Oracle VirtualBox and libvirt (Linux KVM)
  * IP addresses of virtual machines are now configured via DHCP
  * Supported Linux Distributions
    - CentOS 6
    - CentOS 7
    - Ubuntu 14.04 LTS (Trusty Tahr)[^footnote]
    - Ubuntu 16.04 LTS (Xenial Xerus)
    - Ubuntu 18.04 LTS (Bionic Beaver)
    - Ubuntu 18.10 (Cosmic Cuttlefish)
    - Debian 8 (Jessie)
    - Debian 9 (Stretch)


# Release Notes for Version 1.2.0
  * Added Ubuntu 18.04 LTS (Bionic Beaver)
  * Added Ubuntu 18.10 (Cosmic Cuttlefish)
  * Removed Ubuntu 17.10 (Artful Aardvark) removed - End of Life
  * Documatation https://ansible-development.readthedocs.io/en/latest/ changed.


# Release Notes for Version 1.1.0
  * Initial documentation finished: https://ansible-development.readthedocs.io


# Release Notes for Version 1.0.1
  * Added Ubuntu 14.04 LTS (Trusty Tahr)
  * Added support for readthedocs.io


# Release Notes for Version 1.0.0
  * Initial version which supports
    - CentOS 6
    - CentOS 7
    - Debian 8 (Jessie)
    - Debian 9 (Stretch)
    - Ubuntu 16.04 LTS (Xenial Xerus)
    - Ubuntu 17.10 (Artful Aardvark)


[^footnote]: If you want to use the Vagrant Box/Image for Ubuntu 14.04 LTS with libvirt you must
    convert it with the Vagrant plugin mutate, see [Vagrant provider specific tasks](/provider/libvirt/#required-plugins-for-vagrant-provider-libvirt) for details.
