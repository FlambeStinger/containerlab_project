# Project Summary and Conclusion #
A summary and conclusion on the project

## Summary ##
This personal project started because I wanted to learn more about containers and build a homelab without the need of having the physical equipement. The project's objectives were: 
1) Learn how container networking functions
2) Become more familiar with containers
3) Learn SRLinux
4) Successfully configure a container network

The original toplogy consisted of three nodes and two clients deployed in a spine and leaf architecture, and the setup was deployed and ran inside of a Ubunutu Server virtual machine. Week 1 was focused on learning the fundamentals of SRLinux and deploying a toplogy using Container Lab. However, I encountered problems with deploying the lab such as duplicate interfaces with the same IP Addresses preventing management access, and problems with clients being unable to communicate with each other. Fortuantely, I learned how to troubleshoot the former, but the latter required me to start a newer and smaller toplogy to better learn the intricacies of SRLinux. 

Week 2's focus was learning how to configure a network where clients can communicate with each other on the same LAN and on different VLANs. Starting with the topology, I defined a toplogy with one node and two clients. Clients were assigned to the 172.16.16.0/24 network. I configured the node to forward traffic between its interfaces that the clients were connected to, and I configured the clients with their respective IP Addresses (more details in Week 2 Directory). After verifying that the clients could communicate with each other, I made the toplogy slightly more complex with the addition of another client and the establishment of two VLANs (10 and 20). 

## Conclusion ##
