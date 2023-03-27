# -*- mode: ruby -*-
# vi: set ft=ruby :

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

# vim:set nu expandtab ts=2 sw=2 sts=2:
