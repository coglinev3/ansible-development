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
        # Configure vbguesṫ auto updates
        subconfig.vbguest.auto_update = current_config['vbguest_auto_update']
        subconfig.vm.hostname = "#{box['hostname']}#{i}"
        subconfig.vm.provider "libvirt" do |libvirt, override|
          libvirt.cpus = 1
          libvirt.memory = "512"
          libvirt.nested = false
        end
        subconfig.vm.provider "virtualbox" do |vbox, override|
          # Don't install VirtualBox guest additions with vagrant-vbguest
          # plugin, because this doesn't work under Alpine Linux
          if box["image"] =~ /alpine/
            override.vbguest.auto_update = false
            override.vm.provision "shell",
              inline: "test -e /usr/sbin/dhclient || (echo nameserver 10.0.2.3 > /etc/resolv.conf && apk add --update dhclient)"
          end
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
    end
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

# vim:set nu expandtab ts=2 sw=2 sts=2:
