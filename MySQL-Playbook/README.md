
## Ansible Playbook: Install MySQL 8 on Rocky Linux 8

This Ansible playbook automates the installation and initial configuration of **MySQL 8.0** on a **Rocky Linux 8** host.


### Requirements:

- Ansible installed on the control node.
- A reachable target host (`node1`) running **Rocky Linux 8**.
- Passwordless sudo access or `--ask-become-pass`.


### What It Does: 

1. Download MySQL Repository RPM  - Downloads the MySQL 8.0 repository RPM to `/tmp`.

2. Install MySQL Repository   - Installs the downloaded MySQL repository RPM.

3. Download MySQL GPG Keys  - Downloads 3 official GPG keys to `/etc/pki/rpm-gpg`.

4. Import GPG Keys  - Imports the downloaded GPG keys to the system.

5. Update GPG Key Paths in MySQL Repo File  - Replaces the default GPG key line with specific file paths in the MySQL repo configuration.

6. Disable Built-in MySQL DNF Module   - Disables the system-provided MySQL module to prevent version conflicts.

7. Clean Yum Cache   - Clears all cached packages and metadata.

8. Install MySQL Server and PyMySQL   - Installs the **MySQL 8.0 server** and Python 3 **PyMySQL** library.

9. Enable and Start MySQL Service   - Starts and enables the `mysqld` service.

10. Fetch Temporary Root Password   - Extracts the temporary root password from MySQL logs.

11. Set Custom MySQL Root Password   - Updates the MySQL root password using the temporary one.

12. Configure MySQL to Listen on All Interfaces   - Adds `bind-address = 0.0.0.0` to `/etc/my.cnf` to allow external access.

13. Restart MySQL Service   - Restarts MySQL to apply configuration changes.



### Playbook: 

```
---
- name: Install and Configure MySQL 8 on Rocky Linux 8
  hosts: node1
  become: true

  vars:
    mysql_root_password: Admin@123

  tasks:
    - name: Download MySQL 8 Community Repo RPM
      get_url:
        url: https://repo.mysql.com/mysql80-community-release-el8-1.noarch.rpm
        dest: /tmp/mysql80-community-release-el8-1.noarch.rpm

    - name: Install MySQL 8 Community Repository RPM
      yum:
        name: /tmp/mysql80-community-release-el8-1.noarch.rpm
        state: present

    - name: Download MySQL GPG keys
      get_url:
        url: "{{ item.url }}"
        dest: "{{ item.dest }}"
        mode: '0644'
      loop:
        - { url: "https://repo.mysql.com/RPM-GPG-KEY-mysql",         dest: "/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql" }
        - { url: "https://repo.mysql.com/RPM-GPG-KEY-mysql-2022",    dest: "/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022" }
        - { url: "https://repo.mysql.com/RPM-GPG-KEY-mysql-2023",    dest: "/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023" }

    - name: Import MySQL GPG keys
      rpm_key:
        key: "{{ item }}"
        state: present
      loop:
        - /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
        - /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
        - /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023

    - name: Update GPG keys in MySQL repo file
      replace:
        path: /etc/yum.repos.d/mysql-community.repo
        regexp: '^gpgkey=.*'
        replace: |
          gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
                 file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
                 file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023

    - name: Disable built-in MySQL module
      command: dnf -qy module disable mysql
      args:
        creates: /etc/dnf/modules.d/mysql.module

    - name: Clean yum cache
      command: yum clean all

    - name: Install MySQL 8.0 Server and PyMySQL
      yum:
        name:
          - mysql-community-server
          - python3-PyMySQL
        state: present

    - name: Start and enable MySQL service
      service:
        name: mysqld
        state: started
        enabled: yes

    - name: Get temporary MySQL root password
      shell: grep 'temporary password' /var/log/mysqld.log | awk '{print $NF}'
      register: mysql_temp_pass
      changed_when: false

    - name: Change MySQL root password using shell
      shell: >
        mysql --connect-expired-password -uroot -p'{{ mysql_temp_pass.stdout }}'
        -e "ALTER USER 'root'@'localhost' IDENTIFIED BY '{{ mysql_root_password }}';"
      args:
        executable: /bin/bash

    - name: Allow MySQL to listen on all interfaces
      lineinfile:
        path: /etc/my.cnf
        regexp: '^bind-address\s*='
        line: 'bind-address = 0.0.0.0'
        create: yes
        state: present

    - name: Restart MySQL to apply bind-address change
      service:
        name: mysqld
        state: restarted

```



### Run Playbook: 

```
ansible-playbook mysql8-install.yaml
```



### Post-Installation: Accessing MySQL: (If needed)

```
grep 'temporary password' /var/log/mysqld.log
```


```
mysql -u root -p

ALTER USER 'root'@'localhost' IDENTIFIED BY 'Admin@123';
```



### Ref:
- [MySQL Yum Repository:](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/#repo-qg-yum-fresh-install)



