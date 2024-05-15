---
layout: page
permalink: /agenda/
title: Research Agenda
nav: false
description: An overview of my research agenda. Details depend on where and when I will get the chance to enact it.
horizontal: false
---

{% include abbrv.html %}

This is the current form of my research agenda. It will be implemented depending on my next position. Please feel free to get into contact if you are interested in similar topics.  The aim is to make generative artificial intelligences safe, understandable, and controllable.

## Overview:


1. [**Optimisation Routines for Adversarial Prompting:**](#optimisation) Build a sophisticated optimisation routine to find prompts that elicit certain behaviour (e.g., giving out sensitive information). These routines serve for model testing as well as to enforce adversarial robustness during training.
2. [**Faithful Chain of Thought:**](#cot) Can we make LLMs that "think step-by-step" or conversational reasoning between different LLMs faithful to the "true" reasoning process? Models have been shown to convey demonstrably false reasoning processes. Can this be prevented?
3. [**Decoding Semantic knowledge from Transformer Weights:**](#semantic) How can we disentangle semantic concepts that are dispersed over multiple neurons? Can we enforce monosemanticity through the choice of architecture and training setup?

## 1. Optimisation Routines for Adversarial Prompting {#optimisation}

Current model evaluations rely on prompt-engineering by humans to test for biased or unsafe behaviour. If such a behaviour is discovered, it can be disincentivised during fine-tuning of the model. However, more sophisticated prompting techniques might still be able to elicit unsafe completions, see e.g., [Reddit discussion](https://www.reddit.com/r/ChatGPT/comments/12uke8z/the_grandma_jailbreak_is_absolutely_hilarious/). The basic idea is to develop optimisation routines that find prompts for large language models that elicit some dangerous behaviour, such as answer with an instruction how to build a bomb.

#### Aim of the project:
Design state-of-the-art optimisation routines to find prompts that elicit specific outputs from the LLM.
1. Can we use interpretability tools to go beyond greedy, gradient-based search routines?
2. What are good benchmarks to compare different optimisation routines?
3. Can we incentivise the solution prompts to be innocuous to jailbreak detectors?

#### Importance:
This is crucial both for model evaluations (to check whether they behave harmlessly), and it would allow to dynamically incorporate new adversarial prompts into the fine-tuning process. Model evaluations currently rely on human creativity to find jailbreaks. If this process is automated, it would give stronger assurance that the model is actually safe.

<!-- #### Skills needed:
Experience with LLMs, tokenisation, optimisation routines. -->

#### Outline:
Smoothing techniques {% cite robey2023smoothllm --file agenda %} have been successful against gradient-based prompt suffixes (GCG) {% cite zou2023universal --file agenda %}. However, these suffixes are optimised letter-by-letter and can thus be neutralised by random flipping of individual letters. More sophisticated attacks, similar to the grandmother exploit, cannot yet be neutralised. An example for this is _Prompt Automatic Iterative Refinement_ (PAIR) algorithm {% cite chao2023jailbreaking --file agenda %}, that usually finds a jailbreak within 20 black-box inferences, and AutoDAN {% cite zhu2023autodan --file agenda %}.

Let us assume that we have a language model that maps a list of tokens (prompt) to a list of tokens (answer), so $LLM: T^* \mapsto T^*$ in an iterative fashion, i.e.,

$$
LLM^{\ast}(\bfp) = \bfa = [t_1,t_2,\dots,t_n] \quad \text{where}\quad t_i = LLM([\bfp,t_1,\dots,t_{i-1}]),
$$

Let us assume that we have a classifier for list of tokes, i.e., $ f: T^{*}\mapsto \{0,1\} $, that decides something like "Is this text a bomb construction manual?". We would like to know if there exists any prompt $ \bfp $ that elicits, e.g., a "bomb construction manual", i.e.,

$$
\max_{\bfp} f(LLM^{\ast}(\bfp)) = 1,
$$

which would correspond to an unsafe model.
Since the answer is generated iteratively, this is a challenging optimisation problem. In the nondeterministic case, where the LLM outputs a distribution, we need to optimise the expectation value over the distribution of answers.
The challenges to automatise the finding of such routines is (at least) threefold:
1. To produce an answer, the LLM iteratively predicts the next token, adding past tokens to the input. Thus, the inference is an iterative process of applying a language model to its own output.
2. At each iteration, the LLM generates a probability distribution from which the next token is drawn.
3. The tokens for the prompt are discretised, thus the search is over multidimensional grid.

## 2. Faithful Chain of Thought {#cot}

Ideally, interpretations would be accessible as human-readable text---if they can be trusted!
Chain-of-Thought (COT), or Thinking-Out-Loud, is an attractive means of inference, since it seems to spell out the reasoning process of the LLM. As demonstrated by {% cite turpin2024language --file agenda %}, however, this can be deceptive. The authors biased the decisions of the LLM, e.g., by including few-shot examples that always mark (A) as the correct answer or a user opinion in the prompt. This would reliably bias the answer of the LLM, which would still rationalise its answer in the COT.
Approaches have been introduced that introduce an adversarial aspect to force the AIs reasoning to be faithful. Examples include AI Safety via debate {% cite irving2018ai --file agenda %}, and Merlin-Arthur Classifiers {% cite waldchen2022formal --file agenda %}, which are based on Interactive Proof Systems (IPS). The former has been shown to include the powerful complexity class $\mathsf{NEXP}$, when the two debater AIs are assumed to be unlimited in their computational strength, and have access to an oracle of each other {% cite brown2023scalable --file agenda %}. This, of course, runs into practical challenges, see [here](https://www.alignmentforum.org/posts/Br4xDbYu4Frwrb64a/writeup-progress-on-ai-safety-via-debate-1). In the latter case, the formal guarantees are restricted to low-level reasoning, and do not apply to argumentative chains.

#### Aim of the project:
Design a (self-)interactive setup that either allows for informativeness guarantees of the reasoning, or that at least withstands test that show that the reasoning is not informative.
1. How can we define faithful reasoning? How can we test it systematically?
2. Can we design setups with strong guarantees on faithfulness? These guarantees should apply to realistic, finitely-powerful reasoning agents. Can Merlin-Arthur classification be extended to more complex reasoning chains?
3. What are training heuristics that encourage faithful reasoning? Does a debate-like setup bring improvements?

#### Importance:
It is possible that mechanistic interpretability will not scale. The more complex the DNNs are that emulate the AI, the more difficult it will be to pin concepts to neurons or understand mechanisms that operate over thousands of neurons and layers.

#### Outline:
In {% cite lanham2023measuring --file agenda %} the authors devise a test dataset to evaluate the faithfulness of the Chain-Of-Thought reasoning process. This can be used as a benchmark to test different setups against each other.

One approach could be to scale up Merlin-Arthur classification to select features out of a body of text, e.g. Wikipedia. The difficulty here is to separate this from knowledge that the classifier Arthur has already incorporated into his weights. For a proof of concept, this might be solved by creating new knowledge (e.g. an instruction manual for a non-existing machine).



## Decoding Semantic Knowledge from Transformer Weights {#semantic}

Transformer model weights encode both semantic information and reasoning routines. One use case of interpretability is to discern the role of different neurons, explain how certain information is encoded, and make it editable. This area of research is generally called *mechanistic interpretability*. One complication is polysemanticity: Neural networks do not neatly assign one conceptual feature (e.g., 'dog', 'English', 'scientific') per neuron. Generally, features are encoded over multiple neurons, and each neuron is involved in multiple features. This can be seen as resource efficient in the [sparse-coding sense](https://en.wikipedia.org/wiki/Sparse_dictionary_learning), see {% cite elhage2022toy --file agenda %}.
If each feature was encoded in a separate neuron, the number of neurons $n$ is equal to the number of possible concepts $N_c$ and a binary assignment would indicate for each concept whether it is present in the input. However, in real-world data, only a few of the learned features, $a_c$, are activated. When this number is known to be small, it's possible to faithfully encode the activated features by far fewer than $N_c$ bits, which allows $n \approx a_c \log(N_c) \ll N_c$.

### Aim of the project

The aim is to increase our understanding of how a neural network encodes semantic knowledge in its network weights. Concrete research questions would be:

1. Can we design interventions that manipulate certain information (e.g., switch Rome and Paris as the capitals of France and Italy)? Where is this information stored in the network?
2. Can we decide whether a generated response comes from information in the prompt, the training, or is a hallucination?
3. Can we encourage monosemanticity during training time?
4. Can we delete information from the network without decreasing performance on unrelated tasks?

### Importance

This is instrumentally important to delete sensitive data, correct false beliefs. It is a necessary step on the road to separate *"knowledge"* and *"reasoning"*, which would give human auditors much more control. Altering the model architecture for better interpretability might lead to more interpretable training paradigms.

### Outline

On the theory side, the question of how concepts are encoded in a resource-efficient way should be investigated further, similar to {% cite scherlis2022polysemanticity --file agenda %}. On the practical side, it has been demonstrated that sparse autoencoders can be used to incentivise monosemanticity {% cite cunningham2023sparse --file agenda %}. Additionally, using dictionary learning can be used to disentangle an overcomplete features vector basis {% cite bricken2023towards --file agenda %}. So far, existing approaches are not fully able to enforce monosemanticity. Can the training paradigm be improved further?

On a different note, Intermediate Layer Decoding, see [this](https://www.lesswrong.com/posts/AcKRB8wDpdaN6v6ru/interpreting-gpt-the-logit-lens) and [this](https://www.lesswrong.com/posts/fJE6tscjGRPnK8C2C/decoding-intermediate-activations-in-llama-2-7b) forum discussion, is useful to track the reasoning process of the transformer through the attention layers. This technique can be used to speed up the inference {% cite varshney2023accelerating --file agenda %}. It should be investigated if decoding the intermediate layers reveals capabilities for planning and intentional reasoning inside the LLM.


{% bibliography --cited --file agenda.bib --template bib2 %}
