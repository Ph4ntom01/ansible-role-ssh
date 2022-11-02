# Ansible Role: SSH

This role sets the environment on ansible hosts (machines managed by ansible) by creating a common user and setting needed keys.

- Creates a user to all the hosts groups (master_node and nodes)
- Adds him in sudoer group
- Configures sudo without password
- Creates ed25519 ssh key on master node
- Defines the authorized_keys file
- Add all nodes to the known_hosts file (in the master node)
- Defines a custom name for the ssh keys

## Requirements

### Root Access

This role requires root access, so either run it in a playbook with a global `become: yes`, or invoke the role in your playbook like:

```yml
- hosts: master_node:nodes
  roles:
    - role: ansible-role-ssh
      become: yes
```

Note that you can also set the `become: yes` value in the ansible.cfg file, but use a local file located in the same directory as your playbook.

#### Prompt

We use interactive input (prompt) for the user password. The benefit is that the password is automatically encrypted in SHA-512 by default (see the [example playbook](#example-playbook)).

Other values are available :

- `bcrypt`
- `md5_crypt`
- `sha256_crypt`
- `sha512_crypt`

More information [here](https://docs.ansible.com/ansible/latest/user_guide/playbooks_prompts.html#hashing-values-supplied-by-vars-prompt).

### Modules

- [authorized_key](https://galaxy.ansible.com/ansible/posix)
- [openssh_keypair](https://galaxy.ansible.com/community/crypto)

## Nodes Groups

### Master node

Although this node is used as a simple control node to connect to the clients, It is unique and used to create the ssh keys. Then, the public key is sent to the client nodes.

If the ansible playbook is launched again (by adding some new client nodes), the keys are not generated again.

### Nodes

These nodes are the clients nodes. The master node can connect to them (by using the user account created by this role) without being prompted for a user and a password.

## Role Variables

### Default values (`defaults/main.yml`)

User configuration, the user can be set as a sudo user :

```yml
# User configuration.
usr_name: "user"
usr_dir: "/home/{{ usr_name }}"
is_usr_sudo: false
```

SSH configuration (with custom key names) :

```yml
ssh_dir: "{{ usr_dir }}/.ssh"
ssh_priv_key: "{{ ssh_dir }}/{{ usr_name }}"
ssh_pub_key: "{{ ssh_priv_key }}.pub"
```

## Role Tasks

The role is divided into different tasks, the `tasks/main.yml` file include these tasks :

```yml
---
# User configuration.
- include_tasks: user.yml

# Setup/install tasks.
- include_tasks: create_key.yml
- include_tasks: setup_key.yml

# SSH configuration.
- include_tasks: ssh.yml
```

## Example Playbook

```yml
- hosts: master_node:nodes
  become: yes
  roles:
    - ansible-role-ssh

  vars_prompt:
    - name: usr_passwd
      prompt: Enter user password
      unsafe: yes
      private: yes
      encrypt: sha512_crypt
      confirm: yes
```

## Example Inventory

```yml
[master_node]
node1

[nodes]
node2
node3
node4
node5
```
