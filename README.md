# prometheus_server

Master: [![Build Status](https://travis-ci.org/sansible/prometheus_server.svg?branch=master)](https://travis-ci.org/sansible/prometheus_server)
Develop: [![Build Status](https://travis-ci.org/sansible/prometheus_server.svg?branch=develop)](https://travis-ci.org/sansible/prometheus_server)

* [ansible.cfg](#ansible-cfg)
* [Installation and Dependencies](#installation-and-dependencies)
* [Tags](#tags)
* [Example](#example)

This role installs Prometheus Server.

For more information about Prometheus please visit
[https://prometheus.io/](https://prometheus.io/).



## ansible.cfg

This role is designed to work with merge "hash_behaviour". Make sure your ansible.cfg contains these settings

```INI
[defaults]
hash_behaviour = merge
```


## Installation and Dependencies

This role will install `sansible.users_and_groups` for managing `prometheus_server` user.

To install run `ansible-galaxy install sansible.prometheus_server` or add this to your `roles.yml`

```YAML
- name: sansible.prometheus_server
  version: v1.0
```

and run `ansible-galaxy install -p ./roles -r roles.yml`


## Tags

This role uses two tags: **build** and **configure**

* `build` - Installs Prometheus Server and all it's dependencies.
* `configure` - Configure and ensures that the Prometheus Server service is running.


## Example

```YAML
- name: Install Prometheus Server
  hosts: sandbox

  pre_tasks:
    - name: Update apt
      become: yes
      apt:
        cache_valid_time: 1800
        update_cache: yes
      tags:
        - build

    - name: Create Prometheus directories
      become: yes
      file:
        path: "{{ item }}"
        state: directory
        mode: 0644
      with_items:
        - /home/prometheus/rules
        - /home/prometheus/tgroups
      tags:
        - build

    - name: Place Prometheus rules on server
      become: yes
      copy:
        src: "{{ item }}"
        dest: /home/prometheus/rules
        mode: 0644
      with_fileglob:
        - "{{ playbook_dir }}/files/rules/*.rules"
      tags:
        - build

    - name: Place Prometheus service discovery configuration on server
      become: yes
      copy:
        src: "{{ item }}"
        dest: /home/prometheus/tgroups
        mode: 0644
      with_fileglob:
        - "{{ playbook_dir }}/files/sd/*.json"
        - "{{ playbook_dir }}/files/sd/*.yml"
        - "{{ playbook_dir }}/files/sd/*.yaml"
      tags:
        - build

  roles:
    - name: sansible.prometheus_server
      environment: dev
      path:
        data: /home/prometheus/data
```
