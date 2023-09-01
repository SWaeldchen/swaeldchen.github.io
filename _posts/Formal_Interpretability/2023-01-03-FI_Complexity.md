---
layout: post
title:  "Interactive Classification 5: Computational Complexity"
date:   2023-01-03 00:00:00 +0100
categories: [research]
tags: [optimization, ml, uniform convexity, abstract]
math: true
comments: true
giscus_comments: true
author: Stephan Wäldchen
---

{% include abbrv.html %}

This post is part 5 of my series on <a href="/blog/2023/FI_start/">Interactive Classification</a>

*TL;DR:
We explain the computational complexity of interpreting neural network classifier.

1. Finding small precise features is a hard task even if done only approximately.
1. We can use heuristics as long as we can verify that the heuristic has succeeded a posteriori.
1. To audit a Merlin-Arthur classifier the auditor needs comparable computational resources as the designer of the classifier.



## Computational Complexity of finding features with High precision

On its face, it is not surprising that finding small features that have high precision is an NP-hard task, since it implies combinatorial search over sets of input variables.

We have shown this explicitely in {% cite waldchen2021computational --file formal_interpretability %}. That paper still uses the term $\delta$-relevant features, which is equivalent to a feature with a precision of $\delta$. We also show the stronger result that the smallest set of features with precision $\delta$ cannot be approximated better than $d^{1-\alpha}$ unless $\SP=\SNP$, where $d$ is the input dimension and $\alpha>0$. Note that for $\alpha=0$ we get the trivial approximation of simply taking that whole input as feature with perfect precision. This means that one cannot prove for any procedure that it systematically finds small precise features should they exist. This holds even for two-layer neural networks.

Instead of selecting the smalles set (cardinality-minimal), one can relax the question to selecting a set that cannot be made smaller by omitting any elements from it (inclusion-minimal). For monotone classificers, this makes the problem straight-forward to solve, as shown in {% cite shih2018symbolic --file formal_interpretability %}, as one can simply successively omit input variables from a feature until any further omission would reduce the precision below $\delta$. The authors additionally show that this is efficiently possible for classifiers represented as <a href="https://en.wikipedia.org/wiki/Binary_decision_diagram">Ordered Binary Decision Diagrams (OBDDs)</a>.

## The Result by Blanc et al.

While we have shown that there are networks and inputs for which finding small precise features is a hard task, a surprising result by {% cite blanc2021provably --file formal_interpretability %} shows for a random input it is feasible in polynomial time with high probability.
The caveat here is the size of the found feature, which is polynomial in the size of the smallest precise feature. In fact, it grows so quickly that is unusuable in practice if there does not exist a precise feature that is orders of magnitude smaller than the whole input dimension. Nevertheless, this is an impressive result connecting interesting topics, such as stabiliser trees, implicit representation etc.


## How to overcome computational barrier?

The computational complexity can be ignored. Instead, we can use a heuristic method to select a feature and confirm high precision afterwards. The Merlin-Arthur framework is a heuristic as well in this regard, as we are not guaranteed to converge to a setup with high completeness and soundness, but we can easily check whether this has been achieved.

This is similar to the training process of neural networks. Designing a classifier with high accuracy is a computationally hard task. But SGD is a method that reliably succeeds in practice, and one can confirm success via the test set evaluation. Completness and soundness thus take the role of the test accuracy and confirm not only good performance bu also interpretability.

## Use of Arthur-Merlin Classifiers as Explainable classifiers

Let us come back to the main reason we introduce the Merlin-Arthur classifier for formal interpretability. We want a setup that is provably explainable, especially for the case when the designer of the classifier wants to hide the true reasoning of the classifier. This is important for commercial classifiers, e.g. for hiring decicions. An auditor would want to check if the reason a candidate was hired or rejected was not based on protected features like gender or race.

We have seen proved that if a sound and complete Merlin-Arthur classifiers has to exchange informative features. An auditor could confirm the soundness with their own Morgana, as to make sure that the setup is actually sound.

In our theorems we have seen that the precision bound depends on the relative success rate of Merlin and Morgana. This means that this scheme is successful as long as the company designing the classifier and the auditor have comparable computational resources. This again reflected in the AFC, since we have shown that determining the size of the AFC is as hard as exploiting it. This again reflects that the certification of the Merlin-Arthur classifier works as long as the auditor has comparable computational resources as the firm they are auditing. On the other hand, the auditor does not need to model the datamanifold that the classifier operates on. This task is potentially much harder and has to be done for every new classification task, compared with just designing a good search routine for Morgana.

<a href="/blog/2023/FI_Relative_Strength/">&#9664; Previous Post</a>

### References

{% bibliography --cited --file formal_interpretability.bib --template bib2 %}
