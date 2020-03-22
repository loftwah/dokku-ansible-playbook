# dokku-ansible-playbook

Run ansible playbooks during deployment.

This plugin can be useful when you need to provision your server before or after a deployment of your application (or on [any hook](https://github.com/dokku/dokku/blob/master/docs/development/plugin-triggers.md), just raise an issue and let's add it) and you prefer to use Ansible instead of Bash for certain tasks. Bash can still be the right tool for other things but sometimes, it can be become tricky to manage the idempotent case in Bash.

For example, you can make use of the official [ansible-dokku](https://github.com/dokku/ansible-dokku/) roles.

## Requirements

- dokku 0.19.13+
- Debian based system (uses `apt` package manager for installing dependencies)

## Installation

```shell
$ dokku plugin:install https://github.com/decentral1se/dokku-ansible-playbook.git
$ dokku plugin:install-dependencies
```

## Usage

All files must be placed within the `ansible` folder of your git repository. Everything is copied into `$DOKKU_LIB_ROOT/data/ansible/$APP` on the `post-extract` hook. Dokku will make sure that your Ansible plays are run on various hooks against the Dokku server localhost.

- `requirements.yml`: what role dependencies to download before running your plays.
- `pre-deploy.yml`: play run before a deployment
- `post-deploy.yml`: play run after a deployment
- `post-delete.yml`: play run after an application delete
- `vars/...`: variable files (you'll need to include manually with the [include_vars](https://docs.ansible.com/ansible/latest/modules/include_vars_module.html) module)

## Passwords

Ansible uses the [vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) password file which can be used to decrypt secrets.

To get started with enabling this, you should generate a vault password for your self and run the following on your Dokku host.

```bash
$ dokku ansible-playbook:vault-pass
```

Then you can start to encrypt your passwords on your local machine with the following.

```bash
$ ansible-vault \
  encrypt_string \
  --vault-password-file ansible/.vault.sh \
  --name mysecretname \
  mysecretvalue
```

Where `ansible/.vault.sh` might look like this.

```bash
#!/bin/bash

set -eu -o pipefail

echo "my-cool-vault-password"
```

Then for example, if you want to pass a sudo password, you might include a `vars/ansible_become_password.yml`.

```yaml
---
ansible_become_password: !vault ...
```

## Permissions

Since the `dokku` user account runs the plays on the host, you will need to deal with sudo permissions when you want to use `become: true` to run a privilege escalation to the root account. You can give your `dokku` user account passwordless sudo access but that would give a lot of power to people who can get access to that user account. A solution to this can be to add your `dokku` to the sudoers group, give the account a password (`passwd dokku && usermod -aG sudo dokku`) and pass `ansible_become_password` in as a variable.

## Injected variables

Same as the plugin available variables but in your Ansible plays.

- `dokku_lib_root`

## Example

### ansible/requirements.yml

```yaml
---
- src: dokku_bot.ansible_dokku
  version: v2020.3.15
```

### ansible/vars/ansible_become_password.yml

```yaml
---
ansible_become_pass: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  34396236353735666531323238656533643465303131663464613162396333313836363630666266
  6539323631656635333864316166633064633366323936610a656137616334313534333635313232
```

### ansible/pre-deploy.yml

```yaml
---
- hosts: all
  tasks:
    - name: Load variables
      include_vars:
        dir: "{{ dokku_lib_root }}/data/ansible/gitea/vars/"
        extensions:
          - yml

    - name: Configure the foobar environment
      dokku_config:
        app: foobar
        restart: false
        config:
          FOO: "BAR"

    - name: Setup host group
      group:
        name: barfoo
        system: true
        state: present
      become: true
```
