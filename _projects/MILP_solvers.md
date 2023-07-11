---
layout: page
title: Mixed Integer Linear Programs for Adversarial Robustness
description: Can we use MILP-Solvers to find
img:
importance: 1
category: Robustness
---



Use Mixed Integere (Linear) Programs to get certified robustness bounds.


## The Problem:


The problem we try to solve is
$$
 \max_{\nkl{\delta} \in C} f(\bfx),
$$
where $f$ is given as a neural network and $C$ is some convex region. This problem arises from questions of adversarial robustness, formal verification of networks to interpretability.

Generally, this is a $\SNP$-hard problem and any general solution strategy will take exponentially long in the worst case. However, real-world trained neural networks do not necessarily represent the worst-case. Thus a heuristic algorithm like Gradient Descent works reasonably well in solving this problem.

However, gradient descent does not give us any bounds on how close we are to the optimal $\bfdelta$. We would like to design an algorithm that combines formal guarantees with speedy solutions for most real-world instances.

## Using MIP Solvers

A ReLU neural network can be rewritten as a Mixed Integer Linear Program when there exists a general bound on the size of the variables in each layber, see [this paper](https://arxiv.org/pdf/1712.06174.pdf) for instructions.

The general idea is to introduce binary variables that signify on the sign of the pre-Rely neuron and thus which linear region of the ReLU is applied. When all these sign variables are fixed the neural network becomes a linear problem.

Of course, searching over all possible assignments takes exponentially long in the number of ReLU-activated neurons. Solving the whole network at once might therefore be only feasible for small problems.

## The Constraint Propagation Algorithm (CPA)

This is where the new constrain propagation algorithm comes in.

*Preparation:*
Get a rough first bound on all the post-Relu activations $a^(l)_j$ in every layer $l$ by using the inequality

$$ 0 \leq a^(l)_j \leq c^(l)_j $$

with

$$
 c^(l)_j = \bfw^{(l),+}_j \bfc^(l) + b^(l)_j
$$

and

$$
c^(1)_j = sum_i \bfw^{(l),+}_{ij} \max(x_i) + \bfw^{(l),-}_{ij} \min(x_i),
$$

where $\bfw^{(l),+}$ and $\bfw^{(l),-}$ are the positive and negative part of the weight matrix in layer $l$ and $\max(x_i)$ and $\min(x_i)$ are the upper and lower bound on the input.

This will give us a reasonable a priori bound on all the activations. This bound is then used to transform the network into a MIP.

The idea of the CPA is to iteratively make these bounds tighter in the relevant direction.

**The algorithm:**

We are starting in the output layer with dimension 1. The *active direction* $d$ is thus a just a number, $1$ for maximisation and $-1$ for minimisation.

1. In layer l we assume to have:
    A vector $\bfd$ pointing in the active direction for this layer.
    A convex bounding region $A$ for the neurons in layer $l-1$
1. solve the MIP

$$
 \max_{\bfa \in A} d^T text{ReLU}(W^{(l)T} \bfa + \bfb^{(l)})
$$

which gives solution $\bfa^*$, value $v^*$ and a set of active constraints.
1. We can thus add

$$
 \bfd^{T} \bfx \leq v^*
$$

as a new constraint for layer $l$.
1. We get the relevant direction for layer $l-1$ from the solution vector $\bfa^*$ and the set of active constraints. How exactly the new direction arises should be investigated, but one simple heuristic is the sum of all the normal vectors for the active constraints pointing outward from the convex region.
1. Proceed to layer $l-1$.

Once we have reached the input layer, we begin back at the output layer. The whole procedure will be repeated until we either have a feasible point that



The constraint propagation algorithm loops over the following steps backwards through the networks




## Combination with ResNet Architecture:

The speed of each single MIP-solving step depends on the number of ReLU neurons. Neurons with a simple linear activation do not make the MIP problem harder. The ResNet architecure mixes neurons with ReLU acitvations and linear activations. We might reach a sweet spot for training performance and solver speed by trading the depth of the network against the number of ReLU units per layer.


Research Questions:
1. Is the layer-wise strategy faster than just solving the whole DNN-MIP at once?
1. What are the most complex datasets that can be solved in reasonable time?
1.
1. What is the optimal ResNet trade-off?
1.
