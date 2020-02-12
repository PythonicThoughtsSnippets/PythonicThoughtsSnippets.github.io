---
title: "PTS 9: Args packing after non-default arguments
tags: [python, python functions, functions, arguments, keyword arguments, positional arguments]
author: "Joao MC Teixeira"
---
[Back to top](https://pythonicthoughtssnippets.github.io)

Some days ago I was writing a Python function; it came out something like this:

```python
# toy example of the function I wrote
def myfunc(arg1, arg2, arg3='some string', *args):
    # do magic
    return
```

And I went one doing my life normally. Couple of hours later, flash came to my mind and I just stopped. _Whait_, did that just work? Indeed my function was passing all the tests I had set, but I was just puzzled.

Did you notice why? Is really that obvious, and I was not aware?

The function above just defines a _non-default_ argument **before** an argument packing operation. Was that expected?

Definitively my _brain_ wrote it naturally, so naturally that my conscious me didn't even noticed, but I tell that this was the first time I wrote this syntax in the almost 5 years I Python. There are basically two revelations here:

The **first** is that we can actually `*args` after a non-default argument `arg3='something'`. I had never seen this example before in any tutorial, _how-to_, whatsoever, about argument packing and unpacking with `*args` and `**kwargs`. The most complex example provided in _tutorials_ out there is:

```python
def myfunc(arg1, arg2, *args, kw1=1, kw2='string', kw3=None, **kwargs):
    # magic happens
    return
```

**Secondly**, my _brain_ is getting more Pythonic than I! I was actually writing something completely new for me without even realizing; simply, expecting it to work. And only noticed some hours later.

Did you knew this was possible? [Let me know](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/issues/9).


