
# About Gila

Gila is an OpenHPC-based cluster. Most [nodes](./running.md#gila-compute-nodes) run as virtual machines, with the exception of the Grace Hopper nodes, in a local virtual private cloud (OpenStack). Gila is allocated for NLR workloads. Check back regularly as the configuration and capabilities for Gila are augmented over time.


## Gila Access and Allocations

 **A specific allocation is not needed for NLR employee use of Gila.** All NLR employees with an HPC account automatically have access to Gila and can use the *aurorahpc* allocation to run jobs. If you do not have an HPC account already and would like to use Gila, please see the [User Accounts](https://www.nlr.gov/hpc/user-accounts) page to request an account. 

The aurorahpc allocation does have limited resources allowed per job. These limits are dynamic, and can be found in the MOTD displayed when you log in to Gila. Please note that this allocation is a shared resource. If excessive usage reduces productivity for the broader user community, you may be contacted by HPC Operations staff. If you need to use more resources than allowed by the aurorahpc allocation, or work with external collaborators, you can request a specific allocation for your project. To do so, please email [hpc-requests@nlr.gov](mailto:hpc-requests@nlr.gov) and include a short description of the work you intend to do on Gila as well as your project handle if you already have an allocation on a different NLR HPC system. 

#### For NLR Employees:
To access Gila, log in to the NLR network and connect via ssh to:

    gila.hpc.nlr.gov

To use the Grace Hopper nodes, connect via ssh to:

    gila-arm.hpc.nlr.gov

#### For External Collaborators:
There are no external-facing login nodes for Gila. There are two options to connect:

1. Connect to the [SSH gateway host](https://www.nlr.gov/hpc/ssh-gateway-connection.html) and log in with your username, password, and OTP code. Once connected, ssh to the login nodes as above.
1. Connect to the [HPC VPN](https://www.nlr.gov/hpc/vpn-connection.html) and ssh to the login nodes as above.

!!! warning "Former Vermilion Users"
    Your `~/.bashrc` file was transferred from Vermilion to Gila. Please review any customizations to ensure compatibility with Gila.

## Get Help with Gila

Please see the [Help and Support Page](../../help.md) for further information on how to seek assistance with Gila or your NLR HPC account. 

## Building Code

Do not build or run code on login nodes. Login nodes have limited CPU and memory available. Use a compute node or GPU node instead. Simply start an [interactive job](../../Slurm/interactive_jobs.md) on an appropriately provisioned node and partition for your work and do your builds there.

