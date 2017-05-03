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
  - If you **do not have** `v1.9.4` at a minimum installed when you run `vagrant up` [Vagrant] will attempt to load `hashicorp/boot2docker` within [Virtualbox] and will not leverage (native) locally installed [Docker]. When this happens these will not work properly *(at this time)*. After researching this quickly
  it looks like the *reason/workaround* is related to [this](https://www.vagrantup.com/docs/docker/configuration.html#force_host_vm).
- [Virtualbox]

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

Tips And Tricks
---------------

- Updating `/etc/hosts` on each container spun up for multi-system testing.
You may have a need to do some testing with an [Ansible] role which requires
multiple systems and need name resolution working between containers. There is
a [Vagrant] plugin called `vagrant-hostmanager` but I did't get the results
that I was looking for. So I put together this bit of `pre-tasks` in the
`playbook.yml` which will take care of this for us. Or you can use this for
additional use cases.
```
---
- hosts: all
  vars:
  pre_tasks:
    - name: Installing Network Related Packages
      apt:
        name: "{{ item }}"
        state: "present"
      become: true
      with_items:
        - 'iputils-ping'
        - 'net-tools'
      when: ansible_os_family == "Debian"

# Capturing ip address of eth0 inside container
    - name: Capturing IP
      shell: "ifconfig eth0 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}'"
      register: "_ip_address"
      become: true
      changed_when: false

# We use /etc/hosts.tmp to collect our hosts because we cannot directly make
# changes to /etc/hosts
    - name: Checking If /etc/hosts.tmp Exists
      stat:
        path: "/etc/hosts.tmp"
      register: "_etc_host_tmp"

    - name: Creating /etc/hosts.tmp If It Does Not Exist
      file:
        path: "/etc/hosts.tmp"
        state: "touch"
      become: true
      when: not _etc_host_tmp['stat']['exists']

    - name: Updating /etc/hosts.tmp
      lineinfile:
        path: "/etc/hosts.tmp"
        regexp: "^127.0.0.1"
        line: "127.0.0.1 localhost"
      register: "_etc_host_tmp_updated_localhost"
      become: true
      with_items: '{{ play_hosts }}'

    - name: Updating /etc/hosts.tmp
      lineinfile:
        path: "/etc/hosts.tmp"
        regexp: "^{{ hostvars[item]['_ip_address']['stdout'] }}"
        line: "{{ hostvars[item]['_ip_address']['stdout'] }} {{ hostvars[item]['inventory_hostname'] }}"
      register: "_etc_host_tmp_updated"
      become: true
      with_items: '{{ play_hosts }}'

# We do this to overwrite the current /etc/hosts
    - name: Updating /etc/hosts
      shell: "cat /etc/hosts.tmp > /etc/hosts"
      become: true
      when: >
            _etc_host_tmp_updated['changed'] or
            _etc_host_tmp_updated_localhost['changed']
```

[Ansible]: <https://www.ansible.com>
[Docker]: <https://www.docker.com>
[Vagrant]: <https://www.vagrantup.com/>
[Vagrant-Box-Templates]: <https://github.com/mrlesmithjr/vagrant-box-templates>
[Virtualbox]: <https://www.virtualbox.org/>
