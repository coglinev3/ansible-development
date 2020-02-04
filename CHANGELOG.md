# Release 2.3.0

  * Added Fedora 30
  * Added Fedora 31
  * Added `config.yml` to configure some Vagrant environment options

# Release 2.2.1

  * Set hostmanager.manage_host back to false by default to avoid problems on
  Windows systems (see https://ansible-development.readthedocs.io/en/master/vagrantfile/#configure-the-hostmanager-plugin).

# Release 2.2.0

  * Added CentOS 8

# Release 2.1.0

  * Added Ubuntu 19.04 (Disco Dingo)
  * Added Debian 10 (Buster)
  * Removed Ubuntu 18.10 (Cosmic Cuttlefish) - End of Life

# Release 2.0.1

  * Add a hint to documentation how to switch off the parallel installation the
  first time when using provider libvirt, to avoid provisioning on the
    master node before all clients are up and running (see [initial provisioning](https://ansible-development.readthedocs.io/en/latest/getting_started/#initial-provisioning)).

# Release 2.0.0
  * Boxes are configurable with YAML file boxes.yml
  * Supports now tow vagrant provider: Oracle VirtualBox and libvirt (Linux KVM)
  * IP addresses of virtual machines are now configured via DHCP

# Release 1.2.0
  * Added Ubuntu 18.04 LTS (Bionic Beaver)
  * Added Ubuntu 18.10 (Cosmic Cuttlefish)
  * Removed Ubuntu 17.10 (Artful Aardvark) - End of Life
  * Documatation https://ansible-development.readthedocs.io/en/latest/ changed.

# Release 1.1.0
  * Initial documentation finished: https://ansible-development.readthedocs.io

# Release 1.0.1
  * Added Ubuntu 14.04 LTS (Trusty Tahr)
  * Added support for readthedocs.io

# Release 1.0.0
  * Initial version which supports
    - CentOS 6
    - CentOS 7
    - Debian 8 (Jessie)
    - Debian 9 (Stretch)
    - Ubuntu 16.04 LTS (Xenial Xerus)
    - Ubuntu 17.10 (Artful Aardvark)
