---
layout: post
title:  "Formal Interpretability 1: Preliminaries"
date:   2023-05-09 00:00:00 +0100
categories: [research]
tags: [interpretability, feature importance attribution, fia, modelling problem]
math: true
comments: true
author: Stephan Wäldchen
---



**TL;DR:**
We present an overview of the current approaches and hurdles for formal interpretability:
1. Feature Importance Attribution
1. Shapley values and maximum mutual information
1. The modelling problem and manipulating explanations
1. Computational complexity of these approaches
<!--more-->

<!--- *Written by Stephan Wäldchen.* -->

### Interpretable AI

An interpretable AI systems allows the human user to understand its reasoning process. Examples are decision trees, sparse lnear models and $$k$$-nearest neighbors.

The standard bearer of modern machine learning, the **Neural Networks**, while achieving unprecedented accuracy, is nevertheless considered a **black box**, which means its reasoning is not made explicit.
While we mathetically understand exactly what is happening in a single neuron, the interplay of thousands of these neurons results in behaviour that cannot be predicted straigtforward way {% cite signaltrain %}.
Compare this with how we exactly understand how an AND-gate and a NOT-gate work and how each program of finite length can be expressed as a series of these gates, yet we cannot understand a program just from reading the cuircuit plan.

Interpretability research aims to remedy this fact by accompanying a decision, e.g. such as a classification, with addititional information that describes the reasoning process.

One of the most prominent approaches is feature importance maps, which, for a given input, rate the input features for their imporatance to the model output.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/decision_list.png" alt="img1" style="float:center; margin-right: 5%; width:95%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** An example of a decision list taken from [Rudin2019] used to predict whether a delinquent will be arrested again. The reasoning of the decision list is directly readable.)
{:.figcap}

### Feature Importance Attribution

For a given classifier and an input, **feature importance attribution** (FIA) or feature importance map aims to highlight what part of the input is relevant for the classifier decision on this input. The idea is that generally only a small part of the input is actually important. If for example a neural network decides whether an image contains a cat or a dog, only the part of the image displaying the respectiv animal should be considered important.
This consideration omits in which way the important features were considered. It can thus be seen as the **lowest level** of the reasoning process.

There are quite a lot of practical approaches that derive feature importance values for neural networks, see []. However, these methods are defined purely **heuristically**. They come without any defined target properties for the produced attributions. Furthermore, it has been demonstrated that these methods can be **manipulated** by clever designs of the neural network.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/lrp_example.png" alt="img1" style="float:center; margin-right: 5%; width:80%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** Feature attribution map generated with LRP for a Fisher Vector Classifier (FV) and a Deep Neural Network (DNN). One can see that the FV decides the boat class based mostly on the water. Will this classifier generalise to boats without water? From [Lapuschkin2016])
{:.figcap}

### Manipulation of Heuristic Feature Importance

We are talking about manipulations in the follwing sense: Given that I have a neural network classifier $$\Phi$$ that performs well for my purposes, I want another classifier $$\Phi^\prime$$ that performs equally well but with a completely arbitrary feature importance.

These heuristic FAMs all make implicit assumptions on the data distribution (some of them do that in a layer-wise fashion), see [Lundberg2017].

All these heuristic explanation methods can be manipulated with the same trick: Basically keep the on-manifold behaviour constant, but change the off-manifold behaviour to influence the interpretations.

Example from [Slack2020]: $$\Phi$$ is a discriminatory classifier, $$\Psi$$ is a completely fair classifier and there is a helper function that decides if an input is on-manifold, belongs to a subspace of typical sample $$\mathcal{X}$$. They define

$$ \Phi^\prime(\mathbf{x}) =
\begin{cases}
 \Phi(\mathbf{x}) & \text{if}~ \mathbf{x}\in \mathcal{X}, \\
 \Psi(\mathbf{x}) & \text{otherwise.}
\end{cases}
$$

Now $$\Phi^\prime$$ will almost always discriminate since for $$\mathbf{x}$$ that lie on the manifold, whereas the explanations will be dominated by the fair classifier $$\Psi$$, since most samples for the explanations are not on manifold. Thus the FAM highlights the innocuous features instead of the discriminatory ones.

<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/off-manifold.png" alt="img1" style="width:50%">
</div>
**Figure 1.** On-manifold data samples (blue) and off-manifold LIME-samples (red) for the COMPAS dataset; from [Dimanov2020].
{:.figcap}

Formal approaches to interpretability thus need to make the underlying data distribution explicit.

### Formal definition of feature importance

There are two main approaches to feature attribution:
1. Shapley values
1. Prime Implicants
1. Maximal Mutual Information

**Shapley Values** are a value attribution method from cooperative game theory. It is the unique are the unique method that
satisfies the following desirable properties: linearity, symmetry, null player and efficiency (Shapley, 2016). The idea is that a set of players achieve a common value. This value is to be fairly distributed to the players according to their importance. For this one considers every possible subset of players, called a coalition and the value this coalition would achieve.
Thus, to define Shapley Values one needs a so called *characteristic function*, a value function that is defined on a set as well as all possible subsets.
For $$d$$ players, let $$\nu: 2^{[d]} \rightarrow \mathbb{R}$$. Then $$\phi_i$$, the Shapley value of the $$i$$-th player is defined as

$$
    \phi_{\nu,i} = \sum_{S \subseteq [d]\setminus\{i\}} \begin{pmatrix} d-1 \\ |S| \end{pmatrix}^{-1} ( \nu(S \cup \{i\}) - \nu(S) ).
$$

Thus the Shapley values sum over all marginal contributions of the $$i$$-th player for ever possible coalition.
In machine learning, the players correspond to features and the coalitions to subsets of the whole input. The explicite training of a characteristic function has been used in the context of simple two-player games to compare with heuristic attribution methods in [].
However, generally in machine learning, the model cannot evaluate subsets of inputs. For a given input $$\mathbf{x}$$ and classification function $$f$$, [] define $$\nu$$ over expectation values:

$$
    \nu_{f,\mathbf{x}}(S) = \mathbb{E}_{\mathbf{y}\sim \mathcal{D}}[f(\mathbf{y})\,|\, \mathbf{y}_S = \mathbf{x}_S ] = \mathbb{E}_{\mathbf{y}\sim \mathcal{D}|_{\mathbf{x}_S}}[f(\mathbf{y})].
$$


<div style="display: flex; justify-content: center;">
  <img src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/Shapley.png" alt="img1" style="float:center; margin-right: 1%; width:100%">
  <p style="clear: both;"></p>
</div>
**Figure 1.** Illustration of the idea of Shapley Values. For three players the pay-off for each possible coalition is shown. [Source](https://clearcode.cc/blog/game-theory-attribution/)
{:.figcap}


**Prime Implicants**

A series of appraoches considers hwo much a subset of the features of $$\mathbf{x}$$ already determine the function output $$f(\mathbf{x})$$. One of the most straight-forward approaches are the prime implicant explanations [D] for Boolean classifiers. An implicant is a part of the input that determines the output of the function completely, no matter which value the rest of the input takes. A prime implicant is an implicant that cannot be reduced further by omitting features.

This concept is tricky to implement for very the highly non-linear neural networks, as small parts of an input can often be manipulated to give a completely different classification, see []. Thus, prime implicant explanations need to cover almost the whole input, and are thus not very informative.

Probabilistic prime implicants have thus be introduced. As a relaxed notion, they only require the implicant to determine the function output with some high probability $$\delta$$. For continuously valued fucntions $$f$$ this can be further relaxed to being close to the original value in some fitting norm. One is then often interested in the most informative subset of a given maximal size:

$$
 S^* = \text{argmin}_{S: |S|\leq k} D_{f,\mathbf{x}}(S) \quad \text{where} \quad D_{f,\mathbf{x}}(S) = \|f(\mathbf{x}) - \mathbb{E}_{\mathbf{y}\sim \mathcal{D}|_{\mathbf{x}_S}}[f(\mathbf{y}) ]\|
$$

In the language of Shapley values, we are looking for a small coalition that already achieves a value close to the whole set of players.
There is a natural trade-off between the maximal set size $$k$$ and the achievable distortion $$D(S^*)$$.

This concept can be refined without the arbitrariness of the norm by taking the mutual information.

**Maximal Mutual Information**

Mututal information measures the mutual dependence between two variables. In the context of the inpt features, it can be defined for a given subset S as as

$$
 I_{\mathbf{x} \sim \mathcal{D}}[f(\mathbf{x}); \mathbf{x}_S] = H_{\mathbf{x} \sim \mathcal{D}}[f(\mathbf{x})] - H_{\mathbf{x} \sim \mathcal{D}}[f(\mathbf{x}) ~|~ \mathbf{x}_S],
$$

where $$H_{\mathbf{x} \sim \mathcal{D}}[f(\mathbf{x})]$$ is the a priori entropy of the classification decision and $$H_{\mathbf{x} \sim \mathcal{D}}[f(\mathbf{x}) ~|~ \mathbf{x}_S]$$ is the conditional entropy given the input set.
When the conditional entropy is close to zero, the mutual information takes its maximal value as the pure a priori entropy.

$$
 H_{\mathbf{x} \sim \mathcal{D}}[f(\mathbf{x}) ~|~ \mathbf{x}_S] = - \sum_{l} p_l \log(p_l) \quad \text{where} \quad p_l = \mathbb{P}_{\mathbf{y} \sim \mathcal{D}}[f(\mathbf{y}) = l ~|~ \mathbf{y}_S = \mathbf{x}_S],
$$

where $$l$$ runs over the domain of $$f$$.
Similarly to the prime implicant explanations, one is often interested to find a small subset of the input that ensures high mutual information with the output:

$$
 S^* = \text{argmax}_{S: |S|\leq k} I_{\mathbf{x} \sim \mathcal{D}}[f(\mathbf{x}); \mathbf{x}_S].
$$

### The modeling Problem

All three presented methods to calculate the conditional probabilities $$ \mathcal{D}_{\mathbf{x}_S}$$ for all subsets $$S$$ in question. For synthetic datasets these probabilities can be known, for realistic datasets however, these probabilities require explicit modeling of the conditional data distribution. This has been achieved practically with variational autoencoders [] or generative adversarial networks. Let us call these approximations
$$\mathcal{D^\prime}|_{\mathbf{x}_S}$$.

## Practical Problems

There are basically two practical approaches to the modelling problem. The first is taking a simplified distribution that is independent of the given features:
 \[
 \P_{\bfy\sim\CD}(\bfy_{S^c} ~|~ \bfy_S = \bfx_S) = \prod_{i \in S^c} p(y_i).
 \]
This has been the approach taken for example in [Fong2017],[MacDonald2019] and [Ribeiro2016]. The problem here is that for certain masks this can create features that are not there in the original image, see Figure 4 for an illustration. This can actually happen even when unintended in case of an optimiser solving for small distortion $$D_{f,\mathbf{x}}$$, as shown in Figure 4.


<div style="display: flex; justify-content: center;">
  <embed src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/bird_mask.png" alt="img1" style="float:center; margin-right: 1%; width:50%">
  <p style="clear: both;"></p>
</div>
**Figure 4.** The optimised mask to convince the classifier of the (correct) bird class constructs a feature that is not present in the original image, here a bird head looking to the left inside of the monocrome black wing of the original; from [Macdonald2021].)
{:.figcap}



<div style="display: flex; justify-content: center;">
  <embed src="{{site.url }}{{site.baseurl }}/assets/img/merlin_arthur/failures.png" alt="img1" style="float:center; margin-right: 1%; width:50%">
  <p style="clear: both;"></p>
</div>
**Figure 5.** Illustration of the idea of Shapley Values. For three players the pay-off for each possible coalition is shown. [Source](https://clearcode.cc/blog/game-theory-attribution/)
{:.figcap}

## Theoretical Problems

$$
D_{\text{KL}}(\mathcal{D}^\prime, \mathcal{D})
$$



Since we want a formal approach with a bound on the calculated Shapley values, distortion or mutual information, we need a distance bound between
$$\mathcal{D^\prime}|_{\mathbf{x}_S}$$ and
$$\mathcal{D}|_{\mathbf{x}_S}$$ in some fitting norm, e.g. the total variation or Kullback-Leibler divergence. This is hard to achieve, since to establish such bounds one would need many samples of the dataset conditions on each of the exponentially many subsets.

Taking any image $$\mathbf{x}$$ from Imagenet for example and conditioning on a subset $$S$$ of pixels, there probably exists no second image with the same values on $$S$$ when the set size is larger than 20. These conditional distributions thus cannot be sampled in most high-dimensional datasets and no quality bounds can be derived.

In the next post we discuss how this problem can be overcome by replacing the modeling of the




### Computational Complexity






### References

[Rudin2019] Rudin, Cynthia. "Stop explaining black box machine learning models for high stakes decisions and use interpretable models instead." Nature machine intelligence 1.5 (2019): 206-215. [pdf](https://www.nature.com/articles/s42256-019-0048-x)

[Lapuschkin2016] Lapuschkin, Sebastian, et al. "Analyzing classifiers: Fisher vectors and deep neural networks." Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2016. [pdf](https://openaccess.thecvf.com/content_cvpr_2016/papers/Bach_Analyzing_Classifiers_Fisher_CVPR_2016_paper.pdf)

[Lundberg2017] Lundberg, Scott M., and Su-In Lee. "A unified approach to interpreting model predictions." Advances in neural information processing systems 30 (2017) [pdf](https://proceedings.neurips.cc/paper/2017/file/8a20a8621978632d76c43dfd28b67767-Paper.pdf)

[Slack2020] Slack, Dylan, et al. "Fooling lime and shap: Adversarial attacks on post hoc explanation methods." Proceedings of the AAAI/ACM Conference on AI, Ethics, and Society. 2020. [pdf](https://dl.acm.org/doi/pdf/10.1145/3375627.3375830)

[Dimanov2020] Dimanov, Botty, et al. "You shouldn’t trust me: Learning models which conceal unfairness from multiple explanation methods." (2020). [PDF](https://www.repository.cam.ac.uk/bitstream/handle/1810/301751/FAIA-325-FAIA200380.pdf?sequence=4)

[Heo2019] Heo, Juyeon, Sunghwan Joo, and Taesup Moon. "Fooling neural network interpretations via adversarial model manipulation." Advances in Neural Information Processing Systems 32 (2019). [PDF](https://proceedings.neurips.cc/paper/2019/file/7fea637fd6d02b8f0adf6f7dc36aed93-Paper.pdf)

[Anders2020] Anders, Christopher, et al. "Fairwashing explanations with off-manifold detergent." International Conference on Machine Learning. PMLR, 2020. [pdf](http://proceedings.mlr.press/v119/anders20a/anders20a.pdf)

[Fong2017] Fong, Ruth C., and Andrea Vedaldi. "Interpretable explanations of black boxes by meaningful perturbation." Proceedings of the IEEE international conference on computer vision. 2017. [pdf](https://openaccess.thecvf.com/content_ICCV_2017/papers/Fong_Interpretable_Explanations_of_ICCV_2017_paper.pdf)

[MacDonald2019] MacDonald, Jan, et al. "A rate-distortion framework for explaining neural network decisions." arXiv preprint arXiv:1905.11092 (2019). [pdf](https://arxiv.org/pdf/1905.11092)

[Ribeiro2016] Ribeiro, Marco Tulio, Sameer Singh, and Carlos Guestrin. "Model-agnostic interpretability of machine learning." arXiv preprint arXiv:1606.05386 (2016). [pdf](https://arxiv.org/pdf/1606.05386.pdf?source=post_page)

[Macdonald2021] Macdonald, Jan, Mathieu Besançon, and Sebastian Pokutta. "Interpretable neural networks with frank-wolfe: Sparse relevance maps and relevance orderings." arXiv preprint arXiv:2110.08105 (2021). [pdf](https://arxiv.org/pdf/2110.08105)

{% bibliography --cited %}
{% bibliography file:references.bib %}

{% reference ruby, /_bibliogaphy/references.bib %}
