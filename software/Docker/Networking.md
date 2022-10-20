	# Overview

Networking in docker is a complex topic, which requires some previous knowledge in OSI layers 2 & 3.

Additionally, to isolate a container from its host, docker provides with some virtualization features which create a network with the actual host. 

In this article, I will summarize the different types of networking docker provides, and how some of these functionalities work.

There are 3 main netoworking concepts in docker:
- Bridge
- Host
- Overlay

The links used to obtain the information presented in this article are:

> Youtube Video NetworkChuck - 7 docker types of networking:
> https://www.youtube.com/watch?v=bKFMS5C4CG0&ab_channel=NetworkChuck
> 
> Docker Docs - Docker network containers:
> https://docs.docker.com/engine/tutorials/networkingcontainers/
> 
> Argus-Sec - Docker Networking behind the scenes:
> https://argus-sec.com/docker-networking-behind-the-scenes/
> 
> Docker Documentation - Docker and iptables: 
> https://docs.docker.com/network/iptables/
> 
> StackExchange - What is Masquerade? 
> https://askubuntu.com/questions/466445/what-is-masquerade-in-the-context-of-iptables
> 
> iptables tutorial (Chapter 6 & 7) - traversing of tables and chains / The state machine (connection tracking) 
> 
> Linux Network Administrator - Great Explanation of the NAT process: 
> https://www.oreilly.com/openbook/linag2/book/ch11.html


# Important Concepts

**Layer 2** - Related to the MAC address of computers within the same Local Area Network (LAN). A protocol which is commonly used in this layer is ARP, which broadcasts to all computers in a LAN, to query which computer has a certain IP address.

**Layer 3** - Is the network layer, and allows routing between LANs in a Wide are Network (WAN).

**Network Address Translation (NAT)** - Changes either the source or the destination IP address in a datagram packet. After the translation, when it receives a packet in response, it forwards the packet back to the source host. There are 2 types of NATs Source NAT, and Destination NAT (SNAT, DNAT). Masquerading is a type of SNAT where instead of switching the Source NAT of the packet to a static IP address, it uses the IP of an interface which is specified. 

*Examples*:
	
Source NAT (SNAT): Here the host analysing the packet, if a certain rule is satisfied, it changes the Source IP and Port of the datagram packet to include its IP address. This is commonly done to provide internet access to multiple computers within a LAN through a single public IP address. When a packet is received back on the same IP and Port, it looks up on its NAT table, which host and port the first packet in the transaction was sent out from, and directs it to the host.

Destination NAT (DNAT): The host analysing the packet, when a given rule is satisfied, it changes the destination IP and port of the datagram packet. This is useful if we want to proxy the data to another server which is accessible by the computer currently analysing the packet. If we intend to proxy the data to another server then we have to guarantee the data passes through this host on the way back. Otherwise the Destination IP receiving the packet would see the original Source IP and respond to that IP, which is not what the source host expects. For this reason, to accurately proxy data, DNAT has to be performed combined with SNAT or Masquerading.


**Bridge**: A bridge is a device that can connect computers together through layer 2 communication. A physical device of this type is a switch, which can provide with additional functionality to virtually seperate LANs within the same switch (virtual LANs \[VLANS\]). This is done to minimize flooding when broadcasting Layer 2 within a LAN. 

The bridge uses information to understand through which interface it should forward data which is intended for a 

A bridge can also be a virtual device, 