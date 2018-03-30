# Install and upgrade official Vagrant Boxes

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

On some Vagrant boxes the VirtualBox Guest Additions are not preinstalled.
Therefore you have to install the
[vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) plugin first.

```bash
vagrant plugin install vagrant-vbguest
```
