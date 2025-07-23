## PHP 7.4 Installation on Rocky Linux 8.8 Using Ansible

This Ansible playbook installs and configures **PHP 7.4** and commonly used extensions on **Rocky Linux 8.8** using the **Remi repository**.



### What It Does:

- Installs EPEL repositories
- Download Remi repositories
- Downloads and imports all necessary Remi GPG keys
- Installs Remi repository RPM compatible with Rocky Linux 8.8
- Disables default built-in PHP module
- Enables Remi’s PHP 7.4 module
- Installs PHP 7.4 and common extensions
- Verifies PHP version after installation


---

### PHP Extensions Installed:

- `php`, `php-cli`, `php-common`
- `php-mysqlnd`, `php-pdo`, `php-devel`
- `php-gd`, `php-mbstring`, `php-xml`
- `php-curl`, `php-zip`, `php-bcmath`
- `php-fpm`



### Supported System:

- **OS**: Rocky Linux 8.8
- **PHP Version**: 7.4 (via `php_version` variable)


### Run Playbook::
```
ansible-playbook php-install.yaml
```


### License:
This project is licensed under the MIT License.

