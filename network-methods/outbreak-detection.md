---
layout: post
title: Outbreak Detection in Networks
---

## Introduction
Outbreak detection in networks has many applications in real life. For example, where should we place sensors to quickly detect contaminations in a water distribution network? Which person's blog should we follow to avoid missing important stories in  a social network? Which cities should be monitored to prevent epidemics. These seemingly different problems have a general goal: given a dynamic process spreading over a network, we want to select a set of nodes to detect the process efficiently.

The following figure shows an example:
\\TODO

## Problem Setup
The outbreak detection problem is defined as below:
- Given: a graph $$G(V,E)$$ and data on how outbreaks spread over this $$G$$ (for each outbreak $$i$$ we knew the time $$T(u,i)$$ when the outbreak $$i$$ contaminates node $$u$$).
- Goal: select a subset of nodes $$S$$ that maximize the expected reward:

$$
\max_{S\subseteq U}f(S)=\sum_{i}p(i)\cdot f_{i}(S)
$$

$$
\text{subject to }cost(S)\leq B
$$

where

- $$p(i)$$: probability of outbreak $$i$$ occuring
- $$f_{i}(S)$$: rewarding for detecting outbreak $$i$$ using "sensors" $$S$${% include sidenote.html id='note-outbreak-detection-problem-setup' note='It is obvious that $$p(i)\cdot f_{i}(S)$$ is the expected reward for detecting the outbreak $$i$$' %}
- $$B$$ total budget of placing "sensors"


**Reward** can be one the following three:

- Minimize time to detection
- Maximize number of detected propagations
- Minimize number of infected people

**Cost** is context dependent. Examples are:

- Reading big blogs is more time consuming
- Placing a sensor in a remote location is expensive

## Outbreak Detection Formalization

### Objective Function for Sensor Placements

Define a **penalty $$\pi_{i}(t)$$** for detecting outbreak $$i$$ at time $$t$$, which can be one of the following:{% include sidenote.html id='note-outbreak-detection-penalty-note' note='Notice: in all the three cases detecting sooner does not hurt! Formally, this means, for all three cases, $$\pi_{i}(t)$$ is monotonically nondecreasing in $$t$$.'%}

- **Time to Detection (DT)**
  - How long does it take to detect a contamination?
  - Penalty for detecting at time $$t$$: $$\pi_{i}(t)=t$$

- **Detection Likelihood (DL)**
  - How many contaminations do we detect?
  - Penalty for detecting at time $$t$$: $$\pi_{i}(0)=0$$, $$\pi_{i}(\infty)=1$${% include sidenote.html id='note-penalty-dl' note='this is a binary outcome:  $$\pi_{i}(0)=0$$ means we detect the outbreak and we pay 0 penalty, while $$\pi_{i}(\infty)=1$$ means we fail to detect the outbreak and we pay 1 penalty.'%}

- **Population Affected (PA)**
  - How many people get contaminated?
  - Penalty for detecting at time $$t$$: $$\pi_{i}(t)=$$ number of infected nodes in outbreak $$i$$ by time $$t$$

The objective **reward function $$f_{i}(S)$$ of a sensor placement $$S$$** is defined as penalty reduction:

$$
f_{i}(S)=\pi_{i}(\infty)-\pi_{i}(T(S,i))
$$

where $$T(S,i)$$ is the time when the set of "sensors" $$S$$ detects the outbreak $$i$$.

### Claim1: $$f(S)=\sum_{i}p(i)\cdot f_{i}(S)$$ is nondecreasing
For all $$A\subseteq B\subseteq V$$ ($$V$$ is all the nodes in $$G$$), $$T(A,i)\geq T(B,i)$$, and

$$
f_{i}(A)-f_{i}(B)=\pi_{i}(\infty)-\pi_{i}(T(A,i))-[\pi_{i}(\infty)-\pi_{i}(T(B,i))]
$$

$$
=\pi_{i}(T(B,i))-\pi_{i}(T(A,i))
$$

Because, $$\pi_{i}(t)$$ is monotonically nondecreasing in $$t$$ (see sidenote 2), $$f_{i}(A)-f_{i}(B)<0$$. Therefore, $$f_{i}(S)$$ is nondecreasing.

Then, it is obvious that $$f(S)=\sum_{i}p(i)\cdot f_{i}(S)$$ is also nondecreasing, since $$p(i)\geq 0$$.


### Claim2: $$f(S)=\sum_{i}p(i)\cdot f_{i}(S)$$ is submodular
This is to proof for all $$A\subseteq B\subseteq V$$ $$x\in V \setminus B$$:

$$
f(A\cup \{x\})-f(A)\geq f(B\cup\{x\})-f(B)
$$

There are three cases when sensor $$x$$ detects the outbreak $$i$$:
1. $$T(B,i)\leq T(A, i)<T(x,i)$$ ($$x$$ detects late): nobody benefits. That is $$f_{i}(A\cup\{x\})=f_{i}(A)$$ and $$f_{i}(B\cup\{x\})=f_{i}(B)$$. Therefore, $$f(A\cup \{x\})-f(A)=0= f(B\cup\{x\})-f(B)$$
2. $$T(B, i\leq T(x, i)<T(A,i))$$ ($$x$$ detects after $$B$$ but before $$A$$):so $$x$$ only helps to improve the solution of $$A$$ but not $$B$$. Therefore, $$f(A\cup \{x\})-f(A)\geq 0 = f(B\cup\{x\})-f(B)$$
3. $$T(x, i)<T(B,i)\leq T(A,i)$$ ($$x$$ detects early): $$f(A\cup \{x\})-f(A)=[\pi_{i}(\infty)-\pi_{i}(T(x,t))]-f_{i}(A)$$$$ \geq [\pi_{i}(\infty)-\pi_{i}(T(x,t))]-f_{i}(B) = f(B\cup\{x\})-f(B)$${% include sidenote.html id='note-submodularity-proof1' note='Inequality is due to the nondecreasingness of $$f_{i}(\cdot)$$, i.e. $$f_{i}(A)\leq f_{i}(B)$$ (see Claim1).'%}

Therefore, $$f_{i}(S)$$ is submodular. Because $$p(i)\geq 0$$, $$f(S)=\sum_{i}p(i)\cdot f_{i}(S)$$ is also submodular.{% include sidenote.html id='note-submodularity-proof1' note='Fact: a non-negative linear combination of submodular functions is a submodular function.'%}

We know that the HIll Climbing Algorithm works for optimizing problems with nondecreasing submodular objectives. However, it does not work well in this problem:

- Hill Climbing only works for the cases that each sensor costs the same. For this problem, each sensor has cost $$c(s)$$.
- Hill Climbing is also slow: at each iteration, we need to re-evaluate marginal gains of all nodes.

Hence, we need a new fast algorithm that can handle cost constraints.

## CELF: Algorithm for Optimziating Submodular Functions Under Cost Constraints
### Bad Algorithm 1: Hill Climbing that ignores cost
**Algorithm**

- Ignore sensor cost $$c(s)$$
- Repeatly select sensor with highest marginal gain
- Do this until the budget is exhausted

**This can fail arbitrarily bad!** Example:
- Given $$n$$ sensors and a budget $$B$$
- $$s_{1}$$: reward $$r$$, cost $$B$$
- $$s_{2}$$,..., $$s_{n}$$: reward $$r-\epsilon$$, cost $$\epsilon$$ ($$\epsilon$$ is an arbitrary positive small number)
- Hill Climbing always prefers $$s_{1}$$ to other cheaper sensors, resulting in an arbitrarily bad solution with reward $$r$$ instead of the optimal solution with reward $$\frac{B(r-\epsilon)}{\epsilon}$$ when $$\epsilon \rightarrow 0$$.

### Bad Algorithm 2: optimization using benefit-cost ratio
**Algorithm**
- Greedily pick the sensor $$s_{i}$$ that maximizes the benefit to cost ratio until the budget runs out. That is always pick

$$
s_{i}=\arg\max_{s\in(V\setminus A)}\frac{f(A_{i-1}\cup\{s\})-f(A_{i-1})}{c(s)}
$$

**This can fail arbitrarily bad!** Example:
- Given 2 sensors and a budget $$B$$
- $$s_{1}$$: reward $$2\epsilon$$, cost $$\epsilon$$
- $$s_{2}$$: reward $$B$$, cost $$B$$
- Then the benefit ratios for the first selection are: 2 and 1, respectively
- This algorithm will pick $$s_{1}$$ and then cannot afford $$s_{2}$$, resulting in an arbitrarily bad solution with reward $$2\epsilon$$ instead of the optimal solution $$B$$ when $$\epsilon \rightarrow 0$$.

### Solution: CELF (Cost-Effective Lazy Forward-selection)
**CELF** is a two pass greedy algorithm [[Leskovec et al. 2007]](https://www.cs.cmu.edu/~jure/pubs/detect-kdd07.pdf):
- Get solution $$S'$$ using unit-cost greedy (Bad Algorithm 1)
- Get solution $$S''$$ using benefit-cost greedy (Bad Algorithm 2)
- Final solution $$S=\arg\max[f(S'), f(S'')]$$

**Approximation Guarantee**
- CELF achieves $$\frac{1}{2}(1-\frac{1}{e})$$ factor approximation.
