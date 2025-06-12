---
title: Welcome
layout: template
filename: index
order: 1
---

## Content
1. [About the cluster](#about-the-cluster)
2. [Technical details](#technical-details)
3. [Contacts](#contacts)

## About the cluster 
Orthus cluster is small experimental and development cluster located at the Centre for Informatics and Computing and the Division of Electronics at the Ruđer Bošković Institute in Zagreb, Croatia.
The cluster is acquired via two National Science Foundation projects:

- [HRZZ-UIP-2020-02-4559](https://www.irb.hr/eng/Scientific-Support-Centres/Centre-for-Informatics-and-Computing/Projects/HRZZ/Scalable-high-performance-algorithms-for-future-heterogeneous-distributed-computer-systems) "Scalable high-performance algorithms for future heterogeneous distributed computer systems"
- [HRZZ-UIP-2020-02-1623](https://www.irb.hr/eng/Divisions/Division-of-Electronics/Laboratory-for-Machine-Learning-and-Knowledge-Representation/Projects/Exploring-Interactions-Between-Regulatory-Variants-in-Human-Disease-Context) "Exploring Interactions Between Regulatory Variants in Human Disease Context"

## Technical details

The Orthus cluster currently consists of a login node, one compute node and a storage node. Both the login and compute nodes run the Rocky Linux 9 as the OS, have [Slurm](https://slurm.schedmd.com/documentation.html) for resource management and job scheduling and [FreeIPA](https://www.freeipa.org/) for identity management / single sign-on. Additonally much of the HPC functionallity is proviced by the [OpenHPC](https://openhpc.community/) project, specifically using [this](Install_guide.pdf) version of the OpenHPC installation guide.

For information of software avalable on the cluster see the [applications and software section](applications.md) section and for running batch jobs see the [Job scheduling section](running.md) of this documentation. 

1. **Compute node (GPU)**
    - 2 x Intel(R) Xeon(R) Gold 6240R CPU @ 2.40GHz
    - 48 computer cores
    - 4 x NVIDIA A100 PCI 40GB HBM2e memory
    - 512 GB main memory
    - 2 x 10 Gb/s network adapter

2. **Storage node**
    - QNAP TS-1886-XU
    - 10 x 8 TB
    - RAID 6
    - 4 x 10 Gb/s network adapter

3. **Network infrastructure**
    - Mikrotik CRS312-4C+8XG-RM
    - 12 x 10 Gb/s ports

## Contacts

Location: Ruđer Bošković Institute, Croatia, Zagreb


### Emails
General information: [orthus-info@irb.hr](mailto:orthus-info@irb.hr)

For technical question, user support and requests to install new applications: [orthus-users@irb.hr](mailto:orthus-users@irb.hr) (only for registered users)

---
<img align="left" width="400" height="185" src="https://mojoblak.irb.hr/s/gifFHzfM9gwNxx9/download/HRZZ-eng.jpg">
<img align="right" width="215" height="150" src="https://mojoblak.irb.hr/s/9CPc6HojToCyxet/download/IRB-logo.jpg">
