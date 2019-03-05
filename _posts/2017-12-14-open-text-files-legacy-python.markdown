---
layout: single
title: "Open text files in legacy Python"
date:   2017-12-14 11:32:51
tags: technology python
header:
  overlay_image: assets/images/2017-12-14-typewriter.jpeg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Matthias Bach
---

At a first glance, **opening files** in Python is easy. All you have to do is call the built-in function `open()` and then you start reading from the file. However, often the **content** to be read is **text**, not just a binary stream. In this case encodings become relevant. In [modern Python](https://docs.python.org/3/) this is not much of an issue, as the `open()` has a parameter `encoding` that allows to specify just that. Even if unspecified, it will use the value from `locale.getpreferredencoding(False)` which will usually make everything behave just as expected. Sadly, [legacy Python](https://docs.python.org/2/) is still around and it does not offer this parameter.

So how can **encodings** of a file be handled properly? The trivial approach, of course, would be to open the files in binary mode and manually utilize the `encode()` and `decode()` methods to take care of the encoding. This, however, is tedious and breaks at the moment you have to pass the file pointer to another library that expects that it can directly read from or write to it. Curiously, [legacy Python's file object](https://docs.python.org/2.7/library/stdtypes.html#file-objects) does have [the encoding attribute](https://docs.python.org/2.7/library/stdtypes.html#file.encoding) and conversion logic. It is just not possible to set it when using the standard built-in `open()` function. Luckily, there is a function providing this functionality: `[codecs.open()`](https://docs.python.org/3.6/library/codecs.html#codecs.open). [This function](https://docs.python.org/3.6/library/codecs.html#codecs.open) is available across Python versions and should thus be the first choice when handling text files.