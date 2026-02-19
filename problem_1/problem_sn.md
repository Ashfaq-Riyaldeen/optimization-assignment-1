# Problem 1 — HPC Cloud Server Resource Allocation (Knapsack Problem)

---

## Scenario

**LankaCloud Pvt Ltd** is a cloud computing provider based in Colombo, Sri Lanka. The company operates a
High-Performance Computing (HPC) server that can host managed application instances for its clients.
The server has a limited compute capacity shared among all hosted applications.

LankaCloud offers three types of managed application hosting packages:

| k | Application Type        | vCPU per instance | Monthly Profit per instance |
|---|-------------------------|-------------------|-----------------------------|
| 1 | Web Application Server  | 4 vCPU            | Rs. 9,000                   |
| 2 | Mobile API Server       | 6 vCPU            | Rs. 13,000                  |
| 3 | ML Inference Server     | 5 vCPU            | Rs. 10,000                  |

The HPC server has a total capacity of **12 vCPU units**. Multiple instances of the same application
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
| Web Application Server | 8 cores   | 16 GB | 80 GB   | **4 vCPU** |
| Mobile API Server      | 12 cores  | 24 GB | 120 GB  | **6 vCPU** |
| ML Inference Server    | 10 cores  | 20 GB | 100 GB  | **5 vCPU** |

Total server capacity: 24 CPU cores / 48 GB RAM / 240 GB storage = **12 vCPU units**.

---

## Mathematical Formulation

**Parameters:**

- N = 3 (number of application types)
- C = 12 (total server capacity in vCPU units)
- w₁ = 4, w₂ = 6, w₃ = 5 (vCPU consumed per instance)
- r₁ = 9,000, r₂ = 13,000, r₃ = 10,000 (monthly profit in Rs. per instance)

**Decision Variables:**

- xₖ = number of instances of application type k to host (non-negative integer), k = 1, 2, 3

**Objective Function:**

> Maximise Z = 9,000 x₁ + 13,000 x₂ + 10,000 x₃

**Capacity Constraint:**

> 4x₁ + 6x₂ + 5x₃ ≤ 12

**Integrality Constraint:**

> x₁, x₂, x₃ ∈ {0, 1, 2, ...}

Since xₖ can take any non-negative integer value (each application type can be hosted multiple times),
this is an **unbounded integer programme** — not solvable by the Simplex method. We solve it using
**Dynamic Programming**.

---

## Dynamic Programming Formulation

**Stages:** Application types k = 1, 2, 3 (processed from k = 3 down to k = 1)

**State:** Remaining server capacity i, where 0 ≤ i ≤ 12

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

## Stage 3 — ML Inference Server (w₃ = 5, r₃ = Rs. 10,000)

V₃(i) = max { 10,000 · x₃ : x₃ ≥ 0, 5x₃ ≤ i } = 10,000 × ⌊i/5⌋

| Remaining Capacity (i) | Optimal x₃* | V₃(i) (Rs.) |
|------------------------|-------------|-------------|
| 0                      | 0           | 0           |
| 1                      | 0           | 0           |
| 2                      | 0           | 0           |
| 3                      | 0           | 0           |
| 4                      | 0           | 0           |
| 5                      | 1           | 10,000      |
| 6                      | 1           | 10,000      |
| 7                      | 1           | 10,000      |
| 8                      | 1           | 10,000      |
| 9                      | 1           | 10,000      |
| 10                     | 2           | 20,000      |
| 11                     | 2           | 20,000      |
| 12                     | 2           | 20,000      |

---

## Stage 2 — Mobile API Server + ML Inference (w₂ = 6, r₂ = Rs. 13,000)

V₂(i) = max { 13,000 · x₂ + V₃(i − 6x₂) : x₂ ≥ 0, 6x₂ ≤ i }

| i  | x₂=0: V₃(i) | x₂=1: 13k+V₃(i−6) | x₂=2: 26k+V₃(i−12) | Optimal x₂* | V₂(i) (Rs.) |
|----|-------------|-------------------|---------------------|-------------|-------------|
| 0  | 0           | —                 | —                   | 0           | 0           |
| 1  | 0           | —                 | —                   | 0           | 0           |
| 2  | 0           | —                 | —                   | 0           | 0           |
| 3  | 0           | —                 | —                   | 0           | 0           |
| 4  | 0           | —                 | —                   | 0           | 0           |
| 5  | 10,000      | —                 | —                   | 0           | 10,000      |
| 6  | 10,000      | 13,000            | —                   | 1           | 13,000      |
| 7  | 10,000      | 13,000            | —                   | 1           | 13,000      |
| 8  | 10,000      | 13,000            | —                   | 1           | 13,000      |
| 9  | 10,000      | 13,000            | —                   | 1           | 13,000      |
| 10 | 20,000      | 13,000            | —                   | 0           | 20,000      |
| 11 | 20,000      | **23,000**        | —                   | 1           | 23,000      |
| 12 | 20,000      | 23,000            | **26,000**          | 2           | 26,000      |

> Key observations:
> - At i = 6–9: one Mobile API (Rs. 13,000) beats one ML Inference (Rs. 10,000).
> - At i = 10: two ML Inference (Rs. 20,000) beats one Mobile API (Rs. 13,000).
> - At i = 11: one Mobile API + one ML Inference (13,000 + 10,000 = Rs. 23,000) is best.
> - At i = 12: two Mobile API (26,000) beats all other combinations.

---

## Stage 1 — All Application Types (Capacity = 12 vCPU)

V₁(12) = max { 9,000 · x₁ + V₂(12 − 4x₁) : x₁ ≥ 0, 4x₁ ≤ 12 }

| x₁ | 9,000 × x₁ (Rs.) | Remaining capacity (12 − 4x₁) | V₂(12 − 4x₁) (Rs.) | Total (Rs.)    |
|----|------------------|-------------------------------|---------------------|----------------|
| 0  | 0                | 12                            | 26,000              | 26,000         |
| 1  | 9,000            | 8                             | 13,000              | 22,000         |
| 2  | 18,000           | 4                             | 0                   | 18,000         |
| **3**  | **27,000**   | **0**                         | **0**               | **27,000** ← max |

**Optimal decision at Stage 1:** x₁* = 3

---

## Traceback — Recovering the Optimal Solution

| Step | Stage | Remaining capacity | Optimal xₖ* | vCPU used | Profit earned           |
|------|-------|--------------------|-------------|-----------|-------------------------|
| 1    | k = 1 | 12                 | x₁* = **3** | 3 × 4 = 12| 3 × 9,000 = Rs. 27,000  |
| 2    | k = 2 | 12 − 12 = 0        | x₂* = **0** | 0         | 0                        |
| 3    | k = 3 | 0                  | x₃* = **0** | 0         | 0                        |

**Total vCPU used:** 12 / 12 (server fully utilised)

---

## Optimal Solution

> **Host 3 Web Application Server instances.**
>
> **Maximum monthly profit = 3 × Rs. 9,000 = Rs. 27,000**

---

## Greedy Comparison

A greedy approach ranks applications by profit-per-vCPU (value-to-weight ratio):

| Application Type    | Profit / vCPU                | Greedy rank |
|---------------------|------------------------------|-------------|
| Web Application     | 9,000 / 4 = **2,250 Rs/vCPU** | 1st        |
| Mobile API Server   | 13,000 / 6 ≈ **2,167 Rs/vCPU** | 2nd       |
| ML Inference Server | 10,000 / 5 = **2,000 Rs/vCPU** | 3rd       |

**Greedy execution (capacity = 12 vCPU):**

1. Pick Web Application (4 vCPU) → remaining: 8 vCPU, profit: Rs. 9,000
2. Pick Web Application (4 vCPU) → remaining: 4 vCPU, profit: Rs. 18,000
3. Pick Web Application (4 vCPU) → remaining: 0 vCPU, profit: Rs. 27,000

**Greedy result:** x₁ = 3, x₂ = 0, x₃ = 0 → **Rs. 27,000**

In this instance, greedy and DP arrive at the same solution because the highest-ratio item (Web
Application, w₁ = 4) divides the capacity (C = 12) exactly, leaving no wasted capacity. However,
the greedy heuristic provides **no guarantee of optimality** in general — the Dynamic Programming
approach guarantees the globally optimal solution for any input.

| Method  | x₁ | x₂ | x₃ | vCPU used | Monthly Profit |
|---------|----|----|----|-----------|----------------|
| Greedy  | 3  | 0  | 0  | 12 / 12   | Rs. 27,000     |
| **DP**  | **3** | **0** | **0** | **12 / 12** | **Rs. 27,000** |
