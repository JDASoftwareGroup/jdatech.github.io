---
layout: single
title:  "Write-Up of the PyConDE & PyData Berlin 2019 conference"
date:   2020-01-26 20:00:00 +0100
tags: conference python talk
header:
  overlay_image: assets/images/background-banner-blur-conference-399158.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Sebastian Neubauer
author_profile: true
---
# Write-Up of the PyConDE & PyData Berlin 2019 conference


As many of us are working from home these days, we thought it might be a good idea to share our highlight talks of the 
PyCon.DE & PyData Berlin 2019 conference, so you have something interesting to watch during your breaks. 
While there is so much good content, let's all hope we don't have the time to watch conference recordings very soon, 
because we can go out with friends and familiy again!


Also, the CfP for the 2020 PyCon.DE & PyData Berlin conference is now open (submit your proposal 
[here](https://de.pycon.org/)). There is a chance that you have time to write a proposal, so take these talk 
recommendations as an inspirations for your own submission.


PyCon.DE is a Python conference held in Germany where attendees can meet to learn about new and upcoming Python 
libraries, tools, software, and data science.
In 2019 PyCon.DE & PyData Berlin took place in Berlin and had more than 1000 participants basically doubling the amount 
of attendees as compared to the 2018 conference in Karlsruhe. As almost all Python conferences this event was run and organised by 
volunteers contributing free time and supported by community sponsors contributing work time of their employees. 
Sebastian Neubauer from Blue Yonder is part of the organizing team.

Python is basically used throughout the entire Blue Yonder stack and attending this conference allows to stay on top of 
latest developments in the Python world. Furthermore, since Blue Yonder profits a lot from the open source community, 
the company was a Gold sponsor at this year's conference and thus present with a booth at the venue. In total, eleven 
Blue Yonder associates from the German offices in Karlsruhe and Hamburg traveled to the conference, talked to colleagues 
and potential candidates at the booth, and learned new things attending the talks. Furthermore, three Blue Yonder 
associates gave a talk themselves:

 * Florian Jetter: **"Kartothek – Table management for cloud object stores powered by Apache Arrow and Dask"**  
    Link: [https://www.youtube.com/watch?v=8wAH52rlADI](https://www.youtube.com/watch?v=8wAH52rlADI)

 * Sebastian Neubauer: **"6 Years of Docker: The Good, the Bad and Python Packaging"**  
    Link: [https://www.youtube.com/watch?v=kgqdOftkZ-E](https://www.youtube.com/watch?v=kgqdOftkZ-E)

 * Stefan Maier: **"Embrace uncertainty! Why to go beyond point estimators for valuable ML applications"**  
    Link: [https://www.youtube.com/watch?v=Ght8AVFxBwk](https://www.youtube.com/watch?v=Ght8AVFxBwk)
 
 
## Highlights

There were so many good talks, so here is a very incomplete compilation of our highlights in random order. Thanks to 
Stefan Maier, Sven Fritz, Vasu Sharma, Phillip Sonntag, Jakob Herpel, Aniruddh Goteti and Lucas Rademaker for their 
contribution to this list.

### Title: TBC

Link: [https://www.youtube.com/watch?v=zZXSGzlVxvU](https://www.youtube.com/watch?v=zZXSGzlVxvU)

Speaker: James Powell

Summary: James is a "rockstar" among the python speakers, that much that he is not in the need for handing in an abstract or even a title and still being accepted. He is well known for his fast-paced presentations only using vim. This time he talked about metaprogramming in python, but there is no way around to watch it to get a clue what it is about.

### Title: How MicroPython went into space

Link: [https://www.youtube.com/watch?v=Vh_5Lz1mLq8](https://www.youtube.com/watch?v=Vh_5Lz1mLq8)

Speaker: Christine Spindler

Summary: It is just mind-blowing in how many fields python drains into. Not only there is MicroPython which brings python to embedded devices, but this project now is under certification for the use in space!! So, not long until python will steer and control satellites and the space stations. Awesome!

### Title: Automating feature engineering for supervised learning? Methods, open-source tools and prospects.

Link: [https://www.youtube.com/watch?v=XK7mKGWSue8](https://www.youtube.com/watch?v=XK7mKGWSue8)

Speaker: Thorben Jensen

Summary: The talk basically reported a case study comparing three open source libraries ``tsfresh``, ``featuretools`` and ``TPOT``. At the end of the day, ``TPOT`` seemed superior in most categories, including the degree of automation. However, the case study was only carried out on a single data set. When asked about the potential biases of this dataset, the speaker said that ``tsfresh`` does provide the better support for time series problems, and that this may be more critical for other datasets.
(Shameless plug: ``tsfreh`` is a very popular Blue Yonder open source project: [https://github.com/blue-yonder/tsfresh](https://github.com/blue-yonder/tsfresh))

### Title: Docker and Python - A Match made in Heaven

Link: [https://www.youtube.com/watch?v=Q2u1wcfmlzw](https://www.youtube.com/watch?v=Q2u1wcfmlzw)

Speaker: Dr. Hendrik Niemeyer

Summary: A very detailed and well structured introduction into docker and its usage for python development. Many pointers to useful tools and frameworks and some good introductory live demos.

### Title: 10 Years of Automated Category Classification for Product Data

Link: [https://www.youtube.com/watch?v=R-ts_mxGKOg](https://www.youtube.com/watch?v=R-ts_mxGKOg)

Speaker: Johannes Knopp

Summary: Johannes shows the historic evolution of their product category classification at "solute GmbH", a Karlsruhe based competitor prices service provider. I love how he openly talks about failed attempts, historic quirks and problems they faced over the years.

### Title: Dr. Schmood's Notebook of Python Calisthenics and Orthodontia

Link: [https://www.youtube.com/watch?v=BAsL_cbkQDc](https://www.youtube.com/watch?v=BAsL_cbkQDc)

Speaker: David Schmudde

Summary: I have to admit, the title would not have pulled me into this talk as I have no clue what the words are about. As I was session chair, I was lucky enough to attend it. In fact, it was about the state handling in python and why some complain about "messed up variable states" in IPython notebooks. He also suggests to embrace immutability to solve the situation and I love immutability!

### Title: Python 2020+

Link: [https://www.youtube.com/watch?v=fOdCxum-qLA](https://www.youtube.com/watch?v=fOdCxum-qLA)

Speaker: Łukasz Langa

Summary: In this keynote, Łukasz gives a unique look behind the scenes of how Python, especially cpython, is developed and he gives some predictions where the future will, or should go. But you should definitively watch the talk because of the many anecdotes shared by him. BTW: In the hallway track, Łukasz, the creator of "black" assured me he also doesn't like some of the formatting decisions, but it is "black". Even he is not able to argue with it, it's "black"!

### Title: Rethinking Open Source in the Era of Cloud & Machine Learning

Link: [https://www.youtube.com/watch?v=QMJIh-voWng](https://www.youtube.com/watch?v=QMJIh-voWng)

Speaker: Peter Wang

Summary: Having the CEO of a big software company give the keynote at a community driven conference is at least unexpected. But Peter Wang has definitively proven that it was a very good choice. In his talk "Rethinking Open Source in the Era of Cloud & Machine Learning" he is deep-diving into how to sustainably run an open source project, commercial or uncommercial. A must see for everyone who is interested in the hidden forces behind the tectonic shifts of the  IT landscape in the recent years.

### Title: Are you sure about that?! Uncertainty Quantification in AI

Link: [https://www.youtube.com/watch?v=LCDIqL-8bHs](https://www.youtube.com/watch?v=LCDIqL-8bHs)

Speaker: Florian Wilhelm

Summary: The talk introduced the concepts of aleatoric and epistemic uncertainty. It compared various methods for uncertainty estimates according to several categories, such as performance, implementation effort etc. A simple, one variable toy dataset was used to evaluate these methods in practice. Some methods apparently showed a poor performance such as Monte-Carlo dropouts. I personally would have like to learn on why some methods performed better or worse on the dataset or not and how this generalizes to real-world datasets. However, based on later conversations with the speaker, this seems a tough problem for some of the methods used. What I definitely learned was how to give an easy explanation on the difference between aleatoric and epistemic uncertainty, and on quantile regression to a broad audience. And it was the first time I had been given such a systematic overview on uncertainty quantification.

### Title: Time series modelling with probabilistic programming

Link: [https://www.youtube.com/watch?v=tLao3IthrfI](https://www.youtube.com/watch?v=tLao3IthrfI)

Speaker: Sean Matthews, Jannes Quer

Summary: The talk presented an extrapolation problem in demand forecasting (the aggregated demand for drugs). It was remarkably different from many other data science talks in several respects.

1.  It did rather deal with seemingly old-school methods on a small dataset.
2.  The model choice was done extremely deliberately. For example, the speaker first applied standard methods such a Gaussian process regression and then demonstrated the need to go beyond, since the data had a secular event at the end of the sample data. In the end, he came up with a custom state-space model, and I would need to explore the literature a little further to really understand his final solution.
3.  The speaker was extremely explicit on the methods chosen and about the implementation, although the problem was not an academic one, but occurred in an industry context. (He showed parts of his `pystan` code explicitly.)
4.  The modelling was more of a one-off undertaking and not conceived for contributing to a productive model pipeline that is automatically retrained regularly. When I asked the speaker if he would recommend fitting the same model again after one year, the answer was a clear "No, since I don't know the future, I can't tell whether the model would still perform well then."

Although all this seems pretty much old school, I believe that we ML youngsters can learn a lot from the first generation data scientists: Asking whether we understand why a certain model performs best, and considering also well established methods and not only the ones at their current hype.

### Title: Automated Feature Engineering and Selection in Python

Link: [https://www.youtube.com/watch?v=4-4pKPv9lJ4](https://www.youtube.com/watch?v=4-4pKPv9lJ4)

Speaker: Franziska Horn

Summary: The speaker gave an overview on the typical feature engineering workflow in machine learning. She illustrated how this can be automated useing the [autofeat](https://github.com/cod3licious/autofeat) library. My impression was that this is a viable workflow when you start off with a new dataset, to get a good first iteration and to gain insight into the dataset. It however it seems not to be a tool that can automate away feature engineering completely when you are aiming for best-in-class predictions with a high degree of reliability and retraceability. Nevertheless it could a huge time-saver on the way to that goal.

### Title: Why you don’t see many real-world applications of Reinforcement Learning.

Link: [https://www.youtube.com/watch?v=HqK0wmIceH4](https://www.youtube.com/watch?v=HqK0wmIceH4)

Speaker: Yurii Tolochko

Summary: This talk was geared at pointing out the shortcomings of current reinforcement learning approaches for real-world problems. However, the speaker mainly talked about model free approaches. Examples were RIL steered robots walking into walls, RIL players not trying to complete the game because they could increase their score by continuing. These shortcoming fall into the following categories:

*   The algorithms require too much CPU time for exploration.
*   Modelling the rewards realistically can be cumbersome. Form what the speaker presented, it seemed that a model-free, formalistic mindset creates a tendency within the community to overlook this then starting a project.

The speaker also mentioned that even in the classic examples, such as alpha-go, the supremacy of RIL algorithms above human agents was often not really proven statistically. (For example, Google just reported one game where alpha-go beat the go world champion, without giving evidence that this was not just a coincidence, but statistically significant.)

Altogether, the speaker's conclusions seemed a little too pessimistic to me. For example, he said dynamic pricing based on reinforcement learning was basically impossible, which BY has already proven to be wrong. Nevertheless, the talk was very informative and I found it helpful to gain a sober view on reinforcement learning despite its current hype.

### Title: Law, ethics and machine learning - a curious ménage à trois

Link: [https://www.youtube.com/watch?v=FK-FmuEi5BU](https://www.youtube.com/watch?v=FK-FmuEi5BU)

Speaker: Dr. Benjamin Werthmann

Summary: First of all it really stood out of the mass of talks because Benjamin is not a Python enthusiast or even developer, but a lawyer. Benjamin gave an useful overview on GDPR (General Data Protection Regulation) and it's implications. What really made his talk stay in my mind was his call to action in the end. His message: the open source community must actively participate in topics of law and ethics in machine learning in order to get satisfying results, otherwise lawyers will do it.

### Title: Refactoring in Python: Design Patterns and Approaches

Link: [https://www.youtube.com/watch?v=ZzKaFJxiDzA](https://www.youtube.com/watch?v=ZzKaFJxiDzA)

Speaker: Tin Marković

Summary: Tim currently works at [Kiwi.com](http://Kiwi.com) and focussed his talk on reducing "code debt" by refactoring the code base regularly. The key elements he touched upon were easy wins, like automating some checks by using `black`, `mypy`, `coala`, etc., patterns that hint to "smelly code", and possible reasons ranging from "historical reasons" to "high priority urgent hacky requests". This is often not easily apparent, however, tools like "SonarQube" can help in identifying sections of code that might need refactoring. Another element that he talked about are over use of decorators, which may seem like a good idea but can lead to non-obvious functionality, and recommended that they be only limited, and shouldn't at least alter the function signature and calls. Another suggestion that I liked was around code reviews, in which it may be good idea to structure the reviews at overall scope, followed by system scope, and finally code scope. This can help save coding time in case the architecture or overall scope needs to be changed. Overall, the focus was to always keep an eye out for code debt and try to improve little by little.

### Title: 10 ways to debug Python code

Link: [https://www.youtube.com/watch?v=cokP4XAhcwo](https://www.youtube.com/watch?v=cokP4XAhcwo)

Speaker: Christoph Deil

Summary: Christoph's talk was targeted at beginners, as mentioned in the description, and more towards educating the audience about what they could focus to try learning if they're not sure where to start. He started with a high level overview of how Python executes code and how keeping that in mind and revisiting it can help clarify a few things to ourselves. Other standard tools include using `pdb`, `ipdb` for `ipython`, and of course, if one used an IDE, PyCharm and VS Code also provides visual debugging, which he himself prefers. If nothing seems to work, a rubber duck can be used to explain the error - which surprising also solves the problem and in case of unavailability, a colleague can be used as a replacement for the rubber duck. The talk could be beneficial for someone who is new and/or resorts to print statements.

### Title: Want to have a positive social impact as a data scientist?

Link: [https://www.youtube.com/watch?v=LMh3i1EXhp8](https://www.youtube.com/watch?v=LMh3i1EXhp8)

Speaker: Ellen Koenig

Summary: Ellen is a part of "Data Science for Social Good", Berlin chapter, and her talk was quite personal. She took the example of a relatable hypothetical lady "Alina" who wants to make a positive difference to the society but is not sure what to do. Ellen explained a structure approach any of us can take starting with identifying the social causes we feel strongly about and then picking 1-2 most pressing ones. The next steps would be to look for institutes or groups who are already working in the field and trying to initiate contact with them. With a specific personal goals articulated, like "reducing the number of homeless people in Berlin by 100", one can think of our skills and in what way can we help the already established groups. For example, the group may not even measure their effectiveness as specific as that, which is where Alina can help them. The most striking two things about her talk were - her advise was generic yet structured enough to be adapted to our personal interests and how just dedicating a little time, perhaps 2 hours a week, if that's all one can manage, can also lead to a positive effect and one need not feel overwhelmed and going on is more important.

### Title:  Apache Airflow for beginners

Link: [https://www.youtube.com/watch?v=YWtfU0MQZ_4](https://www.youtube.com/watch?v=YWtfU0MQZ_4)

Speaker: Varya

Summary: A good intro session if you do not know anything about Airflow yet. The basic building blocks of Airflow, like DAGs, Operators and Tasks are explained and a simple example shows how you can use them.

### Title:  Static Typing in Python

Link: [https://www.youtube.com/watch?v=p-nhGq-Wwv8](https://www.youtube.com/watch?v=p-nhGq-Wwv8)

Speaker: Dustin Ingram

Summary: Did you ever want static types in Python? Dustin Ingram shows you how to do it, as he explains the type system in Python. He discusses the pros and cons of static typing and how the Python typing module can help with bigger code bases.

### Title:  Is it me or the GIL?

Link: [https://www.youtube.com/watch?v=-5bRSrCMyH0](https://www.youtube.com/watch?v=-5bRSrCMyH0)

Speaker: Christoph Heer

Summary: Do you know what the GIL is? If not, this expert talk might be for you. Christoph Heer explains the global interpreter lock in Python and when it helps or limits us.   
He then takes a deep dive in the inner workings of the GIL to show us how he found out if the GIL really was the reason for the performance problems he was facing.

### Title: Mock Hell

Link: [https://www.youtube.com/watch?v=CdKaZ7boiZ4](https://www.youtube.com/watch?v=CdKaZ7boiZ4)

Speaker: Edwin Jung

Summary: For me, the most useful talk at PyCon. Edwin first presented what usually goes wrong when we start out implementing our unit tests: mocks and patches make it convenient to get rid of dependencies to databases and other external services, and so we use them everywhere. This leads to very brittle tests that break on every change in production code, even if it's a very smalll change. He argued that our confusion stems from a misunderstanding of test-driven development in object-oriented programming and contrasted "mockist" and "classicist" styles of unit testing. I came away from the talk with a lot more ideas about using different test doubles and the tradeoffs we have to make in TDD. By the way: a longer version of the talk was given at PyCon US and the recording is available on Youtube.

### Title: Break your API gently - or not at all

Link: [https://www.youtube.com/watch?v=qDILVhNTuBA](https://www.youtube.com/watch?v=qDILVhNTuBA)  

Speaker: Tim Hoffmann

Summary: Tim is a core developer of matplotlib and gave us a few hints on how to gently evolve a library's API. A key takeaway for me is that we can pro-actively avoid breaking changes by e.g. only using keyword-only arguments in our public functions so that parameters that need to be added later on don't break argument positioning.

### Title: What if I tell you that your specs are broken

Link: [https://www.youtube.com/watch?v=wd0-bPZD52Y](https://www.youtube.com/watch?v=wd0-bPZD52Y)

Speaker: Samuele Maci

Summary: Wouldn't it be great if we could automatically prevent introducing breaking changes in our HTTP API's? Samuele, a backend engineer for Yelp, demonstrated how to do this using Swagger to enforce frequently changing API specifications.

### Title: How strong is my opponent? Using Bayesian methods for skill assessment

Speaker: Darina Goldin

Link: [https://www.youtube.com/watch?v=eKHh4nFpklE](https://www.youtube.com/watch?v=eKHh4nFpklE)

So as the day 2 of conference was ending, I felt like going for a talk which is no related to any of my general interests. After hearing this talk, I am a big fan of operations research now. Darina gave a good overview of players are ranked in the gaming industry and also gave a example of how the good old Elo algorithm ( which had a brief inaccurate cameo in The Social Network) can be used in the gaming industry. I was quite impressed by the examples she used to explain the various ranking algorithms. The Q and A session was quite funny where the audience was thinking of various other (non-competitive applications) for using this algorithm but as far as I know, China uses these algorithms in all the industries to rank their employees (Geez!)

### Title: Time Series Anomaly detection for Bottling Machine Maintenance

Speaker: Andrea Spichtinger

Link: [https://www.youtube.com/watch?v=5ChQ6Ijt8j4](https://www.youtube.com/watch?v=5ChQ6Ijt8j4)

Summary: Interesting industry use case of semi-supervised learning using anomaly detection to detect possible failures of bottling machines. They had a baseline dataset which they used to define the "correct" state for the machines to run, and used this as training. The algorithm then detected anomalies. She explained several of the challenges of this process, and possible future improvements (one possibility was to actually add microphones and use this audio data during training)

### Title: Does hate sound the same in all languages?

Link: [https://www.youtube.com/watch?v=xFTTHCH4yiQ](https://www.youtube.com/watch?v=xFTTHCH4yiQ)

Speaker: Andrada Pumnea

Summary: Andrada concisely walked us through her process of building an NLP (natural language processing) solution detecting hate speech. Her motive was a referendum in Romania in October 2018 to define the family as exclusively heterosexual which lead to massive hate speech against the LGBT community on social media. She explained the issues she encountered while labeling data (social media posts) as hate speech and some examples of her model performance. What I found very interesting was her explanation about NLP models that work on a level of abstraction above a single language e.g. XLM and Laser (they are able to detect linguistic/grammatical structures in multiple languages). She showed performance of various kind of these models on hate speech datasets in several distinct languages. This is a very interesting development and opens the NLP landscape further, as until now the availability and quality of pre-trained NLP models and datasets varied greatly among languages, being much better in English than in language with a relatively small aomunt of speakers (such as Romanian).

### Title: Gaussian Progress

Link: [https://www.youtube.com/watch?v=aICqoAG5BXQ](https://www.youtube.com/watch?v=aICqoAG5BXQ)

Speaker: Vincent Warmerdam

Summary: Vincent provided a very intuitive conception of Gaussian Process, and then gradually extended this intuition to more complex algorithms. It was very well presented (even included successful live coding) and very helpful in understanding this concept.