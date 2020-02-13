---
title: "PTS 9: Args packing after non-default arguments"
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

And I went on doing my life normally. A couple of hours later, a flash came to my mind and I just stopped. _Whait_, did that just work? Indeed my function was passing all the tests I had set, but I was just puzzled.

Do you notice why? Is it really that obvious and I was not aware?

The function above just defines a _non-default_ argument **before** the argument packing operation `*args`. Was that expected?

Definitively my _brain_ wrote it naturally, so naturally that the conscious me didn't even noticed, but I can tell that this was the first time I wrote such syntax in these almost 5 years I Python. There are basically two revelations here:

**Firstly** is that we can actually `*args` after a non-default argument `arg3='something'`. I had never seen this example before in any tutorial, _how-to_, whatsoever, about argument packing and unpacking with `*args` and `**kwargs`. The most complex example provided out there is always of this kind:

```python
def myfunc(arg1, arg2, *args, kw1=1, kw2='string', kw3=None, **kwargs):
    # magic happens
    return
```

For someone experienced in argument packing and unpacking there is no magic in the above example, just everyday living. But the syntax I encountered today was definitively not expected.

**Secondly**, my _brain_ is getting more Pythonic than me! I was actually writing something completely new for me without even realizing; simply, expecting it to work. And only noticed some hours later.

Did you knew this was possible? [Let me know](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/issues/9).


