# Introduction

How to set up VM that are able to (1) communicate with each others, (2) are reachable from the host (your mac). With (1) and (2) done using their name (FQDN).

# DNSMasq Configuration

# VirtualBox Configuration

The set up of the VM requires 2 network adapters : 

- Host-only adapter
- NAT Network

## Create a NAT Network

In VirtualBox global settings, select ```Network > NAT Networks``` and add a new network if none already exists. Then select the network in the list (maybe one already exists, called ```NatNetwork```) and configure it (click on the screwdriver on the right). For example:

- Network Name: ```NatNetwork```
- Network CIDR: ```10.0.2.0/24```

Tick the option ```Supports DHCP```.

## Create a Host-only Network

In VirtualBox global settings, select ```Network > Host-only Networks``` and add a new network if none already exists. Then select the network in the list (maybe one already exists, called ```vboxnet0```) and configure it (click on the screwdriver on the right).

In the ```Adapter``` section, parameters can be set like this to have a /24 network: 

- IPv4 Address: ```10.99.99.1```
- IPv4 Network Mask: ```255.255.255.0```

In the ```DHCP``` section, untick ```Enable Server```. 

## VM Configuration

Select the VM, then  ```Settings > Network``` and enable network adapter 1 and 2. 

Configure ```Adapter 1```:

- Attached to: ```Host-only Adapter```
- Name: ```vboxnet0```

**In ```Advanced``` section, be sure to renew the MAC address by clicking the green arrows.**

Configure ```Adapter 2```:

- Attached to: ```NAT Network```
- Name: ```NatNetwork```

**In ```Advanced``` section, be sure to renew the MAC address by clicking the green arrows.**

## Guest configuration (Ubuntu)

As root, modify ```/etc/hostname``` with the FQDN wanted (```guest01.vm``` in our case).

File ```/etc/network/interfaces``` must contain the following lines: 

```
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet dhcp
```

Be sure interface names are the one listed using  ```ifconfig -a```

# Known Issues

## Routing

Check routes that are set up in the VM. You must have something like this:

```
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.1        0.0.0.0         UG    100    0        0 eth1
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
10.99.99.0      0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

where ```10.0.2.1``` is your default gateway provided by the NAT networking. The default destination has to be set up this way. 

Having the dnsmasq address (*i.e.* ```10.99.99.1```) instead is not ok. If so, you can swap the corresponding lines (```eth1``` and ```eth0```)' into ```/etc/udev/rules.d/70-persistent-net.rules```. *Certainly not the best way to do, but it worked for me.*

## MAC Address conflict

Be sure that MAC addresses on the VM are different from other running VM. 

```
$ ifconfig -a | grep HWaddr
eth0      Link encap:Ethernet  HWaddr 08:00:27:cf:3d:69  
eth1      Link encap:Ethernet  HWaddr 08:00:27:46:02:dc  
```

This happens when cloning a VM whithout checking the "Reinitialize the MAC address of all network cards".