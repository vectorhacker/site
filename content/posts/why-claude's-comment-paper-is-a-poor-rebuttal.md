---
title: "Why Claude's Comment Paper Is a Poor Rebuttal"
date: 2025-06-15T19:39:41-04:00
draft: false
toc: false
images:
tags:
  - ai
  - llm
  - reasoning
---

Recently Apple [published a paper](https://machinelearning.apple.com/research/illusion-of-thinking) on LRMs (Large Reasoning Models) and how they found that "that LRMs have limitations in exact computation" and that "they fail to use explicit algorithms and reason inconsistently across puzzles." I would consider this a death blow paper to the current push for using LLMs and LRMs as the basis for AGI. Subbaro Kambhampati and Yann LeCun seem to agree. You could say that the paper [knocked out LLMs](https://garymarcus.substack.com/p/a-knockout-blow-for-llms). More recently, a comment paper showed up on [Arxiv](https://arxiv.org/html/2506.09250v1) and shared around X as a rebuttal to Apple's paper. Putting aside the stunt of having Claude Opus as a co-author (yes, I'm not kidding), the paper in itself is a poor rebuttal for many reasons which we shall explore, but mainly for missing the entire point of the paper and prior research by AI researchers such as Professor [Kambhampati](https://cotopaxi.eas.asu.edu/).

## Mathematical Errors

Firstly the paper makes some *key* mathematical errors. As [Andreas Kirsch](https://x.com/BlackHC) points out on X, it makes the claim that token growth is predicted by the following:

$$
T(N) \approx 5(2^N - 1)^2 + C
$$

namely, it predicts a quadratic token growth for solutions of [Towers of Hanoi](https://en.wikipedia.org/wiki/Tower_of_Hanoi). The reality is that the growth of tokens is linear regardless of the number of steps required to solve. In fact, Gemini 2.5 Pro outputs a solution in under 10k tokens for \(n=10\) discs.

See Andreas's post on X:

{{<x user="BlackHC" id="1933442363197706658" >}}

## Confused Between Mechanical Execution and Reasoning Complexity

The rebuttal conflates solution length with computational difficulty. As Kirsch points out, different puzzles have vastly different complexity profiles:

- **Tower of Hanoi**: Requires \(2^N-1\) moves, but has branching factor 1 and requires no search - just mechanical execution with "trivial \(O(1)\) decision process per move"
- **River Crossing**: Requires ~4N moves, but has branching factor >4 and is NP-hard, requiring complex constraint satisfaction and search

This explains why models might execute 100+ Hanoi moves while failing on 5-move River Crossing problems - the latter requires genuine reasoning while the former is mechanical execution.

## Contradictions

The rebuttal paper's own data contradicts its thesis. Its own data shows that models can generate long sequences when they choose to, but in the findings of the original Apple paper, it finds that models *systematically* choose **NOT** to generate longer reasoning traces on harder problems, effectively just giving up. It's not explained why models would self-impose token limits that hurt their performance. In the original study's findings it says "Despite operating well below their generation length limits with ample inference budget available, these models fail to take advantage of additional inference compute" (Shojaee et al., p.8).

In the rebuttal's experiments it shows that it can "Solve Tower of Hanoi with 15 disks. Output a Lua function that prints the solution when called. Results: Very high accuracy across tested models (Claude-3.7-Sonnet, Claude Opus 4, OpenAI o3, Google Gemini 2.5), completing in under 5,000 tokens" (p.3) which directly contradicts its original claim that models are fundamentally constrained by token limits, because this proves that models ARE able to solve complex problems efficiently given appropriate output formats.

The claims in this rebuttal paper create a paradox. If models can solve hard problems efficiently, and models recognize when to truncate output, then why do they systematically choose inefficient approaches that lead to failure? This supports the original study's thesis about reasoning limitations, not the constraint thesis put forth in the rebuttal.

## Completely Missing the Point and Narrow Focus

The main point of Apple's paper was to identify systematic reasoning patterns, not evaluate accuracy. Indeed, they point out that reasoning benchmarks thus far have inherent limitations because of their leakiness and narrow focus on final solutions, rather than on the general process of reasoning.

> Reasoning models initially increase their thinking tokens proportionally with problem complexity. However, upon approaching a critical threshold—which closely corresponds to their accuracy collapse point—models counterintuitively begin to reduce their reasoning effort despite increasing problem difficulty" (Shojaee et al., p.8)

It instead completely ignores this finding and offers no explanation as to why models would systematically reduce computational effort when faced with harder problems. This suggests fundamental scaling limitations with current LRM architectures.

Further, the rebuttal does not address the Apple paper's complexity regime patterns that it identified consistently across models or even how token limits explain these.

## Conclusions and Further Reading

Concluding, this comment paper fails to address the key findings of the original paper and focuses too narrowly on contradicting elements. It fundamentally misses the point of Apple's paper, and indeed other papers including by Subbaro Kambhampati, and most recently Georgia Tech, which identify *fundamental* limitations with current LLM architectures.

I suggest reading up further on [Kambhampati's research](https://scholar.google.com/scholar?hl=en&as_sdt=0%2C5&q=subbarao+kambhampati+reason+llm&btnG=) into LLM reasoning to get a better idea of why LLM/LRMs are fundamentally limited in reasoning ability. Also read [Gary Marcus's blog](https://garymarcus.substack.com/p/llms-dont-do-formal-reasoning-and) on Substack and the Georgia Tech paper [here](https://arxiv.org/pdf/2506.07936).
