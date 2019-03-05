---
layout: single
title: "PyCon.DE 2018 Part 3: Python with and without Pants"
date:   2019-01-08 12:50:28
tags: conference python
header:
  overlay_image: assets/images/data storm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Peter Hoffmann
author_profile: true
---

In the 3rd Part of the PyCon.DE 2018 series, we will talk about building, packaging, and deploying Python code with Pants and PEX. 

PyCon.DE is where Pythonistas in Germany can meet to learn about new and upcoming Python libraries, tools, software, and data science. In 2018 we had more than 500 participants in Karlsruhe, a city where approximately 3600 IT companies with more than 36000 jobs exist.  

In this talk we cover a rather controversial topic: monorepos and their build tools, to be precise, we will focus on Pants. [Pants](https://www.pantsbuild.org) is a build system for large or rapidly growing code bases. It supports all stages of a typical build (bootstrapping, dependency resolution, compilation, linting, ...) and allows users to organize their code via targets for binaries, libraries, and tests. For Python programmers who want to easily manipulate and distribute hermetically sealed Python environments (PEXes), Pants is the right choice. 

First, we will introduce Pants in a company-wide monorepo setting. After that, we will focus on important Python-centric features and then describe them in detail. 

The talk concludes with a discussion of use cases for Pants outside of a monorepo. Stephan Erb is a software engineer driven by the goal to make Blue Yonder's data scientists more productive. 

Stephan holds a master's degree in computer science from the Karlsruhe Institute of Technology. He is a PMC member of the Apache Aurora project and tweets at [@ErbStephan](https://twitter.com/erbstephan).