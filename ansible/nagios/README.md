ansible-nagios
==============
Ansible Playbook for setting up the Nagios monitoring system and clients

**What does it do?**
   - Automated deployment of Nagios on CentOS or RHEL
     * Generates service checks, and monitored hosts from Ansible inventory
     * Wraps Nagios in SSL via Apache
     * Templating inspired from the official [ansible examples](https://github.com/ansible/ansible-examples)

**Requirements**
   - RHEL7 or CentOS7+ server/client with no modifications
     - Fedora 23 or higher needs to have ```yum python2 python2-dnf libselinux-python``` packages.
       * You can run this against Fedora clients prior to running Ansible ELK:
       - ```ansible fedora-client-01 -u root -m shell -i hosts -a "dnf install yum python2 libsemanage-python python2-dnf -y"```

**Notes**
   - Sets the ```nagiosadmin``` password to ```changeme```, you'll want to change this.
   - Implementation is very simple, with only the following server types generated right now:
     - out-of-band interfaces (Dell iDRAC, IPMI etc) (ping/ssh)
     - generic servers (ping, ssh, users, load, swap)
     - webservers (http check, ping, ssh, users, load, swap)
     - network switches (ping/ssh)
   - I do not setup the ```contacts.cfg``` file for notifications

**Nagios Server Instructions**
   - Clone repo and setup your hosts file
```
git clone https://github.com/redhat-performance/ops-tools
cd ansible/nagios
sed -i 's/host-01/yournagioshost/' hosts
```
   - Add any hosts for checks in the ```hosts``` inventory
   - Note that you need to add ```ansible_host``` aliases for switches, out-of-band interfaces and anything that typically doesn't support Python and Ansible fact discovery.
```
[webservers]
webserver01

[switches]
switch01 ansible_host=192.168.0.100
switch02 ansible_host=192.168.0.102

[oobservers]
webserver01-idrac ansible_host=192.168.0.105

[servers]
server01
```
   - Run the playbook
```
ansible-playbook -i hosts install/elk.yml
```
   - Navigate to the server at https://yourhost/nagios
   - Default login is ```nagiosadmin / changeme``` unless you changed it in ```install/group_vars/all.yml```

**TODO**
   - Write equivalent ```nagios-client``` playbooks for NRPE
   - Expand checks
   - Add support for new server types
   - Allow for Nagios/HTTP ports to be configurable
