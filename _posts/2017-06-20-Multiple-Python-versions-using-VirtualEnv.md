---
layout: post
title:  Running multiple Python Versions Using Virtual Envs
date:  Tue Jun 20 13:30:25 IST 2017
categories: python
tags: python virtualenv
author: Abhinav Kumar
mathjax: true
---

Running multiple python versions using Virtual Envs

step1: Download python versions you want to run.

step2: virtualenv -p {python_location} {env_name}

step3: (for mac) . env_name/bin/activate

For example (Running Python 3.6):
```
~ abhinavkumar$ virtualenv -p /usr/local/bin/python3.6 py36
Running virtualenv with interpreter /usr/local/bin/python3.6
Using base prefix '/Library/Frameworks/Python.framework/Versions/3.6'
New python executable in /Users/abhinavkumar/py36/bin/python3.6
Also creating executable in /Users/abhinavkumar/py36/bin/python
Installing setuptools, pip, wheel...done.
~ abhinavkumar$ . py36/bin/activate
(py36) ~ abhinavkumar$ which python
/Users/abhinavkumar/py36/bin/python   
Python 3.6.1 (v3.6.1:69c0db5050, Mar 21 2017, 01:21:04)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
```
Running python 2.7
```
~ abhinavkumar$ virtualenv -p /usr/bin/python2.7 py27
Running virtualenv with interpreter /usr/bin/python2.7
New python executable in /Users/abhinavkumar/py27/bin/python
Installing setuptools, pip, wheel...done.
~ abhinavkumar$ . py27/bin/activate
(py27) ~ abhinavkumar$ python
Python 2.7.10 (default, Oct 23 2015, 19:19:21)
[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
```
You don't need to do this everytime, this is one time job. Once created you just have activate it and once done you can deactivate.

Additionally working with virtualenv help you to segregate your different packages versions, without messing up with your systems settings.