# -*- mode: ruby -*-
# vi: set ft=ruby :

# Parse options
require 'getoptlong'

opts = GetoptLong.new(
  [ '--el6-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--el7-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--ubuntu-bionic-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--ubuntu-cosmic-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--ubuntu-trusty-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--ubuntu-xenial-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--debian-jessie-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--debian-stretch-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--provision', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--no-provision', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '-f', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '-h', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--box-version', GetoptLong::OPTIONAL_ARGUMENT ],
)

# Set defaults
el6_nodes = 1
el7_nodes = 1
ubuntu_bionic_nodes = 1
ubuntu_cosmic_nodes = 1
ubuntu_trusty_nodes = 1
ubuntu_xenial_nodes = 1
debian_jessie_nodes = 1
debian_stretch_nodes = 1

# Set options
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
  { # Official Ubuntu 18.10 (Cosmic Cuttlefish)
    :image => 'ubuntu/cosmic64', :start => true, :nodes => ubuntu_cosmic_nodes,
    :ip_offset => 170, :hostname => 'ubuntu-cosmic-node', :vbox_name => 'Ubuntu (Cosmic) - Node'
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
          # Disconnecting the serial port solves the slow boot problem of some
          # distributions.
          ## vbox.customize ["modifyvm", :id, "--uartmode1", "disconnected"]
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

# vim:set nu expandtab ts=2 sw=2 sts=2:
