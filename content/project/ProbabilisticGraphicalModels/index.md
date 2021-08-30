---
date: "2021-04-27T00:00:00Z"
external_link: ""
image:
  caption:
  focal_point: Smart
links:
- icon: github-square
  icon_pack: fab
  name: GitHub
  url: https://github.com/ranibasna

# slides: example
summary: Using Probabilistic Graphical Models for Inference
tags:
- PGM
- Machine Learning
- Bayesian Network

title: Probabilistic Graphical Models for Inference
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""
---

Due to the typical highly heterogeneous clinical and epidemiological data generating
processes, multiple correlations/dependencies between variables and between response
variables arise. Conventional regression models have a limited capacity in capturing such
dependent multi-factorial relationships. In this project, we plan to leverage the powerful
approach of probabilistic graphical models (PGMs) such as BNs and Hidden Markov Models
for conducting statistical inference. PGMs Models learn joint multivariate distributions over
large numbers of random variables that interact with each other. So far, we have used BN.
Bayesian network is a PGM that models the associations between all covariates with all
variables being potentially dependent. Bayes rule is usually employed, which allows the
factorization of the joint probability distribution of the involved variables, leading to a
probabilistic graph directed acyclic graph (DAG). The overarching goal of these models is to
estimate the conditional dependence of different variables, interactions, and therefore under
specific assumptions may reveal potential causal links between variables. This flexibility
gives BNs an advantage over other approaches for estimating the risk of disease with clinical
relevance and allows core decision pathways.
