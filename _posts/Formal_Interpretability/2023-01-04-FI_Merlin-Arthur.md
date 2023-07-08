---
layout: post
title:  "Formal Interpretability 2: Interactive Classification"
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
$\newcommand{\ekl}[1]{\mathopen{}\left[ #1 \right]\mathclose{}}$
$\newcommand{\E}{\mathbb{E}}$
$\renewcommand{\P}{\mathbb{P}}$
$\newcommand{\morg}{\widehat{M}}$
$\newcommand{\CA}{\mathcal{A}}$
$\newcommand{\CM}{\mathcal{M}}$

{% include abbrv.html %}

<style>
  .figcap {
    font-size: 0.9em;
  }
</style>

This post is part 2 of my series on <a href="/blog/2023/FI_start/">Formal Interpretability</a>.

*TL;DR:
We present interactive classification as an approach to define informative features without modelling the data distribution explicitly

1. Why we need an adversarial aspect to the prover
1. Abstract definition of features
1. A min-max theorem
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
**Figure 2.** An example of a decision list taken from {% cite rudin2019stop --file formal_interpretability %} used to predict whether a delinquent will be arrested again. The reasoning of the decision list is directly readable.)
{:.figcap}

If prover and verifier are purely cooperative, Merlin can decide the class and communicate it over an arbitrary code! The feature selected for this code need not have anything to do with the features that Merlin used to decide the class. See Figure 1 for an Illustration. We showed that this happens in practice in ...

However, any such strategy can be exploited by an adversarial prover (Morgana) to convince Arthur of the wrong class. The intuition is this: Let us assume the verifier accepts a feature as proof of a class that is uncorrelated with the class. Then this feature must also appear in datapoints of a different class. Morgana can then select this feature in the different class and convince the Arthur to give the wrong classification.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/strategy.svg" alt="img1" style="float:center; width:100%">
  <p style="clear: both;"></p>
</div>
**Figure 3.** An example of a decision list taken from {% cite rudin2019stop --file formal_interpretability %} used to predict whether a delinquent will be arrested again. The reasoning of the decision list is directly readable.)
{:.figcap}

Now we want to pack this intuition into theory!

### Theoretical Setup

What exactly constitutes a feature can be up for debate. Most common are features that are defined as partial input vectors, like a cutout from an image. There are more abstract definitions such as anchors {% cite ribeiro2018anchors --file formal_interpretability %}  or queries {% cite chen2018learning --file formal_interpretability %}. Here, we leave our features completely abstract and define them as a set of datapoints. This can be interpreted as the set of datapoints that contain the feature, see Figure 4 for  an illustration.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/house.svg" alt="img1" style="float:center; width:50%">
  <p style="clear: both;"></p>
</div>
**Figure 4.** An example of a feature defined in two different ways: on the left via concrete pixel values, on the right as a set of all images that have these pixel values. The set definition, however, allows to construct much more general features. We can, for example, include shifts and rotations of the pixel values, as well as other transformations, by expanding the set.
{:.figcap}

So form now on we assume that our data space $D$ comes equipped with a feature space $\Sigma \subset 2^{D}$, which is a set of subsets of $D$. In terms of precision (see <a href="/blog/2023/FI_Preliminaries/">former post</a>) we can say that a feature has high precision if it contains datapoints mostly belonging to the same class. Such a feature is highly informative of the class.

#### Definitions for Prover (Feature Selector) and Verifier (Feature Classifier)

**Feature Selector:**
For a given dataset $D$, we define a *feature selector* as a map $M:D \rightarrow \Sigma$ such that for all $\bfx \in D$ we have $ \bfx \in M(\bfx)$. This means that for every data point $\bfx \in D$ the feature selector $M$ chooses a feature that is present in $\bfx$. We call $\CM(D)$ the space of all feature selectors for a dataset $D$.

**Feature Classifier:**
We define a *feature classifier* for a dataset $D$ as a function $A: \Sigma \rightarrow \skl{-1,0,1}$. Here, $0$ corresponds to the situation where the classifier is unable to identify a correct class. We call the space of all feature classifiers $\CA$.

We can extend the definition of the precision of a feature to the expected precision of a feature selector, which will allows us to evaluate the quality of the feature selectors and measure the performance of our framework.  

$$
\ap_{\CD}(M) := \E_{\bfx\sim \CD} \ekl{\P_{\bfy\sim\CD}\ekl{c(\bfy) = c(\bfx) \,|\, M(\bfx) \subseteq \bfy}}.
$$

The expected precision $\ap_{\CD}(M)$ can be used to bound the expected conditional entropy and mutual information of the features identified by Merlin.

$$
\E_{\bfx \sim \CD} [I_{\bfy\sim\CD}(c(\bfy); M(\bfx) \subseteq \bfy)] \geq H_{\bfy\sim\CD}(c(\bfy)) - H_b(\ap_{\CD}(M)).
$$

We can now state our first result of our investigation.

#### A Min-Max Theorem:

For a feature classifier $A$ (Arthur) and two feature selectors $M$ (Merlin) and $\widehat{M}$ (Morgana) we define

$$
\label{eq:am:min_max}
 E_{M,\widehat{M},A} := \skl{x \in D\,\middle|\,
 A\kl{M(\bfx)} \neq c(\bfx) ~\odr  A\kl{\widehat{M}(\bfx)} = -c(\bfx)},
$$

which is the set of all datapoints where either Merlin cannot convince Arthur of the right class, or Morgana can convince him of the wrong class, in short, the datapoints where Arthur fails.


**Min-Max Theorem:** Let $M\in \CM(D)$ be a feature selector and let

 $$
 \epsilon_M = \min_{A \in \CA} \max_{\widehat{M} \in \CM} \,\P_{\bfx\sim \CD}\ekl{\bfx \in E_{M,\widehat{M},A}}.
 $$

 Then a set $D^{\prime}\subset D$ with $\P_{\bfx\sim \CD}\ekl{\bfx \in D^\prime} \geq 1-\epsilon_M$ exists such that for
 $$\CD^\prime = \CD|_{D^\prime}$$. we have

 $$
  \ap_{\CD^\prime}(M) = 1, \quad \text{thus}\quad H_{\bfx,\bfy\sim\CD^\prime}(c(\bfy) \;|\; \bfy \in M(\bfx)) = 0.
 $$

This means that, if Merlin and Arthur cooperate successfully (i.e. small $\epsilon_M$), then there is a set that covers almost the whole original set (up to $\epsilon_M$) conditioned on which the features selected by Merlin determine the class perfectly.

Now, the formulation of this theorem is a bit curious. Why can we not directly state something like

$$
  \ap_{\CD}(M) \geq 1-\epsilon_{M}?
$$

The problem lies in a potential quirk in the dataset that makes it hard to connect the informativeness of the whole feature set to the individual features.

### Key Insights:

1. Without an adversarial prover, Merlin and Arthur can communicate through an arbitrary code of features that are unrelated to the true class.
1. We can derive a min-max theorem that connects the completeness and soundness of Arthur and Merlin's strategy to the informativeness of the features.
1. Crucially, compared to ..., we do not rely on the unrealistic assumption that the features are uncorrelated.

### References


{% bibliography --cited --file formal_interpretability.bib --template bib2 %}
