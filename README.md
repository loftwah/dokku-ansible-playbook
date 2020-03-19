# dokku-ansible-playbook

Run ansible playbooks during deployment.

This plugin can be useful when you need to provision your server before or after a deployment of your application (or on [any hook](https://github.com/dokku/dokku/blob/master/docs/development/plugin-triggers.md), just raise an issue and let's add it) and you prefer to use Ansible instead of Bash. For example, you make use of the [ansible-dokku](https://github.com/dokku/ansible-dokku/) roles.

## Requirements

* dokku 0.19.13+
* Debian based system (uses `apt` package manager for installing dependencies)

## Installation

```shell
$ dokku plugin:install https://github.com/decentral1se/dokku-ansible-playbook.git
$ dokku plugin:install-dependencies
```

## Usage

All files must be placed within the `ansible` folder of your git repository.

* `requirements.yml`: what role dependencies to download before running your plays.
* `pre-deploy.yml`: play run before a deployment
* `post-deploy.yml`: play run after a deployment

Notes:

* Everything is copied into `$DOKKU_LIB_ROOT/data/ansible/$APP` on the `post-extract` hook.
* Dokku will make sure that your Ansible plays are run on the right hook against the Dokku server localhost.
* The `ansible-galaxy` install command is run with `--force` to ensure a clean slate on each hook execution.

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
    - name: Create foobar group
      group:
        name: foobar
        system: true
        state: present
```
