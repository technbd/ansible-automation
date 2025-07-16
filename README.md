## Ansible:
Ansible is an open source automation tools. Ansible can help you with configuration management, application deployment, task automation and also IT orchestration. It is very simple to setup, efficient and more powerful.

It's widely known for being **simple**, **agentless**, and **powerful** — perfect for managing both small setups and large infrastructures.


_Ansible is an open-source automation tool used for:_
- Configuration management
- Application deployment
- Task automation
- Cloud provisioning
- Orchestration



### How Ansible Works:
- **YAML files** (called playbooks) to define automation tasks.
- **SSH** to connect to remote systems (no agent needed on managed nodes).
- A **control node** (your machine or server with Ansible installed) that pushes tasks to the managed nodes (target servers).


### Core Components:

| Component     | Description                                                                  |
| ------------- | ---------------------------------------------------------------------------- |
| **Inventory** | List of managed nodes (in INI, YAML, or dynamic format).                     |
| **Playbook**  | YAML file defining a series of tasks for Ansible to run.                     |
| **Task**      | A single automation step (e.g., install a package).                          |
| **Module**    | A reusable unit of work (e.g., `apt`, `yum`, `copy`, `template`).            |
| **Role**      | A collection of tasks, files, templates, and variables structured for reuse. |
| **Facts**     | System information collected from the managed nodes.                         |



### Use Cases:
- Install packages on multiple servers.
- Configure services (like NGINX, PostgreSQL).
- Deploy web applications.
- Set up Docker containers.
- Patch security updates automatically.
- Provision infrastructure on AWS, Azure, GCP.


### Key Benefits:
- **Agentless**: Uses SSH; no agent needed on nodes.
- **Idempotent**: Running the same playbook multiple times gives the same result.
- **Human-readable**: YAML-based configuration.
- **Extensible**: Easily integrate with custom scripts and roles.



### Prerequisites:
- One Ansible Control Node: The Ansible control node is the machine you’ll use to connect to and control the Ansible hosts over SSH. 
- One or more Ansible Hosts: An Ansible host is any machine that your Ansible control node is configured to automate. 
- A non-root user with `sudo` privileges. 
- Passwordless Authentication


#### Environments: 
- Controller node: 192.168.10.190
- Managed Node: 192.168.10.193



```
vim /etc/hosts

192.168.10.190  ansible-master
192.168.10.193  node1
192.168.10.194  node2
```




### Step-1: Install Ansible (on Controller Node):

#### On RHEL/Rocky: 

```
dnf install -y epel-release
```


```
yum install net-tools bash-completion -y 
```


```
dnf install ansible -y
```



#### On Ubuntu:

_To configure the PPA on your system and install Ansible run these commands:_
```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
```


```
sudo apt install ansible -y 
```


```
ansible --version
```



### Step-2: Passwordless Authentication:

#### Generate SSH Key (on Controller node):

```
ssh-keygen 
```


#### Copy the Public Key to Managed node:
```
ssh-copy-id root@192.168.10.193
ssh-copy-id root@192.168.10.194
```



### Step-3: Configuring Ansible Hosts:

#### Create Ansible Inventory File:

```
cp /etc/ansible/hosts /etc/ansible/hosts.bak
```


_Syntax:_
```
[group_name]
alias ansible_ssh_host=your_server_ip
```



```
vim /etc/ansible/hosts


### Add lines end of the file: 
[web]
node1 ansible_host=192.168.10.193

[db]
node2 ansible_host=192.168.10.194


[infra:children]
web
db
```



### Test the Connection:


_Check ping Validation:_
```
ansible -m ping node1
ansible -m ping node2
ansible -m ping all

ansible -m ping web
ansible -m ping db
ansible -m ping web:db

ansible -m ping infra
```


_Listing Inventory Hosts:_
```
ansible --list-hosts all
ansible --list-hosts web
ansible --list-hosts db
ansible --list-hosts infra
```



```
ansible -m shell -a 'free -h' node1
ansible -m shell -a 'free -h' web

ansible -m shell -a "uptime" web

ansible -m command -a "uptime" all
ansible -m command -a "uptime" 'node1'

ansible -m command -a "uname -r" 'node1'
ansible -m command -a "useradd user2" 'node1'
```




### Create a Simple Playbook:

```
- name: Install Apache on web servers
  hosts: web
  become: yes
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present

    - name: Ensure Apache is running
      service:
        name: httpd
        state: started
        enabled: true
```




_Check for the Syntax errors:_
```
ansible-playbook --syntax-check install-httpd.yaml
```


_To run the playbook:_
```
ansible-playbook install-httpd.yaml
```





### Ref:
- [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)



