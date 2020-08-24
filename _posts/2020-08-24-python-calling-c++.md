---
layout: single
title: "Python calling C++"
date:   2020-08-24 12:55:28
tags:  python c++ technology
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Aditya Gupta
author_profile: true
---

# Python calling C++

##What is the motivation for accessing C++ code from Python?

Blue Yonder has few projects in which few segments of code are in Python and few in C++ and hence it makes for an important use case to find some methodology to make these two languages communicate with each other efficiently. It is extremely crucial to find a tool for Python's seamless integration with the code written in C++.
 
Before diving deeper into how to access C++ code from Python, let us try and understand why or under what circumstances would we want to do that: 

1. **You already have a large, tested, stable library written in C++** that you’d like to take advantage of in Python. This may be a communication library or a library to solve a specific purpose in the project. 

2. **You want to speed up a section of your Python code** by converting a critical section to C++. Not only does C++ have faster execution speed, but it also allows you to break free from the limitations of the Python Global Interpreter Lock (GIL). 

3. **You want to use Python test tools** to do large-scale testing of their systems. 

One of the method to access C++ code from Python is to write a python interface and place python bindings on it in order to give python access, we can do that or use a pre-built tool such as BoostPython library which is much easier to do.  But before going into details of this method let’s see what are the possible ways of combining Python and C++.

![python calling c++](/assets/images/2020-08-24-python-calling-c++.png)

##What are possible ways of combining Python and C++? 

There are two basic models for combining C++ and Python: 

1. **Extending,** in which the end-user launches the Python interpreter executable and imports Python extension modules written in C++. It’s like taking a library written in C++ and giving it a Python interface so Python programmers can use it. From Python, these modules look just like regular Python modules. Extending is writing a shared library that the Python interpreter can load as part of an import statement. 

2. **Embedding,** in which the end-user launches a program written in C++ that in turn invokes the Python interpreter as a library subroutine. It’s like adding scriptability to an existing application. Embedding is inserting calls into your C or C++ application after it has started up in order to initialize the Python interpreter and call back to Python code at specific times. 

Note that even when embedding Python in another program, extension modules are often the best way to make C/C++ functionality accessible to Python code, so the use of extension modules is really at the heart of both models. Embedding takes more work than extending. Extending gives you more power and flexibility than embedding. Many useful python tools and automation techniques are much harder, if not impossible, to use if you're embedding. 

BoostPython library discussed above is used to quickly and easily export C++ to Python such that the Python interface is very similar to the C++ interface. Due to its advantage of being fast and convenient to be able to extend C++ libraries to Python we’ll be learning about it in this blog and how we can use it to make Python and C++ talk to each other. But before learning about it lets first find out what is Python Binding and why is it required. 

##What is Python Binding and why is it required?

Binding generally refers to a mapping of one thing to another. In the context of software libraries, bindings are wrapper libraries that bridge two programming languages, so that a library written for one language can be used in another language. Many software libraries are written in system programming languages such as C or C++. To use such libraries from another language, usually of higher-level, such as Java, Common Lisp, Scheme, Python, or Lua, a binding to the library must be created in that language, possibly requiring recompiling the language's code, depending on the amount of modification needed. Python bindings are used when an extant C or C++ library written for some purpose is to be used from Python. 

To understand why Python bindings are required let’s take a look at how Python and C++ store data and what types of issues this will cause. C or C++ stores data in the most compact form in memory possible. If you use an uint8_t, then the space required to store it would be approximately 8 bits if we don’t take structure padding into account. 

In Python, on the other hand, everything is an object and the memory is heap allotted, integers in Python are big integers and their size may vary according to the value stored in them. This means that our Python bindings will need to convert a C/C++ integer to a Python integer for each integer passed across the boundary. 

The process of transforming the memory representation of an object to a data format suitable for storage or transmission is called Marshalling and Python bindings are doing a process similar to Marshalling when they prepare data to move it from Python to C or vice versa. 

##What is BoostPython library?  

The Boost Python Library is a open source framework for interfacing Python and C++. It allows you to quickly and seamlessly expose C++ classes functions and objects to Python, and vice-versa, using no special tools, just your C++ compiler. It is designed to wrap C++ interfaces non-intrusively, so that you should not have to change the C++ code at all in order to wrap it, making Boost.Python ideal for exposing 3rd-party libraries to Python. The library's use of advanced metaprogramming techniques simplifies its syntax for users, so that wrapping code takes on the look of a kind of declarative interface definition language (IDL). It is designed to be minimally intrusive on your C++ design. In most cases, you should not have to alter your C++ classes in any way in order to use them with Boost.Python. The system should simply reflect your C++ classes and functions into Python. 

The BoostPython Library binds C++ and Python in a mostly seamless fashion. It is just one member of the boost C++ library collection at [http://www.boost.org](http://www.boost.org). Boost.Python bindings are written in pure C++, using no tools other than your editor and your C++ compiler. 

##Relationship to the Python C API 

Python (more specific: CPython, the reference implementation of Python written in C) already provides an API for gluing together Python and C in the form of Python C API. Boost Python is a wrapper for the Python/C API. 

Using the Python/C API, you must deal with passing pointers back and forth between Python and C and worry about pointers hanging out in one place when the object they point to has been thrown away. Boost Python takes care of much of this for you. In addition, Boost Python lets you write operations on Python objects in C++ in OOP style.

##Simple Example

So now that we are done with the theoretical aspects of it, let’s get our hands dirty with some coding and see how it can be used through a short example. 

Before we get to the actual coding part let’s see what the prerequisites are to run a program with a boost library: 

1. [Boost](https://www.boost.org/) (version >= 1.3.2) 

2. [Python](http://www.python.org/) (version >= 2.2) 

3. A C++ compiler for your platform, e.g. [GCC](https://gcc.gnu.org/) or [MinGW](http://www.mingw.org/) 

Suppose we have the following C++ API which we want to expose in Python:

```cpp
#include <string> 

namespace 
{   
    // Avoid cluttering the global namespace. 

    // A couple of simple C++ functions that we want to expose to Python. 
    std::string greet() { return "hello, world"; } 
    int square(int number) { return number * number; }
} 
```

Here is the C++ code for a python module called getting_started1 which exposes the API. 

```cpp
#include <boost/python.hpp> 
using namespace boost::python; 
 
BOOST_PYTHON_MODULE(getting_started1) 
{ 

    // Add regular functions to the module. 
    def("greet", greet); 
    def("square", square); 
} 
```  
That's it! If we build this shared library and put it on our PYTHONPATH we can now access our C++ functions from Python. 

```python
>>> import getting_started1 
>>> print getting_started1.greet()
 
hello, world 

>>> number = 11 
>>> print number, '*', number, '=', getting_started1.square(number) 

11 * 11 = 121 
``` 

So, as you can see from the example above the only additional library required to run our program is the Boost library apart from the regular C++ and Python compiler and all you need to do is compile and build the C++ library and import it in Python as a regular Python library and boom you’re ready to go.

##Exposing classes written in C++ to Python 

Now that we have explored running a simple Hello World program in Python which is written in C++, let’s see how we can do the same with C++ classes through an example. 

Assume we want to expose the below written C++ class to Python.    
```cpp
struct World 
{ 
    void set(std::string msg) { mMsg = msg; }
    std::string greet() { return mMsg; }
    std::string mMsg;
}; 
 
using namespace boost::python; 
 
BOOST_PYTHON_MODULE(classes) 
        { 
                class_<World>("World") 
                        .def("greet", &World::greet) 
                        .def("set", &World::set) 
                ; 
        }; 
```  
The corresponding Python code to call the above C++ class from Python would be: 

```python
import classes 
 
t = classes.World() 
t.set("Python says hi to C++.") 
print (t.greet())
``` 
The output of running the above code would be: 

```python
/Users/adityagupta/PycharmProjects/check/venv/bin/python /Users/adityagupta/PycharmProjects/check/Classes.py 

Python says hi to C++. 

Process finished with exit code 0 
``` 

##Conclusion 

This is just the tip of the iceberg and there could be several use cases for making the two languages talk, especially while taking the edge of data science in the existing projects or while using the best features of both the languages.  Although Boost.Python is a great tool for a quick start up, but it has few drawbacks like packaging projects that have native dependencies could be challenging and since it’s a python run-time dependency resolution it might be difficult to use all the C++ features especially templates but having said that it’s a great tool to getting started and running at least the basic C++ code as Python libraries and opening up possibilities to a whole new world. 