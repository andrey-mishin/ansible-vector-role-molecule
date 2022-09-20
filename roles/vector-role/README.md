Role Name
=========

Базовая установка vector на centos7. 
Vector устанавливается в качестве сервиса из пакета.
Скачивание пакета производится с официального сайта https://vector.dev/download/.

Requirements
------------

Зависимостей не покрываемых Ansible или role нет.

Role Variables
--------------

Единственная используемая переменная - это версия пакета vector.
Задана в defaults/main.yml.
```yaml
vector_version: "0.23.0"
```

Dependencies
------------

Никаких зависимостей от других ролей нет.

Example Playbook
----------------

```yaml
- name: Install Vector
  hosts: vector
  handlers:
    - name: Start Vector
      become: true
      ansible.builtin.service:
        name: vector
        state: restarted
  tasks:
    - block:
        - name: Download Vector
          ansible.builtin.get_url:
            url: "https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-1.x86_64.rpm"
            dest: "./vector-{{ vector_version }}-1.x86_64.rpm"
          tags:
            - v-down
        - name: Install Vector
          become: true
          ansible.builtin.yum:
            name:
              - vector-{{ vector_version }}-1.x86_64.rpm
            state: latest
          notify: Start Vector
          ignore_errors: true
          tags:
            - v-inst
```

License
-------

BSD

Author Information
------------------

[Vector](https://vector.dev/docs/about/what-is-vector/)

Role by [Andrey Mishin](https://github.com/andrey-mishin)
