# Working with the Ansible development environment

The starting point for the development of new Ansible roles is the directory
`/vagrant/provisioning` on the Ansible management node, called *master*. This
directory is shared between the virtual host system and the virtual guest
system, meaning that if the master node is reseted or deleted, Ansible roles and
playbooks will be preserved.

All actions described in this documentation will be performed as user *vagrant*
unless otherwise specified, which is the default user in a Vagrant environment.

Before you start with your work, ensure you have saved the environment. Then
you can go back if something goes wrong, see section 
"[Save all virtual machines](/save_restore_delete/ "Save, Restore and Delete")".

## Logon to the mangement node

Change to the project directory, start the environment if it is not already
started, and log in to the ansible management node via ssh.

```bash
cd <path>/ansible-development
vagrant up && vagrant ssh
```

## Environment preparation on Ansible master node

On the management node open the file `/etc/ansible/ansible.cfg` and check the
variables *inventory* and *roles_path*:

```ini
inventory  = /vagrant/provisioning/inventory.ini
roles_path = /vagrant/provisioning/roles:/etc/ansible/roles:/usr/share/ansible/roles
```

If you want to use the [Ansible Vault](http://docs.ansible.com/ansible/2.4/vault.html)
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
    Vagrant's local Ansible privsionier, then you should also uncomment the
    following line in `/vagrant/Vagrantfile`:

        # ansible.vault_password_file = "provisioning/.ansible_vault"


## Check if the environment is ready

Control if the ansible environment is working with:

```bash
ansible all -m ping
```
 
All clients that were up and running and specified in the inventory file
`/vagrant/provisioning/inventory.ini` should positiv respond to this command.


## Create a new Ansible role

If you want to create a new Ansible role put the new role in directory
`/vagrant/provisioning/roles` on the Ansible management node. Remember the
folder `/vagrant` is a shared folder between the virtual host and the
virtual guest system.

```bash
cd /vagrant/provisioning/roles
ansible-galaxy init new-role-name
```

Then edit the new role. After finishing the new role create a *playbook.yml*
file in directory `/vagrant/provisioning`

```bash
cd /vagrant/provisioning
vi playbook.yml
```

Put the new role in the playbook.

```yaml
---
# file: playbook.yml

- hosts: all
  become: yes
  roles:
    - { role: new-role-name }
```

After that you can test the new role/playbook with:

```
./test-playbook playbook.yml
```

The shell script *test-playbook* will execute the following four steps:

1. Testing the role’s syntax.
1. Perform a dry run.
1. Run the role/playbook.
1. Run the role/playbook again, checking to make sure it's idempotent.

Thanks to [Jeff Geerling](https://www.jeffgeerling.com/) for his articles to
Ansible test strategies. He is also the author of [Ansible for DevOps](https://www.jeffgeerling.com/project/ansible-devops), a great book about Ansible.

