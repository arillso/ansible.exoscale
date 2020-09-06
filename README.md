# Ansible Role: exoscale
[![Build Status](https://travis-ci.com/arillso/ansible.exoscale.svg?branch=master)](https://travis-ci.com/arillso/ansible.exoscale)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?style=popout-square)](https://sbaerlo.ch/licence)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-arillso.exoscale-blue.svg?style=popout-square)](https://galaxy.ansible.com/arillso/exoscale)
[![Ansible Role](https://img.shields.io/ansible/role/d/45457.svg?style=popout-square)](https://galaxy.ansible.com/arillso/exoscale)

## Description
[Exoscale](https://www.exoscale.com) is a provider for VPS. This role aims to automate
the creation of infrastructure as automated as possible.

The goal behind this role is to create a way of defining infrastructure as code
in an Ansible playbook, allowing a quick and easy setup of services without
having to deal with infrastructure.

### Usage
Your access credentials are read from your environment variables or from a
config file. For more information, read [this note](https://docs.ansible.com/ansible/latest/modules/cs_instance_module.html#notes). You must define at least the following:
* `CLOUDSTACK_KEY`
* `CLOUDSTACK_SECRET`
* `EXOSCALE_ACCOUNT_EMAIL`
* `CLOUDSTACK_ENDPOINT`

The configuration is read from the host variables.

> NOTE: as of now, you can only manage infrastructure in one zone. If you need
> to manage multiple zones, you will have to use multiple playbooks.

#### Example: simple instance
To create a new machine, you can define a host as follows:
```yaml
# hosts.yml
---
all:
    vars:
        exoscale_zone: "CH-DK-2"
    hosts:
        test.syntro.ch:
            exoscale_ssh_key: test
            exoscale_template: "Linux Ubuntu 18.04 LTS 64-bit"
            exoscale_service_offering: "Micro"
            exoscale_disk: 20
            exoscale_instance: true
```
If you now apply the role to the `test.syntro.ch` host, it will create a new
instance with a disk size of 20 GB using the `test` key in your account. You
should then see the instance appear in the Exoscale GUI.

After the machine has started, you will have to set the `ansible_host` for the
instance to its actual IP. Depending on your security rules setup, you might
need to apply a rule allowing you to access the instance via SSH.

#### Example: advanced configuration
This role also allows you to apply more advanced configurations to your
instances. Note that most of the definitions are globally applied and **should
NEVER be set for a single instance**.

To create an instance pool with a custom set of security groups and a private
network, we can use the following template:

```yaml
# hosts.yml
---
all:
    vars:
        exoscale_zone: "CH-DK-2"
        exoscale_firewalls_definition:
            - name: http_https
              description: default http/https set
              rules:
                    - {'start_port':'80', 'protocol':'tcp', 'type':'ingress'}
                    - {'start_port':'443', 'protocol':'tcp', 'type':'ingress'}
            - name: ssh
              description: default ssh set
              rules:
                    - {'start_port':'22', 'protocol':'tcp', 'type':'ingress'}
        exoscale_networks_definition:
            - name: private_network
    hosts:
        test-1.syntro.ch:
            exoscale_ssh_key: test
            exoscale_template: "Linux Ubuntu 18.04 LTS 64-bit"
            exoscale_service_offering: "Micro"
            exoscale_disk: 20
            exoscale_instance: true
            exoscale_internal_networks:
                - internal
            exoscale_firewalls:
                - http_https
                - ssh
        test-2.syntro.ch:
            exoscale_ssh_key: test
            exoscale_template: "Linux Ubuntu 18.04 LTS 64-bit"
            exoscale_service_offering: "Micro"
            exoscale_disk: 20
            exoscale_instance: true
            exoscale_internal_networks:
                - internal
            exoscale_firewalls:
                - http_https
                - ssh
```

## Installation
```bash
ansible-galaxy install arillso.exoscale
```

## Requirements
You must install the following Python dependencies on the machine executing
the playbook:
* `jmespath`
* `cs`
* `sshpubkeys`

```
pip install cs jmespath sshpubkeys
```

## Role Variables
| Name                            | Default        | Must be Global | Description                             |
| ------------------------------- | -------------- |:--------------:| --------------------------------------- |
| `exoscale_instance`             | `false`        |       🟥       | Determines if this host will be created |
| `exoscale_template`             | **(required)** |       🟥       | The template to be used                 |
| `exoscale_service_offering`     | **(required)** |       🟥       | The service offering (eg. `Micro`)      |
| `exoscale_disk`                 | `10`           |       🟥       | the disksize to be applied              |
| `exoscale_internal_networks`    | `[]`           |       🟥       | the networks this instance is part of   |
| `exoscale_firewalls`            | `[]`           |       🟥       | the security rules applied              |
| `exoscale_ssh_key`              | **(required)** |       🟥       | the name of the public ssh key to use   |
| `exoscale_user_data`            | `''`           |       🟥       | user data applied to the instance       |
| `exoscale_zone`                 | **(required)** |       🟩       | the zone to manage                      |
| `exoscale_networks_definition`  | `[]`           |       🟩       | network definition (List of names)      |
| `exoscale_firewalls_definition` | `[]`           |       🟩       | security groups definition              |
| `exoscale_ssh_key_def`          | `[]`           |       🟩       | public key definition                   |

#### Firewalling Definition
Supply a list of dicts with the following keys:
```yaml
- name: http_https
  description: default http/https set
  rules:
        - {'start_port':'80', 'protocol':'tcp', 'type':'ingress'}
        - {'start_port':'443', 'protocol':'tcp', 'type':'ingress'}
```

The rules have the following keys available:
| Name         | Description                         | Required |
| ------------ | ----------------------------------- |:--------:|
| `start_port` | the start port                      |    🟩    |
| `end_port`   | the end port                        |    🟥    |
| `protocol`   | the proto                           |    🟩    |
| `type`       | the type (`tcp`/`udp`)              |    🟩    |
| `cider`      | the range (defaults to `0.0.0.0/0`) |    🟥    |

#### SSH Keys
Define a list of public keys as follows:
```yaml
exoscale_ssh_key_def:
    - name: docker
      pub: ssh-rsa ...
```

#### User Data
You can specify user data passed to the instances on startup with
`exoscale_user_data`. This comes in handy to initially set up networks or packages
(eg. python). Read more in [this doc article](https://community.exoscale.com/documentation/compute/cloud-init/).

## Dependencies
none

## Example Playbook
```yml
- hosts: all
  roles:
    - arillso.exoscale
```

## Author
- Matthias Leutenegger

## License
This project is under the MIT License. See the [LICENSE](https://sbaerlo.ch/licence) file for the full license text.

## Copyright
(c) 2019, Syntro GmbH
