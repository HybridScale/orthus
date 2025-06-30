---
title: How to access
layout: template
filename: access
order: 2
---

**_The access to the cluster is possible only within the IRB network_**

- Connecting from the IRB network - [Access from the RBI network](#access-from-the-rbi-network).
- Connecting from the outside of the RBI network - [Access from outside the RBI network](#access-from-outside-the-rbi-network).

<!--If you access the cluster from outside the RBI network you will first have to establish a VPN connection to RBI. Information on how to use VPN can be found [here](http://helpdesk.irb.hr/wiki/OpenVPN). Once the VPN is established, the cluster can be accessed following the instructions [Access from the RBI network](<#access-from-the-rbi-network>).-->

## Login node
 - hostname: **orthus.cir.irb.hr**

## Access from the RBI network
### Linux / Mac / Windows
When first requesting access to the cluster you will be asked to provide a public ssh key and then a created an account for you.

Authentication to the cluster is only possible via ssh key, not a password. 
If you do not already have an ssh key you can generate a new one with the following command on your UNIX command line:
```
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
```

The public key can then be see by running:
```
cat ~/.ssh/id_ed25519.pub
```
The output should be emailed to the cluster administrator inorder to get you access to the cluster.

More in depth information on generating and using ssh keys can be found [here](ht<tps://linuxhint.c om/ssh-using-private-key-linux/). 

The cluster can be access from the command line (Linux, Mac or Windows Linux Subsystem) with:
```
ssh <username>@orthus.cir.irb.hr
```

## Access from outside the RBI network

If you access the cluster from outside the RBI network you will first have to establish a VPN connection to RBI. Information on how to use VPN can be found [here](http://helpdesk.irb.hr/wiki/OpenVPN). Once the VPN is established, the cluster can be accessed following the instructions in [Access from the RBI network](<##access-from-the-rbi-network>).
