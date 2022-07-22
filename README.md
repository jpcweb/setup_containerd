Setup Containerd
=========

Install and configure containerd and crictl for kubelet

Requirements
------------

You'll need a Debian based machine and at least a sudo capability

Example Playbook
----------------

```yaml
    - hosts: servers
      become: yes
      roles:
         - { role: setup_containerd, crictl_version: 1.24.0 }
```

Infos
-------

You can find out about the distribution of a machine using
```shell
  ansible -m setup -a filter="ansible_distribution*" machine_or_group
``` 

License
-------

BSD

Author Information
------------------

Jeremie Payet (jpcweb)
