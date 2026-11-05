[CP2K](https://www.cp2k.org) is a quantum chemistry and solid state physics software package that can perform atomistic simulations of solid state, liquid, molecular, periodic, material, crystal, and biological systems. CP2K provides a general framework for different modeling methods such as DFT using the mixed Gaussian and plane waves approaches GPW and GAPW. Supported theory levels include DFTB, LDA, GGA, MP2, RPA, semi-empirical methods (AM1, PM3, PM6, RM1, MNDO, …), and classical force fields (AMBER, CHARMM, …). CP2K can do simulations of molecular dynamics, metadynamics, Monte Carlo, Ehrenfest dynamics, vibrational analysis, core level spectroscopy, energy minimization, and transition state optimization using NEB or dimer method.


## Accessing CP2K on NLR's HPC Clusters

There  are 4 versions of CP2K available on NLR's HPC Clusters.

* CPU Default
* CPU Plumed
* GPU Default
* GPU Plumed

Runs of the GPU version must be launched from GPU login nodes kl5 or kl6. The modules for the GPU versions will only be visible from GPU login nodes. The CPU versions can only be seen/started from CPU login nodes.  

Module load procedure:

### CPU Default
```
module reset
module load cp2k/2026.1-cpu-default
```

### CPU Plumed
```
module reset
module load cp2k/2026.1-cpu-plumed
```

### GPU Default
```
module reset
module load gcc
module load openmpi/5.0.3
module load cp2k/2025.2-gpu-default
```

### GPU Plumed
```
module reset
module load gcc
module load openmpi/5.0.3
module load cp2k/2025.2-gpu-plumed
```

As of 10/26/2026, the GPU versions are build using OpenMPI and must be launched using mpirun instead of srun.  

The primary executable is `cp2k.psmp`.  There are other utilities which are in the same directory as `cp2k.psmp`.  These can be seen by running the command:

```
$(dirname $(which cp2k.psmp))
```

Below we have complete run scripts for the CPU and CPU versions of `cp2k.psmp`. The scripts encapsulate data files so they are runnable as is after you add you account information.  Each script runs both the default and plumed versions which is redundant but done here for completeness.

These examples will run in a few seconds.  

As written, the script will create a new directory for each run, go there and create the data file.  If you want to use this script with your own data remove or replace all of the lines from `cat > cp2k.inp <<'END_INPUT' ` to `END_INPUT`.  (There cannot be a space after END_INPUT or the file creation will fail.)

The name of the input file is on the srun/mpirun line; in this case *cp2k.inp*.  If you have a data file is in the starting directory you'll need to add lines to copy it to your execution directory.  For example after the line `cd $SLURM_JOBID` add:

```
cp ../myinput cp2k.inp
```



## Example run scripts

### Kestrel

??? example "cp2k CPU example with encapsulated data file"

    ```bash
    #!/bin/bash 
    #SBATCH --time=1:00:00 
    #SBATCH --partition=debug 
    #SBATCH --nodes=1 
    #SBATCH --exclusive 
    #SBATCH --account=ACCOUNT
     
    # make a directory for the run and go there 
     
    mkdir -p $SLURM_JOBID 
    cd $SLURM_JOBID 
     
     
    # create an input file 
     
    cat > cp2k.inp <<'END_INPUT' 
    &GLOBAL 
      PRINT_LEVEL MEDIUM 
      PROJECT H2O-4 
      RUN_TYPE ENERGY 
    &END GLOBAL 
     
    &FORCE_EVAL 
      METHOD Quickstep 
      &DFT 
        CHARGE 0 
        LSD 
        &QS 
          METHOD PM6 
          &SE 
            ANALYTICAL_GRADIENTS T 
            &COULOMB 
              CUTOFF [angstrom] 12.0 
              RC_RANGE [bohr] 1.0 
            &END COULOMB 
            &EXCHANGE 
              CUTOFF [angstrom] 4.9325 
              RC_RANGE [bohr] 1.0 
              RC_TAPER [angstrom] 12.0 
            &END EXCHANGE 
          &END SE 
        &END QS 
        &SCF 
          EPS_SCF 1.0E-8 
          MAX_SCF 40 
          SCF_GUESS ATOMIC 
          &OT 
            MINIMIZER DIIS 
            N_DIIS 9 
            PRECONDITIONER FULL_SINGLE_INVERSE 
          &END OT 
          &OUTER_SCF 
            EPS_SCF 1.0E-8 
            MAX_SCF 5 
          &END OUTER_SCF 
          &PRINT 
            &RESTART OFF 
            &END RESTART 
          &END PRINT 
        &END SCF 
      &END DFT 
      &SUBSYS 
        &CELL 
          ABC 9.865 9.865 9.865 
        &END CELL 
        &COORD 
          O                  -0.083   4.218   0.070   H2O 
          H                  -0.431   3.397   0.609   H2O 
          H                   0.377   3.756  -0.688   H2O 
          O                  -0.740  -1.262  -0.029   H2O 
          H                  -1.272  -0.422  -0.099   H2O 
          H                  -0.403  -1.349   0.873   H2O 
        &END COORD 
        &KIND H 
        &END KIND 
        &KIND O 
        &END KIND 
        &TOPOLOGY 
          CONNECTIVITY OFF 
        &END TOPOLOGY 
      &END SUBSYS 
    &END FORCE_EVAL 
    END_INPUT
    
    
    #save our script for a reference 
     
    cat $0 > script 
    
    # a simple timer
    mytime () 
    {
        now=`date +"%s.%N"`
        if (( $# > 0 )); then
            rtn=$(printf "%0.3f" `echo $now - $1 | bc`);
        else
           rtn=$(printf "%0.3f" `echo $now`);
        fi
        echo $rtn
    }
    
    #run it 
    
    module reset
    module load cp2k/2026.1-cpu-plumed 
     
    echo running plumed version
    
    T1=`mytime`
    export OMP_NUM_THREADS=4
    srun -n 8 cp2k.psmp cp2k.inp > cp2k_pl.out 
    DT=`mytime $T1`
    echo RUNTIME $DT
    
    module reset
    module load cp2k/2026.1-cpu-default  
    
    echo running not plumed version
    T1=`mytime`
    export OMP_NUM_THREADS=4
    srun -n 8 cp2k.psmp cp2k.inp > cp2k.out 
    DT=`mytime $T1`
    echo RUNTIME $DT
    ```


??? example "cp2k GPU example with encapsulated data file"

    ```bash
    #!/bin/bash 
    #SBATCH --time=1:00:00 
    #SBATCH --partition=gpu-h100 
    #SBATCH --gres=gpu:h100:4
    #SBATCH --nodes=1 
    #SBATCH --exclusive 
    #SBATCH --account=ACCOUNT

     
    # make a directory for the run and go there 
     
    mkdir -p $SLURM_JOBID 
    cd $SLURM_JOBID 
     
     
    # create an input file 
     
    cat > cp2k.inp <<'END_INPUT' 
    &GLOBAL 
      PRINT_LEVEL MEDIUM 
      PROJECT H2O-4 
      RUN_TYPE ENERGY 
    &END GLOBAL 
     
    &FORCE_EVAL 
      METHOD Quickstep 
      &DFT 
        CHARGE 0 
        LSD 
        &QS 
          METHOD PM6 
          &SE 
            ANALYTICAL_GRADIENTS T 
            &COULOMB 
              CUTOFF [angstrom] 12.0 
              RC_RANGE [bohr] 1.0 
            &END COULOMB 
            &EXCHANGE 
              CUTOFF [angstrom] 4.9325 
              RC_RANGE [bohr] 1.0 
              RC_TAPER [angstrom] 12.0 
            &END EXCHANGE 
          &END SE 
        &END QS 
        &SCF 
          EPS_SCF 1.0E-8 
          MAX_SCF 40 
          SCF_GUESS ATOMIC 
          &OT 
            MINIMIZER DIIS 
            N_DIIS 9 
            PRECONDITIONER FULL_SINGLE_INVERSE 
          &END OT 
          &OUTER_SCF 
            EPS_SCF 1.0E-8 
            MAX_SCF 5 
          &END OUTER_SCF 
          &PRINT 
            &RESTART OFF 
            &END RESTART 
          &END PRINT 
        &END SCF 
      &END DFT 
      &SUBSYS 
        &CELL 
          ABC 9.865 9.865 9.865 
        &END CELL 
        &COORD 
          O                  -0.083   4.218   0.070   H2O 
          H                  -0.431   3.397   0.609   H2O 
          H                   0.377   3.756  -0.688   H2O 
          O                  -0.740  -1.262  -0.029   H2O 
          H                  -1.272  -0.422  -0.099   H2O 
          H                  -0.403  -1.349   0.873   H2O 
        &END COORD 
        &KIND H 
        &END KIND 
        &KIND O 
        &END KIND 
        &TOPOLOGY 
          CONNECTIVITY OFF 
        &END TOPOLOGY 
      &END SUBSYS 
    &END FORCE_EVAL 
    END_INPUT
    
    
    #save our script for a reference 
    cat $0 > script 
    
    # a simple timer
    mytime () 
    {
        now=`date +"%s.%N"`
        if (( $# > 0 )); then
            rtn=$(printf "%0.3f" `echo $now - $1 | bc`);
        else
           rtn=$(printf "%0.3f" `echo $now`);
        fi
        echo $rtn
    }
    
    #run it 
    
    module reset
    module load gcc
    module load openmpi/5.0.3
    module load cp2k/2025.2-gpu-plumed
     
    echo running plumed version
    
    T1=`mytime`
    export OMP_NUM_THREADS=1
    IRFLAG="-mca mtl ^ofi --mca btl ^ofi"
    mpirun $IRFLAG -n 4 cp2k.psmp cp2k.inp > cp2k_pl.out 
    DT=`mytime $T1`
    echo RUNTIME $DT
    
    module reset
    module load gcc
    module load openmpi/5.0.3
    ml cp2k/2025.2-gpu-default  
    
    echo running not plumed version
    T1=`mytime`
    export OMP_NUM_THREADS=1
    IRFLAG="-mca mtl ^ofi --mca btl ^ofi"
    mpirun $IRFLAG -n 4 cp2k.psmp cp2k.inp > cp2k.out 
    DT=`mytime $T1`
    echo RUNTIME $DT    
    ```
