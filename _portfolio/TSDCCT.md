---
layout: single
title:  "Talent Scheduling with Daily Capacity and Changeover Times"
subtitle: "A Pattern Based Integer Linear Programming Approach"
categories: 
  - Jekyll
permalink: /portfolio/talent-scheduling/ 
tags:
  - talent scheduling
  - integer linear programming
  - bin packing
  - travelling salesman problem
  - Gurobi
  - Python
excerpt: "A project done through the Directed Reading Program at the University of Georgia that replicates a pattern based integer linear programming model for the talent scheduling problem."
---
# Overview

In this project, we investigated the talent scheduling problem with consideration of daily capacity and changeover times (TS-DCCT) as proposed by [Zhaoqi Yang, Shunji Tanaka, and Bertrand M.T. Lin](https://doi.org/10.1016/j.omega.2025.103439) and recreated their integer linear program with feasible patterns. We implemented this model in Python interfaced with Gurobi Optimizer like Yang, et al. in order to compare computational results.

This project was done through a Directed Reading Program in the Mathematics Department at the University of Georgia; throughout this project, I was mentored by [Valerio Palamara](https://www.math.uga.edu/directory/people/valerio-palamara).

**Skills Developed:**

- Integer linear programming
- Operations research modeling
- Performance optimization
- Python programming with Gurobi Optimizer
- Numerical experimentation and analysis

---

# Problem Setup

The original talent scheduling problem proposed by [T.C.E. Cheng, J.E. Diamond, and B.M.T. Lin](https://link.springer.com/article/10.1007/BF00940554) in 1993 aims to minimise the holding cost of actors who are shooting a movie. From this, there are 4 parameters:

- $n$ : number of scenes
- $m$ : number of actors
- $c_1,\cdots, c_m$ : holding cost of each actor
- $T$ : a binary matrix of which scenes need which actors

Note that this formulation runs on the assumption that each scene takes one day to shoot. When we do consider a daily working capacity and changeover times, we can consider the sequences of scenes that are feasible in a given day and the time it may take to change the set, do makeup, etc. Since a method of finding these possible sequences was not given in Yang, et al. (2026), we devised our own method.

## Finding Feasible Patterns

A feasible pattern is one in which the scenes and their changeover times can be totaled to be less than or equal to the daily working capacity. To find feasible scene patterns, we must first introduce some new parameters:

- $l_1, \cdots, l_m$ : length of each scene
- $S$ : asymmetric $m \times m$ matrix of changeover times from scene $i$ to scene $j$
- $W$ : daily working capacity
- $D$ : upper bound on the number of days needed to complete filming the scenes
- $H$ : upper bound on the number of scenes in a working day

Since we have an upper bound on how many scenes can be filmed in a day, we restrict the feasible permutation length to no more than $H$. We then found permutations of $n$ of up to length $H$ whose durations and changeover times were less than or equal to $W$.

```python
from itertools import permutations

def finding_P(n, S, durations, W, H):
    valid_perms = []
  
    for length in range(H):
        for perm in permutations(range(n), length+1):
            total = 0
    
            for scene in perm:
                total += durations[scene]
    
            for i in range(len(perm)-1):
                total += S[perm[i], perm[i+1]]
    
            if total <= W:
                valid_perms.append(perm)
  
    return valid_perms
```

From this, we find the list of unique subsets of scenes which is called $\mathcal{P}$, with each scene subset called $P_i$ for $1 \le i \le |\mathcal{P}|$. Now that we have our subsets of scenes, we can use constraints to search through them and assign them to days which they are to be filmed.

## Constraints

Before defining constraints, we must introduce 3 binary decision variables.

- $a_{k,j} = 1$ if scene $j$ is in scene subset $P_k$, $0$ otherwise
- $x_{k,d} = 1$ if scene subset $P_k$ is assigned to day $d$, $0$ otherwise
- $y_{i,d_1,d_2} = 1$ if actor $i$ is held from day $d_1$ to day $d_2$, $0$ otherwise

With this in mind, we can define our first constraint which ensures that there is one feasible pattern assigned to day 1.

$$
\sum_{k=1}^{|\mathcal{P}|} x_{k,1} = 1
$$

Next, we make sure we can only assign a feasible pattern on a day if there is one already assigned to the day previous.

$$
\sum_{k=1}^{|\mathcal{P}|} x_{k,d-1} \geq 
          \sum_{k=1}^{|\mathcal{P}|} x_{k,d},
\quad 2 \leq d \leq D
$$

Now, we want to make sure that scene $j$ is only assigned once to some subset $P_k$.

$$
\sum_{d=1}^{D} \sum_{k=1}^{|\mathcal{P}|} a_{k,j}\, x_{k,d} = 1, 
        \quad 1 \leq j \leq n
$$

And we check the holding period of each actor and ensure there is only one holding duration for each.

$$
\sum_{d_1=1}^{D} \sum_{d_2=d_1}^{D} y_{i,d_1,d_2} = 1, 
          \quad 1 \leq i \leq m
$$

Lastly, we must ensure that the scenes the actors are in are only filmed when the actor is being held.

$$
\sum_{j=1}^{n} \sum_{k=1}^{|\mathcal{P}|} t_{i,j}\, a_{k,j}\, x_{k,d} 
          \leq \sum_{d_1=1}^{d} \sum_{d_2=d}^{D} y_{i,d_1,d_2}, 
          \quad 1 \leq i \leq m,\ 1 \leq d \leq D
$$

## Objective Function

Like in the original problem, we want to minimize the total holding cost of our actors.

$$
\min\quad \sum_{i=1}^{m} c_i \sum_{d_1=1}^{D} \sum_{d_2=d_1}^{D} (d_2 - d_1 + 1) y_{i,d_1,d_2}
$$

- $c_i$ : holding cost of actor $i$
- $(d_2-d_1+1)y_{i,d_1,d_2}$ : the number of days actor $i$ is needed on set

Note that $(d_2-d_1+1)y_{i,d_1,d_2}$ is dictated by $y_{i,d_1,d_2}$ and the cost will only be counted if the current $d_1, d_2$ is the actor's holding duration found within the constraints.

# Computational Study and Results

We implemented the model using Gurobi Optimizer 13.0.1 while interfaced with Python 3.12.8. This setup is similar to that in Yang, et al. (2026) which used Gurobi Optimizer 9.0 interfaced with Python 3.10.9. We reproduced results found on a personal computer running Windows 10 with a Intel(R) Core(TM) i5-9600KF @ 3.70GHz CPU with 32 GB of RAM.

Here, $\rho$ controls the probability an actor is needed in a scene.

| $n$ | $m$ | $W$ | $\rho$ |      Status      |   Cost   | Time (s) |
| :---: | :---: | :---: | :------: | :--------------: | :-------: | :------: |
|  10  |  10  |  50  |   0.3   |     Optimal     | 42,162.0 |  0.415  |
|  10  |  10  |  50  |   0.5   |     Optimal     | 84,293.0 |  0.882  |
|  10  |  10  |  75  |   0.3   |     Optimal     | 42,162.0 |  0.488  |
|  10  |  20  |  50  |   0.3   |     Optimal     | 131,014.0 |  5.652  |
|  15  |  10  |  50  |   0.3   |     Optimal     | 68,596.0 |  17.311  |
|  15  |  10  |  50  |   0.5   |     Optimal     | 117,608.0 |  25.010  |
|  15  |  10  |  75  |   0.3   |     Optimal     | 68,596.0 |  4.997  |
|  15  |  20  |  50  |   0.3   |     Optimal     | 176,073.0 | 841.287 |
|  15  |  20  |  75  |   0.3   |     Optimal     | 176,073.0 |  16.422  |
|  10  |  10  |  75  |   0.5   | Infeasible/Other |    —    |  0.067  |
|  10  |  20  |  50  |   0.5   | Infeasible/Other |    —    |  0.117  |
|  10  |  20  |  75  |   0.3   | Infeasible/Other |    —    |  0.082  |
|  10  |  20  |  75  |   0.5   | Infeasible/Other |    —    |  0.083  |
|  15  |  10  |  75  |   0.5   | Infeasible/Other |    —    |  0.621  |
|  15  |  20  |  50  |   0.5   | Infeasible/Other |    —    |  0.756  |
|  15  |  20  |  75  |   0.5   | Infeasible/Other |    —    |  1.225  |

From the experiments, it emerged that the ILP pattern model cannot handle a large number of scenes $n.$ This scalability issue represents a critical limitation of the model and demonstrated the NP-hardness of the problem. The NP-hardness can be confirmed by noticing that the TS-DCCT problem has been reframed as a bin packing problem (packing scenes into days) which is known to be NP-hard.

It should also be noted that the way we devised to find feasible patterns also has scalability problems for large enough H. The pattern generation approach using `itertools.permutations()` has a time complexity of $O(n!)$, which becomes prohibitive for large $n$. While we mitigate this by capping the maximum pattern length $H$ (reducing the complexity to $\sum_{k=1}^{H} P(n,k)$), the method still faces scalability challenges for large problem instances. This likely explains the "Other" part in the "Infeasible/Other" results that are more common with the higher density $(\rho)$ cases.

<a href="/files/Talent_Scheduling_Griffin_Palamara.pdf" download class="btn btn--primary">
  <i class="fas fa-file-pdf"></i> Download Presentation (PDF)
</a>

<a href="/files/Talent_Scheduling__Poster.pdf" download class="btn btn--primary">
  <i class="fas fa-file-pdf"></i> Download Poster (PDF)
</a>
