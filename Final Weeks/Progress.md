# Final Week Progress # 
Returning to Week 1's topology and applying what I learned from the previous week

## Returning to the Original Topology
As a reminder this network will have three SRLinux nodes with the leaf nodes behaving more as switches while the spine node behave more as a router and two clients on two seperate subnets. Down below you'll find a diagram and a configuration file representing this.

<img width="379" height="363" alt="image" src="https://github.com/user-attachments/assets/3045eae2-b3c0-4440-9453-83a02bf491b3" />
<br></br>

```
name: lab1srl

  topolgy:
    nodes:
      spine1:
        kind: nokia_srlinux
        image: ghcr.io/nokia/srlinux
      leaf1:
        kind: nokia_srlinux
        image: ghcr.io/nokia/srlinux
      leaf2:
        kind: nokia_srlinux
        image: ghcr.io/nokia/srlinux
      client1:
        kind: linux
        image: alpine
      client2:
        kind: linux
        image: alpine 
  links:
    - endpoints: ["spine1: e1-1", "leaf1: e1-1"]
    - endpoints: ["spine1: e1-2", "leaf2: e1-1"]
    - endpoints: ["leaf1: e1-2", "client1: e1-1"]
    - endpoints: ["leaf2: e1-2", "client2: e1-1"]
```
**Note to self' there cannot be any spaces between the node and it's interface or clab will not properly deploy it**
## Configuring the Leaves

```
Leaf1

enter candidate
interface ethernet-1/1
admin-state enable
vlan-tagging true
subinterface 0
admin-state enable
type bridged
vlan encap untagged 
exit all

interface ethernet-1/2
admin-state enable
vlan-tagging true
subinterface 10
admin-state enable
type bridged
vlan encap single-tagged-range low-vlan-id 10 high-vlan-id 20
exit all

network-instance VLAN10
admin-state enable
type mac-vrf
interface ethernet-1/1.0
exit
interface ethernet-1/2.10
exit all
commit save 


Leaf 2 

enter candidate
interface ethernet-1/1
admin-state enable
vlan-tagging true
subinterface 0
admin-state enable
type bridged
vlan encap untagged 
exit all

interface ethernet-1/2
admin-state enable
vlan-tagging true
subinterface 10
admin-state enable
type bridged
vlan encap single-tagged-range low-vlan-id 10 high-vlan-id 20
exit all

network-instance VLAN20
admin-state enable
type mac-vrf
interface ethernet-1/1.0
exit
interface ethernet-1/2.10
exit all
commit save 


Spine
enter candidate 
interface ethernet-1/1
admin-state enable
vlan-tagging true
subinterface 10
admin-state enable 
type bridged 
vlan encap single-tagged-range low-vlan-id 10 high-vlan-id 20

interface ethernet-1/2
admin-state enable
vlan-tagging true
subinterface 20
admin-state enable 
type bridged 
vlan encap single-tagged-range low-vlan-id 10 high-vlan-id 20
exit all 

network-instance VLAN10
admin-state enable 
type mac-vrf 
interface ethernet-1/1.10
exit to network-instance
interface irb0.10
exit all

network-instance VLAN20
admin-state enable
type mac-vrf
interface ethernet-1/2.20
exit to network-instance
interface irb0.20
exit all

interface lo0
admins-state enable
subinterface 0
admin-state enable
ipv4
address 1.1.1.1/32
exit all

interface irb0
admin-state enable
subinterface 10
admin-state enable
ipv4
admin-state enable
address 10.10.10.1/27 
exit to interface 
subinterface 20
admin-state enable
ipv4
admin-state enable
address 10.10.20.1/27
exit all

network-instance default 
admin-state enable
router-id 1.1.1.1
type ip-vrf

commit save 
```
## Modifying the Topology
Unfortuantely it took me a while to realize this, but my current topology is not adequate for setting up and testing tagged traffic across a LAN. The reason behind this has to due with the fact that each leaf only has one client and that there isn't any other client on the same VLAN in the network. Thus to fix this I will remove `spine1`, connect `leaf1` to `leaf2`, and add an additional client to each leaf node. Additionally, each node will have two VLANs, 10 and 20, with one client belonging to VLAN 10 and the other belonging to VLAN 20. Add clients will be assigned a static IP and no intervlan routing will be configured since that is not the focus for this final topology.

<img width="681" height="283" alt="image" src="https://github.com/user-attachments/assets/812b49c5-9b8c-4604-b691-c5112ba81d1d" />

<br></br>
```                                      
name: final_lab

topology:
  nodes:
    leaf1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux
    leaf2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux
    client1:
      kind: linux
      image: alpine
    client2:
      kind: linux
      image: alpine
    client3:
      kind: linux
      image: alpine 
    client4:
      kind: linux
      image: alpine 
  links:
    - endpoints: ["leaf1:e1-1", "leaf2:e1-1"]
    - endpoints: ["leaf1:e1-2", "client4:e1-1"]
    - endpoints: ["leaf1:e1-3", "client1:e1-1"]
    - endpoints: ["leaf2:e1-2", "client3:e1-1"]
    - endpoints: ["leaf2:e1-3", "client2:e1-1"]

```
```
Leaf1 

enter candidate 
interface ethernet-1/{1..3}
admin-state enable 
vlan-tagging true 
exit to root 

interface ethernet-1/3
subinterface 10
admin-state enable 
type bridged 
vlan encap single-tagged vlan-id 10  
exit to root 

interface ethernet-1/2
subinterface 20
admin-state enable 
type bridged 
vlan encap single-tagged vlan-id 20 
exit to root
 
interface ethernet-1/1
subinterface 0
admin-state enable 
type bridged 
vlan encap single-tagged-range low-vlan-id 10 high-vlan-id 20
exit to root 

network-instance Bridge1 
admin-state enable 
type mac-vrf
interface ethernet-1/1.0
exit
interface ethernet-1/2.20
exit
interface ethernet-1/3.10 
exit to root 

commit now 



Leaf2

enter candidate 
interface ethernet-1/{1..3}
admin-state enable 
vlan-tagging true 
exit to root 

interface ethernet-1/3
subinterface 20
admin-state enable 
type bridged 
vlan encap single-tagged vlan-id 20
exit to root 

interface ethernet-1/2
subinterface 10
admin-state enable 
type bridged 
vlan encap single-tagged vlan-id 10 
exit to root
 
interface ethernet-1/1
subinterface 0
admin-state enable 
type bridged 
vlan encap single-tagged-range low-vlan-id 10 high-vlan-id 20
exit to root 

network-instance Bridge1 
admin-state enable 
type mac-vrf
interface ethernet-1/1.0
exit
interface ethernet-1/2.10
exit
interface ethernet-1/3.20 
exit to root
```
## Troubleshooting the Issue
To better isolate the problem with clients in leaf 1 being unable to send traffic to their respective clients in leaf 2 I decided that creating a topology with one leaf switch and two clients on sending VLAN 10 traffic will enable me to pinpoint if the problem is the client or the node. Additionally, it will allow me to see how SRLinux handels tagged traffic.

## Designing the Topology
For this topology `leaf` will connect to `client1` through its e1-1 interface and connect to `client2` through its e1-2 interface. `client1` will be assigned an IPv4 address of 10.10.10.10/24, and `client2` will be assigned 10.10.10.20/24. Both clients will be sending traffic tagged with VLAN10 to the `leaf`. 

<img width="363" height="108" alt="image" src="https://github.com/user-attachments/assets/9e221dc8-46bd-4fb8-8c0f-6388544c4931" />

## Setting Up the Topology
I created a file named `demo_lab.clab.yaml` with `nano` and defined the node, two clients, and their links. Instead of reconfiguring the same attributes for the clients, I discovered in ContainerLab's documentation that you can create a group where you define its attributes and then assign nodes to a group. Nodes will recieve their attirbutes from the group they are assigned to, so this is very useful whenever I have to configure multiple identitcal nodes! 

```
name: demo_lab

topology:
 groups:
  alpine-clients:
   kind: linux
   image: alpine
 nodes:
  leaf:
   kind: nokia_srlinux
   image: ghcr.io/nokia/srlinux
  client1:
   group: alpine-clients
  client2:
   group: alpine-clients

 links:
  - endpoints: ["leaf:e1-1", "client1:e1-1"]
  - endpoints: ["leaf:e1-2", "client2:e1-1"]
```

## Configuring VLAN Interfaces on the Clients
To begin VLAN configuration, I used `sudo docker exec -it clab-demo_lab-client1 /bin/sh` to access client's 1 shell. From there I listed its interfaces to verify that its e1-1 interface was properly deployed. Next, I issued the command 'ip link add link e1-1 name e1-1.10 type vlan id 10` to configure a subinterface that will send and recieve VLAN 10 traffic. After that, I issued `ip add add 10.10.10.10/24 dev e1-1.10` command to assign the VLAN 10 subinterface an IP address. Finally, I enabled the interface with `ip link set dev e1-1.10 up` and verfied that it was up. I performed the same process on `client2`. 
<br><br>
<img width="748" height="377" alt="image" src="https://github.com/user-attachments/assets/558006be-0fc8-4284-9418-3a7f9efb804d" />

## Configuring the Leaf
(Explain what and why Im making the configuration that I did)

```
enter candidate
interface ethernet-1/{1..2}
admin-state enable
vlan-tagging true
subinterface 10 
admin-state enable
type bridged 
vlan encap single-tagged vlan-id 10 
exit to root

network-instance VLAN10
admin-state enable
type mac-vrf
interface ethernet-1/1.10
exit
interface ethernet-1/2.10
exit to root 
commit now 
```

## Testing Connectivity
After configure the `leaf` I wanted to verify that the clients can communicate with each other, so I accessed client1's shell and made it ping `client2`. Fortuantely, the clients were able to ping each other, so I know that my VLAN configuration on the clients were correct in the final_lab topology. Thus the problem must be with leaves. 

<img width="556" height="196" alt="image" src="https://github.com/user-attachments/assets/d29cea96-c8cd-4092-90cc-9623519da4ff" />
