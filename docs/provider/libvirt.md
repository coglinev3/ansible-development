# KVM with QEMU and libvirt

## Overview

### Kernel Virtual Machine

[KVM](https://www.linux-kvm.org/page/Main_Page "KVM") (for Kernel-based Virtual Machine) is a full virtualization solution for Linux on x86 hardware containing virtualization extensions (Intel VT or AMD-V). It consists of a loadable kernel module, kvm.ko, that provides the core virtualization infrastructure and a processor specific module, kvm-intel.ko or kvm-amd.ko.

### QEMU

[QEMU](https://www.qemu.org/ "QEMU") is a generic and open source machine emulator and virtualizer.

QEMU can be used in several different ways. The most common is for [System Emulation](https://www.qemu.org/docs/master/system/index.html#system-emulation "System Emulation"), where it provides a virtual model of an entire machine (CPU, memory and emulated devices) to run a guest OS. In this mode the CPU may be fully emulated, or it may work with a hypervisor such as KVM, Xen, Hax or Hypervisor.Framework to allow the guest to run directly on the host CPU.

### libvirt

[Libvirt](https://libvirt.org/drvqemu.html "libvirt Virtualization API") is a KVM/QEMU driver that can manage any QEMU emulator.

The libvirt project:

* is a toolkit to manage [virtualization platforms](https://libvirt.org/platforms.html "Supported host platforms")
* is accessible from C, Python, Perl, Java and more
* is licensed under open source licenses
* supports [KVM/QEMU](https://libvirt.org/drvqemu.html "KVM/QEMU hypervisor driver"), [Xen](https://libvirt.org/drvxen.html "libxl hypervisor driver for Xen"), [Virtuozzo](https://libvirt.org/drvvirtuozzo.html "Virtuozzo driver"), [VMWare ESX](https://libvirt.org/drvesx.html "VMware ESX hypervisor driver"), [LXC](https://libvirt.org/drvlxc.html "LXC container driver"), [BHyve](https://libvirt.org/drvbhyve.html "Bhyve driver") and [more](https://libvirt.org/drivers.html)
* targets Linux, FreeBSD, Windows and OS-X
* is used by many applications


This Vagrant environment uses KVM with the libvirt driver.

## Requirements

Using libvirt requires the Vagrant plugin [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt "Vagrant Libvirt Provider").

```bash
vagrant plugin install vagrant-libvirt
```

Some boxes have been configured to work with VirtualBox only. With the plugin [vagrant-mutate](https://github.com/sciurus/vagrant-mutate "vagrant-mutate: convert vagrant boxes to work with different providers") you can convert the images from one VM provider to another. Install the plugin as follows:
```bash
vagrant plugin install vagrant-mutate
```

Now you can download a VirtualBox image and convert it to the libvirt format:
```bash
vagrant box add ubuntu/trusty64
vagrant mutate ubuntu/trusty64 libvirt
```

## Snapshots

### Save all virtual machines

Because the libvirt provider does not support snapshots like the VirtualBox
provider, you must use other tools for taking snapshots. In the following
examples the command-line tool `virsh` from libvirt is used to create and
restore snapshots, see [Virsh Command Reference](https://libvirt.org/sources/virshcmdref/html/ "Virsh Command Reference") for details.

Take a snapshot from all virtual machines:
```bash
for vm in $(virsh list --all --name)
do
  virsh snapshot-create-as $vm --name initial-setup
done
```

!!!Note
    Here *initial-setup* is the name for this snapshot. The snapshot list can be
    printed with:

        for vm in $(virsh list --all --name); do echo $vm; virsh snapshot-list $vm; done

### Save a single machine

If only a single machine is to be saved, eg. el6-node1, the following command can be executed:

```bash
virsh snapshot-create-as ansible-development_el6-node1 --name initial-setup

```

!!!attention
    libvirt uses the local directory name, for example `ansible-development` as a prefix for each virtual machine, see

        virsh list --all


### Restore all machines

You can restore the saved state for all machines with:

```bash
for vm in $(virsh list --all --name)
do
  echo reset $vm
  virsh snapshot-revert $vm --snapshotname initial-setup --running
done
```

### Restore a single machine

If you want to reset the virtual machine *el6-node1* to the snapshot
*initial-setup*, enter the following command.

```bash
virsh snapshot-revert ansible-development_el6-node1 --snapshotname initial-setup --running
```

## Delete machines

### Delete the hole Vagrant environment

If you have done your work then the next command stops the running machines
which Vagrant is managing and destroys all resources that were created during
the creation process. After running this command, your computer should be left
at a clean state, as if you never created the guest machines in the first place.

```bash
vagrant destroy -f
```

### Delete a single machine

The removal of a single machine is again shown using the example of *el6-node1*:

```bash
vagrant destroy el6-node1 -f
```
