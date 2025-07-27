# Week 2 Progress #
Starting a newer and simpler lab to better understand SRLinux

## A New Topology
For a few days I've been struggling with getting the clients to successful communicate with each other in the original topology. Though they aren't on the same VLAN and SVIs aren't configured, I expected to see nodes learn the mac addresses of the clients when they attempt to ping each other. However, I didn't see this when looking through the nodes' mac address tables. This is concerning since this means that the nodes aren't properly configured as bridged (switch) interfaces or the clients aren't sending frames over their virtual interface. To better isolate the problem, I will set up a simpler topology with 1 SRLinux node and 2 client containers running Alpine. 

<img width="286" height="317" alt="image" src="https://github.com/user-attachments/assets/ef90c9f8-4c9b-4871-a4a9-c1042c5f251f" />

## Topology Setup
