# Hybrid Deep Learning Based Resource Allocation and Dynamic Task Scheduling with Efficient Security Model for Cloud Computing

> Implementation of the RADTS-HDL framework combining MCSO-based task scheduling, Hybrid Deep Learning resource allocation, and EAES-based hypervisor attack detection for cloud environments.

---

## Table of Contents

- [Overview](#overview)
- [Research Paper](#research-paper)
- [System Architecture](#system-architecture)
- [Algorithms](#algorithms)
  - [MCSO — Task Scheduling](#mcso--task-scheduling)
  - [Hybrid Deep Learning — Resource Allocation](#hybrid-deep-learning--resource-allocation)
  - [EAES — Security & Hypervisor Attack Detection](#eaes--security--hypervisor-attack-detection)
- [Dataset](#dataset)
- [Experimental Setup & Hyperparameters](#experimental-setup--hyperparameters)
- [Results](#results)
- [References](#references)

---

## Overview

Cloud computing faces three core challenges: **efficient task scheduling**, **optimal resource allocation**, and **security against hypervisor attacks**. This project implements the **RADTS-HDL** (Resource Allocation and Dynamic Task Scheduling with Hybrid Deep Learning) framework, which integrates:

- **Modified Chicken Swarm Optimization (MCSO)** — for energy-aware dynamic task scheduling that minimizes execution time and maximizes throughput.
- **Hybrid Deep Learning (HDL) with Butterfly Optimization Algorithm (BOA)** — for intelligent resource allocation using CNN-based state-action modeling across bandwidth and resource load dimensions.
- **Enhanced Advanced Encryption Standard (EAES)** — for cloud data security and real-time hypervisor/VM attack detection using a double-key round-based encryption mechanism.

The proposed RADTS-HDL system outperforms existing methods (EDS, DVMC, and AEHO-EAES) across all key evaluation metrics.

---

## Research Paper

**Title:** A Hybrid Deep Learning Based Resource Allocation (RA) and Dynamic Task Scheduling (DTS) with Efficient Security Model for Cloud Computing (CC)

**Key Contributions:**
1. An efficient short-scheduler (MCSO-TS) using Modified Chicken Swarm Optimization that reduces task completion time and increases throughput.
2. Two Hybrid Deep Learning models for resource allocation that operate over bandwidth and resource load using different architectural constraints.
3. Butterfly Optimization Algorithm (BOA) integrated with Deep Learning for optimal VM selection within the state-action space framework.
4. EAES algorithm for hypervisor and VM attack detection and data security using double-key round-based AES.

---

## System Architecture

```
Cloud Users
    │
    ▼
Request Services ──► Virtualization Layer (VM Migration)
    │
    ▼
┌─────────────────────────────────────────────┐
│         MCSO-TS (Task Scheduling)           │
│  - Short Queue: tasks with DTi < Dm         │
│  - Long Queue:  tasks with DTi >= Dm        │
│  - Minimize completion time (Tc)            │
│  - Maximize VM Utilization (VMU)            │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│     HDL + BOA (Resource Allocation)         │
│  - State Space (SS): Idle VMs               │
│  - Action Space (AS): Allocated VMs         │
│  - CNN for resource state modeling          │
│  - BOA for optimal VM search                │
│  - Overload predictor per PM                │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│         EAES (Security Layer)               │
│  - Double-key AES encryption                │
│  - Round-based dynamic shift rows           │
│  - Hypervisor attack monitoring             │
│  - VM intrusion detection                   │
└─────────────────────────────────────────────┘
    │
    ▼
Response Services ──► Performance Evaluation
```

---

## Algorithms

### MCSO — Task Scheduling

Chicken Swarm Optimization (CSO) is a bio-inspired algorithm that mimics the hierarchical social behavior of chicken flocks. The **Modified** variant (MCSO) incorporates an Adaptive Benefit Factor (ABF) to improve local exploitation.

#### Task Queue Formation

Tasks are partitioned into short and long queues based on their deadline relative to the median deadline value Dm:

```
If DTi < Dm  →  assign to Short Queue
If DTi >= Dm →  assign to Long Queue
```

The objective is to minimize total completion time Tc and maximize VM utilization (VMU).

#### Rooster Movement

Roosters with higher fitness values explore a wider search space:

```
x_i,j^(t+1) = x_i,j^t × (1 + Randn(0, σ²))

σ² = 1,                              if fi ≤ fk
σ² = exp((fk - fi) / (|fi| + ε)),   otherwise
```

#### Hen Movement

Hens follow their group rooster while competing to steal food discovered by others:

```
x_i,j^(t+1) = x_i,j^t + S1 × Rand × (x_r1,j^t - x_i,j^t)
                        + S2 × Rand × (x_r2,j^t - x_i,j^t)

S1 = exp((fi - fr1) / (|fi| + ε))
S2 = exp(fr2 - fi)
```

#### Chick Movement

Chicks forage around their mother hen:

```
x_i,j^(t+1) = x_i,j^t + FL × (x_m,j^t - x_i,j^t)

FL ∈ [0, 2]
```

#### Adaptive Benefit Factor (ABF)

The ABF updates each new candidate solution through a mutualistic interaction between two species Xi and Xj:

```
X_i_new = Xi + rand(0,1) × (Xbest - MutualVector × BF1)
X_j_new = Xj + rand(0,1) × (Xbest - MutualVector × BF2)

MutualVector = (Xi + Xj) / 2
```

BF1 and BF2 are benefit factors randomly set to 1 or 2, representing the degree of advantage for each organism.

---

### Hybrid Deep Learning — Resource Allocation

#### State Space and Action Space

Resources are divided into two categories:

- **State Space (SS):** Idle VMs not yet assigned to any task. These are candidates for task scheduling.
- **Action Space (AS):** VMs currently allocated to tasks (fully or partially).

VM utilization is computed as:

```
VMU = U_VMi_CPU + U_VMi_Memory + U_VMi_I/O
```

VM transitions between spaces follow:

```
VM_Busy → Action Space (AS)
VM_Idle → State Space (SS)
```

#### System State Representation

The state of the DL-based cloud allocation layer for task T at time t is:

```
S_t^T = [SC_t^T, S^T] = [g1_t^T, ..., gK_t^T, S^T]
       = [U_11_t^T, ..., U_1D_t^T, ..., U_MD_t^T, U_T1, ..., U_TD, d^T]
```

where d^T is the estimated task length and K server groups partition the cluster.

The action space for a cluster with N servers is:

```
A = {a | a ∈ {1, 2, ..., N}}
```

#### Butterfly Optimization Algorithm (BOA)

BOA is used to search for the optimal VM within the state space. It models butterfly foraging behavior in two stages:

**Global Search:**
```
x_i^(t+1) = x_i^t + (r² × g* - x_i^t) × fi
```

**Local Search:**
```
x_i^(t+1) = x_i^t + (r² × x_j^t - x_k^t) × fi
```

where `g*` is the current best solution, `fi` is the fragrance of butterfly i, and `r` is a random number in [0, 1]. A switch probability `p` controls the transition between global and local search phases.

#### Fitness Function

The combined fitness for resource allocation balances cost, makespan, and reliability:

```
Fitness = α × cost + β × makespan + γ × reliability

where α, β, γ ∈ [0, 1]
```

---

### EAES — Security & Hypervisor Attack Detection

The Enhanced AES builds on standard AES (128-bit block, variable key: 128/192/256-bit) with a **round-based dynamic shift operation** and **double-key encryption** for stronger cloud security.

#### Key Generation

Input data and encryption key are arranged as 4×4 byte matrices:

```
Key = [k0  k4  k8  k12]
      [k1  k5  k9  k13]
      [k2  k6  k10 k14]
      [k3  k7  k11 k15]
```

#### Round-Based Shift Operation

Unlike standard AES which always shifts rows by fixed positions, EAES dynamically varies the shift based on the current round number:

- **Odd round (e.g., Round 1):** Shift rows by 1 position.
- **Even round (e.g., Round 2):** Shift rows by 2 positions.

#### Encryption Steps per Round

1. **Sub-bytes** — Nonlinear byte substitution using the reversible S-Box.
2. **Round-based Shift Rows** — Dynamic row shifting based on round parity.
3. **Mix Columns** — Column-wise GF(2⁸) matrix multiplication.
4. **Add Round Key** — Bitwise XOR with round key derived from the key schedule.

#### Hypervisor Attack Detection

- Monitors all data exchanges between VMs and the hypervisor continuously.
- Detects anomalies or unauthorized exchanges using cloud request patterns.
- Identifies both external (hypervisor hijacking) and internal (rogue VM) attacks.
- Automatically blocks and mitigates detected threats in real time.

---

## Dataset

The dataset used in this project is **synthetically generated** to simulate realistic cloud data center workloads.

### Dataset Parameters

| Parameter | Description |
|---|---|
| Cloud Users | Users sending task requests to the data center |
| Tasks (T) | Each task defined by CPU, memory, bandwidth, and storage requirements |
| Virtual Machines (VM) | Characterized by processing power, memory, network bandwidth, and storage |
| Physical Hosts (H) | Defined by CPU capacity and memory volume |
| Energy Model | Static + dynamic power; idle host consumes ~70% of peak host power |

### Task Attributes

Each task `T_k` is defined as:
```
T_k = (T_k^p, T_k^m, T_k^n, T_k^s)
```
- `T_k^p` — Processor power required
- `T_k^m` — Main memory required
- `T_k^n` — Network bandwidth required
- `T_k^s` — Storage required

Two task types exist: compute-intensive (high CPU, low memory) and memory-intensive (low CPU, high memory).

### VM Attributes

Each VM `V_j` is characterized as:
```
V_j = (V_j^m, V_j^p, V_j^n, V_j^s)
```

---

## Experimental Setup & Hyperparameters

### Simulation Environment

| Setting | Value |
|---|---|
| Simulator | CloudSim |
| Evaluation Focus | QoS, energy efficiency, security |
| Comparison Baselines | EDS, DVMC, AEHO-EAES |

### MCSO Hyperparameters

| Parameter | Description | Value/Range |
|---|---|---|
| Population Size (N) | Total number of chickens (solutions) | Configurable |
| Rooster Count (RN) | Best RN chickens assigned as roosters | Subset of N |
| Hen Count (HN) | Middle-fitness chickens | Subset of N |
| Chick Count (CN) | Worst CN chickens assigned as chicks | Subset of N |
| Group Update Interval (G) | Time steps between hierarchy updates | Fixed interval |
| FL (Chick Follow Factor) | Controls how closely chicks follow mother | [0, 2] |
| ε (Zero-division constant) | Minimum constant to prevent divide-by-zero | System minimum |
| BF1, BF2 (Benefit Factors) | ABF interaction parameters | 1 or 2 (random) |

### Deep Learning (BOA + CNN) Hyperparameters

| Parameter | Description | Value/Range |
|---|---|---|
| State Space Dimension (D) | Resource type dimensions per VM | CPU + Memory + I/O |
| Server Groups (K) | Number of server cluster partitions | Configurable |
| Switch Probability (p) | BOA transition from global to local search | [0, 1] |
| BOA r (random factor) | Random number for butterfly movement | [0, 1] |
| α (Cost weight) | Fitness function cost parameter | [0, 1] |
| β (Makespan weight) | Fitness function makespan parameter | [0, 1] |
| γ (Reliability weight) | Fitness function reliability parameter | [0, 1] |

### AES/EAES Parameters

| Parameter | Value |
|---|---|
| Block Size | 128 bits (4×4 byte matrix) |
| Key Size Options | 128 / 192 / 256 bits |
| Rounds (128-bit key) | 10 |
| Rounds (192-bit key) | 12 |
| Rounds (256-bit key) | 14 |
| Shift Mode | Round-based dynamic (odd/even round parity) |
| Key Structure | Double-key orientation |

---

## Results

The proposed RADTS-HDL method was evaluated against EDS, DVMC, and AEHO-EAES across five metrics:

| Method | Time Complexity (sec) | Cost Complexity ($/GB) | Throughput (Mbps) | Energy Consumption (J) | Reliability (%) |
|---|---|---|---|---|---|
| EDS | 56 | 0.130 | 0.79 | 36 | 81.70 |
| DVMC | 42 | 0.097 | 0.85 | 25 | 86.30 |
| AEHO-EAES | 28 | 0.058 | 0.93 | 18 | 92.50 |
| **RADTS-HDL** | **20** | **0.037** | **0.96** | **15** | **95.68** |

### Key Findings

- **64.3% reduction in time complexity** compared to EDS.
- **71.5% reduction in cost complexity** compared to EDS.
- **21.5% higher throughput** compared to EDS.
- **58.3% reduction in energy consumption** compared to EDS.
- **13.98 percentage point improvement in reliability** compared to EDS.
- Consistently outperforms AEHO-EAES (the prior proposed method) across all five metrics.

---

## References

1. Venters, W., & Whitley, E. A. (2012). A critical review of cloud computing. *Journal of Information Technology*, 27(3).
2. Durao, F., et al. (2014). A systematic review on cloud computing. *The Journal of Supercomputing*, 68(3).
3. Chang, V., Wills, G., & De Roure, D. (2010). A review of cloud business models and sustainability. *IEEE Cloud Computing*.
4. Gai, K., & Li, S. (2012). Towards cloud computing: a literature review. *MINES*.
5. El-Gazzar, R. F. (2014). A literature review on cloud computing adoption. *IFIP WITD*.
6. Dudin, E. B., & Smetanin, Y. G. (2011). A review of cloud computing. *Scientific and Technical Information Processing*, 38(4).
7. Al-Samarraie, H., & Saeed, N. (2018). A systematic review of cloud computing tools for collaborative learning. *Computers & Education*, 124.
8. Ma, S. (2012). A review on cloud computing development. *Journal of Networks*, 7(2).
9. Arutyunov, V. V. (2012). Cloud computing: history, modern state, and future. *Scientific and Technical Information Processing*, 39(3).
10. Muthunagai, S. U., et al. (2012). Efficient access of cloud resources through virtualization. *ICRTIT*.
11. El Fazziki, A., et al. (2012). A service oriented information system: a model driven approach. *SITIS*.
12. Fang, Y., et al. (2010). A task scheduling algorithm based on load balancing in cloud computing. *WISM*. Springer.
13. Parsa, S., & Entezari-Maleki, R. (2009). RASA: a new grid task scheduling algorithm. *IJDCTA*, 3(4).
14. Selvarani, S., & Sadhasivam, G. S. (2010). Improved cost-based algorithm for task scheduling in cloud computing. *ICCIC*. IEEE.
15. Stankovic, J. A., et al. (1985). Evaluation of a flexible task scheduling algorithm for distributed hard real-time systems. *IEEE Transactions on Computers*, 100(12).
16. Wu, X., et al. (2013). A task scheduling algorithm based on QoS-driven in cloud computing. *Procedia Computer Science*, 17.
17. Li, J. F., & Peng, J. (2011). Task scheduling algorithm based on improved genetic algorithm. *Journal of Computer Applications*, 31(1).
18. Agarwal, D., & Jain, S. (2014). Efficient optimal algorithm of task scheduling in cloud computing. *arXiv:1404.2076*.
19. Melendez, J. O., et al. (2013). A framework for automatic resource provisioning for private clouds. *CCGrid*. IEEE/ACM.
20. Ergu, D., et al. (2013). The analytic hierarchy process: task scheduling and resource allocation in cloud. *The Journal of Supercomputing*, 64(3).
21. Tsai, J. T., et al. (2013). Optimized task scheduling and resource allocation using improved differential evolution. *Computers & Operations Research*, 40(12).
22. Pawar, C. S., & Wagh, R. B. (2012). Priority based dynamic resource allocation in cloud computing. *ISCOS*. IEEE.
23. Annadanam, C. S., et al. (2020). Intermediate node selection for Scatter-Gather VM migration. *ESTIJ*, 23(5).
24. Beloglazov, A., et al. (2012). Energy-aware resource allocation heuristics for cloud data centers. *Future Generation Computer Systems*, 28(5).
25. Meng, X., et al. (2014). A new bio-inspired algorithm: chicken swarm optimization. *ICSI*. Springer.
26. Arora, S., & Singh, S. (2019). Butterfly optimization algorithm: a novel approach for global optimization. *Soft Computing*, 23.
27. Pendli, V., et al. (2016). Improvising performance of Advanced Encryption Standard algorithm. *MobiSecServ*. IEEE.
28. Szefer, J., et al. (2011). Eliminating the hypervisor attack surface for a more secure cloud. *ACM CCS*.
29. Dildar, M. S., et al. (2017). Effective way to defend hypervisor attacks in cloud computing. *ICACC*. IEEE.
30. Elshabka, M. A., et al. (2020). Security-aware dynamic VM consolidation. *Egyptian Informatics Journal*.


