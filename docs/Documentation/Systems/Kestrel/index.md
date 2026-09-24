---
layout: default
title: Kestrel
parent: Systems
has_children: true
hide:
 - toc
---

# About the Kestrel Cluster

Kestrel is configured to run compute-intensive and parallel computing jobs. It is a heterogeneous system comprised of 2,314 CPU-only nodes, and 156 GPU-accelerated nodes that run the Linux operating system (Red Hat Enterprise Linux), with a peak performance of 44 PetaFLOPS.

Please see the [System Configurations](../index.md) page for more information about hardware, storage, and networking.

## Accessing Kestrel
Access to Kestrel requires an NLR HPC account and permission to join an existing allocation. Please see the [System Access](https://www.nlr.gov/hpc/system-access.html) page for more information on accounts and allocations.

Kestrel has two types of login nodes, CPU and GPU, which share the same architecture as the corresponding compute nodes. You should use the CPU login nodes to compile software for use on and to submit jobs to the CPU compute nodes, and the GPU login nodes for GPU jobs.  


#### For NLR Employees:

Users on an NLR device may connect via ssh to Kestrel from the NLR network using:

* kestrel.hpc.nlr.gov (CPU)
* kestrel-gpu.hpc.nlr.gov (GPU)

 This will connect to one of the three login nodes using a round-robin load balancing approach. Users also have the option of connecting directly to an individual login node using one of the following names: 

* kl1.hpc.nlr.gov (CPU)
* kl2.hpc.nlr.gov (CPU)
* kl3.hpc.nlr.gov (CPU)
* kl5.hpc.nlr.gov (GPU)
* kl6.hpc.nlr.gov (GPU)

#### For External Collaborators:
If you are an external HPC user, you will need a [One-Time Password Multifactor token (OTP)](https://www.nlr.gov/hpc/multifactor-tokens.html) for two-factor authentication.

For command line access, you may login directly to **kestrel.nlr.gov**.  Alternatively, you can connect to the [SSH gateway host](https://www.nlr.gov/hpc/ssh-gateway-connection.html) or the [HPC VPN](https://www.nlr.gov/hpc/vpn-connection.html).

To access the GPU login nodes, first connect with one of the methods described above, and then ssh to **kestrel-gpu.hpc.nlr.gov**. 

!!! warning "Windows SSH "Corrupted MAC on input" Error"
    When attempting to SSH, some Windows users experience an error message stating "Corrupted MAC on input" or "message authentication code incorrect." To solve the error, run the ssh command with the flag `-m hmac-sha2-512`. Example below:

    ```
    ssh -m hmac-sha2-512 username@kestrel.hpc.nlr.gov
    ```

    See the [Workaround blog post](../../../blog/2022-12-19-windows_ssh.md) for further details and information.

!!! warning "Login Node Policies"
    Kestrel login nodes are shared resources, and because of that are subject to process limiting based on usage to ensure that these resources aren't being [used inappropriately](https://www.nlr.gov/hpc/inappropriate-use-policy.html). Each user is permitted up to 8 cores and 100GB of RAM at a time, after which the Arbiter monitoring software will begin moderating resource consumption, restricting further processes by the user until usage is reduced to acceptable limits.

## Data Analytics and Visualization (DAV) Nodes

There are eight DAV nodes available on Kestrel, which are nodes intended for HPC applications that require a graphical user interface.  They are not general-purpose remote desktops, and are intended for HPC or visualization software that requires Kestrel.

[FastX](../../Viz_Analytics/virtualgl_fastx.md) is available for HPC users to use graphical applications on the DAV nodes.

To connect to a DAV node using the load balancing algorithim, NLR employees can connect to **kestrel-dav.hpc.nlr.gov**. To connect from outside the NLR network, use **kestrel-dav.nlr.gov**. 


## Get Help With Kestrel
Please see the [Help and Support Page](../../help.md) for further information on how to seek assistance with Kestrel or your NLR HPC account. 
