# Running on Gila

*Learn about compute nodes and job partitions on Gila.*


## Gila Compute Nodes

Gila compute nodes are not configured as exclusive and can be shared by multiple users or jobs. Be sure to request the resources that your job needs, including memory and cores. If you need exclusive use of a node, add the `--exclusive` flag to your job submission. 

!!! info "Important" 
    Please note that Gila's hardware configuration is continuously evolving, and partitions are subject to change. For the most current partition information, use the `sinfo` command. To display detailed configuration information about a node, run `scontrol show node <nodename>`. 

### CPU Nodes

The CPU nodes in Gila are single-threaded virtualized nodes.  There are two sockets and NUMA nodes per compute node, with each socket containing 30 __AMD EPYC Milan__ (x86-64) cores. Each node has 220GB of RAM that can be used. 


### GPU Nodes

There are four types of GPU nodes on Gila, including the Grace Hopper nodes described below. Two types have A100 GPUs and are running on x86-64 __Intel Xeon Icelake CPUs__. There is also one node with an AMD MI210 GPU.

### Grace Hopper Nodes

Gila has 5 NVIDIA Grace Hopper nodes. To use the Grace Hopper nodes, submit your jobs to the `gh` partition from the `gila-arm.hpc.nlr.gov` login node. Each Grace Hopper node has a 72 core NVIDIA Grace CPU and an NVIDIA GH200 GPU, with 96GB of VRAM and 470GB of RAM. They have one socket and NUMA node. 

Please note - the __NVIDIA Grace CPUs__ run on a different processing architecture (ARM64) than both the __Intel Xeon Icelake CPUs__ (x86-64) and the __AMD EPYC Milan__ (x86-64). Any application that is manually compiled by a user and intended to be used on the Grace Hopper nodes __MUST__ be compiled on the Grace Hopper nodes themselves.



## Partitions

Here are the partitions as of 4/2/2026:

| Partition Name       | CPU                       | GPU                      | Qty | RAM    | Cores/node | GPUs/node |
| :--:                 | :--:                      | :--:                     | :--:| :--:   | :--:       | :--:      |
| gpu-intel-a100-40g   | Intel Xeon Icelake        | NVIDIA Tesla A100-40     | 5   | 210 GB | 60         | 1         |
| gpu-intel-a100-80g   | Intel Xeon Icelake        | NVIDIA Tesla A100-80     | 5   | 910 GB | 42         | 8         |
| amd                  | 2x 30 Core AMD Epyc Milan | N/A                      | 36  | 220 GB | 60         | N/A       |
| gh                   | NVIDIA Grace              | GH200                    | 5   | 470 GB | 72         | 1         |
| gpu-amd-mi210        | 2x 30 Core AMD Epyc Milan | AMD MI210                | 2   | 210 GB | 60         | 1         |
| gpu-amd-mi250        | 2x 30 Core AMD Epyc Milan | AMD MI250                | 1   | 1   TB | 48         | 1         |


## Performance Recommendations

Gila is optimized for single-node workloads. Multi-node jobs may experience degraded performance. All MPI distribution flavors work on Gila, with noted performance from Intel-MPI. Gila is single-threaded, and applications that are compiled to make use of multiple threads will not be able to take advantage of this. 

!!! tip "VASP" 
    For performance recommendations and examples specific to VASP jobs on Gila, please see the [VASP documentation page](../../Applications/vasp.md#vasp-on-gila). 

## Example: Compiling a Program on Gila

In this section we will describe how to compile an MPI based application using an Intel toolchain from the module system. Please see the [Modules page](./modules.md) for additional information on the Gila module system.


### Requesting an interactive session
First, we will begin by requesting an interactive session. This will give us a compute node from where we can carry out our work. An example command for requesting such a session is as follows:

```salloc -N 1 -n 60 --mem 60GB --partition=amd --account=aurorahpc --time=01:00:00```

This will request a single node from the AMD partition with 60 cores and 60 GB of memory for one hour. We request this node using the ```aurorahpc``` account that is open to all NLR staff, but if you have an HPC allocation, please replace ```aurorahpc``` with the project handle.

### Loading necessary modules

Once we have an allocated node, we will need to load the initial Intel module for the toolchain `oneapi`. This will give us access to the Intel toolchain, and we will we now load the module ```intel-oneapi-mpi``` to give us access to Intel MPI. Please note, you can always check what modules are available to you by using the command ```module avail``` and you can also check what modules you have loaded by using the command ```module list```. The commands for loading the modules that we need are as follows:

```bash
module load oneapi
module load intel-oneapi-mpi
```

### Copying program files

We now have access to the tools we need from the Intel toolchain in order to be able to compile a program! First, create a directory called `program-compilation` under `/projects` or `/scratch`. 

```bash
mkdir program-compilation
cd program-compilation 
```
Now we are going to copy the `phostone.c` file from `/nopt/nrel/apps/210929a` to our `program-compilation` directory. 

```rsync -avP /nopt/nrel/apps/210929a/example/phostone.c .```

`rsync` is a copy command that is commonly used for transferring files, and the parameters that we put into the command allow for us to see the progress of the file transfer and preserve important file characteristics. 

### Program compilation

Once the file is copied, we can compile the program. The command we need to use in order to compile the program is as follows:

```bash
mpiicx -qopenmp phostone.c -o phost.intelmpi
```

The command ```mpiicx``` is the Intel MPI compiler that was loaded from the module ```intel-oneapi-mpi```, and we added the flag of ```-qopenmp``` to make sure that the OpenMP compiled portions of the program are able to be loaded. We then specified the file name as `phost.intelmpi` using the ```-o``` flag. 

### Submitting a job

The following batch script requests two cores to use two MPI ranks on a single node, with a run time of up to an hour. Save this script to a file such as `submit_intel.sh`, and submit using `sbatch submit_intel.sh`. Again, if you have an HPC allocation, we request that you replace ```aurorahpc``` with the project handle.

??? example "Batch Submission Script - Intel MPI"

    ```bash
    #!/bin/bash
    #SBATCH --nodes=1
    #SBATCH --ntasks=2
    #SBATCH --cpus-per-task=2
    #SBATCH --time=01:00:00
    #SBATCH --mem=20GB
    #SBATCH --account=aurorahpc

    module load oneapi
    module load intel-oneapi-mpi

    srun --cpus-per-task 2 -n 2 ./phost.intelmpi -F
    ```

Your output should look similar to the following

```
MPI VERSION Intel(R) MPI Library 2021.14 for Linux* OS
task    thread             node name  first task    # on node  core
0000      0000    gila-compute-36.novalocal        0000         0000  0001
0000      0001    gila-compute-36.novalocal        0000         0000  0000
0001      0000    gila-compute-36.novalocal        0000         0001  0031
0001      0001    gila-compute-36.novalocal        0000         0001  0030
```


### Compiling with OpenMPI

We can now follow these steps using OpenMPI as well! First, we will unload the Intel modules from the Intel toolchain. We will then load GNU modules and OpenMPI using the `module load` command from earlier. The commands are as follows:

```bash
module unload intel-oneapi-mpi
module unload oneapi
module load gcc
module load openmpi
```

We can then compile the phost program again by using the following commands:

```bash
mpicc -fopenmp phostone.c -o phost.openmpi
```

Once the program has been compiled against OpenMPI, we can go ahead and submit another batch script to test the program:


??? example "Batch Submission Script - OpenMPI"

    ```bash
    #!/bin/bash
    #SBATCH --nodes=1
    #SBATCH --ntasks=2
    #SBATCH --cpus-per-task=2
    #SBATCH --time=01:00:00
    #SBATCH --mem=20GB
    #SBATCH --account=aurorahpc

    module load gcc
    module load openmpi

    srun --cpus-per-task 2 -n 2 ./phost.openmpi -F
    ```




