# Ansible notes for ShakeAlert implementation

## Install

It's much easier to install ansible using pip at the user level.  It allows easy patching by the user, uses least-priviledge concept, and give the user the ability to add plugins and libraries as they wish.

```bash
python3 -m pip install --user ansible
```

External ansible galaxy requirements can then be installed using the requirements file in this project.

```bash
ansible-galaxy install -r requirements.yml
```

This will give you to POSIX and community driven libraries that are very usefull to simplify playbooks/roles.

## Ansible user and SSH keys

It is not recommended to setup the ansible_user and associated ssh key in the configuration.  To meet cyber-security requirements, it far better to let all users run this from their home directory and use their own credentials on the remote systems.  In order to do so, simply setup the .ssh/config under your home directory for ssh key access.  This only applies if using ansible-playbook configuration for pushing changes (not pulling).

```text
Host *.geo.berkeley.edu *.gps.caltech.edu *.gps.caltech.edu *.wr.usgs.gov
  IdentityFile ~/.ssh/ansible_id_rsa
  User chblais
```

## Ansible configuration

This project comes a base ansible.cfg file.  Any commands executed from this directory will use this file.  In other words, it is required to run all ansible commands for the root of the project.   This configuration will identify the default inventory, extra custom python modules (if you desire to create some), and the roles directory.

## Inventories

The structure of the inventories is the one recommended by [Ansible](https://docs.ansible.com/ansible/latest/user_guide/sample_setup.html#alternative-directory-layout).  In brief, each inventories are grouped by their type of environments.  You will see later how easy it is to quickly switch environement with the ansible-playbook command.

In each inventory, there is a main *hosts.yml* file.  Below is an example setup of ShakeAlert.

```yaml
---
all:
  children:
    shakealert:
      children:
        berkeley:
          hosts:
            eew-bk-int1.geo.berkeley.edu:
            eew-bk-int2.geo.berkeley.edu:
        caltech:
          hosts:
            eew-ci-int1.gps.caltech.edu:
            eew-ci-int2.gps.caltech.edu:
        menlo:
          hosts:
            eew-nc-int1.wr.usgs.gov:
            eew-nc-int2.wr.usgs.gov:
        uw:
          hosts:
            eew-uw-int1.ess.washington.edu:
            eew-uw-int2.ess.washington.edu:
        canada:
          hosts:
            eew-cn-int1.seismo.nrcan.gc.ca:
            eew-cn-int2.seismo.nrcan.gc.ca:
```

When using ansible, its ideal to split hosts by groups based on their variables.  For example, in the example above, Canada will not have the same Earthworm institute ID as Caltech.  By doing so, we can set a group specific variable (explained later).

In each envrionment, there is also a *group_vars* and *host_vars* folder.  These are default directory naming pattern defined by Ansible.  In the earlier example, we have a group "canada" with hosts "eew-cn-int1.seismo.nrcan.gc.ca" and "eew-cn-in2.seismo.nrcan.gc.ca".  This means the inventory structure can be defined as followed:

* hosts.yml
* group_vars/
  * all.yml
  * canada.yml
* host_vars/
  * eew-cn-int1.seismo.nrcan.gc.ca.yml
  * eew-cn-int2.seismo.nrcan.gc.ca.yml

The *all.yml* is a global one defined for all hosts.  This is also illustrated in the earlier YAML example.

Under each of these file, you can have role specific variables.  You can also have Ansible vault encrypted information.

## Ansible Vault

It is very important to encrypt any sensitive information.  In this example project, we've setup a sample encrypted YAML file under inventories/integration/group_vars/all/vault.  To encrypt a file, simply run:

```bash
ansible-vault encrypt [filename]
```

or from a new file

```bash
ansible-vault create [filename]
```

You can encrypt/decrypt any type of file.  It is, however, recommended to never use decrypt to edit content and instead use:

```bash
ansible-vault edit [filename]
```

It can happen where a file is not re-encrypted.

## Roles

Ansible roles structure can be generated quickly with the ansible-galaxy command.

```bash
ansible-galaxy role init [name]
```

An ansible role is shareable/modular playbook ideally designed to be shared amongst groups.  They contain their own set of default variables, files, templates, and dependencies cleanly organized and optimized for use with ansible modules.

The structure of a role is as followed:

* defaults = default variables defined by the role, they are lowest level of variable priority meaning that any other overwritting variables defined in group_vars, host_vars, etc will take precedence
* files = files that can be copied (using ansible module "copy") to the remote host
* handlers = handler playbook routines called by tasks with matching "notify" parameters.  These are often used to restart services on state change.
* meta = (ignore) meta information used for hosting on ansible galaxy portal
* tasks = the role specific playbook
* templates = Jinja2 template file that can be copied (using ansible module "template") to the remote host
* tests = (ignore) for testing the playbook on the localhost
* vars = often used for environment specific variables (like OS) if not using the "when" condition part of a task. Can be called using "include_vars" task.  Rarely used.

### Roles - task structure

Through experience, it was determined that it can often be better to leave the main.yml part of the task structure with "import_tasks" to other files.  It can be quite easy to create a long and unreadable YAML configuration file.  As in the example for "earthworm" role in the project, you will notice that each file are defined also by an associated "tags".  This allows quick splitting of sections of the playbook and quick reference of possible tags.  It also remove repeating the tags mutliple times accross multiple tasks.

## Quick tips

### Execute ansible on different environment

In ansible.cfg, the default inventory is often the one with the less risk of execution.  You can easily switch inventory using the command-line by using the -i flag.

```bash
ansible-playbook -i inventories/production site.yml -K --ask-vault-pass
```

Note, in the above, the "-K" will prompt for the sudo password and the "--ask-vault-pass" will be used to decrypt vault content.

### Inventory listing

It might be hard some times to just see the list of hosts/groups. Simply run this:

```bash
ansible-inventory --graph
```

### Running only a tag section

As stated earlier, tags are usefull to run only a small section of the playbook.

```bash
ansible-playbook -i inventories/production site.yml -K --ask-vault-pass --limit eew-cn-int2.seismo.nrcan.gc.ca --tag user_access
```

This will only run the user_access tag (defined under playbooks/shakealert_servers.yml) in host eew-cn-int2.
