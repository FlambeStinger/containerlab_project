# Week 1 Progress #
## Setup
Started following a video on setting up containerlab. Before watching through it, I configured a KVM running Ubuntu Server 24.04.2. Immediately upon start up I ran `sudo apt update` and `sudo apt upgrade`. Next, I installed the packages necessary for running containerlab. Fortunately both the video and containerlab's website provide a command to install it all: `curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"`

## Creating the Topology
First I created a new directory called `clab-labs` then cd into `~/clab-labs`. Afterwards I created another directory named `clab-lab1srl` which is nested inside `clab-labs`. My thought process behind that is that each subdirectory under `clab-labs` will represent different containerlab projects. Regardless, after creating the subdirectory I created the topology file: `lab1.clab.yaml`. The `.clab.yaml` extension makes a file a topolgy file. Finally, I configured the topology file.

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
  links:
    - endpoints: ["spine1: e1-1", "leaf1: e1-1"]
    - endpoints: ["spine1: e1-2", "leaf2: e1-1"]
```
### Topology File Breakdown
- **Name** : Defines the name of the topology 
- **Topology** : Defines topology configurations
- **Nodes** : Defines node configuartions
- **Kind** : Defines the type of NOS/OS clab can expect
- **Image** : Defines the container image that will be pulled and used
- **Endpoints** : Defines how nodes will be connected

## 1st Error
<img width="956" height="113" alt="image" src="https://github.com/user-attachments/assets/0701e7c8-9190-47e8-a425-db99f2b36654" />

It turns out that you can have multiple instances of a container network from the same topology file. I didn't realize this until I thought about veryifing the status of my network with `sudo containerlab inspect -a` :

<img width="908" height="248" alt="image" src="https://github.com/user-attachments/assets/1ccf8e9e-b007-40cf-b4f7-c28dc47d3014" />

Given how little resources I allocated to the VM, it makes sense that I was unable to deploy the lab. Fortuantely, since the lab was already up there wasn't much to fix. Anyways, running multiple container networks could be interesting to play with in the future once I have a better understanding of containerlabs.

## 2nd Error
In my attempt to ssh into `spine1` to start configuartions, I was unfortanetly met with this error message:

<img width="837" height="409" alt="image" src="https://github.com/user-attachments/assets/92231de6-1189-4c76-b607-69917fefb094" />

Thinking that it would be an easy fix, I redeployed my lab using `sudo clab redeploy -t lab1.clab.yaml`. I attempted to ping `spine1` after the redeployment but that didn't do anything. 

<img width="615" height="358" alt="image" src="https://github.com/user-attachments/assets/14780d28-3518-4724-8f4f-a55b54950fdb" />

Obviously the problem is a networking problem, so I decided to take a look at my interfaces.

<img width="772" height="312" alt="image" src="https://github.com/user-attachments/assets/c504e6dc-d9af-4106-9663-e2d7815f8513" />

When looking through the outputs of `ip -br add show && ip -br link show` I noticed that both the `docker0` and the `br-738c2b1c6465` interfaces are DOWN. Also, I find it interesting that there are two bridge interfaces with the same IP address. I'm going to create the same containerlab on a new VM and compare the network configurations between the two VMs.

<img width="755" height="275" alt="image" src="https://github.com/user-attachments/assets/8c921140-cffa-4cd4-8814-7e2e1f936e1e" />

The above image is the network configuration from the new VM. For the most part everything looks about the same when compared to the original. However, the new VM doesn't have duplicate interfaces for the 172.20.20.0/24 network. I speculate that the duplicate interfaces might be the cause of the problem as without it I'm able to ping and SSH into my devices.

<img width="774" height="336" alt="image" src="https://github.com/user-attachments/assets/e63d24fb-1747-431a-9560-af2bdcd91261" />

My theory was right! After deleting the duplicate bridge interface I was able to ping and SSH into my devices! 


## SRLinux Configuration

For my first lab working with SRLinux I'm planning to create two VLANs (VLAN 10 and 20), place their endhosts on their respective VLANs, setup two trunk links between the leaves and spine switches, and configure `spine` to route between the two VLANs. Attached below is the topology detailing IP and VLAN assignment.

<img width="379" height="363" alt="image" src="https://github.com/user-attachments/assets/3045eae2-b3c0-4440-9453-83a02bf491b3" />

### Adding Clients

Before I start configuring the network, I need to add the client devices that I forgot to add when defining the topology. Similar to adding the SRLinux devices, I created and defined the connection, image, and kind for the two nodes `client1` and `client2`. My topology file ended up looking like this after I completed adding the clients.

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

### Configuring Layer2 Interfaces

```
enter candidate (Brings us to candidate mode; where configurations occur)
interface ethernet-1/1 (The interface we want to configure)
admin-state enable (this enables the interface))
subinterface 10 (creates a subinterface ending in .10)
type brideged (sets the link to be a bridge)
admin-state enable (enables the .10 subinterface)
exit all (returns to root)

network-instance layer-2 (this creates a vrf named layer-2)
type mac-vrf (this sets the vrf to behave like a switch)
admin-state enabled (this enables the instance)
interface ethernet-1/1.10 (Adds the ethernet-1/1.10 interface to this vrf instance)
```

### Configuring and Applying VLANs 
