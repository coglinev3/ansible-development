# Configuration options

With the help of the file `config.yml` the most important settings of the
Vagrant environment can be made. This section describes the individual options.

In `config.yml` two dictionary variables are defined:

- **vagrant_config**: Defines the global settings.
- **vagrant_boxes**: Configuration settings for the various Vagrant boxes
  (virtual machines).


## Global settings

Within the dictionary variable *vagrant_config* different environment types can
be defined, for example to distinguish between development and production or
between a Linux and a Windows environment. The *env* key decides which
environment type is currently used.

Example:

!!! info "Global settings within config.yml"
    ```yaml
    ---
    vagrant_config:
      env: 'production'
      staging:
        dynamic_inventory: true
        hostmanager_enabled: false
        hostmanager_manage_guest: true
        hostmanager_manage_host: true
        hostmanager_include_offline: true
        hostmanager_ignore_private_ip: false
        vbguest_auto_update: true
        provider: libvirt
      production:
        dynamic_inventory: true
        hostmanager_enabled: false
        hostmanager_manage_guest: true
        hostmanager_manage_host: false
        hostmanager_include_offline: true
        hostmanager_ignore_private_ip: false
        vbguest_auto_update: false
        provider: virtualbox
    ```


Parameter description:

- **dynamic_inventory**: If *dynamic_inventory* is set to *true*, the
  `provisioning/vagrant.ini` file is used as the Ansible inventory file. It is
  created dynamically, which is recommended. This way, you can add or remove
  new nodes without worrying about customizing the Ansible inventory file. If
  *dynamic_inventory* is set to false, `provisioning/inventory.ini` is used as
  the inventory file. You must manually edit this file each time you add or
  remove new nodes (Ansible clients).
- **hostmanager_enabled**: Starting at Vagrant version 1.5.0, `vagrant up` runs
  hostmanager before any provisioning occurs. If you would like hostmanager to
  run after or during your provisioning stage, you can use hostmanager as a
  provisioner. To disable the default behavior, *hostmanager_enabled* must be
  set to *false*. This allows you to use the provisioning order to ensure that
  hostmanager runs when desired. The provisioner will collect hosts from boxes
  with the same provider as the running box.
- **hostmanager_manage_host**: If the value is *true*, the Hosts file is
  updated on the host's machine. Admin rights are required for this (see
  explanations below).
- **hostmanager_ignore_private_ip**: A machine's IP address is defined by
  either the static IP for a private network configuration or by the SSH host
  configuration. To disable using the private network IP address, set
  *hostmanager_ignore_private_ip* to *true*.
- **hostmanager_include_offline**: If the value is set to *true*, nodes (boxes)
  that are up or have a private ip configured will be added to the `hosts` file.
- **provider**: Defines the Vagrant provider. Valid values are *virtualbox* or
  *libvirt*.
- **vbguest_auto_update**: A VirtualBox specific parameter. Set
  *vbguest_auto_update* to *false* if you do NOT want the correct version of
  VirtualBox Guest Additions to be checked at boot time for each node. If
  *vbguest_auto_update* is *true*, a version check will take place at boot
  time. If the version of the VirtualBox Guest Additions does not match the
  version of VirtualBox, an attempt is made to update the VirtualBox Guest
  Additions. Booting the node may take longer, but the performance of the node
  may be better afterwards.


!!! warning "Write access to the hosts file"
    If you want to manage the hosts file on the host's computer, ensure that
    you have write access to the hosts file.

    **Linux:**

    To avoid being asked for the password every time the `/etc/hosts` file is updated,
    enable passwordless sudo for the specific command that hostmanager uses to
    update the `/etc/hosts` file.

    * Add the following snippet to the sudoers file (e.g. `/etc/sudoers.d/vagrant_hostmanager`):
    ```bash
    Cmnd_Alias VAGRANT_HOSTMANAGER_UPDATE = /bin/cp <home-directory>/.vagrant.d/tmp/hosts.local /etc/hosts
    %<admin-group> ALL=(root) NOPASSWD: VAGRANT_HOSTMANAGER_UPDATE
    ```
    Replace `<home-directory>` with your actual home directory (e.g. /home/joe) and `<admin-group>`
    with the group that is used by the system for sudo access (usually *sudo* on
    Debian/Ubuntu systems and *wheel* on Fedora/Red Hat systems).
    
    * If necessary, add yourself to the `<admin-group>`:
    ```bash
    usermod -aG <admin-group> $USER
    ```
    Replace `<admin-group>` with the group that is used by the system for sudo
    access (see above).


    **Windows:**

    Hostmanager will detect Windows guests and hosts and use the appropriate
    path for the hosts file: `%WINDIR%\System32\drivers\etc\hosts`

    By default on a Windows host, the hosts file is not writable without
    elevated privileges. If hostmanager detects that it cannot overwrite the file,
    it will attempt to do so with elevated privileges, causing the
    UAC prompt to appear.
    To avoid the UAC prompt, open `%WINDIR%\System32\drivers\etc\` in
    Explorer, right-click the `hosts` file, go to "Properties > Security > Edit"
    and give your user Modify permission.

    UAC limitations:

    Due to User Account Control (UAC) limitations, canceling the UAC prompt
    does not cause visible errors. However, when you cancel, the Hosts file is
    not updated.


## Vagrant boxes settings

Every Vagrant environment requires at least one or more so-called boxes. Boxes
are the package format for an operating system image.  You can search for boxes
in  [Vagrant Cloud](https://app.vagrantup.com/boxes/search "Discover Vagrant Boxes").
The boxes to be used are defined in the `config.yml` file using the dictionary
variable *vagrant_boxes*. The *master* key defines the settings for the Ansible
management node. The *clients* key is used to define the settings for the
various Ansible clients.

Example:

!!! info "Client definitions within config.yml"
    ```yaml
    ---
    vagrant_boxes:
      # Ansible management node
      master:
        image: generic/alpine317
        cpus: 1
        vbox_name: 'Management Node'
      # Ansible clients
      clients:
        - image: generic/alpine317
          autostart: true
          hostname: alpine317-node
          cpus: 2
          vbox_name: 'Alpine 3.17 - Node'
          nodes: 3
        - image: generic/alma8
          autostart: true
          hostname: el8-node
          vbox_name: 'EL8 - Node'
        - image: generic/alma9
          autostart: true
          hostname: el9-node
          vbox_name: 'EL9 - Node'
        - image: generic/ubuntu2204
          autostart: true
          hostname: ubuntu2204-node
          memory: '1024'
          vbox_name: 'Ubuntu 22.04 (Jammy Jellyfish) - Node'
          nodes: 2
    ```

You can add your own Vagrant boxes or remove other boxes here if you need.

Parameter description:

- **image**: A Vagrant Box/Image (see  [Vagrant Cloud](https://app.vagrantup.com/ "HashiCorp Vagrant Cloud")).
- **autostart**: By default in a multi-machine environment, `vagrant up` will
  start all of the defined machines. The *autostart* setting allows you to tell
  Vagrant to *not* start specific machines. If *autostart* is set to *false*
  you can manually force the *<node_name>* machine to start by running
  `vagrant up <node_name>`.
- **hostname**: The hostname prefix. The real hostname (`<node_name>`) will be
  *hostname* + *node number*, for instance *alpine317-node2*.
- **cpus**: The CPU count. *Default is 1.*
- **memory**: The main memory size in MB. *Default is 512.*
- **vbox_name**: A VirtualBox specific parameter which defines the name prefix
  in the VirtualBox GUI. The real name will be *vbox_name* + *node_number*. In
  general, the VirtualBox GUI is not needed for Ansible role development and
  testing. But of course you can start and use the VirtualBox GUI.
- **nodes**: The number of nodes for this Box/Image. *Default is 1.*

!!! warning
    If you are not using the dynamic inventory file
    `/vagrant/provisioning/vagrant.ini` (see [below](#environment-type-specific-configuration))
    and you are defining new machines, you must also add them to the Ansible
    inventory file `/vagrant/provisioning/inventory.ini` to use these machines
    with Ansible ,

