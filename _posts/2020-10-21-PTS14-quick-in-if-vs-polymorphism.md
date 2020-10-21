---
title: "PTS 14: Quick assessment of `if` vs. polymorphism - simple case"
tags: [Python, Python 3.8, Python Programming, polymorphism, if-statements]
author: "Joao MC Teixeira"
---
_21.10.2020_ - [Back to top](https://pythonicthoughtssnippets.github.io)


Now that I am back to programming in Python, I want to continue populating the Pythonic Thoughts Snippets web site with more posts. Today I had to decide on whether to apply polymorphism between two functions or use an `if`-statement. I dislike to `if`, and use polymorphism as much as possible. However, some times the overhead does not compensate using the polymorphism. The abstration bellow is an example of it.

I still don't know how much I will need to extend the code with different cases, most likely it won't be extended much more than 2 or 3 cases. So, maybe the `if` can stand still.


```python
from random import choice as randc

list_ = range(100_000)
```


```python
# I need to mask randc into func1
# and receive **kwargs
def func1(a=randc, b=list_, **kwargs):
    return randc(list_)
```


```python
%%timeit
func1(cres=1)
```

    550 ns ± 12.4 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)



```python
%%timeit
if None is None:
    randc(list_)
```

    440 ns ± 8.16 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)



```python
%%timeit
if True is None:
    pass
else:
    randc(list_)
```

    439 ns ± 3.14 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)


With the `if`-statement is just faster. And yes, this time I really need speed.

Cheers,
