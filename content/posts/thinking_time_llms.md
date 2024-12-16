---
title: "About thinking time in LLMs"
date: 2024-12-16
draft: false
ShowToc: true
author: Herocharge
---

### Introduction

Note: citation needed

Let's take a detour and think about how WE(for the humans reading this blog) think.
Our thinking time depends on the question posed. Every question has a certain complexity.  
This complexity isn't necessarily proportional to the length of the question (number of characters).  

For example, something like `"If I have 2 sons and 2 daughters, how many kids do I have?"`(58 characters) is easy,
because it is essentially asking 2+2. Well... you do need to have knowledge of english and the understanding that both daughters and sons count as kids equally(although some seem to disagree).  
Whereas the question `"What is the 13337th digit of PI?"`(32 characters) is much harder because one needs to start computing the digits of PI upto the 13337th digit, which I don't think a human has enough patience to do even if they have all the requisite knowledge.  

#### So how do we measure question complexity?  
Well a good benchmark would be the average amount of time it would take a human to answer the question, but this is infeasible to create such a dataset (right?). What do humans do when we think (ideally?). If there is an easy way to check the correctness of an answer we try to guess (an educated guess?) till we feel like guessing won't work. Then we actually put in some effort, start with all the knowledge we know and then slowly deduce the conclusion from the axioms. We can define the complexity of the question to be the number of steps it would take to deduce the conclusion/answer from the axioms.  
But there maybe more than 1 way to deduce the conclusion......
We could pick the shortest path, but that doesn't capture the fact that humans are on average dumb(oops). We need to take into account the size of search space for the reasoning steps, this is probably directly proportional to the number of axioms we begin with, so the larger the initial knowledge base the more chances for us to make mistakes (Worse when contradictory knowledge exists simultaneously).  

> This actually a very important observation! The complexity of a question doesn't only depend on the 
question but also on the initial knowledge of the person you are asking the question to. For example, for the question "Do you really think the earth is flat?" most people would say "No", but some individuals(cough, flatearthers, cough) might start prepping a powerpoint to explain how you are wrong.

We can take the average number of steps of deduction, this might partially capture the number of possible deduction paths. However the problem with this is that at some point we might need to actually compute all the possible deduction paths and this might actually be pushing NP-hard territory. We went from being practially infeasible to theoretically infeasible? (good job bhargav)

Ok this was a rather long detour, the point is there isn't really a good way to capture complexity of a question a.k.a prompt. This means that thinking time for both humans and LLMs would probably vary a lot on the actual question and any deadline on the thinking time would inevitably make some questions out of scope.

### What is the thinking time of an LLM?
For a long time now, one of the marketed features of LLMs are their context length (number of tokens you can expect an LLM to digest before it starts producing shit, eww weird). The transformer architecture doesn't inherently put a limit on the number of tokens it can generate. You can scale the attention mechanism to however many tokens as long as it fits in memory. Also the inference time scales quadratically with the number of tokens. The real constraint is training for larger context lengths[citation needed]. Why? idk, look it up. Anyway, we can think of the transformer "thinking" once when producing a single token. 

When producing a new token, we deterministically process the weights of the transformer and the already produced tokens.` We can draw a very loose analogy between weights of transformer to initial knowledge base of a person and the already produced tokens to be steps that are deduced prior from initial knowledge base. (TLDR, tokens are deduction steps, weights are axioms)` This would mean longer the context, the more the deduction steps. This just means it's thinking more, no guarantee if it's thinking in the right direction or about relevant things (cough, -insert distracting thing-, cough). 

We can artificially increase the thinking time of the LLM by forcing it to produce special tokens called `thinking tokens` (TODO: cite paper). However, the paper only implements having the same thinking token everywhere, which is counterintuitive because it's like stating the same deduction step everywhere (it actually works decently because our analogy is dogshit.). It's however still not optimal. My friend @vnnm is working on a version of this which can incorporate local context, which should give better results as it's different reasoning steps.

Moving on... 

It would be foolish to not talk about chain of thought and it's sister methods (tree of thought and graph of thought and categories of thought(coming soon 2207)). These methods are more closer to our analogy because we literally beg the LLM to produce intermediate reasoning steps in the form of natural language. The X of thought methods should've been enough, as they only really stop when they are done reasoning and can basically go on forever if you allow for it. (Oh nvm we still have to deal with the limit of context length).   

There is an inherent contradiction here.
> If we increase the amount of initial world knowledge then there is more of a chance to pursue a spurious or inefficient line of deduction, if we decrease the amount of inital world knowledge we might not meet the deadline posed by memory or context length because we need more deduction steps.

One might say increasing the inital world knowledge doesn't automatically make reasoning worse, which I agree with but when we are dealing with probablistic things (the dataset introduces uncertainity), it inevitably makes things worse.


#### So what can we do?
There are two things we can do - have a better initial knowledge base and have a better way to come up with the next deduction step.

A better initial knowledge base would consist of few facts but those which convey a lot of information. We need something like a lossy compression of the knowledge base. This is probably why finetuning works so well, the knowledge base is very specific and small and the tasks the finetuned models perform won't need to work well outside their domain.

A better reasoning process is a much harder thing to actually come up. By better we mean more efficient, i.e., requiring fewer deduction steps(aka tokens) or a single token representing multiple deduction steps. Thinking tokens which are context aware are again a good way to do it because we can train them to be as efficient as we want (easier said than done).

What if I don't want special tokens?  
How does an LLM decide on a particular token (aka deduction step)?  
The transformer does all the processing on prior knowledge and deduction and produces a probability distribution over the possible tokens. Picking one token is a lossy step because we lose a lot of the information that was being conveyed by the probability distribution. Picking the correct token here matters a lot! We can use better sampling techniques like entropy based sampling (another rabbit hole, TODO: cite paper). And things like multiprediction are approaches to use.


### Conclusion
Why restrict ourselves to tokens?  
We might need a better way to represent our deduction steps (again, the analogy isn't perfect but you get the point). We need to focus more on how the LLM selects it's line of thought than trying to make it longer or shorter. And...  


Don't listen to me. I'm dumb.