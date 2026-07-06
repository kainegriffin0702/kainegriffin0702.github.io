---
layout: single
title:  "Talent Scheduling with Daily Capacity and Changeover Times"
subtitle: "A Pattern Based Integer Linear Programming Approach"
categories: 
  - Jekyll
tags:
  - talent scheduling
  - integer linear programming
  - bin packing
  - travelling salesman problem
excerpt: "A project done through the Directed Reading Program at the University of Georgia that replicates a pattern based integer linear programming model for the talent scheduling problem."
---

# Overview

In this project, we investigated the talent scheduling problem with consideration of daily capacity and changeover times (TS-DCCT) as proposed by [Zhaoqi Yang, Shunji Tanaka, and Bertrand M.T. Lin](https://doi.org/10.1016/j.omega.2025.103439) and recreated their integer linear program with feasible patterns. We implemented this model in Python interfaced with Gurobi Optimizer like Yang, et al. in order to compare computational results.

This project was done through a Directed Reading Program in the Mathematics Department at the University of Georgia; throughout this project, I was mentored by [Valerio Palamara](https://www.math.uga.edu/directory/people/valerio-palamara).

---

# Problem Setup

The original talent scheduling problem proposed by [T.C.E. Cheng, J.E. Diamond, and B.M.T. Lin](https://link.springer.com/article/10.1007/BF00940554) in 1993 has 4 parameters:

- $n$: number of scenes
- $m$: number of actors
- $c_1,\cdots, c_m$: holding cost of each actor
- $T$: a binary matrix of which scenes need which actors

Note that this formulation runs on the assumption that each scene takes one day to shoot. When we do consider a daily working capacity and changeover times, we can consider the sequences of scenes that are feasible in a given day. Since a method of finding these possible sequences was not given in Yang, et al. (2026), we devised our own method.

## Finding Feasible Scene Sequences

To find feasible scene patterns, we must first introduce some new parameters:

- $l_1, \cdots, l_m$: length of each scene 
- $S$: asymmetric matrix of changeover times from scene $i$ to scene $j$
- $W$: daily working capacity
- $D$: upper bound on the number of days needed to complete filming the scenes
- $H$: upper bound on the number of scenes in a working day

Since we have an upper bound on how many scenes can be filmed in a day, we restrict the feasible permutation length to no more than $H$. We then found permutations of $n$ of up to length $H$ whose durations and changeover times were less than or equal to $W$.

```python
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

From this, we find the list of unique subsets of scenes which is called $\mathcal{P}$, with each scene subset called $P_i$ for $1 \le i \le |\mathcal{P}|$