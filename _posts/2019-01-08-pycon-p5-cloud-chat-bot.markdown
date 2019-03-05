---
layout: single
title: "PyCon.DE 2018 Part 5: Cloud chat bot for lazy people"
date:   2019-01-08 12:52:28
tags: conference python
header:
  overlay_image: assets/images/dataStorm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Peter Hoffmann
author_profile: true
---

Part 5 of our PyCon.DE 2018 series is about a cloud-based chatbot which will answer easy questions like the health of a service the owner of the bot is responsible for. The bot is written in Python with [Azure bot service](https://azure.microsoft.com/en-us/services/bot-service/), reachable by [Slack](https://slack.com/) and based on information from a [Prometheus](https://prometheus.io/) monitoring system. 

PyCon.DE is where Pythonistas in Germany can meet to learn about new and upcoming Python libraries, tools, software, and data science. In 2018 we had more than 500 participants in Karlsruhe, a city where approximately 3600 IT companies with more than 36000 jobs exist.  

At Blue Yonder we established Slack years ago as our chat application and it greatly increased productivity as it is very easy to contact a person or a group simultaneously. On the other hand, people are more likely to ask simple questions rather than doing a quick Google search. One solution is to educate on how and where to find the answer. The other solution is to use a chatbot which has the advantage that you can import a specific type of response more easily into a conversation than looking up the information and copy and paste it. 

A frequently asked question is: _“Does service X has a problem right now?”_ which we will use to demonstrate the chatbot. The first step is to create a python bot for the Azure bot service. Then, we can either directly use the bot to answer the question or create the response without going to the service health monitoring. In this case, a Prometheus monitoring service has to be queried and the result needs to be transformed into a chat message. 

Bjoern Meier is a software engineer at Blue Yonder GmbH since 2016 after graduating in Computer Science. More correctly you could say he is a DevOps engineer at Blue Yonder where he is developing and operating - among other things - the services for the external data interfaces, preprocessing and data storage to enable the data scientists to run their prediction models. He loves the versatility and ecosystem of python to write e.g. production web apps, data analysis tools or operational scripts. If there was more free time he would like to spend it to dive deeper into functional programming languages like elixir to have a different view on things.