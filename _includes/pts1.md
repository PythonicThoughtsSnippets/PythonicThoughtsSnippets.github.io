# 1: Dictionaries for Control Flow
_19.08.2019_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-1.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

We are all well adapted to the if...elif...else: control flow statements. But these statements some times (well... many times) just become too many or they instead encapsulate many lines of codes inside. At a given moment, or above certain complexity, it becomes awkward and difficult to follow the flow of the code and to retain all the nesting in one's brain's RAM. "Flat is better then nested", isn't it? :)

So, with this idea in mind, one day I stared at my code looking for a way to flat those ifs. Well, I've not invented anything new. What I write here is the result of many readings on books and forums. But the question here is not just reading and understanding others; is, instead, to create a shift in one's mindset after which makes no sense to do otherwise.

Here's how I am feeling about control flow where several options have to be considered: instead of if statements, lets map the flow options under a dictionary!

```python
def translate_ifs(val):
    
    if isinstance(val, str):
        return val
    
    elif isinstance(val, int):
        return str(val)
    
    elif bool(val is False):
        return 'F'
    
    elif bool(val is True):
        return 'T'
    
    else:
        raise SomeError('My control Flow Error!')


# evaluates WITHOUT else statement
def translate_1(val):
    
    d = {
        isinstance(val, str): val,
        isinstance(val, int): str(val),
        bool(val is False): 'F',
        bool(val is True): 'T',
        }
    
    try:
        return d[True]
    except IndexError:
        pass  # handle exception here


# evaluates WITH else statement
def translate_2(val):
    
    d = {
        True: 'Happy Flowers!',
        isinstance(val, str): val,
        isinstance(val, int): str(val),
        bool(val is False): 'F',
        bool(val is True): 'T',
        }
    
    return d[True]


# Because functions in Pyhon are first class objects...
def translate_3(val):
    
    d = {
        True: func1,
        isinstance(val, str): func2,
        isinstance(val, int): func3,
        bool(val is False): func4,
        bool(val is True): func5,
        }
    
    return d[True](val)


# # you can even use partials to prepare function calls
def translate_4(val):
    
    from functools import partial
    
    d = {
        True: partial(func1, some_val),
        isinstance(val, str): partial(func2, some_val),
        isinstance(val, int): partial(func3, some_val),
        bool(val is False): partial(func4, some_val),
        bool(val is True): partial(func5, some_val),
        }
    
    return d[True]()
```
