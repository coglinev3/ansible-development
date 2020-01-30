# Download Vagrant boxes

Vagrant boxes can be downloaded separately from
[Vagrant Cloud](https://app.vagrantup.com/) if needed.

- [Alpine 3.9](https://app.vagrantup.com/generic/boxes/alpine39 "Alpine 3.9")
- [Alpine 3.10](https://app.vagrantup.com/generic/boxes/alpine310 "Alpine 3.10")
- [CentOS 6](https://app.vagrantup.com/centos/boxes/6 "CentOS 6")
- [CentOS 7](https://app.vagrantup.com/centos/boxes/7 "CentOS 7")
- [CentOS 8](https://app.vagrantup.com/generic/boxes/centos8 "CentOS 8")
- [Fedora 30](https://app.vagrantup.com/generic/boxes/fedora30 "Fedora 30")
- [Fedora 31](https://app.vagrantup.com/generic/boxes/fedora31 "Fedora 31")
- [Ubuntu/Trusty64](https://app.vagrantup.com/ubuntu/boxes/trusty64 "Ubuntu 14.04 LTS (Trusty Tahr)")
- [Ubuntu/Xenial64](https://app.vagrantup.com/generic/boxes/ubuntu1604 "Ubuntu 16.04 (Xenial Xerus)")
- [Ubuntu/Bionic64](https://app.vagrantup.com/generic/boxes/ubuntu1804 "Ubuntu 18.04 (Bionic Beaver)")
- [Ubuntu/Disco64](https://app.vagrantup.com/generic/boxes/ubuntu1904 "Ubuntu 19.04 (Disco Dingo)")
- [Debian/Jessie64](https://app.vagrantup.com/debian/boxes/jessie64 "Debian 8 (Jessie)")
- [Debian/Stretch64](https://app.vagrantup.com/debian/boxes/stretch64 "Debian 9 (Stretch)")
- [Debian/Buster64](https://app.vagrantup.com/debian/boxes/buster64 "Debian 10 (Buster)")

```bash
vagrant box add <boxname>
```

If you already have installed the Vagrant boxes you can upgrade them with:

```bash
vagrant box update <boxname>
```

Inside the project directory you can easily update all Vagrant boxes with:
```bash
vagrant box update
```

# Use a new box version

If you want to use a new box version for an existing virtual machine, you must
first destroy the old virtual machine before you can use the new version of a
Vagrant box for that virtual machine.

!!! attention "save your work first"
    Save your work if needed before destroying any machines.


```bash
vagrant destroy el6-node1 -f
vagrant up --provision
```

!!!Note "master node is needed for provisioning"
    If you want to destroy a box and reinstall it, it is not enough to restart
    this box. It is also necessary to start the master node with the option
    `--provision` because the reinstallation via Ansible is done only from
    the master node.

    wrong:
    ```bash
    # this will only bring up the new machine without Ansible provisioning
    vagrant up el6-node1
    ```
    right:
    ```bash
    # this will bring up all machines and executes the provisioners for all
    # machines including the master node
    vagrant up --provision
    
    # or when the nodes are already running
    vagrant provision
    ```

