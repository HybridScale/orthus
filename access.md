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
When first requesting access to the cluster the admin would have requested a public ssh key and created an account for you. Authtication to the cluster is only possible via ssh key, not a password. More information on using ssh keys can be found [here](https://linuxhint.com/ssh-using-private-key-linux/). 

The cluster can be access from the command line (Linux, Mac or Windows Linux Subsystem) with:

```bash
ssh <username>@orthus.cir.irb.hr
```
## Access from outside the RBI network

If you access the cluster from outside the RBI network you will first have to establish a VPN connection to RBI. Information on how to use VPN can be found [here](http://helpdesk.irb.hr/wiki/OpenVPN). Once the VPN is established, the cluster can be accessed following the instructions in [Access from the RBI network](<##access-from-the-rbi-network>).
