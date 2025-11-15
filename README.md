# Hospital-Bed-Simulation
Discrete-Event Simulation of Patient Flow in Hospitals  
**DTU 02443 – Stochastic Simulation**  
**Group 15**

This repository contains an event-driven simulation model of inpatient flow in a hospital with limited bed capacity.  
The project evaluates admission performance, relocation dynamics, and patient losses under the baseline configuration and under an epidemic scenario requiring the creation of a new ward (F*).

The work includes:
- A full discrete-event simulation (DES) model  
- Erlang B–based sizing of an epidemic ward  
- A simulated annealing metaheuristic for optimizing bed reallocations  
- Multi-horizon performance evaluation with confidence intervals  
- Sensitivity analysis on LOS variability, bed distribution, and total capacity  

---

# 1. Project Context

**Course:** 02443 – Stochastic Simulation  
**Institution:** Technical University of Denmark  
**Academic Period:** 2025  
**Deliverables:** Simulation code + optimization + analysis + report  

Hospital inpatient wards behave as *Erlang loss systems*: when no beds are available, patients must be either  
relocated to an alternative ward or lost entirely.

The simulation models:
- Five baseline wards (A–E)  
- Arrival processes (Poisson)  
- Length of stay (exponential or log-normal)  
- Ward-specific relocation probabilities  
- Loss events when all alternatives are full  

An epidemic introduces a new patient type and a new ward **F\***. Because resources are fixed, beds must be reallocated from existing wards, requiring an optimization step.

# 2. Methods and Techniques

### **Simulation**
- Discrete-event simulation based on chronological event processing  
- Events: arrivals and departures  
- Implementation using Python `heapq` (priority queue)  

### **Stochastic models**
- Inter-arrival times: Exponential with rate λₖ  
- Length of stay: Exponential (baseline) or Log-Normal (sensitivity)  
- Blocking probability estimation  
- Erlang B for analytical sizing of ward F\*  

### **Optimization**
- **Simulated annealing** for reallocation of beds  
- Objective function combining:
  - patient losses  
  - relocations  
  - urgency points (ethical fairness)  

### **Performance evaluation**
- Expected admissions  
- Expected relocations  
- Expected losses  
- Probability of full capacity on arrival  
- 95% confidence intervals based on 100 independent replications  

### **Sensitivity analysis**
1. Length-of-stay (LOS) variance  
2. Redistribution of beds (D→C, D→B, equalized layout)  
3. Global total bed count (150–180 beds)  

---

# 3. Model Overview

## 3.1 Baseline System
Five wards (A–E) with:
- Bed capacity Mᵢ  
- Arrival rates λᵢ  
- Mean LOS 1/µᵢ  
- Relocation matrix {pᵢⱼ}  

### Key outputs:
- Ward C is the chronic bottleneck (highest blocking probability)  
- Wards B and E also saturate at long horizons  
- Ward D remains systematically underutilized  

---

# 4. Epidemic Scenario and Ward F\*

A sixth ward **F\*** is introduced with:
- Arrival rate: 13 patients/day  
- LOS: 2.2 days  
- No back-relocation from other wards  
- Must admit ≥ 95% of incoming F\* patients  

### 4.1 Sizing of Ward F\*
Using Erlang B:

- Traffic intensity: A = λ × mean LOS = 28.6  
- Required number of beds such that B(M, A) ≤ 0.05  
- Result: **M\* = 34 beds**

### 4.2 Bed Redistribution via Simulated Annealing
Objective:
- Minimize urgency-weighted losses + relocations  
- Subject to: sum of beds = fixed total  

Optimal solution:
A: 44
B: 33
C: 24
D: 15
E: 15
F*: 34

This configuration ensures:
- Ward F\* blocking ≈ 4–5%  
- High-urgency wards (A and D) protected  
- Low-urgency wards absorb more relocations  

---

# 5. Sensitivity Analysis

## 5.1 LOS Variance Increase  
When switching LOS distribution from exponential to log-normal and increasing σ²:

- Blocking rises nonlinearly, especially in B–C–E  
- Losses ↑ by ~18% at k = 4  
- A and D remain insensitive due to low utilization  

## 5.2 Bed Redistribution  
Scenarios:

- **D→C (+5 beds)** → Best improvement (−12% losses)  
- **D→B (+5 beds)** → Moderate improvement (−10% losses)  
- **Equalized layout** → Dramatic worsening (+75% losses)  

## 5.3 Total Bed Count  
Global capacity varied {150, 165, 170, 180}:

- Increasing from 150 → 165 reduces losses by 45%  
- Gains flatten beyond ~175 beds  
- Major congestion always located in Ward C  
