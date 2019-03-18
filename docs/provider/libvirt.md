# Vagrant Provider libvirt (KVM)

## Vagrant plugins for libvirt

Using [libvirt](https://libvirt.org/drvqemu.html "libvirt Virtualization API") ([KVM](https://www.linux-kvm.org/page/Main_Page "Kernel Virtual Machine")) requires the Vagrant plugin that you install when it is missing (see "vagrant plugin list"):

```bash
vagrant plugin install vagrant-libvirt
```

Some boxes have been configured to work with VirtualBox only. With the Vagrant plugin *mutate* you can convert the images from one VM provider to another. Install the plugin as follows:
```bash
vagrant plugin install vagrant-mutate
```

Now you can download a VirtualBox image and convert it to the libvirt format:
```bash
vagrant box add ubuntu/trusty64
vagrant mutate ubuntu/trusty64 libvirt
```
