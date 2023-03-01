# Howto Install and Configure Cobbler on Centos 6

This document describes the steps to get cobbler up and running on Centos 6
After the installation finnishes we need to perform the following steps before we can install Cobbler:

1. Configure the network interfaces 2. Configure the resolver
3. Disable Selinux
4. Disable Iptables Firewall
5. Install EPEL Yum Repository

###Configure Linux
###Configure the network interfaces
In Redhat 6 Network configurations are handled by network-manager. Since I always install a minimalistic server network-manager is by default not installed and configuration should be done the old fashioned way by edditing the configuration files directly.
First we set the correct hostname and gateway

```
vi /etc/sysconfig/network
```

###Configure the resolver
###Disable selinux
Change the following line from:
```
SELINUX=enforcing

```
to:

```
SELINUX=disabled
```

###Disable iptables
Stop Iptables:
```
service iptables stop
```

Disable iptables:
```
chkconfig iptables off
```

###Install the Fedora EPEL repository
```
rpm -Uhv
http://download.fedora.redhat.com/pub/epel/6/i386/epel-release-6-5.noarch.rp
m
```

###Install Cobbler and prerequisites
Install the required packages:
```
yum install cobbler cobbler-web pykickstart
```

Enable the required services:
```
chkconfig httpd on
chkconfig xinetd on
chkconfig cobblerd on
```

Start the required services:
```
service httpd start
service xinetd start
service cobblerd start
```

###Configure Cobbler and prerequisites 
###Install and Configure ISC Dhcp and Bind

For a complete standalone install server we need to install a local dhcp server and a dns server, we will use the bind dns software and the dhcp software both from ISC. Bind has a track record of being hard to configure because of the miriad of options but since cobbler does all the configuring for us we dont need to worry about that.

First install Bind and Dhcp
```
yum install -y bind bind-utils dhcp
```

Enable Bind
```
chkconfig named on
```

Start Bind
```
service named start
```
Enable Dhcp
```
chkconfig dhcpd on
```
Start Dhcp
```
service dhcpd start
```
Enable dns and dhcp management in cobbler
```
vi /etc/cobbler/modules.conf
```
and configure the dns and dhcp lines to look like the example below
```
[dns]
module = manage_bind
[dhcp]
module = manage_isc
```

Configuring Bind and Dhcp is done via cobbler, open the following files and make the the changes as stated below:
```
vi /etc/cobbler/dhcp.template
```

Make sure the following lines match your network configuration
```
subnet 192.168.1.0 netmask 255.255.255.0 {
     option routers             192.168.1.1;
     option domain-name-servers 192.168.1.1;
     option subnet-mask         255.255.255.0;
     range dynamic-bootp        192.168.1.100 192.168.1.254;
```
After the above changes it's time to restart cobbler
```
service cobblerd restart
```

All changes are now active, let's check if cobbler has any recomendations to get things working:
```
cobbler check
```

Correct settings sugested by Cobbler check command
```
vi /etc/cobbler/settings.conf
```

change the following lines in the settings file to match the example below
```
server: 192.168.1.66
next_server: 192.168.1.66
```

Now we need to install some network bootloaders
```
cobbler get-loaders
```

This will download all required boot loaders and some more exotic boot loaders for other architectures.
Enable the tftpd daemon in xinetd
```
vi /etc/xinetd.d/tftp
```

And change the line:
```
disable = yes
```

to:
```
disable = no
```
Enable the Rsync daemon in xinetd

```
vi /etc/xinetd.d/rsync
```

And change the line:
```
disable = yes
```

to:
```
disable = no
```

Now we can restart xinetd
```
service xinetd restart
```
