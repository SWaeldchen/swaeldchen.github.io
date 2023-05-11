---
layout: page
title: Certifiable Game Plans
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
However, we can extend the discussion with the AI up to polynomially many rounds, which results in the complexity class $\mathsf{IP}$ or *Interactive Proofs*. Coincidentally, $mathsf{IP} is exactly equal to $\mathsf{PSPACE}$.
