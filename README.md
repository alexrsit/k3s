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
  hosts: master1:masterx
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
  vars:
    virtual_ip: 192.168.1.60
    ansible_user: "{{ lookup('env', 'ANSIBLE_USER') }}"
    ansible_ssh_pass: "{{ lookup('env', 'ANSIBLE_SSH_PASS') }}"
    ansible_become_password: "{{ lookup('env', 'ANSIBLE_BECOME_PASSWORD') }}"
  children:
    master1:
      hosts:
        master1:
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
        loadbalancer1:
          keepalived_state: "MASTER"
          keepalived_priority: 200
          ansible_host: 192.168.1.56
        loadbalancer2:
          keepalived_state: "BACKUP"
          keepalived_priority: 100
          ansible_host: 192.168.1.57
