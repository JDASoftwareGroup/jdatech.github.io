---
layout: single
title: "PyCon.DE Part 4: Observing applications with Sentry and Prometheus"
date:   2018-03-16 09:48:10
tags: conference python
header:
  overlay_image: assets/images/2018-03-16-pycon-de-part-4-monitoring.png
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Peter Hoffmann
author_profile: true
---

In Part 4 of our PyCon.DE series we´ll demonstrate how to observe your applications with Sentry and Prometheus. If you have services running in production, something will fail sooner or later. We cannot avoid this completely, but we can prepare for it. In this talk we will have a look at how Sentry and Prometheus can help to get better insight into our systems to quickly track down the cause of failure. 


[PyCon.DE](https://pycon.de) is where Pythonistas in Germany can meet to learn about new and upcoming Python libraries, tools, software and data science. In 2017  Python enthusiasts, programmers and data scientists from around the world joined in Karlsruhe.

{% include video id="f3WO4bpLySs" provider="youtube" %}

When your services starts behaving in a strange way, for example due to bugs introduced in the newly deployed release, you expect to get informed about any issues as soon as possible. Preferably by your own monitoring tool and not by one of your customers. We will have a look at how Sentry and Prometheus can help solving this problem. 


[Sentry](https://sentry.io) is a real-time error tracking system, which can notify you when exceptions in your application occur. Additionally, it provides lots of context so that crashes can be reproduced and fixed very quickly. 


[Prometheus](https://prometheus.io) is a systems and service monitoring system, collecting metrics from all kinds of targets. The collected metrics can help to get insight in what's actually going on in your services. Patrick Mühlbauer is a Software Engineer from Karlsruhe, working in Blue Yonder's Platform team. He likes this devops thing and enjoys instrumenting code to collect metrics and create nice and shiny Grafana Dashboards.