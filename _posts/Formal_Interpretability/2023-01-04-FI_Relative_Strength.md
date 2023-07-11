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

This post is part 3 of my series on <a href="/blog/2023/FI_start/">Interactive Classification</a>.

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

It stands to reason that if both provers are implemented by the same algorithm, their performance should be similar. Of course, as with anything neural network related that might be hard to prove in practice.




### Key Result

With the above two notions of Relative Strength and Asymmetric Feature Correlation in mind, we can provide a key theoretical result of this paper.

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
1. We do not assume our agents to be optimal. In the presented theorem Merlin is allowed to have any arbitrary strategy.
We rather rely on the relative strength of Merlin and Morgana for our bound. We also allow our provers to
select the features with the context of the full data point.

1. We do not make the assumption that features are independently distributed. Instead, we introduce the notion
of Asymmetric Feature Correlation (AFC) that captures which correlations make an information bound
difficult.




### References


{% bibliography --cited --file formal_interpretability.bib --template bib2 %}
