# Week 2 Progress #
Starting a newer and simpler lab to better understand SRLinux

## A New Topology
For a few days I've been struggling with getting the clients to successful communicate with each other in the original topology. Though they aren't on the same VLAN and SVIs aren't configured, I expected to see nodes learn the mac addresses of the clients when they attempt to ping each other. However, I didn't see this when looking through the nodes' mac address tables. This is concerning since this means that the nodes aren't properly configured as bridged (switch) interfaces or the clients aren't sending frames over their virtual interface. To better isolate the problem, I will set up a simpler topology with 1 SRLinux node and 2 client containers running Alpine. 

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
After configuring Leaf1, I verified that the two clients can ping each other. The next step will be adding another client and setting up VLANs. One client will be on VLAN 10 and the other two will be on VLAN20.

<img width="465" height="217" alt="image" src="https://github.com/user-attachments/assets/094c6a26-65fd-4051-9161-7dd4e1b55b58" />

## Setting Up VLANs

