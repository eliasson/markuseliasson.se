+++
Categories = ["Code"]
Description = "Python pip virtualenv in Windows"
Tags = ["Python", "Windows"]
date = "2015-02-24T22:00:33+01:00"
title = "Python pip virtualenv in Windows"

+++

I needed to setup my Windows 8 machine for Python and Django development as I
was fed up with running virtualbox and I really wanted to run PyCharm native
in Windows. I jotted down these simple steps for future reference, or if
someone else needs a brief overview.


## Install python

I chose the official Python Windows distribution, the later versions comes
with pip bundled. All I needed to do was to add two directories to my PATH:


    SET PATH=%PATH%;C:\<path-to-python>\;C:\<path-to-python>\Scripts\


## Setup a new virtualenv

I do not use PowerShell and did not bother trying to find an alternative
for virtualenvwrapper that I use in bash. Instead I just roll my virtualenvs
manually. First thing you need to do is to install virtutalenv:


    pip install virtualenv


Creating a virtual environment by running:


    virtualenv C:\virtualenvs\foo-env


Activate this virtual environment by running:


     C:\virtualenvs\foo-env\Scripts


From this stage it was only a matter of importing my project to PyCharm and
select the right Python Interpreter.

A side note, as I use Makefiles to wrap my Django tasks I can really
recommend [gow](https://github.com/bmatzelle/gow) a lightweight unix util
distribution for windows.
