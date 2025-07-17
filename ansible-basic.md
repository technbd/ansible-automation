## Ansible Basic: 

Ansible is a powerful automation tool used for:

- **Configuration management**: Automate server setup, manage services, and configurations.
- **Application deployment**: Deploy and manage software applications.
- **Orchestration**: Coordinate complex, multi-step workflows, such as CI/CD pipelines.



### Key Components of Ansible:

_Ansible consists of several key components:_
- **Inventory**: A list of servers (hosts) Ansible manages, which can be in the form of an inventory file or dynamic inventory.
- **Module**: Small programs that Ansible runs to perform system changes (e.g., managing files, services).
- **Playbook**: A YAML file that defines a series of tasks to be executed on the hosts.
- **Task**: The individual unit of action in Ansible, such as installing a package or starting a service.
- **Role**: A way to organize tasks, handlers, and variables in a structured way for reuse and sharing.
- **Handlers**: Tasks triggered by changes, usually at the end of playbook execution.
- **Variables**: Store data dynamically, allowing reusability and customization.
- **Templates**: Generate dynamic files using the Jinja2 templating engine.



### What Does "Push-Based" Mean?
In a push-based model, the controller node (192.168.10.190) pushes configurations and commands over SSH to managed nodes (e.g., 192.168.10.193), without any agents running on the managed nodes.



### Ansible is Push-Based by Default:
- You write a playbook or run a command on the controller.
- Ansible uses SSH to connect to the managed node.
- Tasks are executed immediately on the target.
- No daemon or agent is needed on the managed system.



### Push-Based Method: 

_Push-Based Ansible, which consists of these key components:_
- Ad-hoc Command : Quick one-time task
- Playbook       : Declarative config for multi-step tasks to be executed in sequence on one or more managed nodes.





## Ansible Modules:

An Ansible module is a reusable, standalone script Ansible runs to perform a specific task (e.g., create a user, install a package, modify a file, manage services). Ansible modules are the building blocks used in ad-hoc commands and playbooks. They perform system tasks like installing packages, managing users, services, files, etc.

_You use modules with:_
- Ad-hoc commands
- Playbooks



_List all modules installed:_
```
ansible-doc -l
```


_To view specific module:_
```
ansible-doc <module_name>

ansible-doc file
```


### Core System Modules:
Used to manage files, packages, users, services, etc.

| Module       | Purpose                                     |
| ------------ | ------------------------------------------- |
| `file`       | Manage file properties (permissions, owner) |
| `copy`       | Copy files to remote nodes                  |
| `template`   | Deploy Jinja2 templates                     |
| `user`       | Manage user accounts                        |
| `group`      | Manage groups                               |
| `service`    | Control system services                     |
| `dnf`, `yum` | Install/remove packages (RedHat)            |
| `apt`        | Install packages (Debian/Ubuntu)            |
| `command`    | Run simple shell commands                   |
| `shell`      | Run shell with features like pipes          |
| `stat`       | Get file properties                         |



####  Networking Modules: 

| Module                       | Purpose                      |
| ---------------------------- | ---------------------------- |
| `firewalld`                  | Manage Firewalld             |
| `iptables`                   | Manage iptables rules        |
| `nmcli`                      | Control NetworkManager       |
| `community.general.hostname` | Set hostname                 |
| `ufw`                        | Manage UFW (Ubuntu firewall) |
| `ping`                       |  |


#### Storage Modules:

| Module   | Purpose                   |
| -------- | ------------------------- |
| `mount`  | Mount/unmount filesystems |
| `parted` | Partition disks           |
| `lvg`    | LVM volume group          |
| `lvol`   | LVM logical volume        |


#### Database Modules:

| Module         | Purpose                      |
| -------------- | ---------------------------- |
| `postgresql_*` | Manage PostgreSQL users, DBs |
| `mysql_*`      | Manage MySQL/MariaDB         |
| `mongodb_*`    | Manage MongoDB resources     |



#### Security Modules:

| Module           | Purpose                    |
| ---------------- | -------------------------- |
| `seboolean`      | Set SELinux booleans       |
| `selinux`        | Manage SELinux mode        |
| `authorized_key` | Add public SSH key to user |
| `iptables`       | Manage firewall rules      |
| `pam_limits`     | Set PAM limits             |



#### Container Modules:

| Module             | Purpose                   |
| ------------------ | ------------------------- |
| `docker_container` | Manage Docker containers  |
| `docker_image`     | Manage Docker images      |
| `podman_container` | Manage Podman containers  |
| `k8s`              | Manage Kubernetes objects |




#### Cloud Modules:

| Cloud        | Common Modules/Collections               |
| ------------ | ---------------------------------------- |
| AWS          | `amazon.aws.ec2`, `ec2_instance`, `s3`   |
| Azure        | `azure.azcollection.azure_rm_*`          |
| Google Cloud | `google.cloud.gcp_compute_instance`      |
| VMware       | `community.vmware.vmware_guest`          |
| OpenStack    | `openstack.cloud.server`, `volume`, etc. |




#### Monitoring / Notification Modules:

| Module       | Purpose                |
| ------------ | ---------------------- |
| `slack`      | Send messages to Slack |
| `mail`       | Send email alerts      |
| `debug`      | Print output to CLI    |
| `logentries` | Log to Logentries      |



#### Source Control Modules:

| Module       | Purpose                   |
| ------------ | ------------------------- |
| `git`        | Clone/pull repositories   |
| `subversion` | Checkout SVN repositories |




---
---



## Ad-hoc Commands: 

Ad-hoc commands are one-liner commands run from the controller to perform quick tasks on managed nodes — without writing a playbook.

- `-host-pattern` : the managed hosts to run against
- `-m` : Specify module to run
- `-a` : Provide module arguments to required by the module
- `-i` : Specify inventory file
- `--become`, `-b` : in Ansible is used to **escalate privileges**, meaning it allows Ansible to execute tasks with `sudo` or **root permissions** on the target (managed) node.
- `-u` : remote SSH user


_Syntax:_
```
ansible [-i INVENTORY] [server] [-m MODULE] {-a MODULE_OPTIONS}

ansible [host-pattern] -m [module] -a “[module options]”
ansible <host-pattern> -m <module> -a "<arguments>" [options]
```



### `shell` and `command` Modules:

a. `shell` Module:
- Runs via `/bin/sh` (or defined shell)
- Less secure if user input is passed (risk of shell injection)
- Slightly slower due to shell overhead
- Supports **pipes, redirection, globs, variables**, etc.

b. `command` Module (Recommended for Simple Commands):
- Safe: No shell is invoked
- Faster and more secure
- No shell features (e.g., pipes `|`, redirects `>`, variables `$VAR`, etc.)


#### `shell` Modules:
_Create a directory:_
```
ansible node1 -m shell -a "mkdir -p /opt/myapp"

ansible node1 -m shell -a "ls -l /opt"
```


_Check disk/memory usage:_
```
ansible node1 -m shell -a 'df -h' 
ansible node1 -m shell -a 'df -h /dev/sda2' 

ansible node1 -m shell -a 'free -h'

ansible node1 -m shell -a 'uptime'
```


_Run multiple commands with pipe:_
```
ansible node1 -m shell -a "cat /etc/passwd | grep root"
```


_Redirect output to a file:_
```
ansible node1 -m shell -a "date >> /tmp/date.log" --become
```



_To executes the raw shell command for service:_
```
ansible node1 -m shell -a "systemctl stop httpd" --become
ansible node1 -m shell -a "systemctl start httpd" --become
ansible node1 -m shell -a "systemctl restart httpd" --become
```


_Check listening ports:_
```
ansible node1 -m shell -a "netstat -tlpn"
```


#### `command` Modules:

_Create a directory:_
```
ansible node1 -m command -a "mkdir -p /opt/mydir"

ansible node1 -m command -a "ls -l /opt"
```


_Copy a file on the remote machine (using cp):_
```
ansible node1 -m command -a "cp /etc/hosts /tmp/hosts.bak" --become

ansible node1 -m command -a "cat /tmp/hosts.bak" --become
```



_Run a script:_
```
ansible node1 -m command -a "/tmp/backup.sh" --become
```



_Check system uptime:_
```
ansible node1 -m command -a 'uptime' 

ansible node1 -m command -a "free -h"
```





### `file` Modules: 

The Ansible file module is used to manage files and file properties such as permissions, ownership, symbolic links, directories, etc., on the managed nodes.

_Common Use Cases:_
- Create a directory
- Delete a file or directory
- Change ownership or permissions
- Create empty files
- Create a symbolic link


_Create a Directory:_
```
ansible web -m file -a 'path=/root/mydir state=directory'
```


_Delete a Directory:_
```
ansible web -m file -a "path=/root/mydir state=absent"
```


_Creates directory with proper permissions:_
```
ansible web -m file -a "path=/root/mydir2 state=directory owner=user2 group=user2 mode=0754"
```


_Create a File:_
```
ansible node1 -m file -a 'path=/root/mydir/myfile.txt state=touch'
```


Change Ownership and Permissions of a File:
```
ansible node1 -m file -a "path=/root/mydir/myfile.txt owner=user2 group=user2 mode=0655"
```


_Delete a File:_
```
ansible node1 -m file -a 'path=/root/mydir/myfile.txt state=absent' --become
```


_Create a Symbolic Link:_
```
ansible node1 -m file -a "src=/root/mydir/myfile.txt dest=/tmp/myfile.txt state=link"
```




### `copy` Modules: 
The Ansible copy module is used to **copy files from the controller node to a managed node**. It's commonly used in ad-hoc commands and playbooks.


_Copy a File to Remote Node:_
```
ansible node1 -m copy -a 'src=/etc/hosts dest=/tmp/hosts'
```


_Copy a File and Set Permissions:_
```
ansible node1 -m copy -a "src=/root/ansible.txt dest=/tmp/ansible.txt owner=user2 group=user2 mode=0644"
```


_Copy a Script and Make it Executable:_
```
ansible node1 -m copy -a "src=./backup.sh dest=/tmp/backup.sh mode=0755" --become
```


_Copy a Directory:_
The `copy` module does **not support** copying entire directories directly. Instead, use `synchronize` or `unarchive` for that. For directory-level copying, use `ansible.posix.synchronize` (requires `rsync`).

```
dnf install rsync -y  ### install on both machines (controller and managed nodes)

ansible node1 -m synchronize -a "src=/etc/ansible dest=/tmp" --become
ansible node1 -m ansible.posix.synchronize -a "src=/etc/ansible dest=/tmp recursive=yes" --become
```



### `user` and `group` Modules: 

_Create a User:_
```
ansible node1 -m user -a "name=devops state=present"

ansible node1 -m user -a "name=devops1 shell=/bin/bash create_home=yes" --become
```

_Delete a User:_
```
ansible node1 -m user -a "name=devops1 state=absent remove=yes" --become
```



_Create a Group:_
```
ansible node1 -m group -a "name=dev state=present" --become
```


_Delete a Group:_
```
ansible node1 -m group -a "name=dev state=absent" --become
```


_Create User with a Group:_
```
ansible node1 -m user -a "name=devops2 group=dev create_home=yes" --become
```


_Verify the Result:_
```
ansible node1 -m shell -a "id devops2" --become
```



_Set Encrypted Password:_

Explanation:
- `-6` : Use SHA-512 hashing (as per glibc crypt)
- `-salt NaJMuL` : Salt value (custom string)
- `mypassword` : The plain text password to hash

```
openssl passwd -6 -salt NaJMuL mypassword

or,

openssl passwd --method=SHA512 --salt=NaJMuL mypassword
```


```
ansible node1 -m user -a "name=user4 state=present password='<HASHED_PASSWORD>'" --become
```



### `yum` and `dnf` Modules:

_Install a Package:_
```
ansible node1 -m yum -a "name=telnet state=present"

ansible node1 -m dnf -a "name=htop state=present" --become

ansible node1 -m yum -a "name=telnet state=installed"
```


_Install Multiple Packages:_
```
ansible node1 -m dnf -a "name=vim,wget,curl state=present" --become
```


_Install from a `.rpm` File:_
- The `.rpm` file must exist on the remote node already. You can upload it with the `copy` module first.

```
ansible node1 -m dnf -a "name=/tmp/custom.rpm state=present" --become
```


_Update a Package (install and upgrade to latest version):_
```
ansible node1 -m dnf -a "name=telnet state=latest"
```


_Remove (Uninstall) a Package:_
```
ansible node1 -m dnf -a "name=telnet state=absent"
```




### `service` Modules:

a. `service` Module (Generic/Legacy):
- Supports both `SysV` and `systemd` init systems.
- Works across a wide range of OS types (RedHat, Ubuntu, older distros).
- Detects and delegates to the correct service manager (systemd, upstart, etc.).
- Portable, but not as precise with systemd features.
- Best for: Cross-platform compatibility and older distros (CentOS 6, Debian 7, etc.)


b. `systemd` Module (Preferred for Rocky Linux 8/9+):
- Directly uses `systemctl` to manage services.
- Full support for `systemd` features (like `masked`, `daemon_reload`, `scope`).
- More reliable and powerful on modern Linux systems using systemd (which includes Rocky Linux, RHEL 7+).
- Best for: Modern systems like Rocky Linux and when you need full control over systemd


_Syntax:_
```
ansible <host> -m service -a "name=<service_name> state=<state>" --become
```


_Start a Service:_
```
ansible node1 -m service -a "name=httpd state=started" --become

ansible node1 -m service -a "name=httpd state=started enabled=yes" --become
```


_Stop a Service:_
```
ansible node1 -m service -a "name=httpd state=stopped"

ansible node1 -m service -a "name=firewalld state=stopped enabled=no"
```


_Restart a Service:_
```
ansible node1 -m service -a "name=httpd state=restarted"
```


_`systemd` Module:_
```
ansible node1 -m systemd -a "name=httpd state=started enabled=yes"
ansible node1 -m systemd -a "name=httpd state=stopped"

ansible node1 -m systemd -a "name=httpd state=restarted"
```



### `stat` Modules: 
The `stat` module in Ansible is used to check the status of a file or directory on a remote host.

```
ansible node1 -m stat -a "path=/etc/hosts"

ansible node1 -m stat -a "path=/var/log"
```




### `selinux` Modules:

_Check SELinux Status:_
- This will return current mode: `enforcing`, `permissive`, or `disabled`.
```
ansible node1 -m setup -a "filter=ansible_selinux"
```


_Change SELinux Mode (Temporarily):_
```
ansible node1 -m selinux -a "policy=targeted state=permissive" --become

ansible node1 -m ansible.posix.selinux -a "policy=targeted state=permissive" --become
```



### `firewalld` Modules:

_Add a Service (e.g., HTTP):_
```
ansible node1 -m firewalld -a "service=http permanent=yes state=enabled immediate=yes" --become
```


_Add a Port:_
```
ansible node1 -m firewalld -a "port=8080/tcp permanent=yes state=enabled immediate=yes" --become
```


_Change Default Zone:_
```
ansible node1 -m command -a "firewall-cmd --get-default-zone" --become

ansible node1 -m command -a "firewall-cmd --set-default-zone=public" --become
```



_Remove a Service/Port:_
```
ansible node1 -m firewalld -a "service=http permanent=yes state=disabled immediate=yes" --become

ansible node1 -m firewalld -a "port=8080/tcp permanent=yes state=disabled immediate=yes" --become
```


_Reload firewalld:_
```
ansible node1 -m command -a "firewall-cmd --reload" --become

ansible node1 -m command -a "firewall-cmd --list-all" --become
```



_Add a Rich Rule:_
```
ansible node1 -m firewalld -a "rich_rule='rule family=ipv4 source address=192.168.10.44 port port=22 protocol=tcp accept' permanent=yes state=enabled immediate=yes" --become
```


_Remove a rich rule:_
```
ansible node1 -m firewalld -a "rich_rule='rule family=ipv4 source address=192.168.10.44 port port=22 protocol=tcp accept' permanent=yes immediate=yes state=disabled" --become
```




### `ufw` Modules:
Ansible does not have a built-in `ufw` module officially maintained like `firewalld`. But there are some ways to manage UFW (Uncomplicated Firewall) on Ubuntu/Debian systems via Ansible:

_Enable UFW and allow SSH:_
```
ansible node1 -m shell -a "ufw enable" --become

ansible node1 -m shell -a "ufw allow ssh" --become
```


_Deny a port:_
```
ansible node1 -m shell -a "ufw deny 8080" --become
```


_Reset UFW:_
```
ansible node1 -m shell -a "ufw reset" --become
```




### `ping` Modules:

```
ansible all -m ping
ansible node1 -m ping

ansible node1 -m shell -a "ping -c 5 8.8.8.8"
```



### `mount` Modules:

_Syntax:_

```
ansible <host> -m mount -a "path=<mount_point> src=<device_or_remote> fstype=<type> opts=<options> state=<mounted|unmounted|present|absent>" --become
```


_Mount a device temporarily (does not modify fstab):_
```
ansible node1 -m mount -a "path=/data src=/dev/sdb1 fstype=ext4 state=mounted" --become

ansible node1 -m mount -a "path=/data src=/dev/my_vg01/data_lv01 fstype=ext4 state=mounted" --become
```


```
ansible node1 -m shell -a "df -h"
```


_Mount and add to /etc/fstab (persistent mount):_
```
ansible node1 -m mount -a "path=/data src=/dev/my_vg01/data_lv01 fstype=ext4 opts=defaults state=present" --become
```


_Unmount a mount point but keep fstab entry:_
```
ansible node1 -m mount -a "path=/data state=unmounted" --become
```



_Remove mount from fstab (does not unmount):_
```
ansible node1 -m mount -a "path=/data state=absent" --become
```



_Mount an NFS share persistently:_
```
ansible node1 -m mount -a "path=/nfs_share src=192.168.1.10:/export nfs opts=_netdev,state=mounted" --become
```



### `postgresql` Modules:

Ensure `psycopg2` or `psycopg2-binary` is installed on the managed host:
```
dnf install python3-psycopg2

Or,

pip3 install psycopg2-binary
```


_Create a PostgreSQL Database:_
```
ansible node1 -m community.postgresql.postgresql_db -a "name=mydb state=present login_user=postgres" --become
```


_Create a PostgreSQL User:_
```
ansible node1 -m community.postgresql.postgresql_user -a "name=myuser password=mypass state=present login_user=postgres" --become
```


_To Change Database Owner:_
```
ansible node1 -m community.postgresql.postgresql_db -a "name=mydb owner=myuser login_user=postgres" --become
```



_Grant Privileges to User on DB:_
```
ansible node1 -m community.postgresql.postgresql_privs -a "db=mydb privs=ALL type=database roles=myuser state=present login_user=postgres" --become
```



_Drop a Database:_
```
ansible node1 -m community.postgresql.postgresql_db -a "name=mydb state=absent login_user=postgres" --become
```


_Drop a User:_
```
ansible node1 -m community.postgresql.postgresql_user -a "name=myuser state=absent login_user=postgres" --become
```





### `mysql` Modules:

Ensure `PyMySQL` is installed on the managed host:

```
dnf install -y python3-PyMySQL

or,

pip3 install PyMySQL
```


_Create a Database:_
```
ansible node1 -m community.mysql.mysql_db -a "name=mydb state=present login_user=root" --become

ansible node1 -m community.mysql.mysql_db -a "name=mydb2 state=present login_user=root login_password=Admin@123" --become
```


_Create a MySQL User:_
```
ansible node1 -m community.mysql.mysql_user -a "name=myuser password=mypassword priv='*.*:ALL' state=present login_user=root login_password=Admin@123" --become
```


_Grant Privileges to a User on a DB:_
```
ansible node1 -m community.mysql.mysql_user -a "name=myuser password=mypassword priv='mydb.*:ALL' state=present login_user=root login_password=Admin@123" --become
```



_Drop a Database:_
```
ansible node1 -m community.mysql.mysql_db -a "name=mydb state=absent login_user=root login_password=Admin@123" --become
```


_Delete a User:_
```
ansible node1 -m community.mysql.mysql_user -a "name=myuser state=absent login_user=root login_password=Admin@123" --become
```





### `docker_image` Modules:
Here’s how to use the Ansible `docker_image` module (from `community.docker`) in ad-hoc mode to manage Docker images.

_Pull a Docker Image:_
```
ansible node1 -m community.docker.docker_image -a "name=nginx source=pull" -b
ansible node1 -m community.docker.docker_image -a "name=nginx-alpine source=pull" -b
```


_Build an Image from a Dockerfile:_
- `build.path=/opt/myapp` : Directory containing the `Dockerfile`
- `name=mycustom_image` : Name for the new image

```
ansible node1 -m community.docker.docker_image -a "name=mycustom_image build.path=/opt/myapp" -b
```


_Remove an Image:_
```
ansible node1 -m community.docker.docker_image -a "name=nginx state=absent" -b
```




### `docker_container` Modules:

Here’s a simple ad-hoc Ansible example using the `docker_container` module (from the `community.docker` collection) to run a Docker container:


_Start Nginx Container (Port 8080 → 80):_
```
ansible node1 -m community.docker.docker_container -a "name=nginx image=nginx-alpine state=started published_ports=8080:80 restart_policy=always" -b
```


_Start a Container with Env Variable and Volume:_
```
ansible node1 -m community.docker.docker_container -a "name=myapp image=myapp:latest state=started env='APP_ENV=prod' volumes='/opt/myapp:/app'" -b
```


_Stop and Remove a Container:_
```
ansible node1 -m community.docker.docker_container -a "name=nginx state=absent" -b
```




Ansible is a powerful tool for automation, configuration management, and orchestration, and can be easily scaled from small-scale environments to large enterprise systems with tools like Ansible Tower.










