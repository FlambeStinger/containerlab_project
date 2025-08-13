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
