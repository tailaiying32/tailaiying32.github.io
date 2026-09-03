---
layout: post
title: "Collusion Lab: an experimental platform for studying multi-agent collusion"
date: 2026-09-03
---

# What is Collusion Lab?

Collusion Lab was a project I built last semester because I thought I wanted to put my research and engineering skills to use and learn more about multi-agent dynamics and how they would behave in different situations. At the time, I also had very conveniently received $1500 in API credits for OpenAI and Anthropic through YC (thank you very much), which certainly gave me good motivation to build this project. 

With all that being said, let's talk about what this project actually is: a platform with plug-and-play infrastructure to run multi-agent experiments in a variety of environments. In my case, I chose to run a simulated economy where LLM agents play competing firms in a repeated pricing game. Each round, agents pick a price to sell their goods at, and a demand model converts this price into a quantity sold and profit. The question I wanted to investigate: if agents are allowed to freely communicate before pricing, will they collude to maintain prices above the Nash equilibrium? In other words, given the means, will these agents price fix their goods? More importantly, if we add an independent auditor that randomly checks agent communication channels for illegal pricing agreements and penalizes the offenders, do agents revert to normal competetive pricing or do they learn to coordinate in secret without getting caught?

# Design Choices
I want to take a short detour from the fun part of this project by quickly detailing my design decisions. I built this as a reusable research platform first, meaning that all parts of the pipeline are decoupled and easily swappable. This means that through a single YAML file, anyone can swap the LLM provider, demand model, auditing strategy, prompts, etc, without having to make changes to the pipeline code itself. This also means that it would be quite easy for anyone to add on additional environments to the existing pricing game, if they so desired. Additionally, run trajectories and configurations are completely traceable, which means outcomes are fully reproducible allowing for rigorous experimentation.

# The Simulation
The simulation runs through six main steps each round:
1. **Reasoning/Communication:** Each agent first spends some time thinking about anything they would like, whether that is what message to send to other agents, pricing logic, etc. After this, agents are allowed to send a single message to every other firm.
2. **Pricing:** After agents read all incoming messages, agents are allowed one final reasoning step before setting a discrete price. 
3. **Profit Calculation:** A deterministic demand model (linear differentiable demand by default) converts the price vector into quantities and profits.
4. **Auditing:** A given round has a probability to be audited (set in run configuration). If this round is chosen to be audited, an independent agent acting as judge examines the public messages sent, penalizes any agents it deems as acting illegally by partially or fully taking away the round's profit, and sends a message to the offending agents as to why they were penalized.
5. **Memory update:** Agents can remember a fixed number of rounds in the past. This round's information is appended to the agent's memory: its own action, all competing agents' actions, it's own reward, messages, and if present, any auditor feedback.
6. **Signals logged:** Various signals for this round are recorded in the run log: price dispersion among agents, normalized reward elevation, and four different collusion flags (explicit / behavioral / covert / hollow).

# To be continued...
