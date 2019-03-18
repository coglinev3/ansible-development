# The whole Vagrantfile

This document describes the base of this development environment - the 
`Vagrantfile`. Every section of this file and how you can use it will be explaind
step by step.

!!! note "Vagrantfile: The whole file"
    ```ruby
    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    
    # Include box configuration from YAML file boxes.yml
    require 'yaml'
    boxes = YAML.load_file(File.join(File.dirname(__FILE__), 'boxes.yml'))
    
    Vagrant.configure(2) do |config|
    
      # define the order of providers 
      config.vm.provider "virtualbox"
      config.vm.provider "libvirt"
    
      # configure the hostmanager plugin
      config.hostmanager.enabled = false
      config.hostmanager.manage_host = true
      config.hostmanager.manage_guest = true
      config.hostmanager.ignore_private_ip = false
      config.hostmanager.include_offline = true
    
      # Box Configuration for Ansible Clients
      boxes.each do |box|
        (1..box["nodes"]).each do |i|
          config.vm.define "#{box['hostname']}#{i}", autostart: box["start"] do |subconfig|
            subconfig.vm.box = box["image"]
            subconfig.vm.synced_folder ".", "/vagrant", disabled: true
            subconfig.vm.hostname = "#{box['hostname']}#{i}"
            subconfig.vm.provider "libvirt" do |libvirt, override|
              libvirt.memory = "512"
              # get DHCP-assigned private network ip-address
              override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
                if hostname = (vm.ssh_info && vm.ssh_info[:host])
                  `vagrant ssh "#{box['hostname']}#{i}" -c "hostname -I"`.split()[0]
                end
              end
            end
            subconfig.vm.provider "virtualbox" do |vbox, override|
              vbox.gui = false
              vbox.name = "#{box['vbox_name']} #{i}"
              vbox.linked_clone = true
              vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
              override.vm.network "private_network", type: "dhcp"
              # get DHCP-assigned private network ip-address
              override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
                if hostname = (vm.ssh_info && vm.ssh_info[:host])
                  `vagrant ssh "#{box['hostname']}#{i}" -c "hostname -I"`.split()[1]
                end
              end
            end
            subconfig.vm.provision :hostmanager
            # The Vagrant timezone configuration doesn't work correctly.
            # That's why I use the solution from Frédéric Henri, see: https://stackoverflow.com/questions/33939834/how-to-correct-system-clock-in-vagrant-automatically
            # You have to replace 'Europe/Berlin' with the timezone you want to set.
            subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime"
          end
        end
      end
    
      # Box configuration for Ansible Management Node
      config.vm.define "master", primary: true do |subconfig|
        subconfig.vm.box = 'centos/7'
        subconfig.vm.hostname = "master"
        subconfig.hostmanager.ip_resolver = proc do |vm, resolving_vm|
          if hostname = (vm.ssh_info && vm.ssh_info[:host])
            `vagrant ssh -c "hostname -I"`.split()[1]
          end
        end
        subconfig.vm.provider "libvirt" do |libvirt, override|
          libvirt.memory = "1024"
          override.vm.synced_folder ".", "/vagrant", type: "nfs"
          override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
            if hostname = (vm.ssh_info && vm.ssh_info[:host])
              `vagrant ssh -c "hostname -I"`.split()[0]
            end
          end
        end
        subconfig.vm.provider "virtualbox" do |vbox, override|
          vbox.memory = "4096"
          vbox.gui = false
          vbox.name = "Management Node"
          vbox.linked_clone = true
          vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
          override.vm.network "private_network", type: "dhcp"
          override.vm.synced_folder ".", "/vagrant", type: "virtualbox", SharedFoldersEnableSymlinksCreate: false
          override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
            if hostname = (vm.ssh_info && vm.ssh_info[:host])
              `vagrant ssh -c "hostname -I"`.split()[1]
            end
          end
        end
        subconfig.vm.provision :hostmanager
        subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime"
        subconfig.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "provisioning/bootstrap.yml"
          # ansible.provisioning_path = "/vagrant"
          ansible.verbose = false
          # ansible.vault_password_file = "provisioning/.ansible_vault"
          # ansible.ask_vault_pass = true
          ansible.limit = "all" # or only "nodes" group, etc.
          ansible.install = true
          ansible.inventory_path = "provisioning/inventory.ini"
          # pass environment variable to ansible, for example:
          # ANSIBLE_ARGS='--extra-vars "system_update=yes"' vagrant up
          ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV['ANSIBLE_ARGS']
        end
      end
    
    end
    ```

# Define Vagrant Boxes

Every Vagrant development environment requires a box. You can search for boxes
at [Vagrant Cloud](https://app.vagrantup.com/boxes/search "Discover Vagrant Boxes").
The variable `boxes` gets the box definitions from the external yaml file called `boxes.yml`

!!! note "Vagrantfile: Include Box Definitions"
    ```bash
    # Include box configuration from YAML file boxes.yml
    require 'yaml'
    boxes = YAML.load_file(File.join(File.dirname(__FILE__), 'boxes.yml'))

    ```
Here is an excerpt from the file `boxes.yml`

!!! note "Content of boxes.yml"
    ```yaml
    ---
    - image: centos/6
      start: true
      hostname: el6-node
      vbox_name: 'EL6 - Node'
      nodes: 1
    - image: centos/7
      start: true
      hostname: el7-node
      vbox_name: 'EL7 - Node'
      nodes: 1
    - image: ubuntu/trusty64
      start: true
      hostname: ubuntu-trusty-node
      vbox_name: 'Ubuntu (Trusty) - Node'
      nodes: 1

      ...

    ```

You can add your own Vagrant boxes or delete other boxes here if you need.

Attribute description:

- **image**: A official Vagrant Box/Image, see  [Vagrant Cloud](https://app.vagrantup.com/)
- **start**: Should the virtual maschien be started on `vagrant up`
- **hostname**: The hostname prefix. The real hostname will be *hostname* + *node number*
- **vbox_name**: Then name prefix in Oracles VirtualBox GUI
- **nodes**: The number of nodes for this os.

!!! attention
    If you define new machines, you must also define them in the Ansible
    inventory file `/vagrant/provisioning/inventory.ini` in order to use the
    machine with Ansible.

# Order of Vagrant providers

If you want to test both providers, then you must specify the order of the
providers to use.

!!! note "Vagrantfile: Order of providers"
    ```bash
      config.vm.provider "virtualbox"
      config.vm.provider "libvirt"
    
    ```
!!! attention
    You can't use both providers at the same time.

# Configure the hostmanager plugin

[vagrant-hostmanager](https://github.com/devopsgroup-io/vagrant-hostmanager) is a Vagrant plugin that manages the hosts file on guest machines (and optionally the host). Its goal is to enable name resolution of multi-machine environments.

!!! note "Vagrantfile: Configure hostmanager plugin"
    ```bash
      config.hostmanager.enabled = false
      config.hostmanager.manage_host = true
      config.hostmanager.manage_guest = true
      config.hostmanager.ignore_private_ip = false
      config.hostmanager.include_offline = true
    ```
Attribute description:

- **hostmanager.manage_host**: Update the `hosts` file on the hosts's machine.
- **hostmanager.manage_guest**: Update the `hosts` file on the guest machines.
- **hostmanager.ignore_private_ip**: A machine's IP address is defined by either the static IP for a private network configuration or by the SSH host configuration. To disable using the private network IP address, set config.hostmanager.ignore_private_ip to true.
- **hostmanager.include_offline**: If the attribute is set to true, boxes that are up or have a private ip configured will be added to the hosts file.
- **hostmanager.enabled**: Starting at version 1.5.0, vagrant up runs hostmanager before any provisioning occurs. If you would like hostmanager to run after or during your provisioning stage, you can use hostmanager as a provisioner. This allows you to use the provisioning order to ensure that hostmanager runs when desired. The provisioner will collect hosts from boxes with the same provider as the running box.

!!! note "Example"
    ```bash
    # Disable the default hostmanager behavior
    config.hostmanager.enabled = false

    # ... possible provisioner config before hostmanager ...

    # hostmanager provisioner
    config.vm.provision :hostmanager

    # ... possible provisioning config after hostmanager ...
    ```


# Start up Ansible clients

Next, Ansible clients are started using the defined boxes in variable `boxes`. This
requires two loops:

- one outer loop for all box definitions (operating systems)
- one inner loop for every node of a operating system type

Within a `Vagrantfile` the construct `array.each do |index|` or 
`(1..#end).each do |index|` will be used for every box definition.

!!! note "Vagrantfile: Loops for boxes and nodes"
    ```ruby
    boxes.each do |box|
      (1..box[:nodes]).each do |i|

      ...
      
      end
    end
    ```

Then, for each node, the definition of the virtual machine comes:

- which image to use, 
- the hostname of the virtual machine, 
- provider specific configuration, like memory or private networks,
- configuration of provisioners

and so on.

!!! note "Vagrantfile: definition for each node"
    ```ruby
       config.vm.define "#{box['hostname']}#{i}", autostart: box["start"] do |subconfig|
         subconfig.vm.box = box["image"]
         subconfig.vm.synced_folder ".", "/vagrant", disabled: true
         subconfig.vm.hostname = "#{box['hostname']}#{i}"
         # configuration for provider libvirt
         subconfig.vm.provider "libvirt" do |libvirt, override|
           libvirt.memory = "512"
           # get DHCP-assigned private network ip-address
           override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
             if hostname = (vm.ssh_info && vm.ssh_info[:host])
               `vagrant ssh "#{box['hostname']}#{i}" -c "hostname -I"`.split()[0]
             end
           end
         end
         # configuration for provider VirtualBox
         subconfig.vm.provider "virtualbox" do |vbox, override|
           vbox.gui = false
           vbox.name = "#{box['vbox_name']} #{i}"
           vbox.linked_clone = true
           vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
           override.vm.network "private_network", type: "dhcp"
           # get DHCP-assigned private network ip-address
           override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
             if hostname = (vm.ssh_info && vm.ssh_info[:host])
               `vagrant ssh "#{box['hostname']}#{i}" -c "hostname -I"`.split()[1]
             end
           end
         end
         # execute the provisioners for each node
         subconfig.vm.provision :hostmanager
         subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime"
       end
    ```

!!! attention
    The Vagrant timezone configuration doesn't work correctly.  That's why I
    use the solution from [Frédéric Henri](https://stackoverflow.com/users/4296747/fr%c3%a9d%c3%a9ric-henri)
    with the shell provisioner, see [stackoverflow](https://stackoverflow.com/questions/33939834/how-to-correct-system-clock-in-vagrant-automatically "How to correct system clock in vagrant automatically")
    You have to replace 'Europe/Berlin' with the timezone you want to set.


# Start up Ansible master with provisioning of clients

At last comes the configuration of the Ansible Management node called `master`.
The Ansible provisioner from this machine will setup each node.

!!! note "Vagrantfile: Master node definition"
    ```ruby
      # Box configuration for Ansible Management Node
      config.vm.define "master", primary: true do |subconfig|
        subconfig.vm.box = 'centos/7'
        subconfig.vm.hostname = "master"
        subconfig.hostmanager.ip_resolver = proc do |vm, resolving_vm|
          if hostname = (vm.ssh_info && vm.ssh_info[:host])
            `vagrant ssh -c "hostname -I"`.split()[1]
          end
        end
        subconfig.vm.provider "libvirt" do |libvirt, override|
          libvirt.memory = "1024"
          override.vm.synced_folder ".", "/vagrant", type: "nfs"
          override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
            if hostname = (vm.ssh_info && vm.ssh_info[:host])
              `vagrant ssh -c "hostname -I"`.split()[0]
            end
          end
        end
        subconfig.vm.provider "virtualbox" do |vbox, override|
          vbox.memory = "4096"
          vbox.gui = false
          vbox.name = "Management Node"
          vbox.linked_clone = true
          vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
          override.vm.network "private_network", type: "dhcp"
          override.vm.synced_folder ".", "/vagrant", type: "virtualbox", SharedFoldersEnableSymlinksCreate: false
          override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
            if hostname = (vm.ssh_info && vm.ssh_info[:host])
              `vagrant ssh -c "hostname -I"`.split()[1]
            end
          end
        end
        subconfig.vm.provision :hostmanager
        subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime"
        # provisioning of each node with Ansible
        subconfig.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "provisioning/bootstrap.yml"
          ansible.verbose = false
          # ansible.vault_password_file = "provisioning/.ansible_vault"
          # ansible.ask_vault_pass = true
          ansible.limit = "all" # or only "nodes" group, etc.
          ansible.install = true
          ansible.inventory_path = "provisioning/inventory.ini"
          # pass environment variable to ansible, for example:
          # ANSIBLE_ARGS='--extra-vars "system_update=yes"' vagrant up
          ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV['ANSIBLE_ARGS']
        end
      end
    ```

The Ansible provisioner executes the playbook `bootstrap.yml` from the directory `provisioning`. As Ansible inventory file `inventory.ini` is used.

!!! attention "provisioning/inventory.ini"
    If you change the images or the number of nodes in `boxes.yml`, you must also change the ansible inventory file` inventory.ini`.
    For example, if you change the number of CentOS 6 nodes from one to three,
    you should add two more nodes to the group [el6-nodes] to start and configure them.
    ```dosini
    #  ansible inventory file
    
    [management-node]
    localhost   ansible_connection=local ansible_host=127.0.0.1
    
    [el6-nodes]
    el6-node1	ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.el6-node1
    el6-node2	ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.el6-node2
    el6-node2	ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.el6-node3
    
    ...

    ```

    After that you can start an provision then new nodes with:
    ```bash
    vagrant up --provision
    ```

