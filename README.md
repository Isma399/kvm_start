# kvm_start
#Start KVM, bond, bridge
# Config network
#Install bridge-utils
yum -y install bridge-utils

# Config interface em1 et em2
cat > /etc/sysconfig/network-scripts/ ifcfg-em1 >> EOF
DEVICE="em1"
IPV4_FAILURE_FATAL="no"
IPV6INIT="no"
IPV6_FAILURE_FATAL="no"
NAME="em1"
UUID="558f62ba-dcf4-4f72-9d1a-b524a445cae5"
ONBOOT="yes"
NM_CONTROLLED="no"
HWADDR=C8:1F:66:EA:BB:E1
BOOTPROTO="none"
MASTER=bond0
SLAVE=yes
DNS1="195.83.247.29"
DNS2="195.83.247.21"
DOMAIN="univ-brest.fr"
EOF

cat > /etc/sysconfig/network-scripts/ ifcfg-em2 >> EOF
DEVICE=em2
NAME=em2
UUID=ba050409-d6c8-4211-abcb-a71d7e836ccc
DEVICE=em2
NM_CONTROLLED="no"
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
DNS=195.83.247.29
DOMAIN=univ-brest.fr
ONBOOT=yes
EOF

# Config bond0
cat > /etc/sysconfig/network-scripts/ifcfg-bond0 >> EOF
DEVICE=bond0
BONDING_MASTER=yes
BOOTPROTO=none
ONBOOT=yes
NAME="Bond0"
ONPARENT=yes
BONDING_OPTS="mode0 miimon=100"
DNS=195.83.247.29
DOMAIN=univ-brest.fr
EOF

# Config bridges
cat > /etc/sysconfig/network-scripts/ifcfg-vlan15 >> EOF
DEVICE=vlan15
ONBOOT=yes
TYPE=Bridge
DELAY=0
IPADDR=192.168.1.48
NETMASK=255.255.255.0
GATEWAY=192.168.1.18
DEFROUTE=yes
NM_CONTROLLED=no
NAME="System vlan15"
DNS=195.83.247.29
DOMAIN=univ-brest.fr
EOF

cat > /etc/sysconfig/network-scripts/ifcfg-vlan5 >> EOF
DEVICE="vlan5"
ONBOOT="yes"
TYPE=Bridge
BOOTPROTO=none
PREFIX=24
DNS1=195.83.247.29
DOMAIN=univ-brest.fr
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME="System vlan5"
EOF

# One bond per vlan
cat > /etc/sysconfig/network-scripts/ifcfg-bond0.15
DEVICE=bond0.15
BOOTPROTO=none
ONBOOT=yes
VLAN=yes
TYPE=Ethernet
BRIDGE=vlan15
EOF

cat > /etc/sysconfig/network-scripts/ifcfg-bond0.5
DEVICE=bond0.5
BOOTPROTO=none
ONBOOT=yes
VLAN=yes
TYPE=Ethernet
BRIDGE=vlan5
EOF

# Add bond modules
# miimon = 7 mode (failover, roundrobin, ..)
cat > /etc/modprobe.d/bonding >> EOF
options bond0 mode=0 miimon=0
EOF

cat > /etc/modules-load.d/wdsm.conf >> EOF
bonding
bridge
tun
8021q
EOF

# Disable de network manager 
systemctl stop NetworkManager.service 
systemctl disable NetworkManager.service

# Enable network service
systemctl enable network.service 
systemctl start network.service

reboot
# Config validation
cat /proc/net/vlan/bond0.5
cat /proc/net/bonding/bond0
