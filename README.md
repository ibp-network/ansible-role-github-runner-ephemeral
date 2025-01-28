# Ephemeral Github Runner Ansible role
This ansible role installs and configures rootless docker and a self-hosted ephemeral github runner in a secure way.


Scripts used used in this for the `runner` service are from @alvicsam and his [github-runner-docker-ephemeral](https://github.com/alvicsam/github-runner-docker-ephemeral/) repo.
By default with the [docker-github-actions-runner](https://github.com/myoung34/docker-github-actions-runner) setup variables are not safe from exfiltration. Read his [blogpost](https://dev.to/alvic/ephemeral-self-hosted-github-actions-runners-42ma) for more information.
```
### Security

It is known that environment variables are not safe from
exfiltration. If you are using this runner make sure that any
workflow changes are gated by a verification process (in the actions
settings) so that malicious PR's cannot exfiltrate these.
```
To solve this we are using @alvicsam's solution and are going to run another docker container in parallel, which will be run in a rootless docker setup.

## Role Requirements
---
This role currently only supports Debian.

**requirements.yaml**
```yml
---
roles:
  - name: github-runner-ephemeral
    src: https://github.com/ibp-network/ansible-role-github-runner-ephemeral.git
    scm: git
    version: main
```

```bash
ansible-galaxy install -r requirements_collections.yml
```

## Using the Role
---
### Playbook Example
```yml
---
- name: Github Playbook
  hosts: github_runner
  gather_facts: true
  roles:
    - role: github-runner-ephemeral
```
### Inventory Example
```yml
---
all:
  children:
    github_runner:
      hosts:
        git1:
          ansible_host: gitrun-001.amforc.com
          runner_org: amforc
          runner_name: "gitrun-001"
```
### Vault Example
```yml
---
runner_app_id: 12345
runner_app_login: amforc

runner_app_private_key: |
  -----BEGIN RSA PRIVATE KEY-----
  -----END RSA PRIVATE KEY-----
```

### Variables
---
The following are the ones that **have** to be set
| Variable      | Description      |
| ------------- | ------------- |
| `runner_name` | The name of the ephemeral runner, this will show up in the org in github |
| `runner_org` | The github organisation this runner will join. For example `ibp-network` or `amforc` |
| `runner_app_id` | The github application ID |
| `runner_app_login` | The github application login id, if not specified, the same value as `runner_org` can be used |
| `runner_app_private_key` | The github application private key |

Some of the default ones, can be changed to your needs
| Variable      | Default | Description      |
| ------------- | ------------- | ------------- |
| `runner_scope` | `org` | The scope the runner will be registered on |
| `runner_labels` | `self-hosted,ephemeral,debian` | A comma separated string to indicate the labels. |
| `runner_build` | `myoung34/github-runner:debian-bookworm` | Using debian by default, if you want ubuntu: `myoung34/github-runner:ubuntu-jammy`. More options [here](https://github.com/myoung34/docker-github-actions-runner?tab=readme-ov-file#docker-artifacts) |

### Creating a Github app
---
If a project wants a ephemeral runner, run by the IBP they first need to create a Github app and provide `runner_app_id`, `runner_app_login`, `runner_app_private_key` to the IBP member that will run their instance.

- Go to: Github > org > settings > developer settings > github apps > new github app
  - Enter name
  - Homeage url can be any place holder
  - `callback url` can be left empty
  - `webhook` falase
  - *Organization permissoins*: `Self-hosted runners Read & Write`
  - *Where can this GitHub App be installed?*: `Only on this account`
  - **create**
- copy the `app id`
- Go to `private keys`, generate a new one.
- Go to  `Install App` and install ap in org.

Send all the new necessary information in a secure matter to the IBP member that will run your instance.
