# dokku-ansible-playbook

Run ansible playbooks during deployment.

## Requirements

* dokku 0.19.13+
* [dokku-supply-config](https://github.com/dokku-community/dokku-supply-config)
* Debian based system (uses `apt` package manager for dependencies)

## Installation

```shell
$ dokku plugin:install https://github.com/josegonzalez/dokku-supply-config.git
$ dokku plugin:install https://github.com/decentral1se/dokku-ansible-playbook.git
```

## Usage

All files must be placed within the `.dokku/ansible` folder of your git repository.

* `requirements.yml`: what role dependencies to download before running your plays.

The following hooks are supported (add `.yml` to the hook name in `.dokku/ansible`):

* `pre-deploy`
* `post-deploy`

## Example

### .dokku/ansible/requirements.yml

```yaml
---
- src: dokku_bot.ansible_dokku
  version: v2020.3.15
```

### .dokku/ansible/pre-deploy.yml

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
