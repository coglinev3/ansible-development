# Directory structure

The following files and directories are the main components of this Multi-VM Vagrant environment.
```
.../ansible-development/
    ├── provisioning/
    │   ├── roles/
    │   │   ├── coglinev3.ansible-common
    │   │   ├── coglinev3.vagrant-ansible-init
    │   │   │   ...
    │   │   └── your_ansible_role 
    │   ├── bootstrap.yml
    │   ├── inventory.ini
    │   ├── requirements.yml
    │   └── test-playbook
    ├── boxes.yml
    ├── config.yml
    └── Vagrantfile
```
Within the ansible management node, called master, the host directory `<your_path>/ansible-development` is provided under the path `/vagrant`.

- **boxes.yml**: Contains the Box definitions für this Multi-VM environment.
- **config.yml**: With config.yml you can configure some options for your Vagrant environment without editing the Vagrantfile.
- **Vagrantfile**: Every Vagrant environment needs a [Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/ "Vagrantfile"). The primary function of the Vagrantfile is to describe the type of machines required for a project, and how to configure and provision these machines. 
- **provisioning/requirements.yml**: Requirements for installing needed Ansible roles from [Ansible Galaxy](https://galaxy.ansible.com/ "Ansible Galaxy is Ansible’s official hub for sharing Ansible content.") (or your own git repository)
- **provisioning/inventory.ini**: [Ansible Inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html "Ansible Inventory file")
file, which is only used if you set option `dynamic_inventory` to `false` (see
section [Environment type specific configuration](#environment-type-specific-configuration))
in file *config.yml*
- **provisioning/bootstrap.yml**: [Ansible Playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html "Ansible PLaybook") for the initial deployment of all nodes, including the master node
- **provisioning/roles**: Directory for the [Ansible Roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html "Ansible Roles")
- **provisioning/test-playbook**: Shell script for testing new playbooks and
roles

!!! Note "Vagrantfile"
    The heart of this environment is the `Vagrantfile`. Every section of this file and how you can use it will now be explaind step by step int the next sections.

!!! attention "Don't change the Vagrantfile"
    Be careful when changing the Vagrantfile. A small mistake and the
    environment is unusable. In most cases, there is no need to change the
    Vagrantfile. Instead, use the options in `boxes.yml` and `config.yml`.

# Define Vagrant Boxes

Every Vagrant development environment requires a box. You can search for boxes
at [Vagrant Cloud](https://app.vagrantup.com/boxes/search "Discover Vagrant Boxes").
The variable `boxes` gets the box definitions from the external YAML file called `boxes.yml`

!!! Note "Vagrantfile: Include Box Definitions"
    ```bash
    # Include box configuration from YAML file boxes.yml
    require 'yaml'
    boxes = YAML.load_file(File.join(File.dirname(__FILE__), 'boxes.yml'))

    ```
Here's a snippet from the file `boxes.yml`

!!! Note "Content of boxes.yml"
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

You can add your own Vagrant boxes or remove other boxes here if you need.

Attribute description:

- **image**: A official Vagrant Box/Image, see  [Vagrant Cloud](https://app.vagrantup.com/ "HashiCorp Vagrant Cloud")
- **start**: Defines if the virtual machine should be started on `vagrant up`
- **hostname**: The hostname prefix. The real hostname will be *hostname* + *node number*
- **vbox_name**: The name prefix in Oracles VirtualBox GUI. The real name will be *vbox_name* + *node_number*
- **nodes**: The number of nodes for this Box/Image

!!! attention
    If you are not using the dynamic inventory file
    `/vagrant/provisioning/vagrant.ini` (see [below](#environment-type-specific-configuration))
    and you are defining new machines, you must also add them to the Ansible
    inventory file `/vagrant/provisioning/inventory.ini` to use these machines
    with Ansible ,


# Environment type specific configuration

A few settings can be made using `config.yml` without changing the
Vagrantfile.

!!! Note "Vagrantfile: Include environment configuration"
    ```bash
    # Include environment configuration from YAML file config.yml
    require 'yaml'
    config = YAML.load_file(File.join(File.dirname(__FILE__), 'config.yml'))
    # Get current environment from variable 'env'
    current_config = config['vagrant_config'][config['vagrant_config']['env']]
    ```

The variable *env* in `config.yml` controls which environment type is currently
used.

!!! Note "config.yml"
    ```yaml
    ---
    vagrant_config:
      env: 'production'
      staging:
        hostmanager_manage_host: true
        hostmanager_include_offline: true
        vbguest_auto_update: true
        dynamic_inventory: true
      production:
        hostmanager_manage_host: false
        hostmanager_include_offline: true
        vbguest_auto_update: false
        dynamic_inventory: true
    ```

With `config.yml` you can define different environments with different values for:

- **hostmanager_manage_host**: Update the `hosts` file on the hosts's machine.
- **hostmanager_include_offline**: If the attribute is set to `true`, boxes that are up or have a private ip configured will be added to the hosts file.
- **vbguest_auto_update**: Set vbguest_auto_update to `false`, if you do NOT want to check the correct version of VirtualBox Guest Additions on the guest system when booting the machine
- **dynamic_inventory**: If `dynamic_inventory` is set to *true* the file `provisioning/vagrant.ini` will be uses as the Ansible inventory file. It will be dynamicly build, which is recommended. This way, you can add or remove new hosts in [boxes.yml](#define-vagrant-boxes) without having to worry about customizing the Ansible inventory file too. If `dynamic_inventory` is set to *false* `provisioning/inventory.ini` will be used as inventory file. You have to change this file manualy every time you add or remove new hosts (Ansible Clients). 


!!! attention "Update the hosts file on the hosts's machine."
    If you want to manage the hosts file on the host's computer, ensure that you have write access to the hosts file, before you set
    ```bash
    hostmanager_manage_host: false
    ```

    **Linux:**

    To avoid being asked for the password every time the hosts file is updated,
    enable passwordless sudo for the specific command that hostmanager uses to
    update the hosts file.

    * Add the following snippet to the sudoers file (e.g. `/etc/sudoers.d/vagrant_hostmanager`):
    ```bash
    Cmnd_Alias VAGRANT_HOSTMANAGER_UPDATE = /bin/cp <home-directory>/.vagrant.d/tmp/hosts.local /etc/hosts
    %<admin-group> ALL=(root) NOPASSWD: VAGRANT_HOSTMANAGER_UPDATE
    ```
    Replace <home-directory> with your actual home directory (e.g. /home/joe) and `<admin-group>`
    with the group that is used by the system for sudo access (usually sudo on
    Debian/Ubuntu systems and wheel on Fedora/Red Hat systems).
    
    * If necessary, add yourself to the `<admin-group>`:
    ```bash
    usermod -aG <admin-group> <user-name>
    ```
    Replace <admin-group> with the group that is used by the system for sudo
    access (see above) and <user-name> with you user name.


    **Windows:**

    Hostmanager will detect Windows guests and hosts and use the appropriate
    path for the hosts file: `%WINDIR%\System32\drivers\etc\hosts`

    By default on a Windows host, the hosts file is not writable without
    elevated privileges. If hostmanager detects that it cannot overwrite the file,
    it will attempt to do so with elevated privileges, causing the
    UAC prompt to appear.
    To avoid the UAC prompt, open `%WINDIR%\System32\drivers\etc\` in
    Explorer, right-click the `hosts` file, go to Properties > Security > Edit
    and give your user Modify permission.

    UAC limitations:

    Due to limitations caused by UAC, cancelling out of the UAC prompt will not cause any
    visible errors, however the hosts file will not be updated.

# Order of Vagrant providers


This Vagrant environment supports two providers: VirtualBox and libvirt.
VirtualBox is the default provider, so it comes first.

!!! Note "Vagrantfile: Order of providers"
    ```bash
      config.vm.provider "virtualbox"
      config.vm.provider "libvirt"
    
    ```
!!! attention
    You can't use both providers at the same time.

If you want to use libvirt you can change the order in the Vagrantfile or
specіfy libvirt via environment variable `VAGRANT_DEFAULT_PROVIDER`,
for example:
```bash
VAGRANT_DEFAULT_PROVIDER=libvirt vagrant up
```
Another way is to specify the provider with the option `--provider`
```bash
vagrant up --provider libvirt
```

# Configure the hostmanager plugin

[vagrant-hostmanager](https://github.com/devopsgroup-io/vagrant-hostmanager) is
a Vagrant plugin that manages the hosts file on guest machines (and optionally
on the host). Its goal is to enable name resolution of multi-machine environments.

!!! Note "Vagrantfile: Configure hostmanager plugin"
    ```bash
      config.hostmanager.enabled = false
      config.hostmanager.manage_guest = true
      config.hostmanager.ignore_private_ip = false
    ```
Configuration options

- **hostmanager.enabled**: Starting at Vagrant version 1.5.0, vagrant up runs hostmanager before any provisioning occurs. If you would like hostmanager to run after or during your provisioning stage, you can use hostmanager as a provisioner. This allows you to use the provisioning order to ensure that hostmanager runs when desired. The provisioner will collect hosts from boxes with the same provider as the running box.
- **hostmanager.manage_guest**: Update the `hosts` file on the guest machines.
- **hostmanager.ignore_private_ip**: A machine's IP address is defined by either the static IP for a private network configuration or by the SSH host configuration. To disable using the private network IP address, set config.hostmanager.ignore_private_ip to true.

!!! Note "Use Vagrant plugin hostmanager as provisioner"
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

!!! Note "Vagrantfile: Loops for boxes and nodes"
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

!!! Note "Vagrantfile: definition for each node"
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

!!! attention "Timezone problem"
    The Vagrant timezone configuration doesn't work correctly.  That's why I
    use the solution from [Frédéric Henri](https://stackoverflow.com/users/4296747/fr%c3%a9d%c3%a9ric-henri)
    with the shell provisioner, see [stackoverflow](https://stackoverflow.com/questions/33939834/how-to-correct-system-clock-in-vagrant-automatically "How to correct system clock in vagrant automatically")
    You have to replace 'Europe/Berlin' with the timezone you want to set.


# Start up Ansible Management Node (master)

Now comes the configuration of the Ansible management node called `master`.
It is the same configuration as for the clients, except that here CentOS 7
is explicitly specified as operating system and the current directory is
mounted with the option synced_folder as directory `/vagrant` within the master
node. Finally, Ansible runs to automatically provision all virtual machines.
The Ansible Provisioner will then be explained in a separate section.

!!! Note "Vagrantfile: Master node definition"
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
        # configuration for provider libvirt
        subconfig.vm.provider "libvirt" do |libvirt, override|
          libvirt.memory = "1024"
          # synced folder via nfs requires the NFS Kernel Server on the host system
          override.vm.synced_folder ".", "/vagrant", type: "nfs"
          override.hostmanager.ip_resolver = proc do |vm, resolving_vm|
            if hostname = (vm.ssh_info && vm.ssh_info[:host])
              `vagrant ssh -c "hostname -I"`.split()[0]
            end
          end
        end
        # configuration for provider VirtualBox
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
        ...
        # Ansible provisioning
        ...
      end
    ```

# Define and create Ansible inventory file

First define which inventory file to use:

* dynamic_inventory = **true**: `provisioning/vagrant.ini`
* dynamic_inventory = **false**: `provisioning/inventory.ini`

If a dynymic inventory file is used create one inventory group for the
management node and one for all other nodes.

!!! Note "Define inventory file"
    ```bash
      if current_config['dynamic_inventory']
        # define dynamic inventory file
        ANSIBLE_INVENTORY_FILE = "provisioning/vagrant.ini"
        
        # create or overwrite inventory file
        File.open("#{ANSIBLE_INVENTORY_FILE}" ,'w') do | f |
          f.write "[management_node]\nlocalhost    ansible_connection=local ansible_host=127.0.0.1\n"
          f.write "\n"
          f.write "[nodes]\n"
        end
      else
        ANSIBLE_INVENTORY_FILE = "provisioning/inventory.ini"
      end
    ```

In the Loop for the Ansible clients (see [Start Ansible clients](#start-up-ansible-clients))
add every node to the dynamic inventory file.

!!! Note "Add every node to the dynamic inventory file"
    ```bash
    # dynamically create the Ansible inventory file
    if current_config['dynamic_inventory']
      File.open("#{ANSIBLE_INVENTORY_FILE}" ,'a') do | f |
        f.write "#{box['hostname']}#{i}    ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.#{box['hostname']}#{i}\n"
      end
    end
    ```

Finally, tell the ansible_local provisioner for the master node which inventory
file should be used and pass the name of the inventory file as an argument for
the Ansible Playbook.

 
!!! Note "Add every node to the dynamic inventory file"
    ```ruby
    subconfig.vm.provision "ansible_local" do |ansible|
      ...
      ansible.inventory_path = "#{ANSIBLE_INVENTORY_FILE}"
      # pass environment variable to ansible, for example:
      # ANSIBLE_ARGS='--extra-vars "system_update=yes"' vagrant up
      ENV["ANSIBLE_ARGS"] = "--extra-vars \"ansible_inventory_file=/vagrant/#{ANSIBLE_INVENTORY_FILE}\""
      ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV['ANSIBLE_ARGS']
      ...
    end # provision ansible_local
    ```

# Provisioning with Ansible

The Ansible provisioner from the master node will setup each node.

!!! Note "Vagrantfile: Ansible provisioning"
    ```ruby
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
    ```

The provisioner executes the playbook `bootstrap.yml` from the directory `provisioning`
while using `inventory.ini`as Ansible inventory file.
If you want to use the Ansible Vault feature with your roles see section "[Use roles with Ansible Vault feature](test_ansible_roles.md#use-roles-with-ansible-vault-feature 'Ansible Vault feature')" for details to activate this feature.

!!! attention "provisioning/inventory.ini"
    If you change the images or the number of nodes in `boxes.yml` and you
    won't use the dynamicaly created inventory file, you must also change the
    ansible inventory file` inventory.ini`.
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

    After that you can start and provision then new nodes with:
    ```bash
    vagrant up --provision
    ```

# The whole Vagrantfile

On the previous sections, the structure of the Vagrant file was explained step by step.
As result here is the whole Vagrantfile.

!!! Note "Vagrantfile: The whole file"
    ```ruby
    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    
    # Include box configuration from YAML file boxes.yml
    require 'yaml'
    boxes = YAML.load_file(File.join(File.dirname(__FILE__), 'boxes.yml'))
    config = YAML.load_file(File.join(File.dirname(__FILE__), 'config.yml'))
    current_config = config['vagrant_config'][config['vagrant_config']['env']]
    
    
    # set defaul Language for each virtual machine to C
    ENV["LANG"] = "C"
    
    Vagrant.configure(2) do |config|
    
      if current_config['dynamic_inventory']
        # define dynamic inventory file
        ANSIBLE_INVENTORY_FILE = "provisioning/vagrant.ini"
        
        # create or overwrite inventory file
        File.open("#{ANSIBLE_INVENTORY_FILE}" ,'w') do | f |
          f.write "[management_node]\nlocalhost    ansible_connection=local ansible_host=127.0.0.1\n"
          f.write "\n"
          f.write "[nodes]\n"
        end
      else
        ANSIBLE_INVENTORY_FILE = "provisioning/inventory.ini"
      end
    
      # define the order of providers 
      config.vm.provider "virtualbox"
      config.vm.provider "libvirt"
    
      # configure the hostmanager plugin
      config.hostmanager.enabled = false
      config.hostmanager.manage_guest = true
      config.hostmanager.manage_host = current_config['hostmanager_manage_host']
      config.hostmanager.include_offline = current_config['hostmanager_include_offline']
      config.hostmanager.ignore_private_ip = false
    
    
      # Box Configuration for Ansible Clients
      boxes.each do |box|
        (1..box["nodes"]).each do |i|
          config.vm.define "#{box['hostname']}#{i}", autostart: box["start"] do |subconfig|
            subconfig.vm.box = box["image"]
            subconfig.vm.synced_folder ".", "/vagrant", disabled: true
            subconfig.vbguest.auto_update = current_config['vbguest_auto_update']
            subconfig.vm.hostname = "#{box['hostname']}#{i}"
            subconfig.vm.provider "libvirt" do |libvirt, override|
              libvirt.cpus = 1
              libvirt.memory = "512"
              libvirt.nested = false
            end # libvirt
            subconfig.vm.provider "virtualbox" do |vbox, override|
              # Don't install VirtualBox guest additions with vagrant-vbguest
              # plugin, because this doesn't work under Alpine Linux
              if box["image"] =~ /alpine/
                override.vbguest.auto_update = false
                override.vm.provision "shell",
                  inline: "test -e /usr/sbin/dhclient || (echo nameserver 10.0.2.3 > /etc/resolv.conf && apk add --update dhclient)"
              end # if alpine
              vbox.gui = false
              vbox.memory = 512
              vbox.cpus = 1
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
            # The Vagrant timezone configuration doesn't work correctly.
            # That's why I use the solution from Frédéric Henri, see: https://stackoverflow.com/questions/33939834/how-to-correct-system-clock-in-vagrant-automatically
            # You have to replace 'Europe/Berlin' with the timezone you want to set.
            subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime"
          end # subconfig
    
          # dynamically create the Ansible inventory file
          if current_config['dynamic_inventory']
            File.open("#{ANSIBLE_INVENTORY_FILE}" ,'a') do | f |
              f.write "#{box['hostname']}#{i}    ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa.#{box['hostname']}#{i}\n"
            end
          end
    
        end # each node
      end # each box
    
      if current_config['dynamic_inventory']
        # finish inventory file
        File.open("#{ANSIBLE_INVENTORY_FILE}" ,'a') do | f |
          f.write "\n"
          f.write "[nodes:vars]\nansible_ssh_user=vagrant\n"
        end
      end
    
      # Box configuration for Ansible Management Node
      config.vm.define "master", primary: true do |subconfig|
        subconfig.vm.box = 'generic/centos8'
        subconfig.vm.hostname = "master"
        subconfig.vm.provider "libvirt" do |libvirt, override|
          libvirt.memory = "1024"
          override.vm.synced_folder ".", "/vagrant", type: "nfs"
        end # libvirt
        subconfig.vm.provider "virtualbox" do |vbox, override|
          vbox.memory = "1024"
          vbox.gui = false
          vbox.name = "Management Node"
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
        subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime"
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

