# Save all virtual machines

Sometimes it makes sense to save the current state of all virtual machines
before testing a new Ansible Role. If something goes wrong, you can go back to
the saved state.

On the virtual host system go to the project directory and do the following:

```bash
cd <path>/ansible-development
vagrant halt
vagrant snapshot save initial-setup
```

!!!Note
    Here *initial-setup* is the name for this snapshot. The snapshot list can be
    printed with:

        vagrant snapshot list


# Save a single machine

If only one single machine is to be saved, for example el6-node1, the following
commands can be executed:

```bash
vagrant halt el6-node1
vagrant snapshot save el6-node1 initial-setup
```


# Restore all machines

You can restore the saved state with:

```bash
vagrant snapshot restore initial-setup
```

# Restore a single machine

To restore a single machine, the name of the machine must explicitly be
specified.

```bash
vagrant snapshot restore el6-node1 initial-setup
```

Here the machine *el6-node1* will be restored to state *initial-setup*.

# Delete the hole Vagrant environment

If you have done your work then the next command stops the running machines
which Vagrant is managing and destroys all resources that were created during
the creation process. After running this command, your computer should be left
at a clean state, as if you never created the guest machines in the first place.

```bash
vagrant destroy -f
```

# Delete a single machine

The removal of a single machine is again shown using the example of *el6-node1*:

```bash
vagrant destroy el6-node1 -f
```
