# Experiment 4: Cache Coherence (MSI, MESI, MOESI) and NUMA Memory Access

## 1. Overview

This experiment explores how cache coherence protocols maintain consistent data across multiple CPU cores and how NUMA (Non-Uniform Memory Access) affects memory access performance.

The experiment focuses on:

- Studying MSI, MESI and MOESI cache coherence protocols.
- Understanding cache-line invalidation and false sharing.
- Comparing adjacent and padded memory layouts.
- Measuring local and remote NUMA memory access.
- Evaluating execution time, correctness and scalability with different thread counts.

## 2. Objectives

1. Understand why cache coherence is necessary in multi-core processors.
2. Simulate MSI, MESI and MOESI protocols and compare their bus transactions.
3. Observe the performance impact of false sharing.
4. Compare adjacent and padded counter layouts.
5. Study NUMA memory placement and its effect on execution time.
6. Analyze performance as the number of threads increases.

## 3. Technologies Used

| Technology | Purpose |
| --- | --- |
| Ubuntu 22.04 LTS | Operating system |
| VMware Workstation Pro | Virtualization |
| GCC Compiler | Compiling C programs |
| OpenMP | Parallel programming |
| Python 3 | Cache coherence simulation |
| numactl | NUMA CPU and memory binding |
| hwloc | Hardware topology information |

## 4. Theory and Concepts

### 4.1 Cache Coherence

In a multi-core processor, each CPU core may have its own private cache. When multiple cores access the same memory location, they may hold separate copies of the data.

If one core modifies its copy, other cores must be informed so they do not continue using stale data.

Cache coherence protocols ensure that all cores maintain a consistent view of shared memory.

### 4.2 MSI Protocol

MSI uses three states:

| State | Meaning |
| --- | --- |
| Modified (M) | Cache has the only valid, modified copy. |
| Shared (S) | Cache holds a clean copy that may be shared. |
| Invalid (I) | Cache line is not valid. |

A write to a shared cache line requires an upgrade transaction and invalidates other copies.

### 4.3 MESI Protocol

MESI extends MSI by introducing the **Exclusive (E)** state.

| State | Meaning |
| --- | --- |
| Modified (M) | Exclusive modified copy. |
| Exclusive (E) | Only copy, but clean. |
| Shared (S) | Clean copy shared with other caches. |
| Invalid (I) | Invalid cache line. |

A cache line in the Exclusive state can transition to Modified silently when the same core writes to it. This avoids an additional bus transaction.

### 4.4 MOESI Protocol

MOESI extends MESI by introducing the **Owned (O)** state.

The Owned state allows one cache to hold a modified line while supplying its data to other caches. This can avoid an immediate write-back to main memory.

The five states are:

- Modified (M)
- Owned (O)
- Exclusive (E)
- Shared (S)
- Invalid (I)

### 4.5 False Sharing

False sharing occurs when two threads modify different variables that happen to occupy the same cache line.

Even though the threads access different variables, cache coherence operates on entire cache lines. This causes unnecessary invalidations and cache-line transfers between cores.

**Solution:** Padding or alignment can place frequently modified variables on separate cache lines.

### 4.6 NUMA (Non-Uniform Memory Access)

In NUMA systems, memory is divided between different nodes.

- **Local memory:** Memory attached to the same NUMA node as the CPU.
- **Remote memory:** Memory attached to another NUMA node.

Local memory access is generally faster because remote access requires communication across an interconnect.

Linux commonly uses a first-touch policy, where memory pages are placed according to the node of the thread that first accesses them.

The `numactl` utility can control CPU and memory placement.

## 5. Programs Implemented

| No. | Program | Description |
| --- | --- | --- |
| 1 | `coherence_sim.py` | Simulates MSI, MESI and MOESI cache coherence protocols. |
| 2 | `false_sharing.c` | Compares adjacent and padded counter layouts. |
| 3 | `numa_sweep.c` | Measures NUMA memory access performance. |
| 4 | `compare_numa.c` | Compares execution time, correctness and scalability. |

## 6. Installation and Setup

Update Ubuntu packages:

```
sudo apt update
```

Install the required tools:

```
sudo apt install build-essential numactl hwloc python3 -y
```

Verify GCC:

```
gcc --version
```

Verify OpenMP support:

```
echo '#include <omp.h>' | gcc -E -fopenmp -xc - > /dev/null && echo "OpenMP OK"
```

Check CPU and NUMA configuration:

```
lscpu | grep -E "Socket|Core|Thread|NUMA"
numactl --hardware
lscpu -e
```

## 7. Compilation

Compile the C programs from the project directory.

### False Sharing

```
gcc -O2 false_sharing.c -fopenmp -o false_sharing
```

### NUMA Memory Sweep

```
gcc -O2 numa_sweep.c -fopenmp -o numa_sweep
```

### Combined Comparison

```
gcc -O2 compare_numa.c -fopenmp -o compare_numa
```

The Python simulator does not require compilation.

## 8. Execution

### Program 1: Cache Coherence Simulation

```
python3 coherence_sim.py 1000
```

This program simulates MSI, MESI and MOESI protocols using two cores and two counters.

It displays:

- Cache-state transitions.
- Bus transactions.
- Invalidations.
- Write-backs to memory.
- Cache-to-cache transfers.

It also compares adjacent and padded memory layouts.

### Program 2: False Sharing

Run with adjacent counters:

```
OMP_NUM_THREADS=2 ./false_sharing adjacent
```

Run with padded counters:

```
OMP_NUM_THREADS=2 ./false_sharing padded
```

This program compares the execution time of two memory layouts and displays the final counter value.

### Program 3: NUMA Memory Sweep

First, identify the available NUMA nodes:

```
numactl --hardware
```

Run with local memory placement:

```
OMP_NUM_THREADS=2 numactl --cpunodebind=0 --membind=0 ./numa_sweep
```

Run with remote memory placement:

```
OMP_NUM_THREADS=2 numactl --cpunodebind=0 --membind=1 ./numa_sweep
```

Replace node numbers according to the available NUMA configuration.

### Program 4: Combined Comparison

```
./compare_numa
```

This program performs two comparisons:

**Table 1: False Sharing**

- Adjacent and padded layouts.
- 1, 2 and 4 threads.
- Final counter value.
- Correctness verification.
- Execution time.

**Table 2: NUMA Memory Sweep**

- 512 MB memory buffer.
- 1 and 2 threads.
- Execution time for multiple passes.

To compare local and remote memory placement:

```
numactl --cpunodebind=0 --membind=0 ./compare_numa
```

```
numactl --cpunodebind=0 --membind=1 ./compare_numa
```

## 9. Observations and Results

### 9.1 Protocol Comparison

| Protocol | Main Observation |
| --- | --- |
| MSI | Requires an upgrade transaction when writing to a shared line. |
| MESI | Exclusive state avoids an upgrade transaction for private data. |
| MOESI | Owned state allows cache-to-cache transfers without an immediate memory write-back. |

The simulation demonstrates that all three protocols maintain consistency but differ in the amount and type of coherence traffic.

### 9.2 False Sharing Comparison

| Layout | Expected Behaviour |
| --- | --- |
| Adjacent | More cache-line bouncing and potentially higher execution time. |
| Padded | Less cache-line interference and generally better performance. |

Both layouts should produce the correct final counter value. Their execution times may differ because of cache coherence traffic.

### 9.3 NUMA Comparison

| Memory Placement | Expected Behaviour |
| --- | --- |
| Local | Generally lower memory access latency. |
| Remote | Generally higher memory access latency on multi-socket systems. |

On a single-socket computer or virtual machine, local and remote memory timings may be similar because virtual NUMA nodes may share the same physical memory.

### 9.4 Performance and Scalability

The comparison program evaluates how execution time changes when the number of threads increases.

The results help identify:

- The performance cost of false sharing.
- The benefits of padding.
- The effect of NUMA memory placement.
- The relationship between thread count and execution time.

**Note:** Actual execution times depend on the hardware, number of CPU cores, virtual machine configuration and system load.

## 10. Screenshots


<img width="1280" height="800" alt="COHERENCE_SIM1" src="https://github.com/user-attachments/assets/1e877340-1baf-4c14-b97a-80a90d622d9f" />
<img width="1280" height="800" alt="COHERENCE_SIM2" src="https://github.com/user-attachments/assets/440b0fcb-b4de-45ae-9ff9-b13c433cecca" />
<img width="1280" height="800" alt="NUMA NODES" src="https://github.com/user-attachments/assets/4ba4e540-762a-4139-ae88-f75c5a4795ed" />
<img width="1280" height="800" alt="Ubuntu 64-bit (2)-2026-11-05-15-47-49" src="https://github.com/user-attachments/assets/15483f6e-5fd3-4862-b14c-07f61a0469f6" />
<img width="1280" height="800" alt="Ubuntu 64-bit (2)-2026-11-05-15-47-56" src="https://github.com/user-attachments/assets/40152bcd-3dec-4cf9-961e-e7e752d04464" />
<img width="1280" height="800" alt="Ubuntu 64-bit (2)-2026-11-05-15-48-57" src="https://github.com/user-attachments/assets/f8cf29c4-2ac1-4d95-900f-436bef020275" />
<img width="1280" height="800" alt="Ubuntu 64-bit (2)-2026-11-05-15-49-10" src="https://github.com/user-attachments/assets/efc518f4-7642-4b3d-a185-c21f061b46ca" />
<img width="1280" height="800" alt="Ubuntu 64-bit (2)-2026-11-05-15-49-36" src="https://github.com/user-attachments/assets/84dbfccb-950b-4602-ae2b-65f1e9c280fd" />


### 10.1 Cache Coherence Protocol Simulation

MSI, MESI and MOESI cache-state transitions and protocol statistics.

### 10.2 False Sharing Experiment

Execution comparison between adjacent and padded counter layouts.

### 10.3 NUMA Memory Sweep

Local and remote NUMA memory access measurements.

### 10.4 Combined Comparison Program

Execution-time, correctness and scalability comparison across different thread counts.

### 10.5 Additional NUMA Results

Additional terminal output for local and remote memory configurations, if captured separately.

> **Note:** Place your actual screenshots inside a folder named `screenshots` in the repository. Update the image filenames above to match your saved files. Remove any screenshot section for which you do not have an image.

## 11. Conclusion

This experiment demonstrates how cache coherence protocols maintain consistent data across multiple CPU cores and how their states affect memory transactions.

The MSI, MESI and MOESI simulation highlights the differences in coherence traffic. The false-sharing experiment demonstrates how placing independent variables on the same cache line can reduce parallel performance, while padding helps minimize this overhead.

The NUMA experiment shows that memory placement can influence access time, particularly on multi-socket systems.

Overall, the experiment highlights the importance of cache coherence, efficient memory layout and NUMA-aware memory placement in designing scalable parallel programs.

---

**Topics:** Cache Coherence \| MSI \| MESI \| MOESI \| False Sharing \| OpenMP \| NUMA \| Parallel Computing
