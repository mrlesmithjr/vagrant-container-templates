<!-- START doctoc generated TOC please keep comment here to allow auto update -->

<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

-   [Vagrant Container Templates](#vagrant-container-templates)
    -   [Purpose](#purpose)
    -   [Requirements](#requirements)
        -   [Software](#software)
    -   [Useful information](#useful-information)
        -   [Docker images](#docker-images)
    -   [Usage](#usage)
        -   [Getting started](#getting-started)
            -   [Clone repo](#clone-repo)
            -   [Choose distro](#choose-distro)
            -   [Customizing environment](#customizing-environment)
                -   [Provisioning](#provisioning)
                -   [Spinning up environment](#spinning-up-environment)
            -   [Tearing down environment](#tearing-down-environment)
    -   [Tips And Tricks](#tips-and-tricks)
        -   [Build images](#build-images)
        -   [`/etc/hosts`](#etchosts)
    -   [License](#license)
    -   [Author Information](#author-information)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Vagrant Container Templates

## Purpose

Spin up different OS container types using [Vagrant](https://www.vagrantup.com)
and learn [Ansible](https://www.ansible.com) at the same time if desired. These
are based on the same concepts that I used for [Vagrant-Box-Templates](https://github.com/mrlesmithjr/vagrant-box-templates).

## Requirements

### Software

-   [Ansible](https://www.ansible.com)
-   [Docker](https://www.docker.com)
-   [Vagrant](https://www.vagrantup.com)
-   [Virtualbox](https://www.virtualbox.org)

> NOTE: If you **do not have** `v1.9.4` at a minimum installed when you run
> `vagrant up` [Vagrant](https://www.vagrantup.com) will attempt to load
> `hashicorp/boot2docker` within [Virtualbox](https://www.virtualbox.org) and
> will not leverage (native) locally installed [Docker](https://www.docker.com).
> When this happens these will not work properly _(at this time)_. After
> researching this quickly it looks like the _reason/workaround_ is related to [this](https://www.vagrantup.com/docs/docker/configuration.html#force_host_vm).

## Useful information

### Docker images

All of the images for each distro are available on [Docker Hub](https://github.com/mrlesmithjr/vagrant-box-templates) which are the default
images that are pulled down for each distro when testing with this repo.

## Usage

### Getting started

#### Clone repo

```bash
git clone https://github.com/mrlesmithjr/vagrant-container-templates.git
cd vagrant-container-templates
```

#### Choose distro

#### Customizing environment

Each distro folder contains a `containers.yml` file which you can change the number
of containers to spin up if desired.

`containers.yaml`:

```yaml
---
- name: 'container0'
  ansible_groups:
    - 'test_nodes'
  build: false
  image: mrlesmithjr/vagrant-containers:xenial64
  provision: false
  # port_forwards:
  #   - guest: '80'
  #     host: '8080'
  #   - guest: '443'
  #     host: '4433'
- name: 'container1'
  ansible_groups:
    - 'test_nodes'
  build: false
  image: mrlesmithjr/vagrant-containers:xenial64
  provision: false
  # port_forwards:
  #   - guest: '80'
  #     host: '8080'
  #   - guest: '443'
  #     host: '4433'
```

##### Provisioning

If you would like to provision the containers when they startup you will need to
set `provision: true` in the `containers.yml`.

##### Spinning up environment

When you are ready to spin up simply:

```bash
cd Ubuntu/xenial64
vagrant up
```

#### Tearing down environment

When you are all done with your [Vagrant](https://www.vagrantup.com) environment
you can quickly and cleanly tear it all down:

```bash
./cleanup.sh
```

## Tips And Tricks

### Build images

When spinning up containers the default is to build them each time. Instead of
doing this you can simply build each distro image that you choose and then change
`containers.yml` to use the image instead.

For example let's build our custom `Ubuntu Xenial` image:

```bash
cd Ubuntu/xenial64
docker build -t localhost/xenial64-vagrant .
```

Now change `Ubuntu/xenial64/containers.yml` to use this new image:

`From`:

```yaml
---
- name: 'container0'
  ansible_groups:
    - 'test_nodes'
  build: false
  image: mrlesmithjr/vagrant-containers:xenial64
  provision: false
  # port_forwards:
  #   - guest: '80'
  #     host: '8080'
  #   - guest: '443'
  #     host: '4433'
```

`To`:

```yaml
---
- name: 'container0'
  ansible_groups:
    - 'test_nodes'
  build: false
  image: localhost/xenial64-vagrant
  provision: false
  # port_forwards:
  #   - guest: '80'
  #     host: '8080'
  #   - guest: '443'
  #     host: '4433'
```

### `/etc/hosts`

Updating `/etc/hosts` on each container spun up for multi-system testing. You
may have a need to do some testing with an [Ansible](https://www.ansible.com)
role which requires multiple systems and need name resolution working between
containers. There is a [Vagrant](https://www.vagrantup.com) plugin called
`vagrant-hostmanager` but I didn't get the results that I was looking for. So I
put together this bit of `pre-tasks` in the `playbook.yml` which will take care
of this for us. Or you can use this for additional use cases.

```yaml
---
- hosts: all
  vars:
  pre_tasks:
    - name: Installing Network Related Packages (Alpine)
      apk:
        name: "{{ item }}"
        state: "present"
        update_cache: true
      become: true
      with_items:
        - 'bind-tools'
        - 'iproute2'
        - 'iputils'
      when: ansible_os_family == "Alpine"

    - name: Installing Network Related Packages (Debian)
      apt:
        name: "{{ item }}"
        state: "present"
      become: true
      with_items:
        - 'iproute'
        - 'iputils-ping'
        - 'net-tools'
      when: ansible_os_family == "Debian"

    - name: Installing Network Related Packages (RedHat)
      yum:
        name: "{{ item }}"
        state: "present"
      become: true
      with_items:
        - 'iproute'
        - 'iputils'
        - 'net-tools'
      when: >
            ansible_os_family == "RedHat" and
            ansible_distribution != "Fedora"

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
  roles:
  tasks:
    - name: Update Apt-Cache
      apt:
        update_cache: true
        cache_valid_time: 3600
      become: true
      when: ansible_os_family == "Debian"
```

## License

MIT

## Author Information

Larry Smith Jr.

-   [@mrlesmithjr](https://www.twitter.com/mrlesmithjr)
-   [EverythingShouldBeVirtual](http://everythingshouldbevirtual.com)
-   mrlesmithjr [at] gmail.com
