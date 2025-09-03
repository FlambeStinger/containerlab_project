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

### Correcting the Misconfigured Tagged Interface
It turned out that it was unecessary to issue `vlan encap single-tagged-range low-vlan-id 10 high-vlan-id 20` on the interfaces connecting the leaves together. When re-reading through SR Linux's documentation, I realized that the intent of this command was for connecting to servers with multiple hosts on seperate VLANs. When I removed the command from both interfaces and disabled their vlan tagging, clients on both VLANs (10 and 20) were able to communicate with their peers! I don't know if the intefaces are tagged interfaces anymore. Nor do I know why `vlan encap single-tagged-range low-vlan-id 10 high-vlan-id 20`  prevented tagged traffic from being sent over, but I suspect that the way you implement this in a data center enviroment differs greatly from that of a campus. Luckily, this provides me with an opportunity to investigate and learn more! 

## Making Small Improvements
Despite the project being completed, there were a few things that I wanted to improve upon. For starters, after creating several topology file, it became very repetitious configuring the same attributes for nodes and clients of the same type. For example, if I wanted to create a very large network, say 20 nodes and 40 clients, it would take a while to set the attributes for all devices. Thus, I needed to look for a more scalable solution. After reading through Container Lab's User Manual, I discovered that there is a way to pass attritbutes to devices at a large scale. There are four degrees of inheritance from most specific to least specific. 

Starting with the most specific, attributes configured at the `Node` level will be local only to that node. So, no other node will inherit the attributes of a configured node. The next degree of inheritance is confgured at the `Group` level. This is the level that I decided to utilize. At the `Group` level, attirbutes such as type, image, and kind can be described for group that the user defined. From there, each node can be assigned to a group where they can inherit their attributes. The `Kinds` level is very similar to the `Group` level but more general. Finally, the `Defaults` level is the least specific degree of inheritance. It defines attributes at a global level, thus all devices will be configured with attirbutes at the `Default` level unless superseded by `Group` or by `Node`. 

#### Example Demostration Using Groups
```
name: example_lab

topology:
 groups:
  leaves:
   kind: nokia_srlinux
   image: ghcr.io/nokia/srlinux
  alpine-clients:
   kind: linux
   image: alpine
 nodes:
  leaf1:
    group: leaves
  leaf2:
    group: leave
  client1:
    group: alpine-clients
  client2:
    group: alpine-clients
 links:
    - endpoints: ["leaf1:e1-1", "leaf2:e1-1"]
    - endpoints: ["leaf1:e1-2", "client1:e1-1"]
    - endpoints: ["leaf2:e1-2", "client2:e1-1"]
```
#### Automating Clients' Network Configuration
Another thing that I wanted to improve was the process for assigning clients their IP addresses. The current process was pasting the client's configuration from a text file on my host. This did make the process slightly more tolerable than retyping the configurations for each client, but I knew that a better solution was out there. Like before, after digging around Container Lab's User Manual, I learned about the `exec` attribute. This attribute can be assigned to node, and commands that a user specified will be executed under this attribute. I assigned each client an exec attribute under their node attribute and pasted the commands for each client from my text file. 

```
name: example_lab

topology:
 groups:
  leaves:
   kind: nokia_srlinux
   image: ghcr.io/nokia/srlinux
  alpine-clients:
   kind: linux
   image: alpine
 nodes:
  leaf1:
    group: leaves
  leaf2:
    group: leave
  client1:
    group: alpine-clients
  client2:
    group: alpine-clients
 links:
    - endpoints: ["leaf1:e1-1", "leaf2:e1-1"]
    - endpoints: ["leaf1:e1-2", "client1:e1-1"]
    - endpoints: ["leaf2:e1-2", "client2:e1-1"]
```
