---
layout: single
title: "PyCon.DE Part 1: Turbodbc - Turbocharged database access for data scientists"
date:   2018-02-28 13:31:55
tags: conference python
header:
  overlay_image: assets/images/dataStorm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Michael König
author_profile: true
---

At last year´s [PyCon.DE](https://de.pycon.org/), from October 25th till 27th, Blue Yonder held several interesting talks around Python. In case you missed out the event, we would like to share the best of our talks at the conference in this and the following four blog articles. 

PyCon.DE is where Pythonistas in Germany can meet to learn about new and upcoming Python libraries, tools, software and data science. In 2017  Python enthusiasts, programmers and data scientists from around the world joined in Karlsruhe. 

This talk introduces the open source Python database module [turbodbc](https://github.com/blue-yonder/turbodbc). It uses standard ODBC drivers to connect with virtually any database and is a viable (and often faster) alternative to "native" Python drivers. 

Python's database API 2.0 is well suited for transactional database workflows, but not so much for column-heavy data science. This talk explains how the ODBC-based turbodbc database module extends this API with first-class, efficient support for familiar NumPy and [Apache Arrow](https://arrow.apache.org) data structures. Briefly recounting the painful story of how data scientists previously used our analytics database, Michael explains why turbodbc was created and what distinguishes it from other ODBC modules. Sketching the flow of data from databases via drivers and Python modules to consumable Python objects, he motivates a few extensions to the standard database API 2.0 that turbodbc has made. These extensions heavily use NumPy arrays and Apache Arrow tables to provide data scientists with both familiar and efficient binary data structures they can further work on. Michael concludes his talk with benchmark results for a few databases. 

If you would like to get more insights, following blog articles might be interesting for you: 

  * [Making of turbodbc (part 1): Wrestling with the side effects of a C API]({{ site.baseurl }}{% post_url 2017-02-20-making-of-turbodbc-part-1-wrestling-with-the-side-effects-of-a-c-api %})
  * [Making of turbodbc (part 2): C++ to Python]({{ site.baseurl }}{% post_url 2017-04-11-making-of-turbodbc-part-2-c-to-python %})
  * [Turbodbc and Apache Arrow]({{ site.baseurl }}{% post_url 2017-06-20-turbodbc-apache-arrow %})

Michael König is Head of Platform at Blue Yonder GmbH. He holds a PhD in physics, practices test-driven development, and digs Clean Code in C++ and Python. In the last five years, he invested more money in table tennis gear than in smartphones.
