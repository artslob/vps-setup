# vps-setup
Project for automatic server setup using [ansible](https://github.com/ansible/ansible).

Tested on Ubuntu Server 18.04.2 LTS and ansible `2.8.0`.

# Launch

First, create
[config](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html)
file `~/.ansible.cfg`:
```
[defaults]
inventory = ~/.ansible.hosts
vault_password_file = ~/.ansible.vault.pass
```
Second, create
[hosts](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
file `~/.ansible.hosts`. Content example:
```
[web]
user@1.1.1.1
```
Create file `~/.ansible.vault.pass` with password for
[ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html).
For example:
```
somesecretpassformyvault
```
Download project :
```
git clone https://github.com/artslob/vps-setup
cd vps-setup
```
For user creation you need create file `secrets.yml` in project root directory with this template:
```
---
vault_password: somepass
vault_salt: somesalt
```
Encrypt it:
```
ansible-vault encrypt secrets.yml
```
Contents of secrets file should be something like this (run `cat secrets.yml`):
```
$ANSIBLE_VAULT;1.1;AES256
31643131623866643738666533313633366533633133353534633461626355366230623339616437
...
```
Finally, run playbook to create user on your server:
```
ansible-playbook 01-create-user.yml -e "host_env=ec2 root_user=ubuntu"
```
Flag `-e` (or `--extra-vars`) provides additional environment variables, which override
default values in playbook.  
Contents of encrypted `secrets.yml` parsed by ansible automatically.
