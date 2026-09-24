# Quantum Computing

NLR provides a GPU-accelerated quantum computing environment on Kestrel through the
`qiskit/aer-gpu` module. The module bundles several frameworks for quantum circuit
simulation and hybrid quantum-classical algorithms:

| Package | Version | Description |
|---------|---------|-------------|
| [Qiskit](https://www.ibm.com/quantum/qiskit) | 2.2.3 | Quantum circuit construction, transpilation, and execution |
| [qiskit-aer-gpu](https://github.com/Qiskit/qiskit-aer) | 0.15.1 | GPU-accelerated statevector, density matrix, and shot simulators (H100, sm_90) |
| [qiskit-algorithms](https://github.com/qiskit-community/qiskit-algorithms) | 0.4.0 | VQE, QAOA, QAE, amplitude estimation primitives |
| [CUDA-Q](https://developer.nvidia.com/cuda-q) | 0.14.0 | NVIDIA's unified quantum-classical programming model (Python + C++) |
| [cuQuantum](https://developer.nvidia.com/cuquantum-sdk) | 26.3.0 | NVIDIA cuStateVec and cuTensorNet GPU backends |
| [PyTorch](https://pytorch.org/) | 2.10.0 | (inherited) for classical ML integration |
| [mpi4py](https://mpi4py.readthedocs.io/) | 4.1.1 | (inherited) for multi-node parallel workflows |

!!! note
    The `qiskit/aer-gpu` module targets the **H100 GPUs (sm_90)** on Kestrel's GPU nodes.

---

## Loading the Module
Please use only a GPU login node or an allocated GPU node to use the quantum computing module. 
```bash
module load qiskit/aer-gpu
python3 your_script.py
```

To see what the module provides:

```bash
module help qiskit/aer-gpu
```

---

## Qiskit Aer GPU Simulation

`qiskit-aer-gpu` replaces the CPU `AerSimulator` with a GPU-accelerated backend.
See [`qae_example_backends.ipynb`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/qae_example_backends.ipynb)
for a worked example comparing CPU and GPU backends for a QAE circuit.

The simplest way to enable GPU simulation is:

```python
from qiskit_aer import AerSimulator
from qiskit.circuit.library import QFT
from qiskit import transpile

simulator = AerSimulator(method='statevector', device='GPU')

qc = QFT(20)  # 20-qubit Quantum Fourier Transform
qc.measure_all()
t_qc = transpile(qc, simulator)
result = simulator.run(t_qc, shots=1024).result()
counts = result.get_counts()
print(counts.most_frequent())
```

For circuits with more than ~25 qubits, enable cuStateVec for better performance:

```python
simulator = AerSimulator(
    method='statevector',
    device='GPU',
    cuStateVec_enable=True,
)
```

---

## CUDA-Q

[CUDA-Q](https://developer.nvidia.com/cuda-q) is NVIDIA's unified quantum-classical
programming framework. Kernels are written in Python (or C++) and JIT-compiled to
run on CPUs, GPUs, or QPU hardware.

### Single GPU

See [`binary_optimizer.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/binary_optimizer.py)
and [`qae.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/qae.py)
for real-world examples using the single-GPU target.

```python
import cudaq

cudaq.set_target('nvidia')   # single H100, complex<float> (fp32)

@cudaq.kernel
def bell():
    q = cudaq.qvector(2)
    h(q[0])
    cx(q[0], q[1])
    mz(q)

result = cudaq.sample(bell, shots_count=1000)
print(result)   # { 00:~500  11:~500 }
```

### Noisy Simulation

CUDA-Q supports trajectory-based noisy simulation using a depolarizing noise model.
See [`binary_optimizer.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/binary_optimizer.py)
for a production example with per-gate noise channels applied to a QAE circuit.

```python
import cudaq

cudaq.set_target('nvidia')

noise = cudaq.NoiseModel()
dep1 = cudaq.Depolarization1(0.0001)   # single-qubit gate error rate
dep2 = cudaq.Depolarization2(0.001)    # two-qubit gate error rate
for g in ['h', 'x', 'y', 'z', 'rx', 'ry', 'rz']:
    noise.add_all_qubit_channel(g, dep1)
for g in ['cx', 'cz', 'cry']:
    noise.add_all_qubit_channel(g, dep2)

result = cudaq.sample(bell, shots_count=1000, noise_model=noise)
```

### Multi-QPU Shot-Parallel (mqpu)

The `mqpu` option assigns shots to multiple GPUs in parallel. Each GPU holds its own
copy of the full statevector and executes an independent subset of shots — useful
for shot-noisy sweeps where the circuit fits in a single GPU's memory.
See [`par_benchmark_2gpu.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/par_benchmark_2gpu.py)
for a DQA benchmark using this strategy on Kestrel H100s.

```python
import cudaq

# Each of the 4 GPUs handles ~shots/4 shots independently
cudaq.set_target('nvidia', option='mqpu')

@cudaq.kernel
def ansatz(theta: float):
    q = cudaq.qvector(28)
    h(q[0])
    ry(theta, q[1])
    cx(q[1], q[0])
    mz(q)

result = cudaq.sample(ansatz, 0.5, shots_count=4096)
cudaq.mpi.finalize()
```

```bash
#SBATCH --account=<your-account>
#SBATCH --time=01:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --mem=320G
#SBATCH --gpus-per-node=4

module load qiskit/aer-gpu
srun python3 my_mqpu_script.py
```

!!! note "mqpu vs mpi4py vs mgpu"
    | Strategy | When to use | Circuit size |
    |----------|-------------|-------------|
    | `mqpu` | Shot-parallel, built-in CUDA-Q, no MPI boilerplate | Fits in 1 GPU |
    | `mpi4py` shot-split | Noisy simulation, fine-grained rank control | Fits in 1 GPU |
    | `mgpu` | Circuit exceeds single-GPU memory | > 30 qubits |

    For noisy circuits that fit in one GPU (≤28 qubits), **`mpi4py` shot-splitting
    typically outperforms `mqpu`** due to lower inter-process overhead.
    See the [parallelization report](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/parallelisation_report.md)
    for benchmarks on Kestrel H100s.

### Multi-GPU Statevector (mgpu)

The `mgpu` option of the `nvidia` target distributes the statevector across multiple
GPUs using cuStateVec, enabling simulation of circuits that exceed single-GPU memory:

| GPUs (H100 80 GB each) | fp32 qubit limit | Total statevector memory |
|------------------------|-----------------|-------------------------|
| 1 | ~33 qubits | ~64 GB |
| 4 | ~35 qubits | ~256 GB (~64 GB/GPU) |
| 8 | ~36 qubits | ~512 GB (~64 GB/GPU) |

On Kestrel, multi-GPU requires GPU-aware MPI via the Cray GTL library. Use the
wrapper script below to set the required environment variables **before** Python
starts — they cannot be set with `os.environ` inside the script.
See [`bench_mgpu_dqa.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/bench_mgpu_dqa.py)
and [`par_benchmark_mgpu.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/par_benchmark_mgpu.py)
for full DQA benchmarking examples.

**[`run_mgpu.sh`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/run_mgpu.sh):**
```bash
#!/bin/bash
export MPICH_GPU_SUPPORT_ENABLED=1
export LD_LIBRARY_PATH=/nopt/cuda/12.4/lib64${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}
export LD_PRELOAD=/opt/cray/pe/mpich/8.1.28/gtl/lib/libmpi_gtl_cuda.so
export CUDAQ_MGPU_LIB_MPI=/opt/cray/pe/mpich/8.1.28/ofi/gnu/10.3/lib/libmpi.so
export CUDAQ_MGPU_COMM_PLUGIN_TYPE=MPICH
exec python3 "$@"
```

**[Script using mgpu](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/test_mgpu.py):**
```python
import cudaq
cudaq.set_target('nvidia', option='mgpu,fp32')

@cudaq.kernel
def large_circuit(n: int):
    q = cudaq.qvector(n)
    h(q[0])
    for i in range(n - 1):
        cx(q[i], q[i + 1])
    mz(q)

result = cudaq.sample(large_circuit, 34, shots_count=256)
cudaq.mpi.finalize()
```

---

## Example Batch Scripts

### Single GPU — Qiskit Aer

```bash
#!/bin/bash
#SBATCH --account=<your-account>
#SBATCH --nodes=1
#SBATCH --ntasks=32
#SBATCH --mem=80G
#SBATCH --gpus=1
#SBATCH --time=01:00:00
#SBATCH --job-name=qiskit_gpu

module load qiskit/aer-gpu
python3 my_qiskit_script.py
```

### Single GPU — CUDA-Q

```bash
#!/bin/bash
#SBATCH --account=<your-account>
#SBATCH --nodes=1
#SBATCH --ntasks=32
#SBATCH --mem=80G
#SBATCH --gpus=1
#SBATCH --time=01:00:00
#SBATCH --job-name=cudaq_single

module load qiskit/aer-gpu
python3 my_cudaq_script.py
```

### Multi-GPU (4 GPUs, single node) — CUDA-Q mgpu

```bash
#!/bin/bash
#SBATCH --account=<your-account>
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --mem=320G
#SBATCH --gpus-per-node=4
#SBATCH --time=02:00:00
#SBATCH --job-name=cudaq_mgpu

module load qiskit/aer-gpu
srun bash run_mgpu.sh my_cudaq_mgpu_script.py
```

### Multi-Node (8 GPUs, 2 nodes × 4 GPUs) — CUDA-Q mgpu

```bash
#!/bin/bash
#SBATCH --account=<your-account>
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128
#SBATCH --mem=320G
#SBATCH --gpus-per-node=4
#SBATCH --time=04:00:00
#SBATCH --job-name=cudaq_mgpu_8gpu

module load qiskit/aer-gpu
srun bash run_mgpu.sh my_cudaq_mgpu_script.py
```

### Multi-Node Shot-Parallel (noisy, mpi4py) — CUDA-Q

For noisy trajectory simulation with many shots, splitting shots across MPI ranks
(one rank per GPU) gives near-linear speedup. See
[`par_benchmark_mpi.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/par_benchmark_mpi.py)
for a full working example.

```bash
#!/bin/bash
#SBATCH --account=<your-account>
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=64
#SBATCH --mem=160G
#SBATCH --gpus-per-node=2
#SBATCH --time=04:00:00
#SBATCH --job-name=cudaq_mpi4py

module load qiskit/aer-gpu
srun python3 my_mpi4py_cudaq_script.py
```

Inside the script, each rank binds to one GPU before importing CUDA-Q:

```python
import os
from mpi4py import MPI
import cudaq

comm = MPI.COMM_WORLD
os.environ['CUDA_VISIBLE_DEVICES'] = os.environ.get('SLURM_LOCALID', str(comm.rank))
cudaq.set_target('nvidia')

my_shots = N_SHOTS // comm.size
counts = cudaq.sample(kernel, *args, shots_count=my_shots, noise_model=noise)

all_counts = comm.gather(counts, root=0)
```

---

## Using Jupyter Notebooks

```bash
module load qiskit/aer-gpu
jupyter lab --no-browser --ip=0.0.0.0
```

See the [Jupyter documentation](../Development/Jupyter/index.md) for instructions on
forwarding the port to your local browser.

A worked example notebook for QAE-based amplitude estimation is available at
[`qae_example.ipynb`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/qae_example.ipynb).

---

## Installing Additional Packages

To add packages on top of the shared environment without modifying it:

```bash
module load qiskit/aer-gpu
python3 -m venv /scratch/$USER/qiskit_env --system-site-packages
source /scratch/$USER/qiskit_env/bin/activate
pip install <your-packages>
python3 your_script.py
```

!!! warning
    Do not run `pip install` directly into the shared module environment. Use a
    personal venv in `/scratch` as shown above.

---

## Performance Notes

- **Noise model limits**: noisy trajectory simulation requires one full statevector
  per shot in GPU memory. At fp32 a single H100 holds up to ~33 qubits (~64 GB);
  at fp64 the limit is ~30 qubits (~32 GB). Use `mgpu,fp32` for noisy circuits
  with more than ~33 qubits.

- **Scaling**: for noisy simulation at ≤28 qubits, shot-splitting via mpi4py
  outperforms mgpu because shots run in parallel rather than sequentially.

- **JIT warm-up**: the first CUDA-Q `sample()` call for a new circuit/device
  combination triggers NVRTC kernel compilation (~30–60 s for 28 qubits). Subsequent
  calls use the cache and are much faster.

---

## Additional Resources

**NVIDIA / IBM documentation:**

- [CUDA-Q documentation](https://nvidia.github.io/cuda-quantum/)
- [Qiskit documentation](https://docs.quantum.ibm.com/)
- [qiskit-aer documentation](https://qiskit.github.io/qiskit-aer/)
- [cuQuantum SDK](https://developer.nvidia.com/cuquantum-sdk)

**NLR example code (Kestrel-tested):**

- [`run_mgpu.sh`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/run_mgpu.sh) — Cray GTL wrapper for mgpu
- [`test_mgpu.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/test_mgpu.py) — GHZ circuit on nvidia mgpu target
- [`bench_mgpu_dqa.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/bench_mgpu_dqa.py) — DQA benchmark: noiseless + noisy mgpu timing
- [`par_benchmark_mgpu.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/par_benchmark_mgpu.py) — scaling benchmark across multiple GPUs with mgpu
- [`par_benchmark_mpi.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/par_benchmark_mpi.py) — DQA benchmark: mpi4py shot-splitting across ranks
- [`par_benchmark_2gpu.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/par_benchmark_2gpu.py) — DQA benchmark: mqpu 2-GPU shot-parallel
- [`parallelisation_report.md`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/parallelisation_report.md) — Full benchmark comparison (mqpu / mpi4py / mgpu) on Kestrel H100s
- [`qae_example_backends.ipynb`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/qae_example_backends.ipynb) — Qiskit Aer GPU vs CPU backend comparison notebook
- [`binary_optimizer.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/binary_optimizer.py) — CUDA-Q QAE-based stochastic optimiser
- [`qae.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/qae.py) — Quantum Amplitude Estimation circuits
- [`resource_estimator.py`](https://github.com/NatLabRockies/quantum_stochastic_programming/blob/main/qiskit_impl/resource_estimator.py) — Gate/qubit resource estimation utilities
