I've been working on a network configuration project in Packet Tracer. I aimed to show my proficiency in setting up various protocols commonly used by companies and enterprises today. These protocols include DNS, DHCP, IPv4 Addressing Scheme, OSPF, VLSM Subnetting, LACP EtherChannel, Inter-VLAN Routing, and Default Routing.

After setting up the protocols, I practiced device hardening by implementing security measures like Port Security, DHCP Snooping, DAI, PortFast, and BPDU Guard. 

The network consists of one internal local router connected to two Layer 3 switches. The switches are connected to two PCs and one server. These PCs represent the number of hosts in each network (50 and 30). There is also an ISP router that is serially connected to the internal router that gives access to the Internet. The two switches have formed an EtherChannel using four FastEthernet cables. 


Physical Layer

The first thing I did was lay out the devices and connect them using FastEthernet and GigabitEthernet Cables. The PCs are connected to the switches with Fa cables and the switches are connected to the router with Gig cables. I made sure all PCs had functioning NICs to connect to the network. 


Data Link Layer

I did use Layer 3 switches but the configurations are still pretty much the same as Layer 2 switches. I configured the switches with their respective VLANs using these commands:

Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name FINANCE
Switch(config)# interface range f0/5-7
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

Then I configured EtherChannel between the two Layer 3 switches for redundancy and increased bandwidth:

Switch(config)# int range f0/1 - 4
Switch(config-if-range)# channel-group 1 mode active
Switch(config)# int po1
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
Switch(config-if)# no shut

The next step was configuring Port Security to restrict access based on MAC addresses:

Switch(config)#interface gi1/0/1
Switch(config-if)#switchport port-security
Switch(config-if)#switchport port-security maximum 3

Then I enabled DHCP snooping & DAI to prevent rogue DHCP servers and ARP spoofing:

Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10
Switch(config)# ip arp inspection vlan 10
Switch(config)# interface fa0/7
Switch(config-if)# ip dhcp snooping trust
Switch(config-if)# ip arp inspection trust
Switch(config)# ip dhcp snooping vlan 10,20,99

Finally, I enabled PortFast on ports connected to end devices and BPDU Guard to protect against potential STP attacks:

S1(config)# interface fa0/1
S1(config-if)# switchport mode access
S1(config-if)# spanning-tree portfast
S1(config-if)# exit
S1(config)# spanning-tree portfast default
S1(config)# exit
S1# show running-config | begin span

S1(config)# interface fa0/1
S1(config-if)# spanning-tree bpduguard enable
S1(config-if)# exit
S1(config)# spanning-tree portfast bpduguard default
S1(config)# end
S1# show spanning-tree summary


Network Layer

I configured the internal router as a router on a stick, basically putting VLAN 10 on gig0/0/0.10 and VLAN 20 on g0/0/1.20. I set up IP addresses for the VLANs using the usable address in the subnet range of 192.168.1.0/26 and 192.168.1.64/27. I came up with these subnets using VLSM subnetting. The Finance department has 50 hosts and the HR department has 30 hosts. Using VLSM, I can subnet these two departments with minimal waste of IP addresses. 

I also set up a default static route (ip route 0.0.0.0 0.0.0.0 192.168.100.2) to the ISP for external connection to the Internet. OSPF was also implemented on the routers with the router ospf 1 command.


Transport/Application Layer

In this layer, I configured a DNS server on the Finance network (VLAN 10) to resolve domain names to IP addresses & a DHCP server on the HR network (VLAN 20) to dynamically assign IP addresses to PCs.


Check My Work



To verify my configurations you can open up my attached file in Packet Tracer and run these tests,

 Inter-VLAN Connectivity:
Verify that PCs in VLAN 10 can ping PCs in VLAN 20.
External Connectivity:
Verify that internal PCs can ping the ISP router and external IP addresses.
DHCP and DNS:
Ensure PCs in VLAN 20 receive IP addresses from the DHCP server.
Test DNS resolution from various PCs.
