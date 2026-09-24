# Running CONVERGE Software

CONVERGE is a computational fluid dynamics (CFD) software package known for its automated meshing, high-fidelity combustion models, and efficiency on complex moving-geometry simulations (like engines, pumps, compressors, injectors, turbines, etc.). For more information about the software's features, see the [CONVERGE
website](https://convergecfd.com/).

The latest version of CONVERGE installed on Kestrel is version 5.1.0 (default).

## Tips

- **Licensing:**  
   NLR has one general-use CONVERGE license available. If you expect to run CONVERGE jobs frequently, we recommend obtaining your own license. To do this, please contact ITS to set up a license server for you and provide the port number and server name. 

- **Running CONVERGE-STUDIO:**  
   The running of CONVERGE GUI (CONVERGE-STUDIO) is not supported on Kestrel as it requires openssl/1.1.1 which is not available.	 
	 

## Running CONVERGE in Batch Mode

To run CONVERGE in batch mode: First, build your simulation and place all input files somewhere within your project directory. Then, create a Slurm script (as shown below) to submit the job. The Slurm script should be saved in the same directory as your input files.

In the following examples, Slurm scripts are provided for submitting CONVERGE jobs using Intel MPI or Open MPI.


### Running CONVERGE with Intel MPI

	#!/bin/bash
	#SBATCH -N 2                       # number of nodes
	#SBATCH --ntasks-per-node=24       # tasks per node
	#SBATCH --partition=shared         # partition
	#SBATCH -A <project handle>        # name of project allocation 
	#SBATCH -o converge-%j.out         # name of standard output file of Slurm 
	#SBATCH -e converge-%j.err         # name of error file of Slurm
	#SBATCH --time=01:00:00            # time
	
	module purge
	module load converge               # load CONVERGE module
	module load intel-oneapi-mpi       # load intel mpi
	unset I_MPI_PMI_LIBRARY            # unset library as the hydra process manager will be found
	
	mpiexec converge-intel-el8 --license super > outputfilename.out
    
### Running CONVERGE with Open MPI

	#!/bin/bash
	#SBATCH -N 2                       # number of nodes
	#SBATCH --ntasks-per-node=24       # tasks per node
	#SBATCH --partition=shared         # partition
	#SBATCH -A <project handle>        # name of project allocation 
	#SBATCH -o converge-%j.out         # name of standard output file of Slurm 
	#SBATCH -e converge-%j.err         # name of error file of Slurm
	#SBATCH --time=01:00:00            # time
	
	module purge
	module load converge               # load CONVERGE module
	module load openmpi                # load intel mpi
	
	mpiexec converge-ompi-el8 --license super > outputfilename.out

### Using Your Own License
If you have your own license installed on NLR’s license server, connect to it by setting the RLM_LICENSE variable in your Slurm script. Your Slurm script will look like the following example for Intel MPI:

	#!/bin/bash
	#SBATCH -N 2                       # number of nodes
	#SBATCH --ntasks-per-node=24       # tasks per node
	#SBATCH --partition=shared         # partition
	#SBATCH -A <project handle>        # name of project allocation 
	#SBATCH -o converge-%j.out         # name of standard output file of Slurm 
	#SBATCH -e converge-%j.err         # name of error file of Slurm
	#SBATCH --time=01:00:00            # time
	
	module purge
	module load converge               # load CONVERGE module
	
	export RLM_LICENSE=<port number>@<host name or IP> # connect to your own license
	
	module load intel-oneapi-mpi       # load intel mpi
	unset I_MPI_PMI_LIBRARY            # unset library as the hydra process manager will be found
	
	mpiexec converge-intel-el8 --license super > outputfilename.out
