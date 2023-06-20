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

$\newcommand{\bfx}{\mathbf{x}}$
$\newcommand{\bfy}{\mathbf{y}}$
$\newcommand{\bfz}{\mathbf{z}}$
$\newcommand{\ap}{\text{Pr}}$
$\newcommand{\CD}{\mathcal{D}}$
$\newcommand{\ekl}[1]{\mathopen{}\left[ #1 \right]\mathclose{}}$
$\newcommand{\E}{\mathbb{E}}$
$\renewcommand{\P}{\mathbb{P}}$
$\newcommand{\morg}{\widehat{M}}$
$\newcommand{\CA}{\mathcal{A}}$
$\newcommand{\CM}{\mathcal{M}}$

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
Now, we explain how this issue can be circumvented by designing an inherently interpretable classifier through an interactive classification setup. This setup will allow us to derive lower bounds on the **precision** of the features in terms of quantities that can be easily estimated on a dataset. {% cite lundberg2017unified --file formal_interpretability %}

### Interactive Classification

The inspiration for interactive classification comes from Interactive Proof Systems (IPS), a concept from Complexity Theory, specifically the [Merlin-Arthur protocol](https://en.wikipedia.org/wiki/Arthur%E2%80%93Merlin_protocol#MA). The prover (**Merlin**) selects a feature from the datapoint and sends it to the verifier (**Arthur**) who decides the class.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/concept.svg" alt="img1" style="float:center; width:50%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** An example of a decision list taken from {% cite rudin2019stop --file formal_interpretability %} used to predict whether a delinquent will be arrested again. The reasoning of the decision list is directly readable.)
{:.figcap}


Crucially, in IPS the prover is unreliable, sometimes trying to convince the verifier of a wrong judgement. We mirror this by having a second prover, **Morgana**, that tries to get Arthur to say the wrong class. Arthur is allowed to say "Don't know!" and thus refraining from classification.
In this context, we can then translate the concepts of *completeness* and *soundness* from IPS to our setting.

- **Completeness:** describes the probability that Arthur classifies correctly based on features from Merlin.
- **Soundness:** is the probability that Arthur does not get fooled by Morgana, thus either giving the correct class or answering ''Don't know!''.

These two quantities can be measured on a finite dataset and are used to lower bound the information contained in features selected by Merlin. Since these are simple scalar quantities, one can easily estimate them on the test set similar to the test accuracy of a normal classifier.

### Cheating with cooperative Prover and Verfier

Interactive classification had been introduced earlier in {% cite lei2016rationalizing --file formal_interpretability %}  and {% cite bastings2019interpretable --file formal_interpretability %} -- without an adversarial aspect. It was then noted in {% cite lei2016rationalizing --file formal_interpretability %} that in that case Merlin and Arthur can "cheat" and use uninformative features to communicate the class, as illustrated in Figure 1.


<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/cheating.svg" alt="img1" style="float:center; width:50%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** An example of a decision list taken from {% cite rudin2019stop --file formal_interpretability %} used to predict whether a delinquent will be arrested again. The reasoning of the decision list is directly readable.)
{:.figcap}

If prover and verifier are purely cooperative, Merlin can decide the class and communicate it over an arbitrary code. The feature selected for this code need not have anything to do with the features that Merlin used to decide the class.
However, any such strategy can be exploited by an adversarial prover (Morgana) to convince Arthur of the wrong class. The intuition is this: Let us assume the verifier accepts a feature as proof of a class that is uncorrelated with the class. Then this feature must also appear in datapoints of a different class. Morgana can then select this feature in the different class and convince the Arthur to give the wrong classification.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/strategy.svg" alt="img1" style="float:center; width:100%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** An example of a decision list taken from {% cite rudin2019stop --file formal_interpretability %} used to predict whether a delinquent will be arrested again. The reasoning of the decision list is directly readable.)
{:.figcap}

Now we want to pack this intuition into theory!

### Theoretical Setup

What exactly constitutes a feature can be up for debate.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/house.svg" alt="img1" style="float:center; width:70%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** An example of a decision list taken from {% cite rudin2019stop --file formal_interpretability %} used to predict whether a delinquent will be arrested again. The reasoning of the decision list is directly readable.)
{:.figcap}

For the sake of simplicity, we focus on the case of image classification and define our data points as corresponding to pixel values in an image.
We can represent any point $\bfx$ in a data set $D$ as a set $\{(1,x_1), (2,x_2), \dots, (d, x_d)\}$ where the first term corresponds to the index of the pixel and the second term is the pixel value.
The class of the data point $\bfx$ is given by $c(\bfx)$.
A feature of $x$ is then represented by a subset of $x$ which corresponds to a collection of pixels. Thus, a data point $x$ contains a feature $z$ if $z \subseteq x$.
Our setup consists of two feature classifiers Merlin, denoted by $M$, and Morgana, denoted by $\widehat{M}$.

With this in mind, we define the notion of Average Precision which will allows us to evaluate the quality of the feature selectors
and measure the performance of our framework.  

$$
\ap_{\CD}(M) := \E_{\bfx\sim \CD} \ekl{\P_{\bfy\sim\CD}\ekl{c(\bfy) = c(\bfx) \,|\, M(\bfx) \subseteq \bfy}}.
$$

The average precision $\ap_{\CD}(M)$ can be used to bound the average conditional entropy and mutual information of the features identified by Merlin.

$$
\E_{\bfx \sim \CD} [I_{\bfy\sim\CD}(c(\bfy); M(\bfx) \subseteq \bfy)] \geq H_{\bfy\sim\CD}(c(\bfy)) - H_b(\ap_{\CD}(M)).
$$

In order to evaluate the performance of Merlin and Morgana as feature selectors we need to first introduce the notions of Asymmetric Feature Correlation (AFC) and the Relative Strength of Merlin and Morgana.


### Asymmetric Feature Correlation



### Relative Strength of Merlin and Morgana






### Asymmetric Feature Correlation
AFC describes a possible quirk of datasets, where a set of features, say $\Phi$, is strongly concentrated in a few data points in one
class (called $A$) and spread out over almost all data points in another class (called $B$). Because of this quirk, Merlin can use the features that are spread over all data points to indicate class $B$. The amount of error that Morgana can then cause is limited to the few datapoints from class $A$ that also contain the features $\Phi$.
This ensures a small error even with potentially un-informative features by leveraging the distribution of the features across the classes.  
We formally define this notion as follows:

Let $(D,\CD,c)$ be a two-class data space, then the asymmetric feature correlation $\kappa$ is defined as

$$
\kappa = \max_{l\in \{-1,1\}} \max_{F \subset D_p} \E_{\bfy \sim \CD_l|_{F^*}}\ekl{\max_{\substack{\bfz \in F \\ \text{s.t. }\bfy \in \bfz}}\kappa_l(\bfz, F)}
$$

with

$$
  \kappa_l(\bfz, F) = \frac{\P_{\bfx \sim \CD_{-l}}\ekl{\bfz \subseteq \bfx \,\middle|\, \bfx \in F^*}}{\P_{\bfx \sim \CD_l}\ekl{\bfz \subseteq \bfx \,\middle|\, \bfx \in F^*}}.
$$

where

$$
F^\ast := \left\{\bfx \in D~|~ \exists~ \bfz \in F: \bfz \subseteq \bfx\right\},
$$

### Relative Strength of Merlin and Morgana
Another important metric that we care about is the relative strength of the Merlin and Morgana classifiers. This is especially important if we intend to apply our setup to real data sets where Merlin and Morgana are not able to find the optimal features are every step.
We can relax the requirement of Morgana to play optimally as long as it can find features almost as successfully as Merlin.
With this in mind, we define the notion of relative success rate as follows
Let $\mathfrak{D}=(D, \CD, c)$ be a two-class data space. Let $A\in \CA$ and $M, \morg \in \CM(D)$.
Then the relative success rate $\alpha$ of $\morg$ with respect to $A,M$ and $\mathfrak{D}$ is defined as

$$
   \alpha := \min_{l\in \{-1,1\}} \frac{\P_{\bfx\sim \CD_{-l}}\ekl{A(\morg(\bfx))=l \,|\, \bfx \in F_l^\ast}}{\P_{\bfx\sim \CD_{l}}\ekl{A(M(\bfx))=l \,|\, \bfx \in F_l^\ast}},
$$

where

$$F_{l} := \{\bfz \in D_p \,|\, \bfz\in M(D_l), A(\bfz)=l\}.$$

### Key Result

With the above two notions of Relative Strength and Asymmetric Feature Correlation in mind, we can provide a key theoretical result of this paper.

Let $\mathfrak{D}=(D, \CD, c)$ be a two-class data space with AFC of $\kappa$ and class imbalance $B$. Let $A\in \CA$, and $M, \widehat{M}\in\CM(D)$ such that $\widehat{M}$ has a relative success rate of $\alpha$ with respect to $A, M$ and $\mathfrak{D}$.
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
