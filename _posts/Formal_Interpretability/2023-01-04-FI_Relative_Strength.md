---
layout: post
title:  "Interactive Classification 4: Relative Strength of Cooperative and Adversarial Prover"
date:   2023-01-04 00:00:00 +0100
categories: [research]
tags: [optimization, ml, uniform convexity, abstract]
math: true
comments: true
giscus_comments: true
author: Stephan Wäldchen
---

{% include abbrv.html %}

$\newcommand{\ap}{\text{Pr}}$
$\newcommand{\morg}{\widehat{M}}$

<style>
  .figcap {
    font-size: 0.9em;
  }
</style>

This post is part 4 of my series on <a href="/blog/2023/FI_start/">Interactive Classification</a>.

*TL;DR:
It's not necessary that Morgana plays perfectly to counter Merlin, only that she is able to find similar features with a comparable success rate. We go over
1. Definition of relative strength
1. Potential computational barriers for Morgana
1. Resultig theorem

<!--more-->

### Relative Strength of Merlin and Morgana
Another important metric that we care about is the relative strength of the Merlin and Morgana classifiers. This is especially important if we intend to apply our setup to real data sets where Merlin and Morgana are not able to find the optimal features are every step.
We can relax thsi requirement in two important ways:
1. She only needs to find the same features that Merlin also uses
1. She only needs to do so at a success rate similar to Merlin

With this in mind, we define the notion of relative success rate as follows.

**Relative Success Rate:** Let $A\in \CA$ and $M, \morg \in \CM(D)$. Then the relative success rate $\alpha$ is defined as

$$
   \alpha := \min_{l\in \{-1,1\}} \frac{\P_{\bfx\sim \CD_{-l}}\ekl{A(\morg(\bfx))=l \,|\, \bfx \in F_l^\ast}}{\P_{\bfx\sim \CD_{l}}\ekl{A(M(\bfx))=l \,|\, \bfx \in F_l^\ast}},
$$

where
$$F_{l} := \{\phi \in \Sigma \,|\, \bfz\in M(D_l), A(\bfz)=l\}$$ is the set that Merlin uses to successfully convince Arthur of class $l$. In plain words: Given that the the datapoint has a feature that Merlin could successfully use, how likely is Morgana to discover this feature relative to Merlin?

It stands to reason that if both provers are implemented by the same algorithm, their performance should be similar. Of course, as with anything neural network related that might be hard to prove in practice. In **Figure 1** we present an example of an exponentially bad relative strength for a as long as Morgana is implemented with a polynomial-time algorithm.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/random_sparse.svg" alt="img1" style="float:center; width:80%">
  <p style="clear: both;"></p>
</div>
<br>
**Figure 1.** Illustration of a dataset where Morgana's task is computationally harder than the one of Merlin and we should expect a very low relative strength. Class $-1$ consists of $k$-sparse images whose pixel values sum to some number $S$. For each of these images, there is a non-sparse image in class $1$ that shares all non-zero values (marked in red for the first image). Merlin can use the strategy to show all $k$ non-zero pixels for an image from class $-1$ and $k+1$ arbitrary non-zero pixels for class $1$. Arthur checks if the sum is equal to $S$ or if the number of pixels equal to $k+1$, otherwise he says "Don't know!". He will then classify with 100\% accuracy. Nevertheless, the features Merlin uses for class $-1$ are completely uncorrelated with the class label. To exploit this, however, Morgana would have to solve the $\SNP$-hard (see {% cite kleinberg2006algorithm --file formal_interpretability %}) subset sum problem to find the pixels for images in class 1 that sum to $S$. The question is not in which class we can find the features, but in which class we can find the features *efficiently*.
{:.figcap}
<br>


### Key Result

With the notions of Relative Strength and Asymmetric Feature Correlation in mind, we can provide a key theoretical result.

**Main Theorem:** Let $D$ be two-class data space with AFC of $\kappa$ and class imbalance $B$. Let $A\in \CA$, and $M, \widehat{M}\in\CM(D)$ such that $\widehat{M}$ has a relative success rate of $\alpha$ with respect to $A, M$ and $D$.
Define

1. Completeness:
$$
\min\limits_{l \in \{-1,1\}}\P_{\bfx \sim \CD_l}\ekl{A(M(\bfx)) = c(\bfx) } \geq 1- \epsilon_c,
$$
2. Soundness
$$
\max\limits_{l \in \{-1,1\}} \P_{\bfx \sim \CD_l}\ekl{A(\morg(\bfx)) = -c(\bfx) } \leq  \epsilon_s.
$$

where $\CD_l$ is the data distribution conditioned on the class $l$.
Then it follows that

$$
\ap_{\CD}(M) \geq 1 - \epsilon_c - \frac{ \kappa \alpha^{-1}\epsilon_s}{1 - \epsilon_c+ \kappa \alpha^{-1}B^{-1}\epsilon_s}.
$$

This result shows that we can bound the performance of the feature selector Merlin (in terms of average precision) in the Merlin-Arthur framework using measurable metrics such as completeness and soundness.

### Key Takeaways
The above theoretical discussion shows two key contributions of our framework.
1. We do not assume our agents to be optimal, but rather that Morgana has a comparable success rate to Merlin. This seems reasonable when we use the same algorithms for both and deal with real-world datasets.

1. We can use the relative strength and the AFC to derive a lower bound on the precision of the features that are selected by Merlin.

<a href="/blog/2023/FI_AFC/">&#9664; Previous Post</a>
<p align="right"><a href="/blog/2023/FI_Complexity/" align="right"> Next Post &#9654;</a></p>

### References


{% bibliography --cited --file formal_interpretability.bib --template bib2 %}
