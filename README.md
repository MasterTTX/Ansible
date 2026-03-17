# Домашняя работа к занятию "Ansible"

## Цель

Попробовать написать свой playbook Ansible и понять, как через него настраиваются сервера.

---

## Что сделал

* написал playbook `homework.yaml`
* сделал inventory (у меня 2 хоста)
* настроил подключение через пользователя `ansible`
* создал группу хостов `netology-ml`
* проверил, что хосты доступны через `ping`
* добавил установку пакетов через переменную:

  * net-tools
  * git
  * tree
  * htop
  * mc
  * vim
* сделал обновление пакетов (`update_cache`)
* скопировал файл `test.txt` на сервера
* создал группы и пользователей:

  * `devops_1`
  * `test_1`
* у пользователей автоматически создаются домашние директории

---

## inventory.ini

```ini
[netology-ml]
vm1 ansible_host=192.168.1.101
vm2 ansible_host=192.168.1.102

[netology-ml:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

## homework.yaml

```yaml
---
- name: Netology ML homework
  hosts: netology-ml
  become: true

  vars:
    packages:
      - net-tools
      - git
      - tree
      - htop
      - mc
      - vim
    users:
      - devops_1
      - test_1

  tasks:
    - name: Check hosts availability
      ansible.builtin.ping:

    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Install packages
      ansible.builtin.apt:
        name: "{{ packages }}"
        state: present
      when: ansible_os_family == "Debian"

    - name: Copy test file
      ansible.builtin.copy:
        src: test.txt
        dest: /tmp/test.txt
        owner: root
        group: root
        mode: '0644'

    - name: Create user groups
      ansible.builtin.group:
        name: "{{ item }}"
        state: present
      loop: "{{ users }}"

    - name: Create users with home directories
      ansible.builtin.user:
        name: "{{ item }}"
        group: "{{ item }}"
        create_home: yes
        shell: /bin/bash
        state: present
      loop: "{{ users }}"
```

---

## test.txt

```text
Hello from Ansible!
```

---

## Как запускал

```bash
ansible-playbook -i inventory.ini homework.yaml
```

---

## Результат

```text
PLAY [Netology ML homework] ****************************************************

TASK [Gathering Facts] *********************************************************
ok: [vm1]
ok: [vm2]

TASK [Check hosts availability] ************************************************
ok: [vm1]
ok: [vm2]

TASK [Update apt cache] ********************************************************
changed: [vm1]
changed: [vm2]

TASK [Install packages] ********************************************************
changed: [vm1]
changed: [vm2]

TASK [Copy test file] **********************************************************
changed: [vm1]
changed: [vm2]

TASK [Create user groups] ******************************************************
changed: [vm1]
changed: [vm2]

TASK [Create users with home directories] **************************************
changed: [vm1]
changed: [vm2]

PLAY RECAP *********************************************************************
vm1 : ok=7 changed=5 failed=0
vm2 : ok=7 changed=5 failed=0
```
