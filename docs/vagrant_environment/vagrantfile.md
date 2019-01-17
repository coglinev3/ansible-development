# The whole Vagrantfile

This document describes the base of this development environment - the 
`Vagrantfile`. Every section of this file and how you can use it will be explaind
step by step.

!!! note "Vagrantfile: The whole file"
    ```ruby
    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    # Define options
    require 'getoptlong'

    opts = GetoptLong.new(
      [ '--el6-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--el7-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--ubuntu-trusty-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--ubuntu-xenial-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--ubuntu-bionic-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--ubuntu-cosmic-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--debian-jessie-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--debian-stretch-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--provision', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '-f', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '-h', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--box-version', GetoptLong::OPTIONAL_ARGUMENT ],
    )

    # Set defaults
    el6_nodes = 1
    el7_nodes = 1
    ubuntu_trusty_nodes = 1
    ubuntu_xenial_nodes = 1
    ubuntu_bionic_nodes = 1
    ubuntu_cosmic_nodes = 1
    debian_jessie_nodes = 1
    debian_stretch_nodes = 1

    # Parse options
    opts.each do |opt, arg|
      case opt
        when '--el6-nodes'
          el6_nodes = Integer(arg)
        when '--el7-nodes'
          el7_nodes = Integer(arg)
        when '--ubuntu-trusty-nodes'
          ubuntu_trusty_nodes = Integer(arg)
        when '--ubuntu-xenial-nodes'
          ubuntu_xenial_nodes = Integer(arg)
        when '--ubuntu-bionic-nodes'
          ubuntu_bionic_nodes = Integer(arg)
        when '--ubuntu-cosmic-nodes'
          ubuntu_cosmic_nodes = Integer(arg)
        when '--debian-jessie-nodes'
          debian_jessie_nodes = Integer(arg)
        when '--debian-stretch-nodes'
          debian_stretch_nodes = Integer(arg)
      end
    end

    # Every Vagrant development environment requires a box. You can search for
    # boxes at https://atlas.hashicorp.com/search.

    boxes = [
      { # Enterprise Linux 6 (RHEL6/CentOS6)
        :image => 'centos/6', :start => true, :nodes => el6_nodes, :ip_offset => 240,
        :hostname => 'el6-node', :vbox_name => 'EL6 - Node'
      },
      { # Enterprise Linux 7 (RHEL7/CentOS7)
        :image => 'centos/7', :start => true, :nodes => el7_nodes, :ip_offset => 230,
        :hostname => 'el7-node', :vbox_name => 'EL7 - Node'
      },
      { # Official Ubuntu Server 16.04 LTS (Xenial Xerus)
        :image => 'ubuntu/xenial64', :start => true, :nodes => ubuntu_xenial_nodes,
        :ip_offset => 220, :hostname => 'ubuntu-xenial-node', :vbox_name => 'Ubuntu (Xenial) - Node'
      },
      { # Official Ubuntu 18.04 LTS (Bionic Beaver)
        :image => 'ubuntu/bionic64', :start => true, :nodes => ubuntu_bionic_nodes,
        :ip_offset => 210, :hostname => 'ubuntu-bionic-node', :vbox_name => 'Ubuntu (Bionic) - Node'
      },
      { # Vanilla Debian 8 "Jessie"
        :image => 'debian/jessie64', :start => true, :nodes => debian_jessie_nodes,
        :ip_offset => 200, :hostname => 'debian-jessie-node', :vbox_name => 'Debian (Jessie) - Node'
      },
      { # Vanilla Debian 9 "Stretch"
        :image => 'debian/stretch64', :start => true, :nodes => debian_stretch_nodes,
        :ip_offset => 190, :hostname => 'debian-stretch-node', :vbox_name => 'Debian (Stretch) - Node'
      },
      { # Official Ubuntu Server 14.04 LTS (Trusty Tahr)
        :image => 'ubuntu/trusty64', :start => true, :nodes => ubuntu_trusty_nodes,
        :ip_offset => 180, :hostname => 'ubuntu-trusty-node', :vbox_name => 'Ubuntu (Trusty) - Node'
      },
      { # Official Ubuntu 18.10 (Cosmic Cuttlefish)
        :image => 'ubuntu/cosmic64', :start => true, :nodes => ubuntu_cosmic_nodes,
        :ip_offset => 170, :hostname => 'ubuntu-cosmic-node', :vbox_name => 'Ubuntu (Cosmic) - Node'
      },
    ]

    Vagrant.configure(2) do |config|

      # Box Configuration (Ansible Clients)
      boxes.each do |box|
        (1..box[:nodes]).each do |i|
          config.vm.define "#{box[:hostname]}#{i}", autostart: box[:start] do |subconfig|
            subconfig.vm.box = box[:image]
            subconfig.vm.synced_folder ".", "/vagrant", disabled: true
            subconfig.vm.provider "virtualbox" do |vbox|
              # vbox.memory = "2048"
              # vbox.cpus = 2
              vbox.gui = false
              vbox.name = "#{box[:vbox_name]} #{i}"
              vbox.linked_clone = true
              vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
            end
            subconfig.vm.hostname = "#{box[:hostname]}#{i}.example.org"
            subconfig.vm.network "private_network", ip: "192.168.56.#{i+box[:ip_offset]}"
            # The Vagrant timezone configuration doesn't work correctly.
            # Thæt's why I use the solution from Frédéric Henri, see: https://stackoverflow.com/questions/33939834/how-to-correct-system-clock-in-vagrant-automatically
            # You have to replace 'Europe/Berlin' with the timezone you want to set.
            subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
          end
        end
      end

      # Ansible Management Node Configuration
      config.vm.define "master", primary: true do |subconfig|
        subconfig.vm.box = 'centos/7'
        subconfig.vm.synced_folder ".", "/vagrant", type: "virtualbox", SharedFoldersEnableSymlinksCreate: false
        subconfig.vm.provider "virtualbox" do |vbox|
          vbox.memory = "4096"
          # vbox.cpus = 2
          vbox.gui = false
          vbox.name = "Management Node"
          vbox.linked_clone = true
          vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
        end
        subconfig.vm.hostname = "master.example.org"
        subconfig.vm.network "private_network", ip: "192.168.56.250"
        # The Vagrant timezone configuration doesn't work correctly.
        # Thæt's why I use the solution from Frédéric Henri, see: https://stackoverflow.com/questions/33939834/how-to-correct-system-clock-in-vagrant-automatically
        # You have to replace 'Europe/Berlin' with the timezone you want to set.
        subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
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

# Options

If necessary, several nodes can be started for each OS type, for example to be
able to map test, staging and production environments or different regions such
as North America, Europe and Asia. This requires module 'getoptlong', with which
various options can be passed to Vagrant.

!!! note "Vagrantfile: Define Options"
    ```ruby
    require 'getoptlong'

    # Define Options
    opts = GetoptLong.new(
      [ '--el6-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--el7-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--ubuntu-trusty-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--ubuntu-xenial-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--ubuntu-bionic-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--ubuntu-cosmic-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--debian-jessie-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--debian-stretch-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--provision', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '-f', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '-h', GetoptLong::OPTIONAL_ARGUMENT ],
      [ '--box-version', GetoptLong::OPTIONAL_ARGUMENT ],
    )
    ```

Since the 'getoptlong' module parses all command line options that are passed to
the `vagrant` command, it also requires default options be specified such as
`-h` for help or `--provision` for provisioning when defining *GetoptLong.new*.
Otherwise an *unknown option* error message will occur.

The transferred options are assigned to variables within the `Vagrantfile`. So
that the variables can also be used if the associated option was not specified
on the command line, default values must first be defined for these variables.


!!! note "Vagrantfile: Set Defaults"
    ```ruby
    # Set defaults
    el6_nodes = 1
    el7_nodes = 1
    ubuntu_trusty_nodes = 1
    ubuntu_xenial_nodes = 1
    ubuntu_bionic_nodes = 1
    ubuntu_cosmic_nodes = 1
    debian_jessie_nodes = 1
    debian_stretch_nodes = 1
    ```
We see that one node is started by default for each operating system. The
default value can also be set to zero if a particular operating system is not
needed.

Now the options can be parsed. Depending on whether a particular option has been
specified or not, the value specified on the command line or the default value
will be used.

!!! note "Vagrantfile: Parse Options"
    ```ruby
    # Parse options
    opts.each do |opt, arg|
      case opt
        when '--el6-nodes'
          el6_nodes = Integer(arg)
        when '--el7-nodes'
          el7_nodes = Integer(arg)
        when '--ubuntu-trusty-nodes'
          ubuntu_trusty_nodes = Integer(arg)
        when '--ubuntu-xenial-nodes'
          ubuntu_xenial_nodes = Integer(arg)
        when '--ubuntu-bionic-nodes'
          ubuntu_bionic_nodes = Integer(arg)
        when '--ubuntu-cosmic-nodes'
          ubuntu_cosmic_nodes = Integer(arg)
        when '--debian-jessie-nodes'
          debian_jessie_nodes = Integer(arg)
        when '--debian-stretch-nodes'
          debian_stretch_nodes = Integer(arg)
      end
    end
    ```

**Examples of command line calls**

Start `3` CentOS 6 nodes and `3` CentOS 7 nodes:

```bash
vagrant --el6-nodes=3 --el7-nodes=3 up
```

Restart CentOS 6 node 2 of 3:

```bash
vagrant --el6-nodes=3 reload el6-node2
```

Destroy CentOS 6 node 2 of 3:

```bash
vagrant --el6-nodes=3 destroy el6-node2 -f
```

!!! attention
    If a command line option is used to handle more nodes than defined by
    default, such as
    ```
    --el6-nodes=3
    ```
    then this option must always be
    specified with every vagrant command, just after the word vagrant.
    Otherwise, an error message occurs because the specified node is not
    found. For example:
    ```bash
    vagrant restart el6-node2
    ```
    will not work, because the default for the variable *el6_nodes* is set to 1
    and therefore the node *el6-node2* can not be found.

# Define Boxes

Every Vagrant development environment requires a box. You can search for boxes
at [Vagrant Cloud](https://app.vagrantup.com/boxes/search "Discover Vagrant Boxes").
The `Vagrantfile` contains a variable named boxes, which defines all vargrant
box/images used in this environment.

!!! note "Vagrantfile: Box Definitions"
    ```bash
    boxes = [
      {
        # Enterprise Linux 6 (RHEL6/CentOS6)
        :image => 'centos/6',
        :start => true,
        :ip_offset => 240,
        :nodes => el6_nodes,
        :hostname => 'el6-node',
        :vbox_name => 'EL6 - Node'
      },
      {
        # Official Ubuntu Server 16.04 LTS (Xenial Xerus)
        :image => 'ubuntu/xenial64',
        :start => true,
        :ip_offset => 220, 
        :nodes => ubuntu_xenial_nodes,
        :hostname => 'ubuntu-xenial-node', 
        :vbox_name => 'Ubuntu (Xenial) - Node'
      },

      ...
    ]
    ```

You can add your own Vagrant boxes or delete other boxes here if you need.

Attribute description:

- **image**: A official Vagrant Box/Image, see  [Vagrant Cloud](https://app.vagrantup.com/)
- **start**: Should the virtual maschien be started on `vagrant up`
- **ip_offset**: The offset for the last 3 ip address digits, e.g. 240 will be convertet to *240* + *node number*. PLease pay attention to the ip_offset attribute to avoid ip address conflicts.
- **nodes**: The number of nodes [1-9] for this os.
- **hostname**: The hostname prefix. The real hostname will be *hostname* + *node number*
- **vbox_name**: Then name prefix in Oracles Virtualbox GUI

!!! attention
    If you define new machines, you must also define them in the Ansible
    inventory file `/vagrant/provisioning/inventory.ini` in order to use the
    machine with Ansible.


# Start up Ansible clients

Next, Ansible clients are started using the previously defined boxes. This
requires two loops:

- one outer loop for all box definitions (operating systems)
- one inner loop for every node of a operating system type

Within a `Vagrantfile` the construct `array.each do |index|` or 
`(1..#end).each do |index|` will be used.

!!! note "Vagrantfile: Loops for boxes and nodes"
    ```ruby
    boxes.each do |box|
      (1..box[:nodes]).each do |i|

      ...
      
      end
    end
    ```

After that comes the virtual machine definition for each node, e. g. which image should be used, the hostname,
private networks, the name within VirtualBox, how many memory should be useѕ ans so on. 

Then, for each node, the definition of the virtual machine comes:

- which image to use, 
- the hostname of the virtual machine, 
- the definition of the private networks, 
- how many stores to use, 
- the name to display in VirtualBox, 
- and so on.

!!! note "Vagrantfile: virtual machine definition"
    ```ruby
    # Definition for each Ansible client
    config.vm.define "#{box[:hostname]}#{i}", autostart: box[:start] do |subconfig|
      subconfig.vm.box = box[:image]
      subconfig.vm.synced_folder ".", "/vagrant", disabled: true
      subconfig.vm.provider "virtualbox" do |vbox|
        # vbox.memory = "2048"
        # vbox.cpus = 2
        vbox.gui = false
        vbox.name = "#{box[:vbox_name]} #{i}"
        vbox.linked_clone = true
        vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
      end
      subconfig.vm.hostname = "#{box[:hostname]}#{i}.example.org"
      subconfig.vm.network "private_network", ip: "192.168.56.#{i+box[:ip_offset]}"
      subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
    end 
    ```

!!! attention
    The Vagrant timezone configuration doesn't work correctly.  That's why I
    use the solution from [Frédéric Henri](https://stackoverflow.com/users/4296747/fr%c3%a9d%c3%a9ric-henri)
    with the shell provisioner, see [stackoverflow](https://stackoverflow.com/questions/33939834/how-to-correct-system-clock-in-vagrant-automatically "How to correct system clock in vagrant automatically")
    You have to replace 'Europe/Berlin' with the timezone you want to set.


# Start up Ansible master with provisioning of clients

Text: 

!!! note "Vagrantfile: Master node definition"
    ```ruby
    # Ansible Management Node Configuration
    config.vm.define "master", primary: true do |subconfig|
      subconfig.vm.box = 'centos/7'
      subconfig.vm.synced_folder ".", "/vagrant", type: "virtualbox", SharedFoldersEnableSymlinksCreate: false
      subconfig.vm.provider "virtualbox" do |vbox|
        vbox.memory = "4096"
        # vbox.cpus = 2
        vbox.gui = false
        vbox.name = "Management Node"
        vbox.linked_clone = true
        vbox.customize ["modifyvm", :id, "--groups", "/Ansible"]
      end
      subconfig.vm.hostname = "master.example.org"
      subconfig.vm.network "private_network", ip: "192.168.56.250"
      ...
      # Provisioning
      ...
    end
    ```

