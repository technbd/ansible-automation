## Ansible Playbook: 

Playbooks are YAML files that define a set of tasks to be executed in sequence on one or more managed nodes. Ansible playbooks are saved as file `.yaml` or `yml`.


_This playbook contains the following elements:_

| Element                | Purpose                                                                                                    |
| ---------------------- | ---------------------------------------------------------------------------------------------------------- |
| `---`                  | Indicates the **start** of a YAML document (playbook).                                                     |
| `name`                 | A **human-readable description** of the play or playbook. Appears in Ansible's output when the play runs.  |
| `hosts`                | Defines which **inventory group or host(s)** to target (e.g., `all`, `webservers`).                        |
| `become: yes`          | Runs the tasks with **elevated privileges** (like `sudo`).                                                 |
| `become_user: root`    | Specifies the **user to become**, similar to running `sudo su - root`. Default is `root` if not specified. |
| `vars`                 | Declares **variables** used in the playbook. In this example, `greeting = Hello World!`.                   |
| `tasks`                | A list of **instructions** that will be executed on the remote host(s).                                    |
| `...`                  | (Optional) Denotes the **end** of the YAML file, though it's rarely used in real-world playbooks.          |



### Playbook Structure:

```
---
- name: <Name of the play>
  hosts: <Target group from inventory>
  become: <true/false - run as sudo?>
  vars:               # Optional variables
    key: value

  tasks:              # Main section
    - name: <Task description>
      <module_name>:    # Example: yum/dnf, copy, service
        option1: value
        option2: value
```


#### Task Structure: 
_Each task typically includes:_

- `name`: A description of the task.
- `module_name`: The Ansible module to use (yum/dnf, copy, service, etc.).
- `arguments`: Options for the module.


```
- name: <Description of the task>         # Required for readability (not execution)
  module_name:                            # Example: yum/dnf, copy, service
    option1: value                        # Module-specific options
    option2: value
```



### Example: Simple Playbook: 

```
- name: Install Apache and start service
  hosts: node1
  become: yes   # use for 'sudo'
  vars:
    http_port: 80  # Define a variable
  tasks:
    - name: Install httpd package
      dnf:
        name: httpd
        state: present

    - name: Start and enable service
      service:
        name: httpd
        state: started
        enabled: true
```


_Syntax check of the playbook:_
```
ansible-playbook httpd-install.yaml --syntax-check 
```


_To perform a dry-run of the playbook:_
```
ansible-playbook httpd-install.yaml --check
```


_To run playbook:_
```
ansible-playbook httpd-install.yaml
```


### Ansible `gather_facts`:

In Ansible, “Gathering Facts” refers to the process of collecting detailed information (facts) about the remote target systems before running tasks.


#### What Are “Facts”?

_Facts are system-level variables automatically collected by Ansible, such as:_

| Fact Name                      | Example Value  | Description             |
| ------------------------------ | -------------- | ----------------------- |
| `ansible_hostname`             | `web01`        | Hostname of the machine |
| `ansible_os_family`            | `RedHat`       | OS family               |
| `ansible_distribution`         | `Rocky`        | Linux distribution name |
| `ansible_default_ipv4.address` | `192.168.1.10` | Default IP address      |
| `ansible_processor_cores`      | `4`            | Number of CPU cores     |


#### When to Use or Skip It
✅ Use when: 
- You want to use system facts in your playbook (e.g., detect OS, RAM, IP).

🚫 Skip when:
- You don’t need facts (faster playbook execution).
- You're working on a system where setup module might fail (e.g., restricted devices).


#### Example: 

```
---
- name: Gather facts demo
  hosts: node1
  become: yes
  gather_facts: true  # default is 'true'

  tasks:
    - name: Print facts
      ansible.builtin.debug:
        var: ansible_facts
        #msg: "This host is running {{ ansible_distribution }}"
```



### `file` Modules Playbook:

#### Create a Directory, Empty File and Set Ownership: 

```
---
- name: File module example playbook
  hosts: node1
  become: yes

  tasks:
    - name: Create a directory
      file:
        path: /opt/demo_dir
        state: directory
        mode: '0755'

    - name: Create an empty file
      file:
        path: /opt/demo_dir/empty_file.txt
        state: touch
        mode: '0644'

    - name: Set file owner and group
      file:
        path: /opt/demo_dir/empty_file.txt
        owner: root
        group: root
```



#### Delete a File and Directory: 

```
---
- name: File module example playbook
  hosts: node1
  become: yes

  tasks:
    - name: Delete the file
      file:
        path: /opt/demo_dir/empty_file.txt
        state: absent

    - name: Delete the directory
      file:
        path: /opt/demo_dir
        state: absent       # Deletes the directory (must be empty)
```



### `copy` Modules Playbook:

- `src`  : is relative to the playbook or role directory.
- `dest` : is the absolute path on the remote host.


#### Copy a File to a Remote Host:
```
---
- name: Copy a file to remote host
  hosts: node1
  become: yes

  tasks:
    - name: Copy hello.txt to /opt directory
      copy:
        src: /tmp/hello.txt
        dest: /opt/hello.txt
        mode: '0644'
```


#### Copy Inline Content as a File: 
```
---
- name: Copy inline content to a file
  hosts: node1
  become: yes

  tasks:
    - name: Write message to file
      copy:
        content: "This is a test file created by Ansible.\nHello World!"
        dest: /opt/test_message.txt
        mode: '0644'
```



#### Copy and Set Ownership and Permissions:

```
---
- name: Copy file with ownership and permissions
  hosts: node1
  become: yes

  tasks:
    - name: Copy a script and make it executable by root
      copy:
        src: /tmp/script.sh
        dest: /opt/script.sh
        owner: root
        group: root
        mode: '0755'
```



#### Copy Directory Recursively:

```
---
- name: Recursively copy a folder
  hosts: node1
  become: yes

  tasks:
    - name: Copy local directory to remote host
      copy:
        src: /opt
        dest: /var/www/html
        mode: '0755'
```






### `user` and `group` Modules Playbook:


#### Create a User:

```
---
- name: Create a user 
  hosts: node1
  become: yes

  tasks:
    - name: Create user 'john'
      user:
        name: john
        shell: /bin/bash
        create_home: yes
        state: present
```


#### Create a Group:

```
---
- name: Create a group
  hosts: node1
  become: yes

  tasks:
    - name: Create a group 'developers'
      group:
        name: developers
        state: present
```


#### Create a User and Add to Group:

```
---
- name: Create user and add to group
  hosts: node1
  become: yes

  tasks:
    - name: Create user 'jon' and add to 'developers' group
      user:
        name: jon
        group: developers
        shell: /bin/bash
        create_home: yes
        state: present
```


#### Add User to Multiple Groups: 

```
---
- name: Add user to multiple groups
  hosts: node1
  become: yes

  tasks:
    - name: Add 'jony' to extra groups
      user:
        name: jony
        groups: developers,devops
        append: yes
```


#### Delete a User and Group:

```
---
- name: Remove user and group
  hosts: node1
  become: yes

  tasks:
    - name: Remove user 'jony'
      user:
        name: jony
        state: absent
        remove: yes

    - name: Remove group 'devops'
      group:
        name: devops
        state: absent
```








### `yum` and `dnf` Modules Playbook:


#### Install a Single Package:
```
---
- name: Install package using yum
  hosts: node1
  become: yes

  tasks:
    - name: Install httpd using yum
      yum:  ## or use 'dnf'
        name: telnet
        state: present
```


#### Install Multiple Packages:
```
---
- name: Install multiple packages
  hosts: node1
  become: yes

  tasks:
    - name: Install git, vim, wget
      yum:
        name:
          - git
          - vim
          - wget
        state: present
```


#### Remove a Package:
```
---
- name: Remove a package
  hosts: node1
  become: yes

  tasks:
    - name: Remove telnet
      yum:
        name: telnet
        state: absent
```


#### Copy `.rpm` from Control Machine and Install:

- This assumes `/tmp/custom_package.rpm` is already present on the remote host.

```
---
- name: Copy and install RPM package
  hosts: node1
  become: yes

  tasks:
    - name: Copy RPM to remote host
      copy:
        src: /opt/custom_package.rpm
        dest: /tmp/custom_package.rpm
        mode: '0644'

    - name: Install RPM using dnf
      dnf:
        name: /tmp/custom_package.rpm
        state: present
```


#### Install `.rpm` from Remote URL:
```
---
- name: Download and install RPM from URL
  hosts: node1
  become: yes

  tasks:
    - name: Download RPM package
      get_url:
        url: https://dl.rockylinux.org/pub/rocky/8/Devel/x86_64/os/Packages/s/screen-4.6.2-4.el8.x86_64.rpm
        dest: /tmp/custom_package.rpm
        mode: '0644'

    - name: Install RPM from downloaded file
      dnf:
        name: /tmp/custom_package.rpm
        state: present
```




### `service` Modules Playbook:

#### Start and Enable a Service:
```
---
- name: Start and enable a service
  hosts: node1
  become: yes

  tasks:
    - name: Install httpd (or replace with nginx)
      yum:
        name: httpd     # nginx
        state: present

    - name: Start and enable httpd/nginx service
      service:
        name: httpd        # nginx
        state: started     # or 'restarted' and 'stopped'
        enabled: yes       # Enables service on boot
```


#### Stop and Disable a Service:
```
---
- name: Stop and disable a service
  hosts: node1
  become: yes

  tasks:
    - name: Stop httpd service
      service:
        name: httpd
        state: stopped     # Immediately stops the service

    - name: Disable httpd at boot
      service:
        name: httpd
        enabled: no        # Prevents it from starting on boot
```



### `selinux` Modules Playbook:

The `selinux` module is used to manage SELinux state (e.g., enforcing, permissive, or disabled) on remote hosts.


#### Disable SELinux (reboot required):

```
---
- name: Disable SELinux
  hosts: node1
  become: yes

  tasks:
    - name: Disable SELinux permanently
      selinux:
        state: disabled   # 'permissive' or 'enforcing'
        policy: targeted  # Usually targeted (default on RHEL)
```



### `firewalld` Modules Playbook:

Make sure `firewalld` is installed and running on your managed hosts:

#### Open a Specific Port (e.g., 80/tcp):

```
---
- name: Open HTTP port using firewalld
  hosts: node1
  become: yes

  tasks:
    - name: Open port 80/tcp permanently and immediately
      firewalld:
        port: 80/tcp
        permanent: yes
        state: enabled
        immediate: yes
```


#### Enable a Firewalld Service (e.g., ssh, http, https): 
```
---
- name: Enable HTTP service
  hosts: node1
  become: yes

  tasks:
    - name: Enable firewalld http service
      firewalld:
        service: http
        state: enabled
        permanent: yes
        immediate: yes
```


#### Add a Rich Rule:
```
---
- name: Add firewalld rich rule
  hosts: node1
  become: yes

  tasks:
    - name: Allow SSH from specific IP using rich rule
      firewalld:
        rich_rule: 'rule family="ipv4" source address="192.168.10.44" port protocol="tcp" port="22" accept'
        state: enabled
        permanent: yes
        immediate: yes
```


#### Remove a Rich Rule:

```
---
- name: Remove firewalld rich rule
  hosts: node1
  become: yes

  tasks:
    - name: Remove specific rich rule (allow SSH from 192.168.10.44)
      firewalld:
        rich_rule: 'rule family="ipv4" source address="192.168.10.44" port protocol="tcp" port="22" accept'
        state: disabled
        permanent: yes
        immediate: yes
```



#### Remove a Port or Service:
```
---
- name: Remove firewalld rule
  hosts: node1
  become: yes

  tasks:
    - name: Remove port 8080/tcp or Service
      firewalld:
        #port: 80/tcp
        service=http
        state: disabled
        permanent: yes
        immediate: yes

```


#### Reload Firewalld:

```
---
- name: Reload firewalld 
  hosts: node1
  become: yes

  tasks:
    - name: Reload firewalld
      command: firewall-cmd --reload
```






### `ufw` Modules Playbook:

```

```










### `mount` Modules Playbook:

#### Mount a Partition Permanently: 
```
---
- name: Mount Volume
  hosts: node1
  become: yes

  tasks:
    - name: Create mount directory if not exists
      file:
        path: /data
        state: directory
        mode: '0755'

    - name: Mount /dev/sdb1 on /data
      mount:
        path: /data
        #src: /dev/sdb1
        src: /dev/my_vg01/data_lv01
        fstype: ext4
        opts: defaults
        state: mounted
```


#### Unmount a Partition:

```
---
- name: Unmount Partition 
  hosts: node1
  become: yes

  tasks:
    - name: Unmount and remove entry from /etc/fstab
      mount:
        path: /data
        state: absent  # or use 'unmounted'
```



### `lineinfile` Modules Playbook:


#### Add a line if it doesn't exist: 

This ensures the line `'192.168.10.194 node4'` exists in `/etc/hosts`. It will **add it if it's not there**, or do nothing if it already is.

```
---
- name: Add a line 
  hosts: node1
  become: yes

  tasks:
    - name: Add a line on hosts file 
      lineinfile:
        path: /etc/hosts
        line: '192.168.10.194 node4'
```


#### Remove a line: 

This removes any line matching `debug=true`.

```
---
- name: Remove a line
  hosts: node1
  become: yes

  tasks:
    - name: Remove a line on hosts file
      lineinfile:
        path: /etc/hosts
        #regexp: '^debug=true'
        regexp: '^192.168.10.194 node4'
        state: absent
```




#### Replace a line matching a pattern: 

1. Playbook: 
```
---
- name: Update configuration
  hosts: node1
  become: yes

  tasks:
    - name: Update on hosts file
      lineinfile:
        path: /etc/hosts
        regexp: '^192.168.10.194 node4'
        line: '192.168.10.194 node5'
```


2. Playbook: Update selinux Configuration

```
---
- name: Update selinux
  hosts: node1
  become: yes

  tasks:
    - name: dislable selinux
      lineinfile:
        path: /etc/selinux/config
        regexp: '^SELINUX='
        line: 'SELINUX=disabled'
```





#### Uncomment and update a config line:


1. Playbook: SSH Port Change: 

- `regexp: '^#?Port\s+'` — This matches both `Port` and `#Port`.
- `line: 'Port 2222'` — This enforces the desired line.
- `backup: yes` — Makes a backup of the original file before editing.


```
---
- name: Update SSH Configuration
  hosts: node1
  become: yes

  tasks:
    - name: Change SSH Port to 2222
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?Port\s+'
        line: 'Port 2222'
        state: present
        backup: yes

    - name: Restart SSH service
      service:
        name: sshd
        state: restarted
```



2. Playbook: PostgreSQL listening on '*': 

This replaces `#listen_addresses = 'localhost'` or `listen_addresses = 'localhost'` with `listen_addresses = '*'`.

```
---
- name: Update PostgreSQL Configuration
  hosts: node1
  become: yes

  tasks:
    - name: Ensure PostgreSQL listening on '*'
      lineinfile:
        path: /var/lib/pgsql/16/data/postgresql.conf
        regexp: '^#?listen_addresses\s*='
        line: "listen_addresses = '*'"
        state: present
        backup: yes

    - name: Restart PostgreSQL service
      service:
        name: postgresql-16
        state: restarted
```






An Ansible Playbook is a simple yet powerful tool to automate IT tasks such as software installation, configuration management, service control, user management, and more.







