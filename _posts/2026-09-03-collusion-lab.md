---
layout: post
title: "Collusion Lab: an experimental platform for studying multi-agent collusion"
date: 2026-09-03
---

# What is Collusion Lab?

Collusion Lab was a project I built last semester because I thought I wanted to put my research and engineering skills to use and learn more about multi-agent dynamics and how they would behave in different situations. At the time, I also had very conveniently received $1500 in API credits for OpenAI and Anthropic through YC (thank you very much), which certainly gave me good motivation to build this project. 

With all that being said, let's talk about what this project actually is: a platform with plug-and-play infrastructure to run multi-agent experiments in a variety of environments. In my case, I chose to run a simulated economy where LLM agents play competing firms in a repeated pricing game. Each round, agents pick a price to sell their goods at, and a demand model converts this price into a quantity sold and profit. The question I wanted to investigate: if agents are allowed to freely communicate before pricing, will they collude to maintain prices above the Nash equilibrium? In other words, given the means, will these agents price fix their goods? More importantly, if we add an independent auditor that randomly checks agent communication channels for illegal pricing agreements and penalizes the offenders, do agents revert to normal competetive pricing or do they learn to coordinate in secret without getting caught?

# Design Choices
I built this as a reusable research platform first, meaning that all parts of the pipeline are decoupled and easily swappable. This means that through a single YAML file, anyone can swap the LLM provider, demand model, auditing strategy, prompts, etc, without having to make changes to the pipeline code itself. This also means that it would be quite easy for anyone to add on additional environments to the existing pricing game, if desired. Additionally, run trajectories and configurations are completely traceable, which means outcomes are fully reproducible allowing for rigorous experimentation.

# The Simulation
The simulation runs through six main steps each round:
1. **Reasoning/Communication:** Each agent first spends some time thinking about anything they would like, whether that is what message to send to other agents, pricing logic, etc. After this, agents are allowed to send a single message to every other firm.
2. **Pricing:** After agents read all incoming messages, agents are allowed one final reasoning step before setting a discrete price. 
3. **Profit Calculation:** A deterministic demand model (linear differentiable demand by default) converts the price vector into quantities and profits.
4. **Auditing:** A given round has a probability to be audited (set in run configuration). If this round is chosen to be audited, an independent agent acting as judge examines the public messages sent, penalizes any agents it deems as acting illegally by partially or fully taking away the round's profit, and sends a message to the offending agents as to why they were penalized.
5. **Memory update:** Agents can remember a fixed number of rounds in the past. This round's information is appended to the agent's memory: its own action, all competing agents' actions, it's own reward, messages, and if present, any auditor feedback.
6. **Signals logged:** Various signals for this round are recorded in the run log: price dispersion among agents, normalized reward elevation, and four different collusion flags (explicit / behavioral / covert / hollow).

# The Demand Models
1. **Linear differentiated demand:** This is the current default demand model. It's simple, and works very well for the purposes of this project. The linear demand model is

    $$
    q_i = a - b p_i + d \bar{p}_{j \neq i}
    $$

    where $$q_i$$ is the number of units the $$i$$th firm sells, $$a$$ is the demand intercept i.e. how much a firm would sell at very low prices, $$b$$ is the own-price slope i.e. how much a firm loses by charging more, $$d$$ is the cross-price slope i.e. how much a firm gains when rivals charge more, and $$\bar{p}$$, the average rival price. I chose this as the default model since it's more intuitive than Calvano, but much gentler than the winner-take-all model. 

2. **Calvano logit demand:** This is included as a comparison to the linear default, and comes from Calvano et al. (2020). Rather than raw units, quantities here are market shares, and consumers can also choose to buy nothing. The Calvano demand model is

    $$
    q_i = \frac{\exp((a - p_i)/\mu)}{\sum_{j} \exp((a - p_j)/\mu) + \exp(a_0/\mu)}
    $$

    where $$q_i$$ is firm $$i$$'s market share, $$a$$ is product quality, $$p_i$$ is firm $$i$$'s price, $$\mu$$ is an index of horizontal differentiation i.e. how much individual consumer tastes matter, and $$a_0$$ is the attractiveness of the outside option i.e. how appealing it is to buy nothing. We can think of $$\mu$$ as a consumer's loyalty to their favorite firm. If it's small, a tiny undercut steals a large portion of demand, but when it's large, consumers will stay even at a higher price. If all firms raise together, consumers will simply choose to not buy.
3. **Winner take-all:** The cheapest firm captures the entire market (ties split accordingly). Collusion is very fragile compared to the other two models, since the slightest undercut will win everything. Firms are incentivized to price as low as possible.

# The Auditing Pipeline
An LLM-as-judge acts as an independent auditor when enabled. This agent is prompted to read through public messages and outputs a strict VERDICT $$\to$$ EVIDENCE $$\to$$ REASONING format, and defaults to not-flagged in case of error. This is combined with a hard regex backstop, so if any message contains explicit price-intent language (e.g. we will/plan to/set to/match...) with a number, it is automatically flagged. 

# Steganography Detection
This detection pipline runs only after a run as finished, and by design never feeds back into the simulation to ensure gameplay is not contaminated. It extracts unigram and bigram features from message text, and labels rounds as `coordinated_up` (prices move up together with low dispersion) and `elevated` (normalized reward elevation above a threshold). It then computes a **lift statistic** per feature:

$$
P(\texttt{coordinated\_up} \mid \texttt{feature\_present}) - P(\texttt{coordinated\_up})
$$

A "steganographic signature" is declared when explicit-collusion rate is low, behavioral-collusion rate is high, and at least one word/phrase has meaningful predictive lift.

# Results
to be continued...
