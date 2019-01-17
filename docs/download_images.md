# Install and upgrade official Vagrant Boxes

The official Vagrant boxes can be downloaded separately from
[Vagrant Cloud](https://app.vagrantup.com/) if needed.

- [CentOS 6](https://app.vagrantup.com/centos/boxes/6 "CentOS 6")
- [CentOS 7](https://app.vagrantup.com/centos/boxes/7 "CentOS 7")
- [Ubuntu/Trusty64](https://app.vagrantup.com/ubuntu/boxes/trusty64 "Trusty Tahr")
- [Ubuntu/Xenial64](https://app.vagrantup.com/ubuntu/boxes/xenial64 "Xenial Xerus")
- [Ubuntu/Bionic64](https://app.vagrantup.com/ubuntu/boxes/bionic64 "Bionic Beaver")
- [Ubuntu/Cosmic64](https://app.vagrantup.com/ubuntu/boxes/cosmic64 "Cosmic Cuttlefish")
- [Debian/Jessie64](https://app.vagrantup.com/debian/boxes/jessie64 "Debian Jessie")
- [Debian/Stretch64](https://app.vagrantup.com/debian/boxes/stretch64 "Debian Stretch")

```bash
vagrant box add centos/6
vagrant box add centos/7
vagrant box add ubuntu/trusty64
vagrant box add ubuntu/xenial64
vagrant box add ubuntu/bionic64
vagrant box add ubuntu/cosmic64
vagrant box add debian/jessie64
vagrant box add debian/stretch64
```

If you already have installed the Vagrant boxes you can upgrade them with:

```bash
vagrant box update <boxname>
```

Inside the project directory you can easily update all Vagrant boxes with:
```bash
vagrant box update
```

!!! attention "Ensure vbguest plugin is installed"
    On some Vagrant boxes the VirtualBox Guest Additions are not preinstalled.
    Therefore, you have to install the
    [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest)
    plugin first before you can use them.

    ```bash
    vagrant plugin install vagrant-vbguest
    ```

