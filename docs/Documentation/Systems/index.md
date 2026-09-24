---
layout: default
title: Systems
has_children: true
order: 4
---

# NLR Systems
NLR operates three on-premises systems for computational work. 

## System Configurations

| Name        | Kestrel |  Swift        | Gila     | 
| :---------- | :------ |  :----------- | :------------- |
| OS          | RedHat Enterprise Linux |  Rocky Linux    | Rocky Linux       |
| Login       | kestrel.hpc.nlr.gov |  swift.hpc.nlr.gov | gila.hpc.nlr.gov |
| CPU         | Dual socket Intel Xeon Sapphire Rapids |  Dual AMD EPYC 7532 Rome CPU | Dual AMD EPYC Milan |
| Cores per CPU Node | 104 cores |  128 cores | Varies by partition | 
| Interconnect | HPE Slingshot 11 | InfiniBand HDR| 25GbE |
| HPC scheduler | Slurm | Slurm | Slurm |
| Network Storage | 95PB Lustre | 3PB NFS | multi-PB Ceph
| GPU         | 156 4x NVIDIA H100 SXM GPUs | 10 4x NVIDIA A100 40GB GPUs | NVIDIA A100, NVIDIA GH200, AMD MI210
| Memory      | 256GB, 384GB, 700GB, 2TB | 256GB(CPU) 1T(GPU) | Varies by partition
| Number of Nodes| 2478 | 484 | 43 |


