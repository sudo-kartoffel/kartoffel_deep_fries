# Tous les packages dont on aura besoin :

```
apt-get update -y
apt-get upgrade -y
apt-get install vim
apt-get install isc-dhcp-server
apt-get remove --auto-remove nftables
apt-get purge nftables
apt-get update
apt-get install iptables iptables-persistent -y
apt-get install isc-dhcp-server
apt-get install tcpdump
```

# Network Config 

## Hostname

```
hostnamectl set-hostname deb
```

## Interfaces /etc/network/interfaces

```
vim /etc/network/interfaces

# Exemple pour du DHCP

#auto eth0
#allow-hotplug eth0
#iface eth0 inet dhcp

# Le reste en static

# GW-INT Adapter 1
auto enp0s3
iface enp0s3 inet static
    address 172.16.2.2/30
    gateway 172.16.2.1

# LAN-INT Adapter 2
auto enp0s8
iface enp0s8 inet static
    address 172.16.0.254/24

# DMZ-INT Adapter 3
auto enp0s9
iface enp0s9 inet static
    address 172.16.1.254/24

systemctl restart networking
```

## Forwarding /etc/sysctl.conf

```
vim /etc/sysctl.conf

net.ipv4.ip_forward=1

service systemd-sysctl restart
```

# Config du serveur DHCP 

## Ajoute les interfaces ou il doit servir /etc/default/isc-dhcp-server

```
vim /etc/default/isc-dhcp-server

# Interfaces pour LAN-INT et DMZ-INT
INTERFACESv4="enp0s8 enp0s9"
#INTERFACESv6=""
```

## Config file /etc/dhcp/dhcpd.conf

```
vim /etc/dhcp/dhcpd.conf

# Show that we want to be the only DHCP server in this network:
authoritative;

### General parameters ###
deny unknown-clients;
option domain-name-servers 1.1.1.1, 8.8.8.8;
default-lease-time 3600;
max-lease-time 14400;

### GW-INT ###

subnet 172.16.2.0 netmask 255.255.255.252 {
}

### LAN-INT ###

subnet 172.16.0.0 netmask 255.255.255.0 {
    range 172.16.0.1 172.16.0.200;
    option subnet-mask 255.255.255.0;
    option routers 172.16.0.254;
    
#    host xub {
#        hardware ethernet xx:xx:xx:yy:yy:yy;
#        fixed-address 172.16.0.1;
#    }
}

### DMZ-INT ###

subnet 172.16.1.0 netmask 255.255.255.0 {
    range 172.16.1.1 172.16.1.200;
    option subnet-mask 255.255.255.0;
    option routers 172.16.1.254;
    
#    host fed {
#        hardware ethernet xx:xx:xx:yy:yy:yy;
#        fixed-address 172.16.1.1;
#    }
}

systemctl enable --now isc-dhcp-server
```





vim /etc/iptables/rules.v4
