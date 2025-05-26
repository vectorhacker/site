---
title: Software Engineers Are Safe... For Now
date: 2024-12-25
draft: false
toc: false
images:
tags:
  - computer science
  - tech
  - ai
---
Take a look at this search of jobs postings at OpenAI for December 25th, 2024. As of the time of this writing, over 150 results in all, 87 results for the term "Software Engineer." This is a few days after the release of their latest frontier model, o3. This tells me that for the foreseeable future, Software Engineering jobs are safe, if not going to continue to be safe from full automation.

![LinkedIn Job Postings for OpenAI Dec 25, 2024](/images/linkedin-job-posts-openai.png)

Many of these roles require several years of experience developing products, proficiency in programming languages and frameworks like JavaScript, React, and Python, standing up infrastructure, proficiency with SQL, and some interest in AI/ML. This tells me that not only are they still looking to hire people who are experts in their respective fields, but that human work is still favorable to machine's for at least the next several years.

Are these engineers using ChatGPT and AI coding assistants? Yes, absolutely, as is and will every other engineer in the near future! Does this mean they don't have be experts in the tech they're working on? Quite the contrary, if anything it means they need to be more deeply familiar with the tech, not less.

## Cost Prohibitive

There's evidence to suggest that while these advancements in reasoning models are amazing, they come at [prohibitive cost, even for tech giants like Microsoft. ](https://www.barrons.com/articles/openai-o3-model-cost-chatgpt-microsoft-ca040997) And so we may just be squeezing what's left out of the current generation of LLMs and are waiting for hardware to improve exponentially. For the time being, the addition of "reasoning" to these models, while improving outputs, adds time to the output, from seconds to minutes for complex questions. With regards to full compute models, it costs nearly $3500 per task and takes ~14 minutes to complete each one for the full-reasoning model on ARC-AGI.

What's more, is that the higher compute models are [making costs less predictable. ](https://techcrunch.com/2024/12/23/openais-o3-suggests-ai-models-are-scaling-in-new-ways-but-so-are-the-costs/) In all, o3 blows past other models in arc-agi at a cost of over $1000 per task, and that's at log scale. It's all to suggest that these models wouldn't be used to solve tasks that humans can do much more cheaply, even with the help of lesser ai assistants, such as programming and software engineering. No, these models will probably be let loose on problems of much higher import and where the cost of compute is no object or at least less of a barrier. Reason being, is feedback loops. Software engineering requires quick iterative feedback loops that could be lost when using these slower reasoning systems.

## Humans Can Still Reason Faster

While these models can allegedly beat human programmers in coding competitions and somewhat match humans in software engineering tasks according to [swe-bench](https://www.swebench.com/), for day to day tasks, humans still reason faster and with enough accuracy whilst still being much cheaper to maintain. For the same cost you could probably pay a couple or even 10s of humans to solve real world software engineering problems, get feedback faster, and get a product out faster. Now give an experienced human software engineer access to a coding assistant like copilot or editor like Cursor and they're flying.

## Humans Still Generalize and Adapt Better

While these models are impressive in their reasoning ability and that they can now adapt to some novel situations, there's [evidence that they're brittle and very much dependent on the patterns they were trained on. ](https://machinelearning.apple.com/research/gsm-symbolic) Not just failing, but dropping performance significantly, something a human that is sufficiently intelligent does not do. What's more, this drastic failure in performance [compounds on itself and lead these models to be extra vulnerable to reasoning chain attacks.](https://arxiv.org/html/2412.11934v1) While you could argue that a human could compound their errors in a similar way, we'd at least have a chance at recognizing out compounding errors early on especially in software engineering where you'd only need to test something before shipping it to figure out if it doesn't work. With an llm you could potentially just trick it to not only give a wrong answer or software solutions, but have it think that it's doing okay.

## Conclusion

Humans software engineers, for the time being, are safe. The day openai and other ai giants stop hiring new engineers, that's the day we should be worried. For now it's actually safer and cheaper to keep hiring human engineers. That's not to say that this will hold forever, but for the foreseeable future, this will hold. So you computer science majors about to graduate, or just starting, keep on trucking, you'll be fine for now.
