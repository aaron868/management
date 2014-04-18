# Management

## Introduction

This is the Ansible playbook for my infrastructure. I'm sharing it so that it will save you some time and so that you will critique it. I'd like this playbook to eventually become a *best practice* configuration for CentOS servers.

It's not entirely plug-n-play. My goal is that you can use these roles yourself unmodified by only changing the values in the `groups_vars/all` file. 

To run these plays I use the following Ansible command: `ansible-playbook -i production.hosts site.yml -K -k`

## Infrastructure Assumptions
This playbook assumes several things about the IT infrastructure. It assumes a largely homogeneous set of servers that run only CentOS 6. These servers are VMs running in XenServer 6.2. The CentOS servers are hardened according to the US DISA STIG for RHEL 6. 

I also locally mirror the default and EPEL repositories. It doesn't take that much space but adds to the speed, stability, and security of my infrastructure. The way yum selects mirrors can product odd results. And I do not trust all mirrors equally.

Even though your infrastructure is likely different than mine, many of these roles should work for you. Please check these closely before using them in your own environment. And please offer suggestions and improvements. 

## Layout

My goal is that the only configuration you have to do is to change the variable in the `group_vars/all` file and in the `host_vars/hostname` file.  The group variables should define your network. The host variables configure an individual server.

I use small roles to make the playbook more modular. I don't use role variables, only host and global ones.

## Roles
Role specific considerations are found here.

### VM
A play to create a Xen VM.

### Base
The CentOS 6.5 configuration. This is fully SCAP (DISA STIG) compliant except for AIDE. I use OSSEC instead. It disables IPv6. It adds the EPEL repository in an unorthodox but well performing way. I mirror the CentOS and EPEL repos locally for speed and security. 

### network
A play to configure CentOS ethernet interfaces. See my [blog post](http://wp.me/p4iDAr-7V) for details. 

### clamav
This role runs clamav once per day via cron. It also updates the signatures once per day. It does not run the clamav daemon.

### iptables
This role uses a helper application called 'ferm' to manage iptables. 

### auditd
This role uses the default CentOS configuration plus the DISA STIG settings. I have not added anything additional customizations.

### dhcp-server
A basic ISC DHCP server configuration.

### kickstart-server
This is meant to be run by the server that stores the kickstart files and serves them from the web. The interesting thing here is the kickstart template. It is SCAP compliant and Xen specific.

### mysql-server
A basic mysql server setup. Automating the mysql post configuration was a huge challenge. I found bits and pieces on this on the web and put them togther. I'm grateful to those whol posted them.

### nginx-server
A basic nginx web server setup. I include the collectd plugin.

### ntp-client
A complete NTP client.

### ntp-server
A, hopefully, well tuned NTP server configuration. I still want to look into crypto for this.

### ossec-client
A basic OSSEC client configuration. Uses out-of-the-box settings with the addition of a few simple extra rules to monitor IPtable and network ports. In your plays make sure the OSSEC server is running before you run the clients. The clients need to register with the server.

### ossec-server
A basic OSSEC server using ut-of-the-box settings. This role is designed to serve as a relay for network devices that only speak syslog. The devices connect to syslog-ng on this server and their logs are stored as files for the OSSEC engine. The logs are then forwarded to the central logging server for archiving. 

See this [blog post](http://sharknet.us/2014/04/15/ansible-ossec-role) for details.

### squid-server
A simple but functional squid server configuration.

### syslog-ng-client
This role replace rsyslog with syslog-ng. It is a drop-in replacement for rsyslog. It also forwards logs to a central log server. I intend to add SSL to this role in the near future so that all log traffic is encrypted.

### syslog-ng-relay
This is designed to be used with the `ossec-server` role. It listens for syslog for network traffic and does two things with it: stores it in a local file and forwards it to another syslog-ng server.

### syslog-ng-server
This role allows the server to act as a central log archive. All device and host logs are sent to this server and stored as files, one per host. The raw logs are stored and could also be sent to a structured storage engine such as Elasticsearch.

### end
As you may have guessed from its name, this role should run at the very end. It allows you to follow the Unix `directory.d` pattern. Roles can add files to a `dir.d` folder and then this role will cause the daemon being configured to restart. This is needed for iptables, collectd, and others. It makes up for the lack of support by Ansible for global handlers.