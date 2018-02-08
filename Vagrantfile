# -*- mode: ruby -*-
# vi: set ft=ruby :

# Parse options
require 'getoptlong'

opts = GetoptLong.new(
  [ '--el6-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--el7-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--ubuntu-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--debian-nodes', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--provision', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '-f', GetoptLong::OPTIONAL_ARGUMENT ],
)

# defaults
el6_nodes = 1
el7_nodes = 1
xenial64_nodes = 1
jessie64_nodes = 1

opts.each do |opt, arg|
  case opt
    when '--el6-nodes'
      el6_nodes = Integer(arg)
    when '--el7-nodes'
      el7_nodes = Integer(arg)
    when '--ubuntu-nodes'
      xenial64_nodes = Integer(arg)
    when '--debian-nodes'
      jessie64_nodes = Integer(arg)
  end
end

# Every Vagrant development environment requires a box. You can search for
# boxes at https://atlas.hashicorp.com/search.
EL6_IMAGE             = "centos/6"
EL7_IMAGE             = "centos/7"
UBUNTU_XENIAL64_IMAGE = "ubuntu/xenial64"
DEBIAN_JESSIE64_IMAGE = "debian/jessie64"
EL6_START             = true
EL7_START             = true
UBUNTU_XENIAL64_START = true
DEBIAN_JESSIE64_START = true
EL6_NODES             = el6_nodes
EL7_NODES             = el7_nodes
UBUNTU_XENIAL64_NODES = xenial64_nodes
DEBIAN_JESSIE64_NODES = jessie64_nodes


Vagrant.configure(2) do |config|

  # CentOS 7 Nodes
  (1..EL7_NODES).each do |i|
    config.vm.define "el7-node#{i}", autostart: EL7_START do |subconfig|
      subconfig.vm.box = EL7_IMAGE
      subconfig.vm.synced_folder ".", "/vagrant", disabled: true
      subconfig.vm.provider "virtualbox" do |vbox|
        # vbox.memory = "2048"
        # vbox.cpus = 2
        vbox.gui = false
        vbox.name = "Ansible EL7 - Node #{i}"
        vbox.linked_clone = true
      end
      subconfig.vm.hostname = "el7-node#{i}.example.org"
      subconfig.vm.network "private_network", ip: "192.168.56.#{i+240}"
      subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
    end
  end

  # CentOS 6 Nodes
  (1..EL6_NODES).each do |i|
    config.vm.define "el6-node#{i}", autostart: EL6_START do |subconfig|
      subconfig.vm.box = EL6_IMAGE
      subconfig.vm.synced_folder ".", "/vagrant", disabled: true
      subconfig.vm.provider "virtualbox" do |vbox|
        # vbox.memory = "2048"
        # vbox.cpus = 2
        vbox.gui = false
        vbox.name = "Ansible EL6 - Node #{i}"
        vbox.linked_clone = true
      end
      subconfig.vm.hostname = "el6-node#{i}.example.org"
      subconfig.vm.network "private_network", ip: "192.168.56.#{i+230}"
      subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
    end
  end

  # Ubuntu/xenial64 Nodes
  (1..UBUNTU_XENIAL64_NODES).each do |i|
    config.vm.define "ubuntu-node#{i}", autostart: UBUNTU_XENIAL64_START do |subconfig|
      subconfig.vm.box = UBUNTU_XENIAL64_IMAGE
      subconfig.vm.synced_folder ".", "/vagrant", disabled: true
      subconfig.vm.provider "virtualbox" do |vbox|
        # vbox.memory = "2048"
        # vbox.cpus = 2
        vbox.gui = false
        vbox.name = "Ansible Ubuntu (Xenial64) - Node #{i}"
        vbox.linked_clone = true
      end
      subconfig.vm.hostname = "ubuntu-node#{i}.example.org"
      subconfig.vm.network "private_network", ip: "192.168.56.#{i+220}"
      #subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
    end
  end

  # Debian/jessie64 Nodes
  (1..DEBIAN_JESSIE64_NODES).each do |i|
    config.vm.define "debian-node#{i}", autostart: DEBIAN_JESSIE64_START do |subconfig|
      subconfig.vm.box = DEBIAN_JESSIE64_IMAGE
      subconfig.vm.synced_folder ".", "/vagrant", disabled: true
      subconfig.vm.provider "virtualbox" do |vbox|
        # vbox.memory = "2048"
        # vbox.cpus = 2
        vbox.gui = false
        vbox.name = "Ansible Debian (Jessie64) - Node #{i}"
        vbox.linked_clone = true
      end
      subconfig.vm.hostname = "debian-node#{i}.example.org"
      subconfig.vm.network "private_network", ip: "192.168.56.#{i+210}"
      #subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
    end
  end

  # Ansible Management Node
  config.vm.define "master", primary: true do |subconfig|
    subconfig.vm.box = EL7_IMAGE
    subconfig.vm.synced_folder ".", "/vagrant", type: "virtualbox"
    subconfig.vm.provider "virtualbox" do |vbox|
      # vbox.memory = "2048"
      # vbox.cpus = 2
      vbox.gui = false
      vbox.name = "Ansible Master"
      vbox.linked_clone = true
    end
    subconfig.vm.hostname = "master.example.org"
    subconfig.vm.network "private_network", ip: "192.168.56.250"
    subconfig.vm.provision :shell, :inline => "sudo rm /etc/localtime && sudo ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime", run: "always"
#    subconfig.vm.provision "ansible_local" do |ansible|
#      ansible.playbook = "provisioning/site.yml"
#      # ansible.provisioning_path = "/vagrant"
#      ansible.verbose = false
#      ansible.vault_password_file = "provisioning/.ansible_vault"
#      # ansible.ask_vault_pass = true
#      ansible.limit = "all" # or only "nodes" group, etc.
#      ansible.install = true
#      ansible.inventory_path = "provisioning/inventory.ini"
#      # pass environment variable to ansible, for example:
#      # ANSIBLE_ARGS='--extra-vars "system_update=yes"' vagrant up
#      ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV['ANSIBLE_ARGS']
#    end
  end
end

# vim:set nu expandtab ts=2 sw=2 sts=2:
