---
date: "2021-06-27T00:00:00Z"
external_link: ""
image:
  caption:
  focal_point: Smart
links:
- icon: github-square
  icon_pack: fab
  name: GitHub
  url: https://github.com/ranibasna/ddk

# slides: example
summary: Orthonormal Basis Selection using Machine Learning.
tags:
- Machine Learning
- Splines
- r package
- Functional Data Analysis
title: Data Driven Knot Selection
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""
---

In implementations of the functional data methods, the effect of the initial choice of an orthonormal basis has not gained much attention in the past. Typically, several standard bases such as Fourier, wavelets, splines, etc. are considered to transform observed functional data and a choice is made without any formal criteria indicating which of the bases is preferable for the initial transformation of the data into functions. In an attempt to address this issue, we propose a strictly data-driven method of orthogonal basis selection. The method uses recently introduced orthogonal spline bases called the splinets obtained by efficient orthogonalization of the B-splines. The algorithm learns from the data in the machine learning style to efficiently place knots. The optimality criterion is based on the average (per functional data point) mean square error and is utilized both in the learning algorithms and in comparison studies. The latter indicates efficiency that is particularly evident for the sparse functional data and to a lesser degree in analyses of responses to complex physical systems.
