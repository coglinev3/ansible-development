# Working with the Ansible development environment

The starting point for the development of new Ansible roles is the directory
`/vagrant/provisioning` on the Ansible management node, called *master*. This
directory is shared between the virtual host system and the virtual guest
system, meaning that if the master node is reseted or deleted, Ansible roles and
playbooks will be preserved.

All actions described in this documentation will be performed as user *vagrant*
unless otherwise specified, which is the default user in a Vagrant environment.

Before you start with your work, ensure you have saved the environment. Then
you can go back if something goes wrong, see section *"Vagrant provider specific tasks"*
for save and restore the state of a virtual machine.

## Environment preparation

### Logon to the mangement node

Change to the project directory, start the virtual machines with vagrant if they are not
already started, and log in to the ansible management node via ssh.

```bash
cd <path_on_your_host_system>/ansible-development
vagrant up && vagrant ssh
```
If the machines are already running, the following command is enough to log into the Ansible management node.

```bash
vagrant ssh
```

### Check if the environment is ready

Control if the ansible environment is working. On the Ansible management node call:

```bash
ansible all -m ping
```
 
All clients that were up and running and dynamicaly added to the inventory file
`/vagrant/provisioning/vagrant.ini` should positiv respond to this command, for example:

```json
localhost | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
el9-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
fedora37-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
debian11-node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

```

!!! attention "Manualy configured inventory file"
    If you use a manualy configured inventory file instead of the dynamicaly
    configured inventory file (which is the default) then you have to specify
    your Ansible clients in `/vagrant/provisioning/inventory.ini` on your own.


## Create a new Ansible role

If you want to create a new Ansible role put the new role in directory
`/vagrant/provisioning/roles` on the Ansible management node. Remember the
folder `/vagrant` is a shared folder between the virtual host and the
virtual guest system.

```bash
cd /vagrant/provisioning/roles
ansible-galaxy init example-role
```

Then edit the main task of the role **example-role**.
```bash
vi example-role/tasks/main.yml
```

Insert your tasks, for example:

```yaml
---
# tasks file for example-role

- name: Show all IPv4 addresses
  debug:
    var: ansible_all_ipv4_addresses
```


Now create a playbook for the new role:

```bash
cd /vagrant/provisioning
vi example-playbook.yml
```

Put the new role in the playbook.

```yaml
---
# file: example-playbook.yml

- hosts: all
  roles:
    - { role: example-role }
```

## Test the new Ansible role

After that you can test the new role/playbook with:

```
./test-playbook example-playbook.yml
```

The shell script *test-playbook* will execute the following four steps:

1. Testing the role’s syntax.
1. Perform a dry run.
1. Run the role/playbook.
1. Run the role/playbook again, checking to make sure it's idempotent.

Thanks to [Jeff Geerling](https://www.jeffgeerling.com/) for his articles to
Ansible test strategies. He is also the author of [Ansible for DevOps](https://www.jeffgeerling.com/project/ansible-devops), a great book about Ansible.

## Use roles with Ansible Vault feature

If you want to use the [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
feature with your roles you have to create an Ansible Vault password file
`/vagrant/provisioning/.ansible_vault` and add the following line in *vagrant's*
`~/.bashrc` file: 

```bash
export ANSIBLE_VAULT_PASSWORD_FILE=/vagrant/provisioning/.ansible_vault
```

You can activate the new environment variable with:

```bash
. ~/.bashrc
```

!!! Note
    If you want to use Ansible's Vault feature while provisioning the whole 
    environment with `vagrant provision` or `vagrant up --provision` using
    Vagrant's local Ansible provsionier, then you should also uncomment the
    following line in the `Vagrantfile`:

        # ansible.vault_password_file = "provisioning/.ansible_vault"


