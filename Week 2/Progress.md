# Week 2 Progress #
Starting a newer and simpler lab to better understand SRLinux

## A New Topology
For a few days I've been struggling with getting the clients to successful communicate with each other in the original topology. Though they aren't on the same VLAN and SVIs aren't configured, I expected to see nodes learn the mac addresses of the clients when they attempt to ping each other. However, I didn't see this when looking through the nodes' mac address tables. This is concerning since this means that the nodes aren't properly configured as bridged (switch) interfaces or the clients aren't sending frames over their virtual interface. To better isolate the problem, I decided to set up a simpler topology with 1 SRLinux node and 2 client containers running Alpine. 

<img width="286" height="317" alt="image" src="https://github.com/user-attachments/assets/ef90c9f8-4c9b-4871-a4a9-c1042c5f251f" />

## Topology Setup
```
name: small_lab

topology:
  nodes:
    leaf1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux
    client1:
      kind: linux
      image: alpine
    client2:
      kind: linux
      image: alpine
  links:
    - endpoints: ["leaf1: e1-1", "client1: e1-1"]
    - endpoints: ["leaf1: e1-2", "client2: e1-1"]
```
## Setting Static IPs for Clients
Before configuring the Leaf1 node, I'll set the IP addresses for the `client1` and `client2`. To begin setting static IPs inside the clients, I use the command `sudo docker exec -it [name of container] /bin/sh` to access my containers. 

### Docker Command Breakdown
- `sudo`: do this as superuser
- `docker`: the program I'm calling
- `exec -it [name of container]`: execute a process in this container, create a virtual TTY interface (`-t`), and display the results of my input (`-i`)
- `/bin/sh`: execute the shell (the process)

The terminal should look like this after executing the command:
<img width="300" height="21" alt="image" src="https://github.com/user-attachments/assets/5f35edc4-cd09-4ea8-9dd5-c3ca7e8f9f0b" />

Now to configure the IP address I need to know the name of the interface that I called `e-1/1` in my topology. To do this, I ran `ip add` which displayed the IP addresses of all my interfaces. It turned out that the clients also labeled the interfaces as `e-1/1`. To set the IP address, I ran `ip addr add 172.16.16.12/24 dev clab-69359b45`. Finally, I ran `ip add` to verify that the interface was succesfully configured. 

<img width="722" height="312" alt="image" src="https://github.com/user-attachments/assets/a34777a3-5ee7-4e03-9549-b29d3d573697" />

## Configuring Leaf1
To configure `leaf1` I SSHed and logged-in using the default credentials. Next I entered the configuration mode (called candidate). From there I configured the interfaces to act as switch interfaces and saved my configurations.

```
enter candidate
interface ethernet-1/1
admin-state enable
subinterface 0
admin-state enable
type bridged
exit all

interface ethernet-1/2
admin-state enable
subinterface 0
admin-state enable
type bridged
exit all

network-instance layer2
admin-state enable
type mac-vrf
interface ethernet-1/1.0
exit
interface ethernet-1.2.0
exit all
commit stay
commit save
```
## Pinging the Two Clients
After configuring Leaf1, I verified that the two clients can ping each other. The next step will be adding another client and setting up VLANs. One client will be on VLAN 16 and the other two will be on VLAN17.

<img width="465" height="217" alt="image" src="https://github.com/user-attachments/assets/094c6a26-65fd-4051-9161-7dd4e1b55b58" />

## Setting Up VLANs
Before setting up the two VLANs, I need to modify the topology file to add `client3` into the network. To do this I will use nano to write to the file and then redeploy the lab with `sudo clab redeploy -t smalllab.clab.yaml`. Below you will see a diagram of the new network.

<img width="284" height="275" alt="image" src="https://github.com/user-attachments/assets/e16cde15-ee6c-4af9-92dc-b234b473bf31" />

### Reconfiguring Client IP Addresses 
Note to self, whenever you redeploy your container lab all of your devices are essentially wiped clean. Due to the addition of a new subnet, it helped a bit but it also cleared `leaf1` configurations. Anyways, I went through all three clients and configured their new IP addresses.

<img width="706" height="550" alt="image" src="https://github.com/user-attachments/assets/a3cf0dae-15c0-4c50-8eea-534b93c3899f" />

## Reconfiguring Leaf1
Borrowing some of the configs from the original topolgy, I used Gemini and SRLinux's documentation to produce the configuration below. You may have noticed that I created two mac-vrf instances named `VLAN16` and `VLAN17`. Unlike Cisco IOS, SRLinux utilizes the concept of a mac-vrf. A mac-vrf is a network instance that creates a virtual swtich inside of the device. The mac-vrf will have its own mac-table and broadcast domain. This is the equivalent of Cisco's way of setting up VLANs (i.e. creating the VLAN then applying it on an interface). Two forward traffic between the two mac-vrf instances and route between the subnets I had to create an Intergrated Routing and Bridging (IRB) interface with two subinterfaces. An IRB interface is an interface that allows for inter-subnet forwarding. An ip-vrf and mac-vrf instances are required for proper IRB configuration. An IRB is the equivalent to a Switch Virtual Interface (SVI). 

```
enter candidate 
interface ethernet-1/{1..3}
admin-state enable
vlan-tagging true
exit

interface ethernet-1/1
subinterface 16
type bridged
vlan encap untagged
exit all

interface ethernet-1/{2..3}
subinterface 17
type bridged
vlan encap untagged
exit all

network-instance VLAN16
admin-state enable
type mac-vrf
interface ethernet-1/1.16
exit all

network-instance VLAN17
admin-state enable
type mac-vrf
interface ethernet-1/2.17
exit
interface ethernet-1/3.17
exit all

interface irb0
admin-state enable
subinterface 16
admin-state enable
ipv4
admin-state enable
address 172.16.16.1/24
exit to interface
subinterface 17
admin-state enable
ipv4
address 172.16.17.1/24
exit all

network-instance default
admin-state enable
router-id 10.10.10.1
type ip-vrf
interface irb0.16
exit
interface irb0.17
exit all

enter candidate 
network-instance VLAN17
interface irb0.17
exit

network-instance VLAN16
interface irb0.16
exit all

interface lo0
admin-state enable
subinterface 0
admin-state enable
ipv4
address 10.10.10.1/32
exit all

commit save 
```
## Troubleshooting Routing Between Subnets
Unfortuantely, clients on one subnet are unable to ping clients on the other subnet. However, they are able to ping all device within their subnet. You will find below two screenshot with the first one showing that clients are able to communicate with the SRLinux node in their subnet and the other one showing how clients are unable to communicate across subnets.

<img width="707" height="737" alt="image" src="https://github.com/user-attachments/assets/33d72fcf-6013-477d-8961-a2749f567b49" />


<img width="481" height="109" alt="image" src="https://github.com/user-attachments/assets/1c2d6a71-c8ef-4e03-9bef-4f5ee7ce14f1" />

Given the issue, I thought that I misconfigured the IRB interface and subinterfaces on `leaf1`. After re-reading through documentation, using Gemini, and reviewing the running configuration, there was nothing wrong with the IRB interface. The next thing I did was checking the device's routing table to understand how the device was forwarding traffic, but its routes were configured correctly. Finally I decided to check on how the clients were sending its trafic and this is where I discovered the issue. 

All clients have three interfaces; one for management, one for loopback, and one for data traffic (the interface where I was sending the pings out of). With this in mind, each client also maintains its own routing table which tells the client where to send its packets depending on the destination IP address. I discovered that the client's default gateway resulted in packets being sent out of its management interface. Thus clients that don't have a route to the 172.16.16.0/24 or 172.16.17.0/24 network will send traffic out of its management interface in the 172.20.20.0/24 network. So the traffic never reaches `leaf1`. 

Fortuantely this is an easy fix, all I needed to do was create a static route to those networks based on the client. To do this on Linux I used the `ip route add [dest network]/[subnet mask] via [gateway address]` command. Then I configured the static routes for the remaining clients. Once I finished, I tried pinging clients on the other subnet and was able to succesfully communicate with them! 

<img width="698" height="466" alt="image" src="https://github.com/user-attachments/assets/d328d492-6676-4bd2-b2d5-2a244db73c18" />

