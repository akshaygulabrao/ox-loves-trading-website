---
layout: post
title:  "My python setup"
date:   2024-07-30 16:05:55 -0400
categories: python
---
A lot of you have been asking my about my python setup (literally no one) using Mac. The hardest part about setting up python is the interdependency between system python in (/usr/bin/python3), homebrew, and pyenv. 

I no longer think homebrew is good for package management in python. The problem comes when you need to deal with multiple versions. A stack that solves this issue really well is pyenv + poetry. Now using poetry and pyenv is hard to starting out and there is a bit of a learning curve. 

Using pyenv with poetry makes reproducing python environments in foreign github repos trivial. You definitely do not want to be messing with conda workspaces every time (which is good for basic data science) or venv (struggle with sharing dependencies across machines). 