# vps-setup
Project for automatic server setup using [ansible](https://github.com/ansible/ansible).

Tested on Ubuntu Server 18.04.2 LTS and ansible `2.8.0`.

# Launch

### 1. Ansible setup

1. Create [config](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html)
file `~/.ansible.cfg`:
    ```
    [defaults]
    inventory = ~/.ansible.hosts
    vault_password_file = ~/.ansible.vault.pass

    [ssh_connection]
    ssh_args = -o ForwardAgent=yes -o ControlMaster=auto -o ControlPersist=60s
    ```
    We will use `ssh-agent` and
    [agent forwarding](https://developer.github.com/v3/guides/using-ssh-agent-forwarding/)
    to apply ssh keys from local machine to remotes. So dont forget to run `ssh-add` before running scripts.

2. Create [hosts](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) file `~/.ansible.hosts`.  
For example:
    ```
    [web]
    user@1.1.1.1
    ```

3. Create file `~/.ansible.vault.pass` with password for
[ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html).  
For example:
    ```
    somesecretpassformyvault
    ```

### 2. Project setup

1. Download project:
    ```
    git clone https://github.com/artslob/vps-setup
    cd vps-setup
    ```

2. Create file `secrets.yml` in project root directory with this template:
    ```
    vault_user_password: somepass
    vault_user_salt: somesalt
    vault_cf_key: deadbeaf01010101010101010101010101fff
    vault_cf_email: user@example.com
    ```
    This file contains secrets for user creation, cloudflare tokens for ssl setup
    ([acme](https://en.wikipedia.org/wiki/Automated_Certificate_Management_Environment)).

3. Encrypt it:
    ```
    ansible-vault encrypt secrets.yml
    ```
    Contents of secrets file should be something like this (run `cat secrets.yml`):
    ```
    $ANSIBLE_VAULT;1.1;AES256
    31643131623866643738666533313633366533633133353534633461626355366230623339616437
    ...
    ```

### 3. Run playbooks

1. Run playbook to create user on your server:
    ```
    ansible-playbook 01-create-user.yml -e "host_env=ec2 root_user=ubuntu"
    ```
    Flag `-e` (or `--extra-vars`) provides additional environment variables, which override
    default values in playbook.  
    Contents of encrypted `secrets.yml` parsed by ansible automatically.

    Few notes:
    * This playbook will not create ssh keys on remote machine. Reason for this is because you should use
    [SSH agent forwarding](https://developer.github.com/v3/guides/using-ssh-agent-forwarding/)
    to exploit your keys from local machine on remotes.
    * Playbook setup switching to `sudo` mode without password for default user.
    * Also setup for all `sudoers` preserving of `SSH_AUTH_SOCK` environment variable to exploit
    [SSH agent forwarding](https://developer.github.com/v3/guides/using-ssh-agent-forwarding/)
    in `sudo` mode.

# Useful links
1. [Ansible vault tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-vault-to-protect-sensitive-ansible-data-on-ubuntu-16-04)
2. [Create user with password](https://stackoverflow.com/questions/19292899/creating-a-new-user-and-password-with-ansible)
3. [Create user accounts and setup ssh keys](http://minimum-viable-automation.com/ansible/use-ansible-create-user-accounts-setup-ssh-keys/)
