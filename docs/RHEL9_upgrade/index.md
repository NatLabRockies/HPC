# Kestrel RHEL9 Upgrade

Kestrel is currently undergoing a phased-in operating system (OS) upgrade from Red Hat Enterprise Linux (RHEL) 8.8 to RHEL 9.4 (RHEL9). At the time of writing in July 2026, most nodes on Kestrel are still using the original RHEL8 OS, but over time more compute nodes will be upgraded to RHEL9 until the entire system is running the newer OS.

The migration from RHEL8 to RHEL9 provides the system with necessary security updates, a critical kernel upgrade to enable the latest versions of various software applications, upgraded CUDA drivers for GPU nodes, and comes with significant changes to the [software module](./modules/index.md) system. 

!!! Note
    The experience of loading and using application modules that are maintained under `/nopt` is *not* expected to change between RHEL8 and RHEL9 on Kestrel. However, **users who compile their own software should especially familiarize themselves with how to navigate the [new RHEL9 module stack](./modules/index.md)**.

## Timeline

The following tables contain the planned migration date for the remainder of the Kestrel nodes:

### Login Nodes 

| Node | Planned Transition Date |
| --- | --- |
| kl1 | Nov. 2, 2026 |
| kl2 | Oct. 28, 2026 |
| kl4 | Nov. 9, 2026 |
| kl6 | Oct. 29, 2026 |

### DAV Nodes 

| Node | Planned Transition Date |
| --- | --- |
| kd1 | Nov. 2, 2026 |
| kd2 | Nov. 2, 2026 |
| kd3 | Nov. 3, 2026 |
| kd4 | Nov. 3, 2026 |
| kd5 | Nov. 4, 2026 |
| kd6 | Nov. 4, 2026 |
| kd8 | Nov. 9, 2026 |

 Kestrel Jupyterhub is planned to be migrated on Nov. 3, 2026.  
## RHEL9 System Software Reference

| Software                | Version(s) |
| ---                     | --- |
| Red Hat Enterprise Linux (RHEL) | 9.4 (ESR) |
| Kernel Release          | 5.14.0-427.13.1 |
| HPE Cray Programming Environment | 25.3 |
| NVIDIA GPU Driver       | 595.58.03 |
| Provided CUDA modules   | 12.8.1 and 13.2 |
| Slurm                   | 25.11.8 |

