# dokku-ansible-playbook

Run ansible playbooks during deployment.

This plugin can be useful when you need to provision your server before or after a deployment of your application (or on [any hook](https://github.com/dokku/dokku/blob/master/docs/development/plugin-triggers.md), just raise an issue and let's add it) and you prefer to use Ansible instead of Bash. For example, you make use of the [ansible-dokku](https://github.com/dokku/ansible-dokku/) roles.

## Requirements

- dokku 0.19.13+
- Debian based system (uses `apt` package manager for installing dependencies)

## Installation

```shell
$ dokku plugin:install https://github.com/decentral1se/dokku-ansible-playbook.git
$ dokku plugin:install-dependencies
```

## Usage

All files must be placed within the `ansible` folder of your git repository. Everything is copied into `$DOKKU_LIB_ROOT/data/ansible/$APP` on the `post-extract` hook. Dokku will make sure that your Ansible plays are run on the right hook against the Dokku server localhost.

- `requirements.yml`: what role dependencies to download before running your plays.
- `pre-deploy.yml`: play run before a deployment
- `post-deploy.yml`: play run after a deployment

## Passwords

You can place a `ansible/.vault.sh` script that produces your [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html password. This file will be copied over to `$DOKKU_LIB_ROOT/data/ansible/$APP` and locked down with the correct read-only permissions for the Dokku user account. This will then be used as the [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) password file which can be used to decrypt secrets.

Don't forget to `chmod +x` it and also **add this file to your `.gitignore`**, you've been warned!

Here's an example `ansible/.vault.sh` file.

```bash
#!/bin/bash

set -eu -o pipefail

echo "my-cool-vault-password"
```

So, if you then encrypt a secret:

```bash
$ ansible-vault \
  encrypt_string \
  --vault-password-file ansible/.vault.sh \
  --name mysecretname \
  mysecretvalue
```

You can place this output in your plays and it can be successfully decrypted on the remote Dokku host.

## Permissions

Since the `dokku` user account runs the plays on the host, you will need to deal with sudo permissions when you want to use `become: true` to run a privilege escalation to the root account. In order to do this, you'll need to 1) run `passwd dokku` as the root user and set an account password and 2) add the `dokku` user account to the sudoers group (`usermod -aG sudo dokku`).

## Example

### ansible/requirements.yml

```yaml
---
- src: dokku_bot.ansible_dokku
  version: v2020.3.15
```

### ansible/pre-deploy.yml

```yaml
---
- hosts: all
  tasks:
    - name: Configure the foobar environment
      dokku_config:
        app: foobar
        restart: false
        config:
          FOO: BAR
      become: true
      become_user: dokku
```
