ansible-nagios
==============
Ansible Playbook for setting up the Nagios monitoring system and clients on CentOS/RHEL.

![Nagios](/ansbile/nagios/image/ansible-nagios.png?raw=true)

## What does it do?
   - Automated deployment of Nagios on CentOS or RHEL
     * Generates service checks, and monitored hosts from Ansible inventory
     * Generates comprehensive checks for the Nagios server
     * Generates comprehensive checks for all hosts/services via NRPE
     * Generates most of the other configs based on jinja2 templates
     * Wraps Nagios in SSL via Apache
     * Sets up proper firewall rules (firewalld or iptables-services)

## Requirements
   - RHEL7 or CentOS7+ for Nagios server.

## Notes
   - Sets the ```nagiosadmin``` password to ```changeme```, you'll want to change this.
   - Creates a read-only user, set ```nagios_create_guest_user: false``` to disable this in ```install/group_vars/all.yml``` 
   - Implementation is very simple, with only the following server types generated right now:
     - out-of-band interfaces *(ping, ssh, http)*
     - generic servers *(ping, ssh, load, users, procs, uptime, disk space)*
     - [ELK servers](https://github.com/sadsfae/ansible-elk) *(same as servers plus elasticsearch and Kibana)*
     - elasticsearch *(same as servers plus TCP/9200 for elasticsearch)*
     - webservers *(http, ping, ssh, load, users, procs, uptime, disk space)*
     - network switches *(ping, ssh)*
     - Dell iDRAC checks courtesy of @dangmocrang [check_idrac](https://github.com/dangmocrang/check_idrac)
       - By default we will check all server health values from iDRAC 7/8 via snmp
   - ```contacts.cfg``` notification settings are in ```install/group_vars/all.yml``` and templated for easy modification.
   - Adding new hosts to inventory file will just regenerate the Nagios configs

## Nagios Server Instructions
   - Clone repo and setup your Ansible inventory (hosts) file
```
git clone https://github.com/redhat-performance/ops-tools
cd ansible/nagios
sed -i 's/host-01/yournagioshost/' hosts
```
   - Add any hosts for checks in the ```hosts``` inventory
   - Note that you need to add ```ansible_host``` entries __only__ for IP addresses for idrac, switches, out-of-band interfaces and anything that typically doesn't support Python and Ansible fact discovery.
   - Anything __not__ an ```idrac```, ```switch``` or ```oobserver``` should use the FQDN (or an /etc/hosts entry) for the inventory hostname or you may see this error:
     - ```AnsibleUndefinedVariable: 'dict object' has no attribute 'ansible_default_ipv4'}```

```
[webservers]
webserver01

[switches]
switch01 ansible_host=192.168.0.100
switch02 ansible_host=192.168.0.102

[oobservers]
webserver01-ilo ansible_host=192.168.0.105

[servers]
server01

[idrac]
database01-idrac ansbile_host=192.168.0.106

```
   - Run the playbook
```
ansible-playbook -i hosts install/nagios.yml
```
   - Navigate to the server at https://yourhost/nagios
   - Default login is ```nagiosadmin / changeme``` unless you changed it in ```install/group_vars/all.yml```

## Known Issues

SELinux doesn't always play well with Nagios, or the policies may be out of date as shipped with CentOS/RHEL.
```
avc:  denied  { create } for  pid=8800 comm="nagios" name="nagios.qh
```
   - If you see this (or nagios doesn't start) you'll need to create an SELinux policy module.
```
# cat /var/log/audit/audit.log | audit2allow -M mynagios
# semodule -i mynagios.pp
```
Now restart Nagios and Apache and you should be good to go.
```
systemctl restart nagios
systemctl restart httpd
```

## Demonstration
   - You can view a video of the Ansible deployment here:

[![Ansible Nagios](http://img.youtube.com/vi/6vfhflwC_Wg/0.jpg)](http://www.youtube.com/watch?v=6vfhflwC_Wg "Deploying Nagios with Ansible")

## iDRAC Server Health Details

   - The iDRAC health check will provide exhaustive health information and alert upon it.

![iDRAC](/ansible/nagios/image/nagios-idrac.png?raw=true)
