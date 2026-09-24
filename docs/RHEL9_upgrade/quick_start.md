# Quick Start

## Accessing RHEL9

All users are encouraged to test their workflows on a set of nodes that are already upgraded to RHEL9 to prepare for the system-wide transition. To do so, you must first be connected to a [dedicated RHEL9 login node](./access.md). Please reach out to [HPC-Help@nlr.gov](mailto:hpc-help@nlr.gov) with any issues or questions.

## Frequently Asked Questions (FAQ)

Please see our dedicated [page](./knownissues.md#frequently-asked-questions-faq) for FAQs about RHEL9 on Kestrel.

## Using Legacy (RHEL8) Software

Kestrel users can still use old (RHEL8) software modules **at their own risk** and with the understanding that those modules **will no longer be maintained** by support staff after the OS upgrade. Refer to the dedicated [Legacy Software](./modules/old_software.md) page for more information.

## Software modules on RHEL9 

!!! Note
    This subsection is targeted towards users who manage their own software environments and need access to specific modules for compilers, MPI, scientific libraries, etc. The new module system still allows for end-use applications to be loaded directly (e.g., `module load vasp`). If you primarily run such end-use applications, you should not notice a major change in your experience using Kestrel.

Kestrel's module system design is [significantly changing](./modules/index.md) with the RHEL9 upgrade. In general, to use the [Cray Programming Environments (CPE)](./modules/cpe.md), you must first run `module load cpe-stack` for the CPE modules to become available. For environments outside of CPE (e.g., [a user-defined one](./modules/user-toolchains.md) built with GNU), you must load the compiler first as a "base module" to reveal software built for that specific toolchain.

!!! Warning
    Mixing CPE and non-CPE environments in the same job or build script is not recommended.

### RHEL9 Module "Cheat Sheet"

Please refer to the non-exhaustive translation table below to note how to load the equivalent module(s) between RHEL8 and RHEL9 for common module operations.

## RHEL8 → RHEL9 Module Translation Table (CPU side)

| RHEL8 module(s) | RHEL9 equivalent | Environment type |
|---|---|---|
| `vasp` | `vasp` | End-use application (unchanged) |
| `lammps` | `lammps` | End-use application (unchanged) |
| `PrgEnv-intel` | `cpe-stack` + `PrgEnv-intel` | CPE |
| `PrgEnv-cray` | `cpe-stack` + `PrgEnv-cray` | CPE |
| `PrgEnv-gnu` | `cpe-stack` + `PrgEnv-gnu` | CPE |
| `gcc-stdalone` | `gcc` | User-defined (NLR toolchain) |
| `intel-oneapi-compilers` | `oneapi` | User-defined (NLR toolchain) |
| `conda` / `anaconda3` | `miniforge3` (conda/mamba inside) | Python / Miniforge |
| `openmpi` (standalone) | `openmpi` (load after gcc/oneapi) | User-defined (NLR toolchain) |
| `application` (flat app tree) | `application/26.05` | Research Applications |
| (n/a — RHEL8-only builds) | `application/rhel8` (compat bridge) | Legacy bridge |
| `cce/17.0.0` (D) | `cpe-stack` + `PrgEnv-cray` | CPE |
| `PrgEnv-aocc` | `cpe-stack` + `PrgEnv-aocc` | CPE |
| `cray-libsci/23.12.5` | `cpe-stack` + `PrgEnv-gnu` (bundles `cray-libsci/25.03.0`) | CPE |
| `cray-mpich/8.1.28` (L,D) | `cray-mpich/8.1.32` / `cray-mpich/9.0.0` | CPE |
| `cray-fftw/3.3.10.6` (D) | `cpe-stack` (bundles `cray-fftw/3.3.10.10`) | CPE |
| `cray-hdf5/1.12.2.9` (D) | `cpe-stack` (bundles `cray-hdf5/1.14.3.5`) | CPE |
| `cray-netcdf/4.9.0.9` | `cpe-stack` (bundles `cray-netcdf/4.9.0.17`) | CPE |
| `intel-oneapi-compilers/2024.1.0` | `oneapi/2025.3.1` | User-defined (NLR toolchain) |
| `intel-oneapi-mpi/2021.12.1-intel` | `intel-oneapi-mpi/2021.17.2` (after `oneapi`) | User-defined (NLR toolchain) |
| `openmpi/5.0.1-gcc` | `openmpi/5.0.3` (load `gcc` first) | User-defined (NLR toolchain) |
| `mpich/4.1-gcc` | `mpich/5.0.0` (load `gcc` first) | User-defined (NLR toolchain) |
| `cmake/3.29.2` (D) | `cmake/3.31.11` | Core module (any toolchain) |
| `git/2.48.1` (D) | `git/2.52.0` | Core module (any toolchain) |
| `python/3.11.4` (D) | `python/3.11.14` (3.10–3.14 also avail) | Core module (any toolchain) |
| `boost/1.85.0-openmpi-gcc` (D) | `boost/1.90.0-mpi` (load `gcc`+`openmpi`) | User-defined (NLR toolchain) |
| `hdf5/1.14.3-openmpi-gcc` | `hdf5/1.14.6-mpi` (load `gcc`+`openmpi`) | User-defined (NLR toolchain) |
| `netcdf-c/4.9.2-openmpi-gcc` (D) | `netcdf-c/4.9.3-mpi` (load `gcc`+`openmpi`) | User-defined (NLR toolchain) |
| `hypre/2.25.0-openmpi-gcc` (D) | `hypre/3.1.0-mpi` (load `gcc`+`openmpi`) | User-defined (NLR toolchain) |
| `matlab/R2023a` | `matlab/R2025a` or `matlab/R2025b` | End-use application |
| `gurobi/13.0.0` (D) | `gurobi/13.0.3` | End-use application (unchanged name) |
| `nwchem/7.0.0` | `nwchem/7.3.1` | End-use application (unchanged name) |
| `ansys/2025R2` (D) | `ansys/2026R1` (D) or `ansys/default` | End-use application (unchanged name) |
| `wrf/4.6.1-cray-mpich-gcc` | `wrf/4.8.0-craype-gnu` (or `4.5.1-craype-gnu`) | End-use application |

> **Note:** this table is non-exhaustive. Always confirm with `module spider <old_module_name>`.

## RHEL8 → RHEL9 Module Translation Table (GPU side)

| RHEL8 module(s) | RHEL9 equivalent | Environment type |
|---|---|---|
| `PrgEnv-nvhpc` | `cpe-stack` + `PrgEnv-nvidia` | CPE |
| `nvhpc-nompi` / `nvhpc-stdalone` | `nvhpc` | User-defined (NLR toolchain) |
| `cuda` (various point versions) | `cuda/12.8.1` (D) or `cuda/13.2` | User-defined (NLR toolchain) |
| `openmpi/4.1.6-nvhpc` | `openmpi/5.0.3-gpu` (with gcc+cuda) | User-defined (NLR toolchain) |
| `gcc-standalone` | `gcc` (native/standalone builds removed from RHEL9 GPU catalog — see [§3](#3-gcc-variant-gotcha-gpu-nodes-only)) | User-defined (NLR toolchain) |
| `conda` / `anaconda3` | `miniforge3` | Python / Miniforge |
| `vasp` (GPU builds) | `vasp/6.5.1_openMP` (D), etc. | End-use application (unchanged) |
| `lammps`-gpu builds | `lammps/22Jul2025-gpu-double\|mixed` etc. | End-use application (unchanged) |
| `application` (flat GPU app tree) | `application/26.05` | Research Applications |
| `nvhpc/23.9` (D) | `nvhpc/26.1` (D) (also 22.7/23.3/24.3/25.11 available) | User-defined (NLR toolchain) |
| `gcc/13.1.0` (compilers_mpi) | `gcc/14.2.0` (Base Modules) | User-defined (NLR toolchain) |
| `cudnn/9.2.0.82-12` (D) | `cudnn/9.17.0.29-13-gpu` (D), also `cudnn/9.17.0.29-12-gpu` | Core module (any toolchain) |
| `intel-oneapi-mpi/2021.13.0-intel` | `intel-oneapi-mpi/2021.17.2` | User-defined (NLR toolchain) |
| `hdf5/1.14.3-openmpi-gcc` (D) | `hdf5/1.14.6-mpi` (load gcc+openmpi) | User-defined (NLR toolchain) |
| `boost/1.85.0-openmpi-gcc` (D) | `boost/1.90.0-mpi` (also 1.84.0-mpi, 1.89.0-mpi available) | User-defined (NLR toolchain) |
| `python/3.12.4` (D) | `python/3.12.13` (3.10–3.14 also avail) | Core module (any toolchain) |
| `cmake/3.29.6` (D) | `cmake/3.31.11` | Core module (any toolchain) |
| `gdb/12.1` | `gdb/17.1` | Core module (any toolchain) |
| `emacs/29.2` | `emacs/30.2` | Core module (any toolchain) |
| `htop/3.2.2` | `htop/3.4.1` | Core module (any toolchain) |
| `hwloc/2.9.3` | `hwloc/2.12.2` (or `2.9.3-mpi` / `2.9.3-gpu-mpi` for MPI-aware builds) | Core module (any toolchain) |
| `forge/25.1.1` (D) | `forge/26.0.1` (D) | End-use application (unchanged name) |
| `julia/1.12.1` (D) | `julia/1.12.1` (also 1.10.4/1.11.4) | End-use application (mostly unchanged) |
| `paraview/5.12.0-egl-server` (D) | `paraview/5.13.3-gui` (D) / `5.12.0-egl-server` still available | End-use application (unchanged name) |

> **Note:** this table is non-exhaustive. Always confirm with `module spider <old_module_name>`.




### Troubleshooting

| Problem | Fix |
|---|---|
| Can't find a module | `module spider <name>` |
| Won't load (deps missing) | `module spider <name>/<version>` then load listed prereqs first |
| Mixed/conflicting toolchains | `module reset`, then reload one workflow only |
| Weird broken shell state | `module reset` (NOT `module purge`) |


### Quick Reference Table 

| Workflow | Commands |
|---|---|
| NLR gcc | `module reset; module load gcc/14.2.0` |
| NLR oneAPI | `module reset; module load oneapi/2025.3.1` |
| CPE | `module reset; module load cpe-stack/25.03; module load PrgEnv-gnu` |
| Miniforge | `module reset; module load miniforge3/26.1.1-3` |
| Container | `module reset; module load apptainer/1.4.1-runonly` |
| Find any package | `module spider <name>` |
| Find package deps | `module spider <name>/<version>` |
| See what's loaded | `module list` |
| Inspect a module | `module show <name>` |
| Undo everything safely | `module reset` |