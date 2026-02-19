# Problem 1 — HPC Cloud Server Resource Allocation (Knapsack Problem)

---

## Scenario

**LankaCloud Pvt Ltd** is a cloud computing provider based in Colombo, Sri Lanka. The company operates a
High-Performance Computing (HPC) server that can host managed application instances for its clients.
The server has a limited compute capacity shared among all hosted applications.

LankaCloud offers three types of managed application hosting packages:

| k | Application Type        | vCPU per instance | Monthly Profit per instance |
|---|-------------------------|-------------------|-----------------------------|
| 1 | Web Application Server  | 3 vCPU            | Rs. 7,000                   |
| 2 | Mobile API Server       | 5 vCPU            | Rs. 12,000                  |
| 3 | ML Inference Server     | 9 vCPU            | Rs. 25,000                  |

The HPC server has a total capacity of **20 vCPU units**. Multiple instances of the same application
type may be hosted simultaneously. LankaCloud wants to determine how many instances of each application
type to host in order to **maximise total monthly profit**, subject to the server's capacity limit.

---

## Resource Abstraction — vCPU Unit

Real HPC servers have multiple resource dimensions: CPU cores, RAM, and storage. To apply the standard
single-constraint Knapsack formulation, all resources are unified into a single **virtual CPU (vCPU)**
metric. Each vCPU unit represents a fixed bundle of:

- 2 physical CPU cores
- 4 GB RAM
- 20 GB SSD storage

Application resource requirements expressed in vCPU units:

| Application Type       | CPU cores | RAM   | Storage | vCPU units |
|------------------------|-----------|-------|---------|------------|
| Web Application Server | 6 cores   | 8 GB  | 40 GB   | **3 vCPU** |
| Mobile API Server      | 10 cores  | 20 GB | 100 GB  | **5 vCPU** |
| ML Inference Server    | 18 cores  | 36 GB | 180 GB  | **9 vCPU** |

Total server capacity: 40 CPU cores / 80 GB RAM / 400 GB storage = **20 vCPU units**.

---

## Mathematical Formulation

**Parameters:**

- N = 3 (number of application types)
- C = 20 (total server capacity in vCPU units)
- w₁ = 3, w₂ = 5, w₃ = 9 (vCPU consumed per instance)
- r₁ = 7,000, r₂ = 12,000, r₃ = 25,000 (monthly profit in Rs. per instance)

**Decision Variables:**

- xₖ = number of instances of application type k to host (non-negative integer), k = 1, 2, 3

**Objective Function:**

> Maximise Z = 7,000 x₁ + 12,000 x₂ + 25,000 x₃

**Capacity Constraint:**

> 3x₁ + 5x₂ + 9x₃ ≤ 20

**Integrality Constraint:**

> x₁, x₂, x₃ ∈ {0, 1, 2, ...}

Since xₖ can take any non-negative integer value (each application type can be hosted multiple times),
this is an **unbounded integer programme** — not solvable by the Simplex method. We solve it using
**Dynamic Programming**.

---

## Dynamic Programming Formulation

**Stages:** Application types k = 1, 2, 3 (processed from k = 3 down to k = 1)

**State:** Remaining server capacity i, where 0 ≤ i ≤ 20

**Optimal Value Function:**

> Vₖ(i) = maximum monthly profit achievable from application types k, k+1, ..., 3
>          with remaining capacity i

**Boundary Condition:**

> V₄(i) = 0  for all i

**Recurrence Relation:**

```
Vₖ(i) = max { rₖ · xₖ + Vₖ₊₁(i − wₖ · xₖ) }
         xₖ ≥ 0,  wₖ · xₖ ≤ i
```

---

## Stage 3 — ML Inference Server (w₃ = 9, r₃ = Rs. 25,000)

V₃(i) = max { 25,000 · x₃ : x₃ ≥ 0, 9x₃ ≤ i }

| Remaining Capacity (i) | Optimal x₃* | V₃(i) (Rs.) |
|------------------------|-------------|-------------|
| 0                      | 0           | 0           |
| 1                      | 0           | 0           |
| 2                      | 0           | 0           |
| 3                      | 0           | 0           |
| 4                      | 0           | 0           |
| 5                      | 0           | 0           |
| 6                      | 0           | 0           |
| 7                      | 0           | 0           |
| 8                      | 0           | 0           |
| 9                      | 1           | 25,000      |
| 10                     | 1           | 25,000      |
| 11                     | 1           | 25,000      |
| 12                     | 1           | 25,000      |
| 13                     | 1           | 25,000      |
| 14                     | 1           | 25,000      |
| 15                     | 1           | 25,000      |
| 16                     | 1           | 25,000      |
| 17                     | 1           | 25,000      |
| 18                     | 2           | 50,000      |
| 19                     | 2           | 50,000      |
| 20                     | 2           | 50,000      |

---

## Stage 2 — Mobile API Server + ML Inference (w₂ = 5, r₂ = Rs. 12,000)

V₂(i) = max { 12,000 · x₂ + V₃(i − 5x₂) : x₂ ≥ 0, 5x₂ ≤ i }

| i  | x₂=0: V₃(i) | x₂=1: 12k+V₃(i−5) | x₂=2: 24k+V₃(i−10) | x₂=3: 36k+V₃(i−15) | x₂=4: 48k+V₃(i−20) | Optimal x₂* | V₂(i) (Rs.) |
|----|-------------|-------------------|--------------------|--------------------|---------------------|-------------|-------------|
| 0  | 0           | —                 | —                  | —                  | —                   | 0           | 0           |
| 1  | 0           | —                 | —                  | —                  | —                   | 0           | 0           |
| 2  | 0           | —                 | —                  | —                  | —                   | 0           | 0           |
| 3  | 0           | —                 | —                  | —                  | —                   | 0           | 0           |
| 4  | 0           | —                 | —                  | —                  | —                   | 0           | 0           |
| 5  | 0           | 12,000            | —                  | —                  | —                   | 1           | 12,000      |
| 6  | 0           | 12,000            | —                  | —                  | —                   | 1           | 12,000      |
| 7  | 0           | 12,000            | —                  | —                  | —                   | 1           | 12,000      |
| 8  | 0           | 12,000            | —                  | —                  | —                   | 1           | 12,000      |
| 9  | 25,000      | 12,000            | —                  | —                  | —                   | 0           | 25,000      |
| 10 | 25,000      | 12,000            | 24,000             | —                  | —                   | 0           | 25,000      |
| 11 | 25,000      | 12,000            | 24,000             | —                  | —                   | 0           | 25,000      |
| 12 | 25,000      | 12,000            | 24,000             | —                  | —                   | 0           | 25,000      |
| 13 | 25,000      | 12,000            | 24,000             | —                  | —                   | 0           | 25,000      |
| 14 | 25,000      | **37,000**        | 24,000             | —                  | —                   | 1           | 37,000      |
| 15 | 25,000      | **37,000**        | 24,000             | 36,000             | —                   | 1           | 37,000      |
| 16 | 25,000      | **37,000**        | 24,000             | 36,000             | —                   | 1           | 37,000      |
| 17 | 25,000      | **37,000**        | 24,000             | 36,000             | —                   | 1           | 37,000      |
| 18 | **50,000**  | 37,000            | 24,000             | 36,000             | —                   | 0           | 50,000      |
| 19 | **50,000**  | 37,000            | 49,000             | 36,000             | —                   | 0           | 50,000      |
| 20 | **50,000**  | 37,000            | 49,000             | 36,000             | 48,000              | 0           | 50,000      |

> Note: At i = 14, x₂ = 1 gives 12,000 + V₃(9) = 12,000 + 25,000 = **Rs. 37,000**, which beats
> the ML-only option of V₃(14) = 25,000. At i = 18, two ML instances (V₃(18) = 50,000) beat any
> Mobile API combination.

---

## Stage 1 — All Application Types (Capacity = 20 vCPU)

V₁(20) = max { 7,000 · x₁ + V₂(20 − 3x₁) : x₁ ≥ 0, 3x₁ ≤ 20 }

| x₁ | 7,000 × x₁ (Rs.) | Remaining capacity (20 − 3x₁) | V₂(20 − 3x₁) (Rs.) | Total (Rs.)    |
|----|------------------|-------------------------------|---------------------|----------------|
| 0  | 0                | 20                            | 50,000              | 50,000         |
| 1  | 7,000            | 17                            | 37,000              | 44,000         |
| **2**  | **14,000**   | **14**                        | **37,000**          | **51,000** ← max |
| 3  | 21,000           | 11                            | 25,000              | 46,000         |
| 4  | 28,000           | 8                             | 12,000              | 40,000         |
| 5  | 35,000           | 5                             | 12,000              | 47,000         |
| 6  | 42,000           | 2                             | 0                   | 42,000         |

**Optimal decision at Stage 1:** x₁* = 2

---

## Traceback — Recovering the Optimal Solution

| Step | Stage | Remaining capacity | Optimal xₖ* | vCPU used | Profit earned    |
|------|-------|--------------------|-------------|-----------|------------------|
| 1    | k = 1 | 20                 | x₁* = **2** | 2 × 3 = 6 | 2 × 7,000 = Rs. 14,000 |
| 2    | k = 2 | 20 − 6 = 14        | x₂* = **1** | 1 × 5 = 5 | 1 × 12,000 = Rs. 12,000 |
| 3    | k = 3 | 14 − 5 = 9         | x₃* = **1** | 1 × 9 = 9 | 1 × 25,000 = Rs. 25,000 |

**Total vCPU used:** 6 + 5 + 9 = **20 / 20** (server fully utilised)

---

## Optimal Solution

> **Host 2 Web Application Servers, 1 Mobile API Server, and 1 ML Inference Server.**
>
> **Maximum monthly profit = Rs. 14,000 + Rs. 12,000 + Rs. 25,000 = Rs. 51,000**

---

## Why Greedy Fails

A greedy approach ranks applications by profit-per-vCPU (value-to-weight ratio):

| Application Type    | Profit/vCPU          | Greedy rank |
|---------------------|----------------------|-------------|
| ML Inference Server | 25,000 / 9 ≈ **2,778** | 1st       |
| Mobile API Server   | 12,000 / 5 = **2,400** | 2nd       |
| Web Application     | 7,000  / 3 ≈ **2,333** | 3rd       |

**Greedy execution (capacity = 20 vCPU):**

1. Pick ML Inference (9 vCPU) → capacity remaining: 11 vCPU, profit: Rs. 25,000
2. Pick ML Inference again (9 vCPU) → capacity remaining: 2 vCPU, profit: Rs. 50,000
3. No application fits in 2 vCPU → **stop**

**Greedy result:** x₁ = 0, x₂ = 0, x₃ = 2 → **Rs. 50,000** (2 vCPU wasted)

| Method  | x₁ | x₂ | x₃ | vCPU used | Monthly Profit |
|---------|----|----|----|-----------|----------------|
| Greedy  | 0  | 0  | 2  | 18 / 20   | Rs. 50,000     |
| **DP**  | **2** | **1** | **1** | **20 / 20** | **Rs. 51,000** |

The greedy heuristic leaves 2 vCPU idle and misses **Rs. 1,000** in monthly profit. Dynamic
Programming finds the true global optimum by exhaustively evaluating all feasible combinations
through backward recursion.
