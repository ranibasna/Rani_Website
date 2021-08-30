---
abstract: The risk of asthma greatly increased during the second half of the last century. Asthma remains a disease with heterogeneous underlying phenotypes, but several environmental and demographic risk factors have been identified. Of these is the independent effects of smoking and socioeconomic status. However, given the known association between socioeconomic status and smoking, the potential interaction between these two factors in relation to the risk of asthma is an important public health question. Elucidating such interactions is not trivial, statistically. However, emerging Bayesian Network analysis is proving valuable in exploring complex data structures and offering new insights to analysis of health-related data. We applied Bayesian Network analysis to address the question of interaction of smoking and social class in relation to asthma and respiratory symptoms based on data derived from the West Sweden Asthma Study and the Obstructive Lung Diseases in Northern Sweden. In this talk, I will discuss the step-by-step approach to implementing the analysis. Independency structure can be identified in Bayesian networks, particularly with Directed Acyclic Graphs (DAGs), which are probabilistic graphical models. DAGs learn the underlying dependency structure represent these as networks with directed connections. We will learn the network structure of the variables including the smoking-socioeconimc variables using a data-driven hill-climbing algorithm with bic-cg score (the corresponding Bayesian Information Criterion score for mixed datasets). We conducted a bootstrap aggregation and model averaging to reduce the number of arcs that are incorrectly included in the Network structure. Then, we fitted the Bayesian network model to learn the related parameters. Finally, we used the approximate inference approach to estimate the conditional probabilities by eliciting a sample of realizations of the modeled variables under specific conditions. We validated our model by first running a cross-validation approach; and second simulating new data and comparing its statistical characteristics with the original data.
address:
  city: Lund
  country: Sweden
#  postcode:
  region: Skåne
# street: 450 Serra Mall
all_day: false
authors: [Rani Basna]
date: "2021-03-24T10:00:00Z"
#date_end: "2021-3-24"
event:  Swedish Network for Register-Based Research meeting
event_url: http://swe-reg.se/springmeeting2021/
featured: false
image:
  caption: 
  focal_point: Right
links:
- icon: twitter
  icon_pack: fab
  name: Follow
  url: https://twitter.com/RaniBasna
- icon: file-alt
  icon_pack: far
  name: Slides
  url: http://swe-reg.se/presentations-from-morning-session/  
- icon: github
  icon_pack: fab
  name: Code
  url: https://github.com/ranibasna/Bayesian-Network-Application-to-SocioEconomic-status

location: Lund
projects:
- internal-project
publishDate: "2021-03-024T00:00:00Z"
# slides: example
summary: Bayesian Network application to airway data. 
tags: [Bayesian Network, Modification Effect]
title: Bayesian Network applications
url_code: ""
url_pdf: ""
#. url_slides: "http://swe-reg.se/presentations-from-morning-session/"
url_video: ""
---

