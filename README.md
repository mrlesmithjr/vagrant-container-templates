Purpose
=======

Spin up different OS container types using [Vagrant] and learn [Ansible] at the
same time if desired. These are based off of the same concepts that I used for
[Vagrant-Box-Templates].

Requirements
------------

- [Ansible]
- [Docker]
- [Vagrant]

Usage
-----

Easily spin up:

```
cd Ubuntu/xenial64
...
vagrant up
```

- In each distro you will find a `containers.yaml` file which resembles the below.
You can modify this to suit your needs.

`containers.yaml`:
```
---
- name: 'container0'
  ansible_groups:
    - 'test_nodes'
  build: true
  provision: false
  # port_forwards:
  #   - guest: '80'
  #     host: '8080'
  #   - guest: '443'
  #     host: '4433'
```
- Change `provision: false` to `provision: true` if you would like to test
[Ansible] related tasks against the specific distro.

- Define `port_forwards` as needed for specific
testing of apps
- If you need to spin up additional containers your `containers.yaml` may look
like:

`containers.yaml`:
```
---
- name: 'container0'
  ansible_groups:
    - 'test_nodes'
  build: true
  provision: false
  # port_forwards:
  #   - guest: '80'
  #     host: '8080'
  #   - guest: '443'
  #     host: '4433'
- name: 'container1'
  ansible_groups:
    - 'test_nodes'
  build: true
  provision: false
  # port_forwards:
  #   - guest: '80'
  #     host: '8080'
  #   - guest: '443'
  #     host: '4433'
```

[Ansible]: <https://www.ansible.com>
[Docker]: <https://www.docker.com>
[Vagrant]: <https://www.vagrantup.com/>
[Vagrant-Box-Templates]: <https://github.com/mrlesmithjr/vagrant-box-templates>
