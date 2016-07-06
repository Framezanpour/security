# security
project of security .describe  iptable


#!/bin/sh

IPT="/sbin/iptables"

### Internet Gateway
INTR=192.168.200.200

###LAN1 INFO START###
#Network:   192.168.88.0/21       11000000.10101000.01011 [000.00000000] (Class C)
#Broadcast: 192.168.95.255        11000000.10101000.01011 [111.11111111]
#HostMin:   192.168.88.1          11000000.10101000.01011 [000.00000001]
#HostMax:   192.168.95.254        11000000.10101000.01011 [111.11111110]
#Hosts/Net: 2046

LAN1=192.168.88.0/21

vLAN1=192.168.91.0/255
vLAN2=192.168.92.0/255
vLAN3=192.168.93.0/255

v1c5=192.168.91.5
v1c110=192.168.91.110
v2c10=192.168.92.10
v2c11=192.168.92.11
v2c220=192.168.92.220
###LAN1 INFO END###

###LAN2 INFO START###
#Network:   192.168.100.0/24      11000000.10101000.01100100 [.00000000] (Class C)
#Broadcast: 192.168.100.255       11000000.10101000.01100100 [.11111111]
#HostMin:   192.168.100.1         11000000.10101000.01100100 [.00000001]
#HostMax:   192.168.100.254       11000000.10101000.01100100 [.11111110]
#Hosts/Net: 254

LAN2=192.168.100.0/24
###LAN2 INFO END###

###SERVER-FARM INFO START###
Network:   192.168.110.0/24      11000000.10101000.01101110 [.00000000] (Class C)
Broadcast: 192.168.110.255       11000000.10101000.01101110 [.11111111]
HostMin:   192.168.110.1         11000000.10101000.01101110 [.00000001]
HostMax:   192.168.110.254       11000000.10101000.01101110 [.11111110]
Hosts/Net: 25

FARM=192.168.110.0/24
DHCP=192.168.110.1
APP1=192.168.110.2
APP2=192.168.110.2
ANTI=192.168.110.3
###SERVER-FARM INFO END###

# Flush any old rules, old custom tables
$IPT --flush
$IPT --delete-chain

# Set default policies for all three default chains => DROP
$IPT -P INPUT DROP
$IPT -P FORWARD DROP
$IPT -P OUTPUT DROP

# Enable free use of loopback interfaces (used for internal communications on localhost)
$IPT -A INPUT -i lo -j ACCEPT
$IPT -A OUTPUT -o lo -j ACCEPT

# Accept UDP packets for DHCP (DHCP port is 67)
$IPT -A FORWARD -p udp --dport 67 -s $DHCP -j ACCEPT

# Accept packets from/to AntiVirus from LAN1 (all vLans)
$IPT -A FORWARD -s $ANTI -d $LAN1 -j ACCEPT
$IPT -A FORWARD -s $LAN1 -d $ANTI -j ACCEPT

# Accept packets to App1 from LAN1 (all vLans)
$IPT -A FORWARD -s $LAN1 -d $APP1 -j ACCEPT

# Accept packets from/to App2 from LAN2
$IPT -A FORWARD -s $APP2 -d $LAN2 -j ACCEPT
$IPT -A FORWARD -s $LAN2 -d $APP2 -j ACCEPT

# vLAN2 should have access to internet gateway
$IPT -A FORWARD -s $vLAN2 -d $INTR  -j ACCEPT
$IPT -A FORWARD -s $INTR  -d $vLAN2 -j ACCEPT

#v1c10 and v1c11 should have access to v1c5
$IPT -A FORWARD -s $v1c10 -d $v1c5 -j ACCEPT
$IPT -A FORWARD -s $v1c11 -d $v1c5 -j ACCEPT

#vLAN3 should have access to v2c220
$IPT -A FORWARD -s $vLAN3 -d $v2c220 -j ACCEPT

#v1c110 should have access to vLAN2
$IPT -A FORWARD -s $v1c110 -d $vLAN2 -j ACCEPT
