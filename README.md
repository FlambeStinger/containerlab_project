# Containerlab Project
My expeirence learning and working with containerlab

## Overview

This will serve to document my experiences working with containerlab and learning container networking. My main objects are: 
1) Learn how container networking functions
2) Become more familiar with containers
3) Learn SRLinux
4) Successfully configure a container network

## Setup
Given the system requirements of the containerlab I decided to setup a KVM running Ubuntu Server 24.04.2.

**KVM Resources:**
```
Hardware:
2 CPUs
4GB of RAM
25GB of storage

OS:
Ubuntu Server 24.04.2
```

**Topology:**

Following SR Linux's Lab, the lab will be a simple spine and leaf topology. All three network devices will be running SR Linux.

<img width="364" height="360" alt="image" src="https://github.com/user-attachments/assets/053ae290-05f4-4745-bdb0-374cf4e175d1" />


## Learning Resources:
- Containerlab - running networking labs with Docker UX: https://www.youtube.com/watch?v=qigCla1qY3k
- Containerlab: https://containerlab.dev/
- SR Linux Lab: https://learn.srlinux.dev/get-started/lab/
- SR Linux VLANs: https://learn.srlinux.dev/blog/2024/vlans-on-sr-linux/
- Exploring the Linux ‘ip’ Command: https://blogs.cisco.com/learning/exploring-the-linux-ip-command
- RedHat: https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf
