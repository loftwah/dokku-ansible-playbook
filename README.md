# dokku-ansible-playbook

Run ansible playbooks during deployment.

## Requirements

* dokku 0.19.13+
* Debian based system (uses `apt` package manager for dependencies)

## Installation

```shell
$ dokku plugin:install https://github.com/decentral1se/dokku-ansible-playbook.git
```

## Usage

All files must be placed within the `ansible` folder of your git repository.

* `requirements.yml`: what role dependencies to download before running your plays.

The following hooks are supported (add `.yml` to the hook name in `ansible`):

* `pre-deploy`
* `post-deploy`

Everything is copied into `$DOKKU_LIB_ROOT/data/ansible/$APP` on the `post-extract` hook.

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
