# ⚡ HPC Benchmarking: Performance, Scalability and Energy Efficiency

Benchmarking of parallel applications on a high-performance computing cluster, comparing **MPI, OpenMP and hybrid parallelisation** on the NAS Parallel Benchmarks (multi-zone) and **GPU workloads with TensorFlow**, with the aim of understanding how each strategy affects execution time, resource usage and energy consumption.

---

## 📌 Overview

**Cluster:** up to 8 nodes (896 CPU cores in total) on two partitions: **GPP** (CPU nodes) and **ACC** (GPU nodes).

**Workloads:**
- **CPU:** three NAS Parallel Benchmarks (NPB) multi-zone kernels: **BT-MZ** (Block-Tridiagonal), **LU-MZ** (Lower-Upper symmetric Gauss-Seidel) and **SP-MZ** (Scalar Pentadiagonal).
- **GPU:** TensorFlow models (**DenseNet121**, **ResNet50**, **VGG19**) tested with two precision settings (basic and mixed).

**Parallel configurations** (each kernel is run under all four, on **4, 6 and 8 nodes**):

| Configuration | MPI processes per node | Threads per process |
|---|---|---|
| **MPI only** | Many | 1 |
| **OpenMP only** | 1 | Many |
| **Hybrid INT1** | Many | 8 |
| **Hybrid INT2** | Fewer | 16 |

**Metrics tracked:** execution time, speedup, parallel efficiency, CPI (cycles per instruction), memory bandwidth, average power draw and estimated energy cost.

**Monitoring:** [EAR (Energy Aware Runtime)](https://www.bsc.es/research-and-development/software-and-apps/software-list/ear-energy-management-framework-hpc) for power and energy tracking, plus custom Python utilities for merging CSV logs and computing derived metrics.

## 🔬 Techniques and findings

| Area | What was done | Main findings |
|---|---|---|
| **Scalability and performance** | BT-MZ, LU-MZ and SP-MZ run under MPI, OpenMP and two hybrid configurations on 4, 6 and 8 nodes. Speedup and parallel efficiency computed from elapsed time. | **Hybrid INT1** consistently gave the best trade-off between runtime and efficiency. MPI-only runs scaled well on CPU (lower CPI), but memory bandwidth dropped as the node count grew. OpenMP-only scaled poorly because of thread synchronisation overhead. |
| **Energy efficiency** | Power and energy collected with EAR for every run and combined with execution time to estimate energy cost. | **INT1** delivered the lowest energy consumption and the highest efficiency. **OpenMP-only** was the least energy efficient in every scenario. |
| **GPU workloads** | DenseNet121, ResNet50 and VGG19 on the ACC partition, with precision toggled between basic and mixed. | **Mixed precision** significantly reduced energy consumption compared with standard precision. |

### Example: BT-MZ on 8 nodes

| Configuration | Elapsed time (s) | Speedup | Efficiency (%) |
|---|---|---|---|
| MPI | 39.94 | 1.00 | 12.50 |
| OpenMP | 26.19 | 1.53 | 19.12 |
| **INT1** | **12.29** | **3.26** | **40.74** |
| INT2 | 14.91 | 2.69 | 33.60 |

Speedup is relative to the MPI-only run. INT1 is about 3.3× faster than MPI-only and more than three times as efficient.

**Overall conclusion:** hybrid MPI + OpenMP with a moderate number of threads (INT1) balances communication and memory pressure best, cutting both runtime and energy. Pure OpenMP is the worst option for energy, and mixed precision is an easy win on GPU.

## 📁 Repository structure

```
.
├── ProyectoHPC.zip    # SLURM scripts, project report, plots and summary tables
└── README.md
```

> ℹ️ Everything is inside `ProyectoHPC.zip`. Unzip it first to access the SLURM scripts (e.g. `run_bt_4nodes.sh`), the Python utilities (`merge_csv.py`, `calculate_energy.py`, `generate_plots.py`) and the TensorFlow scripts with precision toggling.

## ▶️ How to reproduce

**Requirements:** access to a SLURM-based HPC cluster with MPI, OpenMP, the NPB-MZ kernels and EAR installed, plus Python 3 with `pandas` and `matplotlib` for post-processing and a GPU partition with TensorFlow for the GPU experiments.

1. Clone the repository and unzip the project:
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   unzip ProyectoHPC.zip
   ```
2. Launch a kernel benchmark on the cluster:
   ```bash
   sbatch run_bt_4nodes.sh
   ```
3. Copy the EAR energy metrics back to your machine:
   ```bash
   scp mn5:/path/to/results/*.csv .
   ```
4. Merge the logs and compute the derived metrics:
   ```bash
   python merge_csv.py
   python calculate_energy.py
   ```
5. Generate plots and summary tables:
   ```bash
   python generate_plots.py
   ```

## 🛠️ Tools

- **MPI + OpenMP** · hybrid parallelisation of the NPB-MZ kernels
- **SLURM** · job submission on the GPP and ACC partitions
- **EAR (Energy Aware Runtime)** · power, energy, CPI and memory-bandwidth monitoring
- **Python** · `pandas`-style CSV merging, metric computation and plotting
- **TensorFlow** · DenseNet121, ResNet50 and VGG19 with basic and mixed precision

## 📚 References

- NAS Parallel Benchmarks (NPB), NASA Advanced Supercomputing Division.
- EAR: Energy Aware Runtime, Barcelona Supercomputing Center.
- TensorFlow Documentation, https://www.tensorflow.org/

## 👥 Authors

- **Lucas Viner**
- **Oriol Parent**

## 📝 License

This project is licensed under the MIT License.
