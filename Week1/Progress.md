# Week 1 Progress #
## Setup
Started following a video on setting up containerlab. Before watching through it, I configured a KVM running Ubuntu Server 24.04.2. Immediately upon start up I ran `sudo apt update` and `sudo apt upgrade`. Next, I installed the packages necessary for running containerlab. Fortunately both the video and containerlab's website provide a command to install it all: `curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"`

## Creating the Topology
First I created a new directory called `clab-labs` then cd into `~/clab-labs`. Afterwards I created another directory named `clab-lab1srl` which is nested inside `clab-labs`. My thought process behind that is that each subdirectory under `clab-labs` will represent different containerlab projects. Regardless, after creating the subdirectory I created the topology file: `lab1.clab.yaml`. The `.clab.yaml` extension makes a file a topolgy file. Finally, I configured the topology file.

```
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


## 1st Error
<img width="956" height="113" alt="image" src="https://github.com/user-attachments/assets/0701e7c8-9190-47e8-a425-db99f2b36654" />

It turns out that you can have multiple instances of a container network from the same topology file. I didn't realize this until I thought about veryifing the status of my network with `sudo containerlab inspect -a` :

<img width="908" height="248" alt="image" src="https://github.com/user-attachments/assets/1ccf8e9e-b007-40cf-b4f7-c28dc47d3014" />

Given how little resources I allocated to the VM, it makes sense that I was unable to deploy the lab. Fortuantely, since the lab was already up there wasn't much to fix. Anyways, running multiple container networks could be interesting to play with in the future once I have a better understanding of containerlabs.

## 2nd Error
In my attempt to ssh into Spine1 to start configuartions I was unfortanetly met with this error message:

<img width="837" height="409" alt="image" src="https://github.com/user-attachments/assets/92231de6-1189-4c76-b607-69917fefb094" />

## SRLinux Configuration
