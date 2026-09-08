---
layout: post
title: "Production machine learning systems and why they are hard"
date: 2026-09-08
---

I've always been very interested in the field of machine learning. It's super cool to me how we're essentially able, through machine learning, to teach computers to learn from data and make predictions without explicitly being programmed. I took both Intro to Machine Learning and Deep Learning as technical electives during my time at Cornell because of this interest (Machine Learning has since become part of the core CS requirements --- a reflection of the increasing importance of the field in today's world). In these classes, we were taught a huge assortment of different machine learning techniques, models, architectures, etc. While this knowledge is obviously very important and sits at the core of machine learning as a study, it is not all that is needed to create a production level machine learning system. In fact, it's not even *close* to being all that is needed, as shown in the image below.

![An approximation of all that goes into a ML system](https://changyaochen.github.io/assets/images/ml_hidden_debt.png)

In a real world system, there's so much that we have to account for in order for our machine learning model to even work properly. Many of these issues are explained in the famous 2015 paper [Hidden Technical Debt in Machine Learning Systems](https://proceedings.neurips.cc/paper_files/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf) by Sculley et al. at Google. This post is an attempt at explaining some of these issues, and how we can solve them. In the last ten years or so, this field has grown so much that the industry even coined a term for it: "MLOps". We now have engineers --- entire teams, even --- whose sole job is to identify these issues and fix them, before they wreak havoc in production. 

