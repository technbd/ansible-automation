## MongoDB 8.0 Installation on Rocky Linux 8 using Ansible

This Ansible playbook installs and configures MongoDB 8.0 on a Rocky Linux 8 server.



### Prerequisites:

- Ansible installed on your control machine
- A target Rocky Linux 8 host (e.g., `node1`) accessible via SSH
- `become: true` enabled (sudo access)
- Firewalld should be installed and active if you plan to expose MongoDB externally






### What It Does: 

1. Adds the official MongoDB 8.0 YUM repository
2. Installs the `mongodb-org` package
3. Enables and starts the `mongod` service
4. Updates the MongoDB config (`mongod.conf`) to listen on `0.0.0.0` (all interfaces)
5. Verifies the service status
6. Shows the installed MongoDB version



### Run the playbook:

```
ansible-playbook mongodb-install.yaml
```



### Ref:
- [Install MongoDB Community Edition](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-red-hat/)