---
layout: post
title:  "Formal Interpretability 3: Interactive Classification"
date:   2023-01-04 00:00:00 +0100
categories: [research]
tags: [optimization, ml, uniform convexity, abstract]
math: true
comments: true
giscus_comments: true
author: Stephan Wäldchen
---

<style>
  .figcap {
    font-size: 0.9em;
  }
</style>

This post is part 3 of my series on <a href="/blog/2023/FI_start/">Formal Interpretability</a>.

*TL;DR:
We present interactive classification as an approach to define informative features without modelling the data distribution explicitly

1. Interactive Classification with Prover and Verifier: why the prover needs to be unreliable
1. Asymmetric Feature Correlation
1. Do we need a game equilibrium?
<!--more-->


In <a href="/blog/2023/FI_Preliminaries/">a previous post</a> we discussed why it is difficult to model the conditional data distribution. This distribution is used to define feature importance according to high Mutual Information or Shapley Values.
Now, we explain how this issue can be circumvented by designing an inherently interpretable classifier through an interactive classification setup. This setup will allow us to derive lower bounds on the **precision** of the features in terms of quantities that can be easily estimated on a dataset.

### Interactive Classification

The inspiration for interactive classification comes from Interactive Proof Systems (IPS), a concept from Complexity Theory, specifically the [Merlin-Arthur protocol](https://en.wikipedia.org/wiki/Arthur%E2%80%93Merlin_protocol#MA). The **prover** selects a feature from the datapoint and sends it to the verifier, Arthur, who decides the class. Crucially, in IPS the prover is unreliable, sometimes trying to convince the verifier of a wrong judgement. We mirror this by having a second prover, Morgana, that tries to get Arthur to say the wrong class. Arthur is allowed to say "Don't know!" and thus refraining from classification.
In this context, we can then translate the concepts of *completeness* and *soundness* from IPS to our setting. Completeness describes the probability that Arthur classifies correctly based on features from Merlin. Soundness is the probability that Arthur does not get fooled by Morgana, thus either giving the correct class or answering ''Don't know!''.
These two quantities can be measured on a test dataset and are used to lower bound the information contained in features selected by Merlin.

Interactive classification had been introduced earlier in [Lei2016] and [Bastings2019] -- without an adversarial aspect. It was then noted in [Yu2019] that in that case Merlin and Arthur can "cheat" and use uninformative features to communicate the class.


<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/cheating.svg" alt="img1" style="float:center; width:50%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** An example of a decision list taken from {% cite rudin2019stop --file formal_interpretability %} used to predict whether a delinquent will be arrested again. The reasoning of the decision list is directly readable.)
{:.figcap}



### Asymmetric Feature Correlation



### Relative Strength of Merlin and Morgana








### References

[D] Dunn, Joseph C. Rates of convergence for conditional gradient algorithms near singular and nonsingular extremals. SIAM Journal on Control and Optimization 17.2 (1979): 187-211. [pdf](https://epubs.siam.org/doi/pdf/10.1137/0324071?casa_token=mV4qkf9aLskAAAAA:--jyeKNCSwAH5fejuzgJr1im_OXPyesfgPOU1fk-cfmBYZjTdrRSAHHfZEWjQRUaYSI0vNPB7NwY)

[DR] Demyanov, V. F. ; Rubinov, A. M. Approximate methods in optimization problems. Modern Analytic and Computational Methods in Science and Mathematics, 1970.

[DDS] Donahue, M. J.; Darken, C.; Gurvits, L.; Sontag, E. (1997). Rates of convex approximation in non-Hilbert spaces. *Constructive Approximation*, *13*(2), 187-220.

[GH] Garber, D.; Hazan, E. Faster rates for the frank-wolfe method over strongly-convex sets. In 32nd International Conference on Machine Learning, ICML 2015. [pdf](https://arxiv.org/abs/1406.1305)

[GI] Goncharov, V. V.; Ivanov, G. E. Strong and weak convexity of closed sets in a hilbert space. In Operations research engineering, and cyber security, pages 259–297. Springer, 2017.

[GM] Guélat, J.; Marcotte, P. (1986). Some comments on Wolfe's ‘away step’. Mathematical Programming, 35(1), 110-119.

[HL] Huang, R.; Lattimore, T.; György, A.; Szepesvári, C. Following the leader and fast rates in linear prediction: Curved constraint sets and other regularities. In Advances in Neural Information Processing Systems, pages 4970–4978, 2016. [pdf](https://arxiv.org/abs/1702.03040)

[J] Jaggi, M. Revisiting Frank-Wolfe: Projection-free sparse convex optimization. In Proceedings of the 30th international conference on machine learning, ICML 2013. p. 427-435. [pdf](http://www.jmlr.org/proceedings/papers/v28/jaggi13.pdf)

[JN] Juditsky, A.; Nemirovski, A.S. Large deviations of vector-valued martingales in 2-smooth normed spaces. [pdf](https://arxiv.org/abs/0809.0813)

[KDP] Kerdreux, T.; d’Aspremont, A.; Pokutta, S. 2019. Restarting Frank-Wolfe. In The 22nd International Conference on Artificial Intelligence and Statistics (pp. 1275-1283). [pdf](https://arxiv.org/abs/1810.02429)

[LJ] Lacoste-Julien, S. ;  Jaggi, Martin. On the global linear convergence of Frank-Wolfe optimization variants. In : Advances in neural information processing systems. 2015. p. 496-504. [pdf](https://infoscience.epfl.ch/record/229239/files/nips15_paper_sup_camera_ready.pdf)

[P] Polyak, B. T. Existence theorems and convergence of minimizing sequences for extremal problems with constraints. In Doklady Akademii Nauk, volume 166, pages 287–290. Russian Academy of Sciences, 1966.

[ST] Sridharan, K.; Tewari, A. (2010, June). Convex Games in Banach Spaces. In *COLT* (pp. 1-13). [pdf](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.304.5992&rep=rep1&type=pdf)
