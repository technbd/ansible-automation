## Ansible Playbook: Install PostgreSQL 16 on Rocky Linux 8

This Ansible playbook installs and configures **PostgreSQL 16** on a **Rocky Linux 8** host. It uses the official PostgreSQL YUM repository, disables the built-in DNF module, installs necessary packages including `psycopg2`, initializes the database, and configures PostgreSQL to listen on all interfaces.


### Prerequisites

- Ansible installed on the control machine
- Target system: Rocky Linux 8
- SSH access to the target host 
- `become: true` privileges (sudo) on the target host


### What It Does:

- Adds the official PostgreSQL YUM repository
- Imports the GPG key for secure package installation
- Disables the default PostgreSQL DNF module
- Enables the correct PostgreSQL 16 repository explicitly
- Installs PostgreSQL server, client, and Python driver (`psycopg2`)
- Initializes the PostgreSQL database
- Configures PostgreSQL to listen on all IP addresses (`listen_addresses = '*'`)
- Starts and enables the PostgreSQL service


### Playbook:

```
---
- name: Install and Configure PostgreSQL on Rocky Linux 8
  hosts: node1
  become: yes

  vars:
    pg_version: 16

  tasks:
    - name: Download PostgreSQL official YUM repository RPM
      get_url:
        url: https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
        dest: /tmp/pgdg-redhat-repo-latest.noarch.rpm

    - name: Install PostgreSQL YUM repository RPM
      yum:
        name: /tmp/pgdg-redhat-repo-latest.noarch.rpm
        state: present

#    - name: Download PG GPG keys
#      get_url:
#        url:
#        dest:
#        mode: '0644'

    - name: Import PostgreSQL GPG key
      rpm_key:
        #key: https://download.postgresql.org/pub/repos/yum/RPM-GPG-KEY-PGDG
        key: /etc/pki/rpm-gpg/PGDG-RPM-GPG-KEY-RHEL
        state: present

    - name: Install yum-utils
      yum:
        name: yum-utils
        state: present

    - name: Disable built-in PostgreSQL DNF module
      command: dnf -qy module disable postgresql
      args:
        creates: /etc/dnf/modules.d/postgresql.module

    - name: Enable PostgreSQL {{ pg_version }} repo explicitly
      command: dnf config-manager --set-enabled pgdg{{ pg_version }}

    - name: Install PostgreSQL {{ pg_version }} server, client, and psycopg2
      yum:
        name:
          - postgresql{{ pg_version }}
          - postgresql{{ pg_version }}-server
          - python3-psycopg2
        state: present

    - name: Initialize PostgreSQL database
      command: /usr/pgsql-{{ pg_version }}/bin/postgresql-{{ pg_version }}-setup initdb
      args:
        creates: /var/lib/pgsql/{{ pg_version }}/data/PG_VERSION

    - name: Configure PostgreSQL to listen on all interfaces
      lineinfile:
        path: /var/lib/pgsql/{{ pg_version }}/data/postgresql.conf
        regexp: '^#?listen_addresses\s*='
        line: "listen_addresses = '*'"
        state: present

    - name: Enable and start PostgreSQL service
      service:
        name: postgresql-{{ pg_version }}
        state: started
        enabled: yes

```


### Run the Playbook:

```
ansible-playbook postgresql-install.yaml
```



### Customization:
You can customize the PostgreSQL version by changing the `pg_version` variable:

```
vars:
  pg_version: 16
```


### License:
This project is licensed under the **MIT** License. 


### Author:

```
## Author: 

technbd

GitHub: https://github.com/technbd
```


### Ref:
- [PostgreSQL RPM repository GPG key: ](https://yum.postgresql.org/news/pgdg-rpm-repo-gpg-key-update/)

