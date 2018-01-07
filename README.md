# ansible-role-server-update-reboot

Update server to latest packages, reboot server, and wait for the server to start up. Add more roles after this to continue installing/configuring server.  
Works with Redhat/CentOS and Ubuntu.  


Distros tested
------------

* Ubuntu 16.04 - It should work on older versions of Ubuntu/Debian based systems.
* CentOS 7.4


Dependencies
------------

none

ansible-vault
------------

none

.gitignore
------------

```
vi .gitignore
#Insert the following lines
.vaultpass
.retry
```


Default Settings
------------

- **default_sudo_nopass**: yes|no (default yes). yes = passwordless sudo.
- **update_default**: true|false (default true)
- **reboot_default**: true|false (default true)
- **server_upgrade_reboot_wait**: true|false (default true)
- **server_upgrade_ssh_port**: SSH port number (default 22)


Example Playbook server-update-reboot.yml
------------

```
---
- hosts: '{{inventory}}'
  become: yes
  roles:
  - server-upgrade-reboot
  - server-config-xyz
```


Prep
------------

- install ansible
- create keys
- ssh to client to add entry to known_hosts file
- configure client server authorized_keys
- run ansible commands

Usage
------------

Use all defaults to: update, reboot server, and wait for server to start up.
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=all-dev" -i hosts
```

Do not reboot server:
```
ansible-playbook server-update-reboot.yml --extra-vars "inventory=all-dev reboot_default=false" -i hosts
```


## Notes
### RHEL5
RHEL/CentOS 5 has a dependency that needs to be installed: python-simplejson  
This command will use the raw module to install it:
```
ansible centos5 -m raw -a "yum install -y python-simplejson" --ask-pass --su --ask-su-pass --extra-vars="ansible_ssh_user=username123" -i hosts-dev
```

### SELinux
If SELinux is enabled/permissive a dependency is needed: libselinux-python  
This command will use the raw module to install it:
```
ansible centos5 -m raw -a "yum install -y libselinux-python" --ask-pass --su --ask-su-pass --extra-vars="ansible_ssh_user=username123" -i hosts-dev
```
