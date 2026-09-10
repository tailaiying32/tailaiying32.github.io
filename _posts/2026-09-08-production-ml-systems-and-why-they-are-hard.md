---
layout: post
title: "Production machine learning systems and why they are hard"
date: 2026-09-08
---

I've always been very interested in the field of machine learning. It's super cool to me how we're essentially able, through machine learning, to teach computers to learn from data and make predictions without explicitly being programmed. I took both Intro to Machine Learning and Deep Learning as technical electives during my time at Cornell because of this interest (Machine Learning has since become part of the core CS requirements --- a reflection of the increasing importance of the field in today's world). In these classes, we were taught a huge assortment of different machine learning techniques, models, architectures, etc. While this knowledge is obviously very important and sits at the core of machine learning as a study, it is not all that is needed to create a production-level machine learning system. In fact, it's not even *close* to being all that is needed, as shown in the image below. (source: Andrew Ng's [MLOps Course](https://www.coursera.org/learn/introduction-to-machine-learning-in-production))

![An approximation of all that goes into an ML system](https://changyaochen.github.io/assets/images/ml_hidden_debt.png)

In a real-world system, there's so much that we have to account for in order for our machine learning model to work properly. Many of these issues are explained in the famous 2015 paper [Hidden Technical Debt in Machine Learning Systems](https://proceedings.neurips.cc/paper_files/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf) by Sculley et al. at Google. This post is an attempt at explaining some of these issues, and more, and how we can solve them. In the last ten years or so, this field has grown so much that the industry even coined a term for it: "MLOps". We now have engineers --- entire teams, even --- whose sole job is to identify these issues and fix them before they wreak havoc in production.

# ML systems are deeply coupled
One of the biggest issues when it comes to machine learning systems is that things are tightly tangled together. Unlike in traditional software engineering, where strong abstraction and modular design can be enforced and are generally preferred for maintainability, it is very difficult to do the same in ML systems. This concept is commonly referred to as the **CACE principle: Changing Anything Changes Everything**. We will examine some examples of this principle below.

1. ML systems mix a large number of data inputs together, which entangles them and makes the effects of changes hard to isolate. For example, if our system takes in features $$x_1, x_2, ..., x_n$$, and we change the distribution of $$x_i$$ or add/remove a feature, the weights of the other features may need to change. It is impossible to predict how they need to change, only that they will need to change. 
2. We often come across situations where we need to solve some problem that is similar to the original. For these cases, rather than building an entirely new model from scratch, it can be easier to simply allow our new model to take the original model as input and have it learn a small adjustment. This chain of corrections can cascade into a **correction cascade**, in which newer models are dependent on their ancestor models. Once established, a correction cascade makes it extremely difficult to improve the system, as improving any arbitrary component within the cascade can mean degradation of the whole system. 
3. In a similar vein, we also often come across situations where the input of one model comes from the output of another. Like above, we can stack these models arbitrarily on top of each other into what is known as a **model cascade**. This leads to a similar issue where if we tweak an upstream model, it degrades the performance of all downstream models since downstream models were trained on the old output.

# Feedback loops
to be continued...