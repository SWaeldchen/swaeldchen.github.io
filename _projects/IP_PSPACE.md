---
layout: page
title: Arithmetic Reinforcement Learning and Certifiable Game Plans
description: Can we certify the plan of a powerful but possibly adversarial AI?
img:
importance: 1
category: Alignment
---


This is my idea for a new AI alignment project. Basically, I want to answer the followign question:

<blockquote style="text-align:center; font-style:italic; color:#333;">
  "Can a more powerful reasoner convince a human auditor of the validity of a plan, even in a complex environment?"
</blockquote>

To make this work, we need to exploit some principle in the sense of "It's easier to recognise that a proof is correct than to come up with a proof." The first problem is $\mathsf{NP}$-hard, the latter can be solved in polynomial time.

This point was recently raised by Max Tegmark in his Lex Fridman [interview](https://www.youtube.com/watch?v=VcVfceTsD0A&t=1h44m30s).

In case of using AI as theorem provers it is quite straightforward to ensure that it cannot convince a human of a wrong theorem. Foundations of mathematics, e.g. Zermelo-Fraenkel (+ axiom of choice), are a system of deterministic rules, called axioms. The AI can present a reduction of a theorem, for example for $\mathsf{P}\neq\mathsf{NP}$, as a list of intermediate theorems accompanied by the rule that allows one theorem to be derived by the next until it leads back to one fundamental axioms. It is straightforward to check that each rules was applied correctly in each reduction step.

For most real-world plans there are complications however:
1. Most transitions between world states are non-deterministic
1. The world contains other intelligent agents, notably the AI itself.
1. The world model of the AI (the transition rules), might be more accurate than the ones known to the human auditor.
1. It is unclear how long the time horizon is that should be modelled.

At least for the first two problems I can see a way towards a solution. So from now on, we assume that we are have perfect knowledge of how the environment works and only care about the result of the plan up to some finite time $T$.

Two-player games (or multiplayer), even with probabilistic aspects, are $\mathsf{PSPACE}$-complete. As far as we can tell, $\mathsf{PSPACE}$-complete problems are not in $\mathsf{NP}$ and will now allow for a polynomially sized certificate.
However, we can extend the discussion with the AI up to polynomially many rounds, which results in the complexity class $\mathsf{IP}$ or *Interactive Proofs*. Coincidentally, $\mathsf{IP}$ is exactly equal to $\mathsf{PSPACE}$.

This equality is usually demonstrated by reducing TQBL (Totally Quantified Boolean Formulas) to PSPACE, see [wikipedia](https://en.wikipedia.org/wiki/IP_(complexity)#TQBF_is_a_member_of_IP). This reduction involves as a crucial step [Polynomial Identity Testing (PIT)](https://en.wikipedia.org/wiki/Polynomial_identity_testing) via the [Schwartz-Zippel algorithm](https://en.wikipedia.org/wiki/Schwartz%E2%80%93Zippel_lemma) (basically sample randomly over a large-enough finite field).

One can easily transform deterministic two-player games into TQBFs, for non-deterministic games I am not fully sure, but it should be possible, see [here](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjBh_md2O3-AhXB4KQKHSpDA9g4FBAWegQICRAB&url=https%3A%2F%2Fwww.fi.muni.cz%2Fusr%2Fkucera%2Fpapers%2Fqest07.ps&usg=AOvVaw26TwTX8w9Em1wC-UHHo5Wu) and [here](https://research-explorer.ista.ac.at/download/3846/5897/a_survey_of_stochastic_omega-regular_games.pdf).

This TQBF can then be decided via the Interactive Proof reduction. Interestingly, the PIT consistency condition becomes very similar to a arithmetised version of the [Bellman equation](https://en.wikipedia.org/wiki/Bellman_equation). I will add further details to illustrate that point later.

I want to actually implement this for an AI trained with Reinforcement Learning and Gradient Descent.

**Implementation:**
1. Use a simple deterministic game with finite time horizon (e.g. tic-tac-toe, Connect-Four, Hex)
1. Train an AI to not only predict the whether the player can still win given a board situation. Extend this AI to predict the polynomial coefficients for the polynomial identity test, given the previous random samples of the $\mathsf{IP}$-protocol as input.
1. See how well the AI can be trained to actually fulfil the consistency checks.

**Questions:**
1. Can a neural network architecture learn to predict the right polynomial coefficients? Since the coefficients are from a finite field, there is some room to correct errors.
1. Can the whole methods be made robust for a system that makes mistakes sometimes? Could this involve relaxing the consistency condition?
1. Can this be scaled up to more compled games?
1. How can we extend this to non-deterministic games?

**Reasons why I believe this might work:**
1. Neural networks have become quite good at grokking abstract reasoning tasks,
1. As the reasoning capabilities of AI grows, certifying the game plan will become easier, even if the AI comes up with more complicated plans, since it becomes easier to implement the PIT.
