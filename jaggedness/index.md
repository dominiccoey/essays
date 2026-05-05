---
layout: page
title: "On Jaggedness"
permalink: /jaggedness/
---
<style>
.post-content > ul > li + li {
  margin-top: 1em;
}</style>
#### May 4, 2026 


Jaggedness is AI's pattern of pronounced task-dependent strengths and weaknesses relative to humans. Some observations:

- **Jaggedness shapes outlooks.** Some prefer to evaluate AI based on best-case performance, others on worst-case. How one weighs brilliance against idiocy leads to different outlooks on the level and growth of AI capabilities. 
  - *Levels*. We could imagine all combinations of `{evaluate based on min performance, evaluate based on max performance}` x `{pessimist on AI capabilities, optimist on AI capabilities}`. For example, pessimists might steel-man their case by arguing that even given what it’s best at, AI will not be impactful. In practice, emphasis on best vs worst-case performance seems highly correlated with overall views on capabilities.
  - *Growth*. Every model so far makes basic mistakes on some tasks, so worst-case evaluators see slow progress. The peak of performance is growing continuously, so best-case evaluators see fast progress. The gap between best and worst-case performance is also growing: the ceiling keeps rising and the floor remains—for now—well below human-level.
- **Sometimes best-case performance matters more, sometimes worst-case.**
  - Best-case performance matters more if one success is enough. This is true if AI is scaffolded to focus on a single high-value task where one success means the whole project succeeds.
    - Example: Scientific work with cheap verification. A single correct proof or new drug/material with the desired properties resolves the posed research problem.
  - Worst-case performance matters more if one failure can be catastrophic. This is true if AI is operating over a wide domain and a mistake can cause irreversible harm.
    - Example: Chatbot safety. A single case of AI-induced psychosis can have terrible consequences.
  - In some cases outcomes depend on performance across all tasks. Minimizing accidents from self-driving benefits from both raising the ceiling (superhuman signal processing) and the floor (avoiding mistakes humans would never make).
- **High jaggedness makes worst-case evaluation misleading.** With high jaggedness, when worst-case performance matches humans, average and best-case performance will be much beyond human-level. An emphasis on worst-case performance is justified as conservative or cautious in some settings, e.g. minimax statistical decision theory. Here it is a lagging indicator, closer to the opposite of caution.
- **Jaggedness and scaffolding jointly determine economic benefits.** Here’s a toy model.
  - Consider an AI skill vector across $n$ tasks $(q^{AI}_1,\ldots,q^{AI}_n)$ and a human skill vector
$(q^{H}_1,\ldots,q^{H}_n)$.
  - Production is [O-ring](https://en.wikipedia.org/wiki/O-ring_theory_of_economic_development): output is the product of the task-specific productivities. Interpret _scaffolding_ as a costly investment to isolate AI contributions to a specific subset of tasks, as opposed to having either a human or “off-the-shelf” AI do everything. If scaffolding is sufficiently cheap, we choose the best option task-by-task, and achieve maximal output $\prod_{i=1}^n \max \\{q^{H}_i, q^{AI}_i\\}$.
  - If scaffolding is cheap, high jaggedness is good, since tasks AI is bad at will be done by humans. We can reap substantial economic gains well before worst-case performance exceeds human level.
  - Scaffolding may be cheap if AI can scaffold itself and send a task to a human exactly when the human is better. **An AI that can predict when to use AI is economically valuable**.
  - Some [messy jobs](https://substack.com/home/post/p-195270546) may be hard to neatly decompose into separable tasks. Perhaps the best we can do is send bundles of tasks to either humans or AI, rather than individual tasks. Then economic gains will depend on how these bundles relate to relative productivity. If within-bundle variance in AI performance is small, gains remain large. If each bundle mixes performance peaks and troughs, gains will be limited. “I can’t trust AI here—it’s sometimes better but it also makes bad mistakes.”
  - Job messiness is itself endogenous. **If AI can enable finer levels of unbundling, output increases further**. Existing job definitions reflect pre-AI economic forces. Until worst-case AI performance exceeds human-level, there may be large economic incentives to make jobs less “messy” and more decomposable.
- **Jaggedness and autonomy.**
  - Agentic AI requires reliably executing tasks in sequence. In the O-ring setting, we can think of the $q_i$’s being the probability of successfully completing task $i$. Roughly, AI is low-agency if any of the tasks in the sequence has a low success rate relative to humans. By the same logic above driving job restructuring, we should expect the construction of new task sequences on which AI is superhumanly agentic, where each of the tasks has high success probability relative to humans.
- **Jaggedness and safety.**
  - Humans are of course jagged relative to each other. The scary feature of AI is that it might be jagged in an alien way which our institutions are not built to handle. Being good at explaining bioweapons and bad at evaluating moral intent is a dangerous combination.
- **Why does jaggedness exist?**
  - The standard answer: humans and AI have different hardware specs and different inductive biases. Human minds have evolved to excel at tasks which were rewarded in our ancestral environment. But AI has shown steady progress at matching or surpassing human-level abilities on skills which our ancestral environment rewarded: facial recognition, language production, theory of mind, social cognition.
  - What does this mean for science?
    - Plausibly, scientific reasoning ability is another skill which experienced positive selective pressure in our ancestral environment, and our capacity for science is due to this. This includes matters of taste, judgment, and honed intuition about promising research directions. If those other skills selected for in our ancestral environment are being matched and surpassed by AI, why would science be any different?
    - Science might not even be the last task to fall. We might well build an AI which can [replicate Einstein’s annus mirabilis](https://www.ycombinator.com/library/P3-how-to-build-the-future-demis-hassabis) using a 1904 knowledge cutoff, but which remains outmatched by the best human in enterprise SaaS sales.
