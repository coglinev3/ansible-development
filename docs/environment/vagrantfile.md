# The Vagrantfile

The heart of this environment is the `Vagrantfile`. The primary function of the
Vagrantfile is to describe the type of machines required for a project, and how
to configure and provision these machines. Each section of this file is
explained step by step.

!!! warning "Avoid changing the Vagrantfile"
    Be careful when you change the Vagrantfile. One small mistake and the
    environment is unusable. In most cases, it is not necessary to change the
    `Vagrantfile`. Instead, adjust the options in the [config.yml](config_options.md "Configuration options")
    file.

## Read config file

At the beginning the configuration file `config.yml` is read in YAML format. The
general configuration settings are then assigned to the *vagrant_config*
variable. The *ansible_client* and *ansible_master* variables contain the box
configurations for the Ansible client nodes and the management node.

!!! info "Vagrantfile: Include global configuration and box definitions"
    ```bash
    require 'yaml'
    # Include configuration from YAML file config.yml
    config = YAML.load_file(File.join(File.dirname(__FILE__), 'config.yml'))
    vagrant_config = config['vagrant_config'][config['vagrant_config']['env']]
    ansible_client = config['vagrant_boxes']['clients']
    ansible_master = config['vagrant_boxes']['master']

    ```

## Set environment variables

Environment variables can be defined using ENV["Name"] = Value. For example, the
default language can be set with:

!!! info "Vagrantfile: Set environment variables"
    ```ruby
    ENV["LANG"] = "C.UTF-8"
    ```

## Configuration Object

The definition of all machines and all Vagrant parameters is done within the
configuration object.

!!! info "Vagrantfile: Configuration object"
    ```ruby
    Vagrant.configure("2") do |config|
      # ...
    end
    ```

The "2" in the first line above represents the version of the configuration
object `config` that will be used for configuration for that block (the section
between the `do` and the `end`). 
 
## Provider and plugin configuration

Within the configuration object, the global settings for the provider and the
plugins used are defined first.

!!! info "Vagrantfile: Provider and plugin configuration"
    ```ruby
    # define the provider (virtualbox | libvirt)
    config.vm.provider vagrant_config['provider']
  
    # configure the hostmanager plugin
    config.hostmanager.enabled = vagrant_config['hostmanager_enabled']
    config.hostmanager.manage_guest = vagrant_config['hostmanager_manage_guest']
    config.hostmanager.manage_host = vagrant_config['hostmanager_manage_host']
    config.hostmanager.include_offline = vagrant_config['hostmanager_include_offline']
    config.hostmanager.ignore_private_ip = vagrant_config['hostmanager_ignore_private_ip']
    ```

[Vagrant-hostmanager](https://github.com/devopsgroup-io/vagrant-hostmanager) is
a Vagrant plugin that manages the `/etc/hosts` file on guest machines (and
optionally on the host). The configuration of the Hostmanager plugin is done in
the configuration file [config.yml](config_options.md#global-settings "Configuration options").

## Starting Ansible Clients

Ansible clients are then started using the configuration settings defined in
the *ansible_client* variable. This requires two loops:

- one outer loop for all box definitions (operating systems)
- one inner loop for every node of a operating system type

Within the `Vagrantfile` the construct `array.each do |index|` or 
`(1..#end).each do |index|` will be used for every box definition.

!!! Note "Vagrantfile: Loops for boxes and nodes"
    ```ruby
    ansible_client.each do |box|
      # If the nodes option is not set in `boxes.yml`, 1 is used by default.
      box_nodes = box["nodes"] || 1
      (1..box_nodes).each do |i|

      ...
      
      end
    end
    ```

In the inner loop, the following is specified for each node:

- which image should be used
- what is the host name,
- which specific client configuration (e.g., amount of main memory or number of CPUs) is to be used
- which provisioning should be performed, etc.


!!! Note "Vagrantfile: Definitions for each node"
    ```ruby
    config.vm.define "#{box['hostname']}#{i}", autostart: box["autostart"] do |subconfig|
      subconfig.vm.box = box["image"]
      subconfig.vm.synced_folder ".", "/vagrant", disabled: true
      if Vagrant.has_plugin?("vagrant-vbguest")
        subconfig.vbguest.auto_update = current_config['vbguest_auto_update']
      end # if vagrant-vbguest
      subconfig.vm.hostname = "#{box['hostname']}#{i}"
      subconfig.vm.provider "libvirt" do |libvirt, override|
        libvirt.cpus = box["cpus"] || 1
        libvirt.memory = box["memory"] || 512
        libvirt.nested = false
      end # libvirt
      subconfig.vm.provider "virtualbox" do |vbox, override|
        # Don't install VirtualBox guest additions on Alpine Linux with
        # vagrant-vbguest plugin, because this doesn't work under Alpine Linux.
        if box["image"] =~ /alpine/
          if Vagrant.has_plugin?("vagrant-vbguest")
            override.vbguest.auto_update = false
          end # plugin vagrant-vbguest
        end # if alpine
        vbox.gui = false
        vbox.cpus = box["cpus"] || 1
        vbox.memory = box["memory"] || 512
        vbox.name = "#{box['vbox_name']} #{i}"
        vbox.linked_clone = true
        vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
        vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        override.vm.network "private_network", type: "dhcp"
        # get DHCP-assigned private network ip-address
        override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
          if hostname = (vm.ssh_info && vm.ssh_info[:host])
            # detect private network ip address on every Linux OS
            `vagrant ssh "#{box['hostname']}#{i}" -c  "ip addr show eth1|grep -v ':'|egrep -o '([0-9]+\.){3}[0-9]+'"`.split(' ')[0]
          end # if hostname
        end # resolving_vm
      end # virtualbox
      subconfig.vm.provision "time zone data", type: "shell", path: "provisioning/scripts/install_tzdata"
      if Vagrant.has_plugin?("vagrant-timezone")
        subconfig.timezone.value = :host
      end # plugin vagrant-timezone
    end # subconfig
    ```

As can be seen, after starting the clients, the time zone of the virtual guest
systems is set to the time zone of the host system with the help of the
[vagrant-timezone](https://github.com/tmatilai/vagrant-timezone "Time Zone Configuration")
plugin. If the necessary `tzdata` package is not available on the guest
systems, it is installed with the help of the `install_tzdata` script.


## Starting the Management Node

Now follows the configuration for the Ansible management node named `master`.
The configuration is similar to the clients, except that the current directory
is mounted with the `synced_folder` option as directory `/vagrant` inside the
master node.

!!! Note "Vagrantfile: Master node definition"
    ```ruby
    # Box configuration for Ansible Management Node
    config.vm.define "master", primary: true do |subconfig|
      subconfig.vm.box = ansible_master['image']
      subconfig.vm.hostname = "master"
      subconfig.vm.provider "libvirt" do |libvirt, override|
        libvirt.cpus = ansible_master['cpus'] || 1
        libvirt.memory = ansible_master['memory'] || 512
        override.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_udp: false
      end # libvirt
      subconfig.vm.provider "virtualbox" do |vbox, override|
        vbox.cpus = ansible_master['cpus'] || 1
        vbox.memory = ansible_master['memory'] || 512
        vbox.gui = false
        vbox.name = ansible_master['vbox_name'] || 'Management Node'
        vbox.linked_clone = true
        vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
        vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        override.vm.network "private_network", type: "dhcp"
        override.vm.synced_folder ".", "/vagrant", type: "virtualbox", SharedFoldersEnableSymlinksCreate: false
        override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
          if hostname = (vm.ssh_info && vm.ssh_info[:host])
            `vagrant ssh -c "hostname -I"`.split()[1]
          end # if
        end # resolving_vm
      end # virtualbox
      subconfig.vm.provision "time zone data", type: "shell", path: "provisioning/scripts/install_tzdata"
      if Vagrant.has_plugin?("vagrant-timezone")
        subconfig.timezone.value = :host
      end # plugin vagrant-timezone
        ...
        # Ansible provisioning
        ...
      end
    ```

## Create Ansible inventory file

In the [config.yml](config_options.md "Configuration options") file, the
variable *vagrant_config* and the keyword *dynamic_inventory* are used to
specify whether the `inventory.ini` file, which is configured manually, or the
dynamically created inventory file `vagrant.ini` should be used:

* dynamic_inventory = *true*: `provisioning/vagrant.ini`
* dynamic_inventory = *false*: `provisioning/inventory.ini`

When a dynamic inventory file is used, the inventory group *[management_node]*
is created for the management node and the inventory group *[nodes]* for all other nodes.

For detailed information about Ansible inventory files, see [Build Your Inventory](https://docs.ansible.com/ansible/latest/network/getting_started/first_inventory.html).

!!! info "Vagrantfile: Define inventory file"
    ```ruby
    Vagrant.configure("2") do |config|
      if current_config['dynamic_inventory']
        # define dynamic inventory file
        ANSIBLE_INVENTORY_FILE = "provisioning/vagrant.ini"
        
        # create or overwrite inventory file
        File.open("#{ANSIBLE_INVENTORY_FILE}" ,'w') do | f |
          f.write "[management_node]\nlocalhost    ansible_connection=local ansible_host=127.0.0.1\n"
          f.write "\n"
          f.write "[management_node:vars]\n"
          f.write "ansible_python_interpreter=auto_silent\n"
          f.write "\n"
          f.write "[nodes]\n"
        end
      else
        # use the static inventory file
        ANSIBLE_INVENTORY_FILE = "provisioning/inventory.ini"
      end
      # ...
    end
    ```
In the client loop (see [Starting Ansible clients](#starting-ansible-clients)),
each node is added to the dynamic inventory file with the private SSH key to use.

!!! info "Vagrantfile: Add every node to the dynamic inventory file"
    ```ruby
    Vagrant.configure("2") do |config|
      ansible_client.each do |box|
        # If the nodes option is not set in `boxes.yml`, 1 is used by default.
        box_nodes = box["nodes"] || 1
        (1..box_nodes).each do |i|
          # ...
          # dynamically create the Ansible inventory file
          if current_config['dynamic_inventory']
            File.open("#{ANSIBLE_INVENTORY_FILE}" ,'a') do | f |
              f.write "#{box['hostname']}#{i} ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.#{box['hostname']}#{i}\n"
            end
          end
          # ...
        end
      end
      # ...
    end
    ```

Finally, we tell the ansible_local provisioner for the master node which
inventory file to use. The inventory file name is passed as an argument to the
Ansible playbook.

 
!!! Note "Vagrantfile: Set inventory file for playbook"
    ```ruby
    Vagrant.configure("2") do |config|
      # ...
      config.vm.define "master", primary: true do |subconfig|
        # ...
        subconfig.vm.provision "ansible_local" do |ansible|
          # ...
          ansible.inventory_path = "#{ANSIBLE_INVENTORY_FILE}"
          # pass environment variable to ansible, for example:
          # ANSIBLE_ARGS='--extra-vars "system_update=yes"' vagrant up
          ENV["ANSIBLE_ARGS"] = "--extra-vars \"ansible_inventory_file=/vagrant/#{ANSIBLE_INVENTORY_FILE}\""
          ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV['ANSIBLE_ARGS']
          # ...
        end # provision ansible_local
      end # subconfig master
      # ...
    end
    ```

## Provisioning with Ansible

The Ansible provisioner from the master node will setup the master and each
client node.

!!! Note "Vagrantfile: Ansible provisioning"
    ```ruby
    Vagrant.configure("2") do |config|
      # ...
      config.vm.define "master", primary: true do |subconfig|
        # ...
        # provisioning of each node with Ansible
        subconfig.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "provisioning/bootstrap.yml"
          # ansible.provisioning_path = "/vagrant"
          ansible.verbose = false
          # ansible.vault_password_file = "provisioning/.ansible_vault"
          # ansible.ask_vault_pass = true
          ansible.limit = "all" # or only "nodes" group, etc.
          ansible.install = true
          ansible.inventory_path = "#{ANSIBLE_INVENTORY_FILE}"
          # pass environment variable to ansible, for example:
          # ANSIBLE_ARGS='--extra-vars "system_update=yes"' vagrant up
          ENV["ANSIBLE_ARGS"] = "--extra-vars \"ansible_inventory_file=/vagrant/#{ANSIBLE_INVENTORY_FILE}\""
          ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV['ANSIBLE_ARGS']
        end # provision ansible_local
      end # subconfig master
      # ...
    end
    ```

The provisioner runs the `provisioning/bootstrap.yml` playbook using the
defined Ansible inventory file. The *ansible.limit* parameter defines the
systems for which Ansible should run. The value *all* specifies that all
systems including the management node should be provisioned with Ansible. If
the *ansible.install* parameter is set to *true*, the appropriate Ansible
package is installed on the management node before Ansible is run.

If you want to use the Ansible Vault feature with your roles, see the section
"[Use Ansible Vault feature](../test_ansible_roles.md#use-roles-with-ansible-vault-feature 'Ansible Vault feature')"
for details on enabling this feature.

!!! warning "Static inventory file provisioning/inventory.ini"
    Whenever you want to change the images or the number of nodes in
    `config.yml` and the *dynamic_inventory* parameter has the value *false*,
    you must manually change the static inventory file
    `provisioning/inventory.ini`. For example, if you change the number of
    Enterprise Linux 9 nodes from one to three, you should add two more nodes
    to the *[el9-nodes]* group to start and configure them, for example:

    ```yaml
    [management-node]
    localhost   ansible_connection=local ansible_host=127.0.0.1
    
    [el9-nodes]
    el9-node1	ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.el9-node1
    el9-node2	ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.el9-node2
    el9-node2	ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.el9-node3
    
    ...

    ```

    After that you can start and provision the new nodes with:
    ```bash
    vagrant up --provision
    ```

## The whole Vagrantfile

On the previous sections, the structure of the Vagrantfile was explained step by step.
As a result, here is the whole Vagrantfile.

!!! Note "Vagrantfile: The whole file"
    ```ruby
    # -*- mode: ruby -*-
    
    require 'yaml'
    # Include configuration from YAML file config.yml
    config = YAML.load_file(File.join(File.dirname(__FILE__), 'config.yml'))
    vagrant_config = config['vagrant_config'][config['vagrant_config']['env']]
    ansible_client = config['vagrant_boxes']['clients']
    ansible_master = config['vagrant_boxes']['master']
    
    # Set default Language for each virtual machine
    ENV["LANG"] = "C.UTF-8"
    
    Vagrant.configure(2) do |config|
    
      if vagrant_config['dynamic_inventory']
        # define dynamic inventory file
        ANSIBLE_INVENTORY_FILE = "provisioning/vagrant.ini"
        
        # create or overwrite inventory file
        File.open("#{ANSIBLE_INVENTORY_FILE}" ,'w') do | f |
          f.write "[management_node]\nlocalhost    ansible_connection=local ansible_host=127.0.0.1\n"
          f.write "\n"
          f.write "[management_node:vars]\n"
          f.write "ansible_python_interpreter=auto_silent\n"
          f.write "\n"
          f.write "[nodes]\n"
        end
      else
        # use static inventory file
        ANSIBLE_INVENTORY_FILE = "provisioning/inventory.ini"
      end
    
      # define the provider (virtualbox | libvirt)
      config.vm.provider vagrant_config['provider']
    
      # configure the hostmanager plugin
      config.hostmanager.enabled = vagrant_config['hostmanager_enabled']
      config.hostmanager.manage_guest = vagrant_config['hostmanager_manage_guest']
      config.hostmanager.manage_host = vagrant_config['hostmanager_manage_host']
      config.hostmanager.include_offline = vagrant_config['hostmanager_include_offline']
      config.hostmanager.ignore_private_ip = vagrant_config['hostmanager_ignore_private_ip']
    
      # Box Configuration for Ansible Clients
      ansible_client.each do |box|
        box_nodes = box["nodes"] || 1
        (1..box_nodes).each do |i|
          config.vm.define "#{box['hostname']}#{i}", autostart: box["autostart"] do |subconfig|
            subconfig.vm.box = box["image"]
            subconfig.vm.synced_folder ".", "/vagrant", disabled: true
            if Vagrant.has_plugin?("vagrant-vbguest")
              subconfig.vbguest.auto_update = vagrant_config['vbguest_auto_update']
            end # plugin
            subconfig.vm.hostname = "#{box['hostname']}#{i}"
            subconfig.vm.provider "libvirt" do |libvirt, override|
              libvirt.cpus = box["cpus"] || 1
              libvirt.memory = box["memory"] || 512
              libvirt.nested = false
            end # libvirt
            subconfig.vm.provider "virtualbox" do |vbox, override|
              # Don't install VirtualBox guest additions on Alpine Linux with
              # vagrant-vbguest plugin, because this doesn't work under Alpine Linux.
              if box["image"] =~ /alpine/
                if Vagrant.has_plugin?("vagrant-vbguest")
                  override.vbguest.auto_update = false
                end # plugin vagrant-vbguest
              end # if alpine
              vbox.gui = false
              vbox.cpus = box["cpus"] || 1
              vbox.memory = box["memory"] || 512
              vbox.name = "#{box['vbox_name']} #{i}"
              vbox.linked_clone = true
              vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
              vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
              override.vm.network "private_network", type: "dhcp"
              # get DHCP-assigned private network ip-address
              override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
                if hostname = (vm.ssh_info && vm.ssh_info[:host])
                  # detect private network ip address on every Linux OS
                  `vagrant ssh "#{box['hostname']}#{i}" -c  "ip addr show eth1|grep -v ':'|egrep -o '([0-9]+\.){3}[0-9]+'"`.split(' ')[0]
                end # if
              end # resolving_vm
            end # virtualbox
            subconfig.vm.provision "time zone data", type: "shell", path: "provisioning/scripts/install_tzdata"
            if Vagrant.has_plugin?("vagrant-timezone")
              subconfig.timezone.value = :host
            end # plugin vagrant-timezone
          end # subconfig
    
          # dynamically create the Ansible inventory file
          if vagrant_config['dynamic_inventory']
            File.open("#{ANSIBLE_INVENTORY_FILE}" ,'a') do | f |
              f.write "#{box['hostname']}#{i}    ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.#{box['hostname']}#{i}\n"
            end
          end
    
        end # each node
      end # each box
    
      if vagrant_config['dynamic_inventory']
        # finish inventory file
        File.open("#{ANSIBLE_INVENTORY_FILE}" ,'a') do | f |
          f.write "\n"
          f.write "[nodes:vars]\n"
          f.write "ansible_ssh_user=vagrant\n"
          f.write "ansible_python_interpreter=auto_silent\n"
        end
      end
    
      # Box configuration for Ansible Management Node
      config.vm.define "master", primary: true do |subconfig|
        subconfig.vm.box = ansible_master['image']
        subconfig.vm.hostname = "master"
        subconfig.vm.provider "libvirt" do |libvirt, override|
          libvirt.cpus = ansible_master['cpus'] || 1
          libvirt.memory = ansible_master['memory'] || 512
          override.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_udp: false
        end # libvirt
        subconfig.vm.provider "virtualbox" do |vbox, override|
          vbox.cpus = ansible_master['cpus'] || 1
          vbox.memory = ansible_master['memory'] || 512
          vbox.gui = false
          vbox.name = ansible_master['vbox_name'] || 'Management Node'
          vbox.linked_clone = true
          vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
          vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
          override.vm.network "private_network", type: "dhcp"
          override.vm.synced_folder ".", "/vagrant", type: "virtualbox", SharedFoldersEnableSymlinksCreate: false
          override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
            if hostname = (vm.ssh_info && vm.ssh_info[:host])
              `vagrant ssh -c "hostname -I"`.split()[1]
            end # if
          end # resolving_vm
        end # virtualbox
        subconfig.vm.provision "time zone data", type: "shell", path: "provisioning/scripts/install_tzdata"
        if Vagrant.has_plugin?("vagrant-timezone")
          subconfig.timezone.value = :host
        end # plugin vagrant-timezone
        subconfig.vm.provision "shell", inline: <<-SHELL
          echo -n                                       >  /etc/profile.d/ansible.sh
          echo 'export ANSIBLE_PYTHON_INTERPRETER=auto' >> /etc/profile.d/ansible.sh
        SHELL
        subconfig.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "provisioning/bootstrap.yml"
          # ansible.provisioning_path = "/vagrant"
          ansible.verbose = false
          # ansible.vault_password_file = "provisioning/.ansible_vault"
          # ansible.ask_vault_pass = true
          ansible.limit = "all" # or only "nodes" group, etc.
          ansible.install = true
          ## ansible.inventory_path = "provisioning/inventory.ini"
          ansible.inventory_path = "#{ANSIBLE_INVENTORY_FILE}"
          # pass environment variable to ansible, for example:
          # ANSIBLE_ARGS='--extra-vars "system_update=yes"' vagrant up
          ENV["ANSIBLE_ARGS"] = "--extra-vars \"ansible_inventory_file=/vagrant/#{ANSIBLE_INVENTORY_FILE}\""
          ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV['ANSIBLE_ARGS']
        end # provision ansible_local
      end # subconfig master
    
      # populate /etc/hosts on each node
      config.vm.provision :hostmanager
    
    end # config
    ```

