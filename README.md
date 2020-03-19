# dokku-ansible-playbook

Run ansible playbooks during deployment.

## Requirements

* dokku 0.19.13+
* Debian based system (uses `apt` package manager)

## Installation

```shell
$ dokku plugin:install https://github.com/decentral1se/dokku-ansible-playbook.git
```

## Usage

All files must be placed within the `.ansible` folder of your git repository.

* `requirements.yml`: what role dependencies to download before running your plays
* `prepare.yml`: the play to run before the application is built
