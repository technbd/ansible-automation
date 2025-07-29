## Docker Installation on Rocky Linux 8 using Ansible

This Ansible playbook installs and configures Docker on a Rocky Linux 8 host.



### Prerequisites:

- Ansible installed on your control machine
- Target system: Rocky Linux 8 with SSH access
- User must have `sudo` privileges (`become: true` is used)



### What This Playbook Does:

1. Installs required dependencies (`yum-utils`, `device-mapper-persistent-data`, `lvm2`)
2. Adds the official Docker CE YUM repository
3. Installs the latest version of Docker CE and CLI components
4. Installs Docker Compose and Buildx plugins
5. Enables and starts the Docker daemon
6. Prints the installed Docker version




### Run the playbook:

```
ansible-playbook docker-install.yaml
```



### Ref:
- [Install Docker Engine on RHEL](https://docs.docker.com/engine/install/rhel/)
- [Install Docker Engine on Rocky](https://docs.rockylinux.org/gemstones/containers/docker/)
