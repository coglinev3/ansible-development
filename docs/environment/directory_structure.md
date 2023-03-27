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
    │   ├── test-playbook
    │   └── vagrant.ini
    ├── config.yml
    └── Vagrantfile
```

- **config.yml**: You can use [config.yml](config_options.md "Configuration options")
  to define the nodes to use and to make global settings for the Vagrant
  environment without modifying the Vagrantfile.
- **Vagrantfile**: Every Vagrant environment needs a [Vagrantfile](vagrantfile.md "Vagrantfile").
  The primary function of the Vagrantfile is to describe the type of machines
  required for a project, and how to configure and provision these machines. 
- **provisioning/bootstrap.yml**: [Ansible playbook](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html "Ansible pLaybooks")
  for initial provisioning of all nodes, including the master node.
- **provisioning/requirements.yml**: This file contains the required Ansible
  roles needed by `bootstrap.yml`.
- **provisioning/inventory.ini**: Manually configured [Ansible inventory](https://docs.ansible.com/ansible/latest/inventory_guide/index.html)
  file that is used only if the *dynamic_inventory* option is set to *false* in config.yml (see [Configuration Options](config_options.md "Config Options")).
- **provisioning/roles**: Directory for the [Ansible Roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html "Ansible Roles").
- **provisioning/test-playbook**: Shell script for testing new playbooks and
  roles
- **provisioning/vagrant.ini**: Dynamically configured Ansible inventory file
  used when `config.yml` has the *dynamic_inventory* option set to *true*, which
  is the default (see [Configuration Options](config_options.md "Config Options")).


!!! info "Mountpoint /vagrant"
    Within the ansible management node, called master, the host directory `<your_path>/ansible-development` is provided under the path `/vagrant`.
