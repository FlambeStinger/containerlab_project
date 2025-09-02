# Final Weeks Progress # 
Modifying Week 1's lab and applying what I learned

## Modifying the Topology
Unfortuantely it took me a while to realize this, but my current topology is not adequate for setting up and testing tagged traffic across a LAN. The reason behind this has to due with the fact that each leaf only had one client, and each client belonged to a seperate VLAN. Thus to fix this I removed `spine1`, connected `leaf1` and `leaf2` together, and added an additional client to each leaf node. Additionally, on each leaf, one client was assigned to VLAN 10 and the other to VLAN 20. Clients were assigned static IP addresses, and intervlan routing was not configured since that was out of scope of this lab's objective.
<br></br>

<p align="center">
<img width="681" height="283" alt="image" src="https://github.com/user-attachments/assets/812b49c5-9b8c-4604-b691-c5112ba81d1d" />
</p>

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
commit now
```
## Troubleshooting Detour
After making my configurations, I tested the clients' connectivtiy, and like Week 1, clients were unable to send traffic to their peers on the same VLAN. Upon inspection of `leaf1's` and `leaf2's` MAC Address Table, the leaves were learning the sending client's MAC address connected to them but none of the ARP Requests were being forwarded out of leaves' tagged interface. Additionally, the interface was only transmitting traffic, thus this strongly indicated that the problem must be the leaves' tagged interface. 

#### Clients Unable to Communicate
<p>
  <img width="490" height="165" alt="image" src="https://github.com/user-attachments/assets/8e88228c-0884-44ad-81d7-b55e7ca27f77" />
  <img width="490" height="184" alt="image" src="https://github.com/user-attachments/assets/122ea45a-2005-4a4a-949d-08548cc9e16d" />
</p>

#### Leaf 1 and Leaf 2 MAC Tables
<p>
  <img width="915" height="400" alt="image" src="https://github.com/user-attachments/assets/2df36987-767b-4a84-9c05-7ddd05686e1e" />
  <img width="915" height="400" alt="image" src="https://github.com/user-attachments/assets/ff1c9958-9e85-4c1a-8b74-272f6e874f55" />
</p>

#### Leaf 1 and Leaf 2 E1-1 Details
<p>
  <img width="343" height="655" alt="image" src="https://github.com/user-attachments/assets/11b67d95-99d5-4d12-8fcd-3e9c7abde37a" />
  <img width="343" height="655" alt="image" src="https://github.com/user-attachments/assets/ccd6a0a3-2bbd-48cc-afd7-35f227198b97" />
</p>

### Misconfiguring the Trunk/Tagged Interface
