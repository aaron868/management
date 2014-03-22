# Management

## Introduction

This is the Ansible playbook for my infrastructure. I'm sharing it so that it will save you some time and so that you will critique it. I'd like this playbook to eventually become a *best practice* configuration for CentOS servers.

## Infrastructure Assumptions
This playbook assumes several things about the IT infrastructure. It assumes a largely homogeneous set of servers that run only CentOS 6. These servers are VMs running in XenServer 6.2. The CentOS servers are hardened according to the US DISA STIG for RHEL 6. 

I also locally mirror the default and EPEL repositories. It doesn't take that much space but adds to the speed, stability, and security of my infrastructure. The way yum selects mirrors can product odd results. And I do not trust all mirrors equally.

Even though your infrastructure is likely different than mine, many of these roles should work for you. Please check these closely before using them in your own environment. And please offer suggestions and improvements. 

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