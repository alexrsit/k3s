# Loadbalanced K3s Cluster

Installs a loadbalanced k3s cluster on selected hosts. 
Recommended 3 masters, 3 workers and 2 loadbalancer nodes. 
Preferred and tested OS = Ubuntu >=22.04

Use "cleanup" role if the environment is to be destroyed after creation.
Full example below

## Playbook usage

```yaml
- name: Common tasks
  hosts: all
  become: yes
  roles:
  - common

- name: Install loadbalancer
  hosts: loadbalancer
  become: yes
  roles:
  - loadbalancer

- name: Install k3s on masters
  hosts: m1:masterx
  become: yes
  roles:
  - k3s_master

- name: Install k3s on workers
  hosts: worker
  become: yes
  roles:
  - k3s_worker

- name: Cleanup
  hosts: all
  become: yes
  roles:
  - cleanup
```
## Sample inventory with required params
```yaml
all:
  children:
    m1:
      hosts:
        m1:
          ansible_host: 192.168.1.50
    masterx:
      hosts:
        master2:
          ansible_host: 192.168.1.51
        master3:
          ansible_host: 192.168.1.52
    worker:
      hosts:
        worker1:
          ansible_host: 192.168.1.53
        worker2:
          ansible_host: 192.168.1.54
        worker3:
          ansible_host: 192.168.1.55
    loadbalancer:
      hosts:
        lb1:
          ansible_host: 192.168.1.56
        lb2:
          ansible_host: 192.168.1.57
```
Using github actions against local environment for validating and testing idempotency.

## Dynamically fetch hosts/inventory from vSphere
```yaml
plugin: vmware_vm_inventory
strict: False
hostname: "{{ VMWARE_HOST }}"
username: "{{ VMWARE_USER }}"
password: "{{ VMWARE_PASSWORD }}"
validate_certs: False
with_tags: True
hostnames:
- 'config.name'
properties:
- 'config.name'
- 'config.annotation'
- 'guest.ipAddress'
filters:
- "'loadbalancer' in config.annotation or 'master1' in config.annotation or 'masterx' in config.annotation or 'worker' in config.annotation"
groups:
  loadbalancer: "'loadbalancer' in config.annotation"
  master1: "'master1' in config.annotation"
  masterx: "'masterx' in config.annotation"
  worker: "'worker' in config.annotation"
```
Uncomment ansible.cfg line:
```cfg
[defaults]
hostkey_checking = False

[inventory]
enable_plugins = vmware_vm_inventory
```

```
ansible-inventory -i inventory/dynamic.vmware.yml --list
```