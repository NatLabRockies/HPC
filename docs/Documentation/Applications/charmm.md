# Charmm

## Overview

CHARMM (Chemistry at HARvard Macromolecular Mechanics) is a molecular simulation software package used for modeling biological macromolecules, including proteins, nucleic acids, and lipids.

CHARMM 50b2 is installed on Kestrel and supports parallel execution using Intel MPI.


## Loading CHARMM

To load CHARMM on Kestrel, use the following command:

```bash
module load charmm
```

## Running CHARMM with Slurm

CHARMM jobs can be submitted to Kestrel using the Slurm workload manager.

Create a Slurm submission script named `charmm.slurm`:

```bash
#!/bin/bash
#SBATCH --job-name=charmm_test
#SBATCH --account=<your project handdle>
#SBATCH --partition=debug
#SBATCH --nodes=1
#SBATCH --ntasks=4
#SBATCH --time=00:05:00
#SBATCH --output=charmm_test.%j.out
#SBATCH --error=charmm_test.%j.err

module load charmm

export I_MPI_FABRICS=shm:ofi
export FI_PROVIDER=tcp

cd ${SLURM_SUBMIT_DIR}

mpirun -np $SLURM_NTASKS charmm test_charmm.inp test_charmm.out
```

This example requests one compute node with four MPI tasks for five minutes on the `debug` partition.


## Submitting a Job

Ensure that the CHARMM input file `test_charmm.inp` is located in the submission directory.

Submit the job using:

```bash
sbatch charmm.slurm
```

The following output files will be generated:

- `charmm_test.<jobid>.out`: Slurm standard output
- `charmm_test.<jobid>.err`: Slurm standard error
- `test_charmm.out`: CHARMM simulation output

## Additional Information

For more information about CHARMM, visit the [official CHARMM website](https://academiccharmm.org/).
