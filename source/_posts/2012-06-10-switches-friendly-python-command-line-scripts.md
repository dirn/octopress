---
layout: post
title: "Switches: Friendly Python Command Line Scripts"
date: 2012-06-10 20:49
comments: true
categories: [Python, Friendly]
---

`argparse` is an awesome module. It can do a lot of really cool things.
Unfortunately it requires a lot of boilerplate to set up all of the commands
and any arguments associated with them. Plus, once you're done with all of
that, you still need a way to tie each of those commands to some action in
your script.

<!-- more -->

Switches is a Python module that aims to make the whole process easier. It
uses the code you're already writing to cut out the boilerplate.

``` python Add a command
@command
def spam():
    """This function prints 'eggs'"""
    print 'eggs'
```

This is all the code that's needed to add a command to your script called
`spam`. This command doesn't do a whole lot, though, and odds are you're going
to want to add a command that can take some arguments.

``` python Add a command with an argument
# Positional argument
@command
def spam(eggs):
    """This function prints the value of eggs"""
    print eggs

# Optional arguments
@command
def eggs(spam='spam'):
    """This function prints the value of spam"""
    print spam
```

Once all of your commands have been written, invoking the whole thing only
requires one additional line, a call to `commandline()`. `commandline()` takes
one optional (and recommended) parameter, `description`; it will give the
script a nice summary when users passing in the `--help` argument.

{% gist 2908622 %}

Check out the documentation on
[Read the Docs](http://readthedocs.org/docs/switches/en/latest/)
or get started with `pip install switches` or
[Fork it on GitHub](https://github.com/dirn/switches)
