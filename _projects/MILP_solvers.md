---
layout: page
title: Mixed Integer Linear Programs for Adversarial Robustness
description: Make certified robustness bounds for neural networks feasible via a new algorithm based on MILP solvers.
img:
importance: 1
category: Robustness
---
{% include abbrv.html %}
## The Problem:


The problem we try to solve is

$$
 \max_{\bfx \in C} f(\bfx),
$$

where $f$ is given as a neural network and $C$ is some convex region. This problem arises from questions of adversarial robustness, where $
C = \skl{\bfx\in \R^n:~ \nkl{\bfx - \bfx_{\text{sample}} } \leq \epsilon},
$ formal verification of networks and interpretability.

Generally, this is a $\SNP$-hard problem and any general solution strategy will take exponentially long in the worst case. However, real-world trained neural networks do not necessarily represent the worst-case. Thus a heuristic algorithm like Gradient Descent works reasonably well in solving this problem.

However, gradient descent does not give us any bounds on how close we are to the optimal $\bfx$. We would like to design an algorithm that combines formal guarantees with speedy solutions for most real-world instances.

## Using MIP Solvers

A ReLU neural network can be rewritten as a Mixed Integer Linear Program when there exists a general bound on the size of the variables in each layber, see [this paper](https://arxiv.org/pdf/1712.06174.pdf) for instructions.

The general idea is to introduce binary variables that signify on the sign of the pre-Rely neuron and thus which linear region of the ReLU is applied. When all these sign variables are fixed the neural network becomes a linear problem.

Of course, searching over all possible assignments takes exponentially long in the number of ReLU-activated neurons. Solving the whole network at once might therefore be only feasible for small problems.

## The Constraint Propagation Algorithm (CPA)

This is where the new constrain propagation algorithm comes in.

**Preparation:**

Get a rough first bound on all the post-Relu activations $x^{(l)}_j$ in every layer $l$ by using the inequality

$$ 0 \leq x^{(l)}_j \leq c^{(l)}_j $$

with

$$
 c^{(l)}_j = \bfw^{(l),+}_j \bfc^{(l)} + b^{(l)}_j
$$

and

$$
c^{(1)}_j = \sum_i \bfw^{(1),+}_{ij} \max(x_i) + \bfw^{(1),-}_{ij} \min(x_i),
$$

where $\bfw^{(l),+}$ and $\bfw^{(l),-}$ are the positive and negative part of the weight matrix in layer $l$ and $\max(x_i)$ and $\min(x_i)$ are the upper and lower bound on the input variables.

This will give us a reasonable a priori bound on all the activations. This bound is then used to transform the network into a MIP.

The idea of the CPA is to iteratively make these bounds tighter in the relevant direction.

**The algorithm:**

The algorithm loops over alternating forward and a backward pass.

*Forward Pass:*

Propagate a feasible candidate vector from input to output layer and note the resulting candidate for every layer. In the beginning, this vector can be randomly chosen. We can update a lower bound on the target value in the output layer with each forward pass.

*Backward Pass:*

Now, we start at the output layer and go back to the input.

1. For layer l we assume to have:
    * A target vector $\bft$
    * A candidate vector $\bfk$
    * A convex bounding region $C$ for the neurons in layer $l-1$
1. First, try to solve

$$
 \bfx \in C \quad \text{s.t.}\quad \bft = \text{ReLU}(W^{(l)T} \bfx + \bfb^{(l)})
$$

  a) Feasible case:
  Proceed to layer $l-1$ with new target vector $\bfx$.

  b) Infeasible case:
  1. Determine active direction $\bfd = \bft - \bfk$.
  1. Solve linear problem

$$
 \max_{\bfx \in C} \bfd^T \text{ReLU}(W^{(l)T} \bfx + \bfb^{(l)})
$$

which gives solution $\bfx^\ast$, value $v^\ast$ and a set of active constraints.
1. We can thus add

$$
 \bfd^{T} \bfx \leq v^*
$$

as a new constraint for layer $l$.
1. Proceed to layer $l-1$ with new target $\bfx^*$

We start at the output layer $L$ with the (1-dimensional) target $c^L$, and repeat the procedure once we reached the input layer. Note that the constraints on every layer get tighter over time, which leads to increasingly realistic target vectors.

The forward pass increases the lower bound on the maximal output value, while the backward pass decreases the upper bound. We are done when both converge or are sufficiently close.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/constraint_propagation.svg" alt="img1" style="float:center; margin-right: 5%; width:65%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** Illustation for a layer of width 2. The known constraints result in the light green convex area. The dark green arrows mark the linear problem for each ReLU region. The point $a^\ast$ is the solution to the resulting maximisation problem. If this target cannot be reached in the next lower layer, the active direction $\bfd = \bfa^* - \bfk$ is optimised instead, which leads to the next maximisation problem.}


## How to determine the active direction?

This is where a bit of experimentation comes in what works best. My first best guess is to chose as the difference between the candidate and the target. Another possibility is to consider the active constraints for the target vector and choose a vector in a way that maximally reduces the size of the convex region in that corner.


## How to get started?

I think a good start is a neural network of width 2, where the constraints on every level can be easily visualised and more intuition can be gained. An easy beginner task might be a neural network of a single input and with depth 5 that fits some analytical function, like $\sin(x)$ over some compact domain, like $[0,2\pi]$.

## Combination with ResNet Architecture:

The speed of each single MIP-solving step depends on the number of ReLU neurons. Neurons with a simple linear activation do not make the MIP problem harder. The ResNet architecure mixes neurons with ReLU acitvations and linear activations. We might reach a sweet spot for training performance and solver speed by trading the depth of the network against the number of ReLU units per layer.


## Research Questions:
1. Is the layer-wise strategy faster than just solving the whole DNN-MIP at once?
1. What are the most complex datasets that can be solved in reasonable time?
1. How should we choose the active direction?
1. What is the optimal ResNet trade-off?
1. Prove that the upper and lower bounds are always correct (I think this is straightforward).
