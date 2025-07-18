# Week 1 Progress #
## Setup
Started following a video on setting up containerlab. Before watching through it, I configured a KVM running Ubuntu Server 24.04.2. Immediately upon start up I ran `sudo apt update` and `sudo apt upgrade`. Next, I installed the packages necessary for running containerlab. Fortunately both the video and containerlab's website provide a command to install it all: `curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"`

## Creating the Topology
First created a new directory called "clab-labs"

Then cd into ~/clab-labs"

Finally created the topology file:

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
```
### Topology File Breakdown
