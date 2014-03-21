# Management

## Introduction

This is the Ansible playbook for my infrastructure. This infrastructure is built on three elements:

* CentOS 6 
* CentOS 6 configured according to the DISA STIG for RHEL 6
* XenServer 6.2 based VMs

Please check these closely before using them in your own environment. They work well for me but may not for you.

## Layout

My goal is that the only configuration you have to do is to change the variable in the `group_vars/all` file and in the `host_vars/hostname` file.  The group variables should define your network. The host variables configure an individual server.

I use small roles to make the playbook more modular.

## Roles
Role specific considerations are found here.

### clamav
This role runs clamav once per day via cron. It also updates the signatures once per day. It does not run the clamav daemon.

### iptables
This role uses a helper application called 'ferm' to manage iptables. 

### auditd
This role uses the default CentOS configuration plus the DISA STIG settings. I have not added anything additional customizations.

### end
As you may have guessed from its name, this role should run at the very end. It allows you to follow the Unix `directory.d` pattern. Roles can add files to a `dir.d` folder and then this role will cause the daemon being configured to restart. This is needed for iptables, collectd, and others. It makes up for the lack of support by Ansible for global handlers.