# ansible-role-server-update-reboot

Ansible role to update server to latest packages, reboot server, and wait for the server to start up. Add more roles after this to continue installing/configuring server.  
Can also exclude packages from being updated, only update specified packages, or install specified packages.  
Works with Redhat/CentOS and Ubuntu.  

Can be used to update packages for [Meltdown/Spectre Mitigation](#spectremeltdown-mitigation) for Redhat/CentOS 7 and Ubuntu 16.04  

More detailed example can be found in the blog post: [Using Ansible to Update Ubuntu, CentOS, and Redhat](https://ryandaniels.ca/blog/ansible-update-ubuntu-centos-redhat/)  

**Requires**: Ansible 2.7.1 (uses reboot module and 2.7.1 fixes reboot_timeout)

Note:  
This role can reboot the server if there is a kernel update and if the reboot variable is true (reboot is default setting).

## Distros tested

* Ubuntu 18.04 / 16.04
* CentOS & RHEL 7.x

## Group Variables

./group_vars/centos-dev/proxy.yml  
With a proxy:

```yaml
proxy_env:
  http_proxy: http://my.internal.proxy:80
  https_proxy: https://my.internal.proxy:80
```

With no proxy:

```yaml
proxy_env: []
```

## Default Settings

* **debug_enabled_default**: true|false (default false)
* **update_default**: true|false (default true)
* **reboot_default**: true|false (default true)
* **server_update_reboot_pre_delay**: Time (in seconds) to wait before running tasks in this role
* **server_update_reboot_pre_reboot_delay**: Time (in seconds) to wait before rebooting
* **server_update_reboot_post_reboot_delay**: Time (in seconds) to wait after rebooting
* **server_update_reboot_reboot_timeout**: Maximum time (in seconds) to wait for server to reboot

Variables for RHEL/CentOS:

* **server_update_yum_exclude_pkgs**: comma separated string of packages to exclude from update. Can use wildcards. (default [])
* **server_update_yum_install_pkgs**: comma separated string of packages to ONLY update. Can use wildcards. (default '*' meaning all packages)

Variables for Ubuntu:

* **server_update_apt_exclude_default**: true|false. set true if using exclude list below (default false)
* **server_update_apt_exclude_pkgs**: List of packages to not update (each on separate line). Can include wildcard (but use ^ to begin match or a lot will match) to match multiple packages. (default undefined)
* **server_update_apt_default**: full|update_specific|install (default full)
  * full: update all packages using "apt-get dist-upgrade"
  * update_specific: only update from list in variable server_update_apt_install_pkgs
  * install: only install from list in variable server_update_apt_install_pkgs
* **server_update_apt_install_pkgs**: List of packages to ONLY update or install (each on separate line). Can include wildcard to match multiple packages. (default undefined)

## Example Playbook server-update-reboot.yml

Below example playbook will update/reboot one server at a time (using max_fail_percentage and serial variables). If you want to update/reboot everything at once uncomment those lines.

```yaml
---
- hosts: '{{inventory}}'
  max_fail_percentage: 0
  serial: 1
  become: yes
  roles:
#  - stop-applications
  - server-update-reboot
#  - server-config-xyz
#  - start-applications
```

## Prep

* install ansible
* create keys
* ssh to client to add entry to known_hosts file
* configure client server authorized_keys
* run ansible commands

## Usage

### For Redhat/CentOS/Ubuntu

Use all defaults to: update, reboot server, and wait for server to start up:

```bash
ansible-playbook server-update-reboot.yml --extra-vars "inventory=all-dev" -i hosts-dev
```

Same as above, but do not reboot server:

```bash
ansible-playbook server-update-reboot.yml --extra-vars "inventory=all-dev reboot_default=false" -i hosts-dev
```

### For Redhat/CentOS

Update all packages except package(s) specified (for RHEL):

```bash
ansible-playbook server-update-reboot.yml --extra-vars 'inventory=centos-dev server_update_yum_exclude_pkgs="mysql*, bash, openssh*"' -i hosts-dev
```

Only update (or install) specific packages (for RHEL):

```bash
ansible-playbook server-update-reboot.yml --extra-vars "inventory=centos-dev server_update_yum_install_pkgs='kernel-*, iwl*firmware, microcode_ctl, dracut'" -i hosts-dev
```

### For Ubuntu

Update all packages except package(s) specified (for Ubuntu):

```bash
ansible-playbook server-update-reboot.yml --extra-vars 'inventory=ubuntu-dev server_update_apt_exclude_default=true' --extra-vars '{"server_update_apt_exclude_pkgs": [bash, openssl, ^mysql*, ^openssh*]}' -i hosts-dev
```

Only update specific packages (for Ubuntu):

```bash
ansible-playbook server-update-reboot.yml --extra-vars "inventory=ubuntu-dev server_update_apt_default=update_specific" --extra-vars "{'server_update_apt_install_pkgs': [linux-firmware, linux-generic, linux-headers-generic, linux-image-generic, intel-microcode, openssh*]}" -i hosts-dev
```

Only install specific packages (for Ubuntu). Be careful with wildcards:

```bash
ansible-playbook server-update-reboot.yml --extra-vars "inventory=ubuntu-dev server_update_apt_default=install" --extra-vars "{'server_update_apt_install_pkgs': [bash, openssh-server]}" -i hosts-dev
```

## Spectre/Meltdown Mitigation

To patch Redhat/CentOS 7 and Ubuntu 16.04, for [Spectre](https://spectreattack.com/) and [Meltdown](https://meltdownattack.com/) (CVE-2017-5754, CVE-2017-5753, CVE-2017-5715)  
Info from [Ubuntu](https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/SpectreAndMeltdown)  
Info from [Redhat](https://access.redhat.com/security/vulnerabilities/speculativeexecution)  

Or just patch everything using first command above.  

### For Redhat/CentOS 7 (Spectre/Meltdown Mitigation)

```bash
ansible-playbook server-update-reboot.yml --extra-vars "inventory=centos-dev server_update_yum_install_pkgs='kernel-*, iwl*firmware, microcode_ctl, dracut'" -i hosts-dev
```

### For Ubuntu 16.04 (Spectre/Meltdown Mitigation)

```bash
ansible-playbook server-update-reboot.yml --extra-vars "inventory=ubuntu-dev server_update_apt_default=update_specific" --extra-vars "{'server_update_apt_install_pkgs': [linux-firmware, linux-generic, linux-headers-generic, linux-image-generic, intel-microcode]}" -i hosts-dev
```

## Notes

### RHEL5

RHEL/CentOS 5 has a dependency that needs to be installed: python-simplejson  
This command will use the raw module to install it:

```bash
ansible centos5 -m raw -a "yum install -y python-simplejson" --become --ask-pass --become-method=su --ask-become-pass --extra-vars="ansible_ssh_user=username123" -i hosts-dev
```

### SELinux

If SELinux is enabled/permissive a dependency is needed: libselinux-python  
This command will use the raw module to install it:

```bash
ansible centos5 -m raw -a "yum install -y libselinux-python" --become --ask-pass --become-method=su --ask-become-pass --extra-vars="ansible_ssh_user=username123" -i hosts-dev
```
