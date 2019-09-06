# Motivation
I created the Pythonic Thoughts Snippets series out of the need to share my connection with the code I write, how my vision and style evolve, how my mindset for code writing shapes with time in a continuous quest towards writing more maintainable, easy to read, robust and beautiful code. I hope you find these little articles of interest and also share your comments. These posts are the result of many online and book readings, discussions with others and hours spent staring at the code. Please consider I am not a purely computational person, I am a biochemist passionate about coding that have learned my way through this beautiful discipline. - Joao M.C. Teixeira, 2019.

# 0: Parameter Validation on Class Creation
_19.08.2019_
[download the code snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-0.py)

This post serves as tutorial, question to the audience and challenge. I will explain how I solved the problem but I am yet far to be sure about the most correct and Pythonic way to address this question.

For the project I currently have at hands, I have to design a series of classes (I call it modules because of the project's scope, nothing to do with python modules) that can be initiated with a bunch of keyword parameters. I am assigning those parameters to @property attributes inside the class because I want to control the getter and setter methods when accessing those attributes. So far so good and nothing special.

The setter method implement the logic that should accept or reject the values to be assigned to the attributes. This is critical to the project because my classes are bound to a very precise biological and physical meaning. When a wrong value is introduced an error raises.

I want to extend the implementation of the classes to the possibility to force its instantiation even with non-valid values. For that I've introduced a "build" parameter that bypasses the errors risen and assigns the value to its corresponding dunder attribute.

Additionally, I implemented a "testit" option that tests the class instantiation for a given set of parameters and reports on possible value errors by returning Report class object instead of the class itself.  Here's an example:

```python
# Example

c = Module1(param1=1)
# this would rise error because param1 must be
# negative int.

c = Module1(param1=1, testit=True)
# this would return a Report class with a representation
# of the setter validation errors

c = Module1(param1=1, build=True)
# this would bypass the error and instantiate the
# class nonetheless with attribute _param1 set to 1.
```

"MyModule" classes inherit from a ModuleBase class where the logic takes place. I've used a combination of `__new__` and `__init__` methods to implement the desired functionalities.

I don't think that Abstract Factories or Factory Methods are relevant here because I don't look for instantiation of different subclass depending on a context.

Here's how I implemented it, let me know your thoughts and comments, I really appreciate any discussion.

The main problem with this implementation is the need to add an `__init` bool variable because when `return c` the `__init__` method is called again.

```python
import inspect

class ModuleBase:

    __init = False

    def __new__(cls, *args, testit=False, build=False, **kwargs):

        if testit or build:
            c = cls(*args, forceme=True, **kwargs)
            if testit:
                return c._initreport
            elif build:
                return c
        else:
            return super().__new__(cls)

    def __init__(self, frame, forceme=False, **kwargs):

        if not self.__init:
            self._forceme = forceme
            self.__init = True
            self._initreport = ParameterValidationReport()
            self.params, _, _, values = inspect.getargvalues(frame)
            self.params.remove('self')

            for param in self.params:
                try:
                    setattr(self, param, values[param])
                except MyCustomError as e:
                    if forceme:
                        self._initreport.append_msg(repr(e))
                        setattr(self, f'_{param}', values[param])
                    else:
                        raise e

class Module1(ModuleBase):
    def __init__(
            self,
            *,
            param1=-1,
            param2=4,
            **kwargs
            ):

        frame = inspect.currentframe()
        super().__init__(frame, **kwargs)
        return
```

# 1: Dictionaries for Control Flow
_19.08.2019_
[download the code snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-1.py)

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

# 2: Unpacking Conditional Expressions
_15.08.2019_
[download the code snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-2.py)

Hello,

Yesterday (August 15th, 2019) I decided to initiate a series of posts dedicated to my thoughts and experiences with and in Python programming, I called it Pythonic Thoughts Snippets. I had this entry previously in the blog and now I am recovering it to inside the series. It is not exactly a Pythonic Thought, it is actually a Pythonic Encounter, as I originally called it, but I decided to add it to the PTS series so to glue all together.

Here goes the post entry as I originally wrote it:

I’ve been using [Python conditional expressions](https://www.python.org/dev/peps/pep-0308/) for a while now, at least the simplest in the form of:

```
expression1 if condition else expression2
```

However, yesterday I was faced with a situation that although now looks very obvious on the moment of typing this little detail was not sparking in my mind.

```python
# pretty clear code that makes use of list and zip
# to unpack a list of tuples to two separated lists of tuples
# or in this case just a tuple because there is only one element
a = [("1", "2"), ("", "")]
b, c = list(zip(*a))
print(b)
print(c)
('1', '')
('2', '')
```

I wanted to make use of conditional expressions to pass two lists with empty strings to variables b and c in the case the condition was evaluated to False. However, I was receiving an unexpected output (and took me a while to understand from where and why).

So, what is wrong with the following expression?

```python
# here I would expect the same as before, however...
a = [("1", "2"), ("", "")]
# notice that the if True is just an example of 
# a conditional that I was expecting to evaluate to True in
# most cases.
b, c = list(zip(*a)) if True else [""], [""]
print(b)
print(c)
# ¡unexpected output!
[('1', ''), ('2', '')]
['']
```

Why was the two lists with empty strings inside not being passed to b and c? It was only after I posted [a thread in Python-forum.io}(https://python-forum.io/Thread-ternary-operator-with-unpacking) that a spark light on my mind!

Solution: The conditional expression ended in the comma (,), so the result from the conditional was being assigned to b and the `[“”]` to c. Hence, the following code gave the expected output:

```python
a = [("1", "2"), ("", "")]
b, c = list(zip(*a)) if True else ([""], [""])
print(b)
print(c)
('1', '')
('2', '')
```
And that’s it, my yesterday Pythonic encounter. The solution was quite easy and obvious, but, interesting nonetheless. Take home message, sometimes is better to enclose expressions in brackets, it corrects and clarifies things :-P

Cheers

# 3: logging library wide and to end-users
_26.08.2019_
[download the code snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-3.py)

I have start using the [Python logging library](https://docs.python.org/3/library/logging.html) about one year ago it is definitively a powerful lib that you (maybe) should embrace on your code development.

There are countless of web pages with howtos and tutorials on how to set a  logging system for your script. However, there are not so many on how to set a logging system library-wide, thought you can still find some, and the official documentation has examples.

However, I always felt that a precise explanation on how to properly set a logging system library wide is not easy to find. Specially, if your project focus on both library development and nondev end-user distribution.

In this cases it is necessary to set up the logging such that it provides proper output when using the libraries (your project) and when using the software by end-users. Normally, end-users will expect, and you as a developer also need such, some .log file and .debug file with which the communication between users and developers is facilitated. On the other hand, users of your libraries won't need such files and almost certainly a messaged printed to the sys.stdout suffices.

So the question remains on how to properly set up such difference interfaces. At the end, everything looks quite obvious but, honestly, it took me quite a bit of time to figure it out and have the desired behaviour.

It is definitively not straightforward because we will be making use of advanced concepts like Python singletons (that are different from other languages), and the realization that the logging can be set dynamically. :-)

This is the solution I came by. Please share your thoughts if you have a different or better approach. I am far from being a full expert on the Python logging library.

```
# Lets consider the following project structure:
MyApp/
    __init__.py
    base.py (here is where your main API interface goes)
    logger.py
    package1/
        __init__.py
    (... etc ...)
```

Inside logger we define our logger configuration (looks obvious right?). Bellow how I have those of my projects set.

As a common practice in my Pythonic Thoughts Snippets, I am not going into the details of the minor implementations, on doubts please search elsewhere on the web for the particular code snippet, there are countless of amazing explanations; here, we focus on the broader concept.

**logger.py**

```python
import os
import sys
import contextlib
from pathlib import Path
import logging
import logging.handlers

    
info_fmt = logging.Formatter("%(message)s")
    
def init_logger(name):
    
    # here we set the global logger that only print to the
    # SYS STDOUT. In this way when your libraries are covered
    # with a logging system when used by other developers.
    # yet a .log and .debug files are not created.
    
    logger = logging.getLogger(name)
    logger.setLevel(logging.DEBUG)
    
    # print to console
    ch = logging.StreamHandler(stream=sys.stdout)
    ch.setLevel(logging.INFO)
    ch.setFormatter(Logger.info_fmt)
    
    logger.addHandler(ch)
    
    self.log = logger

def set_enduser_logger(logger, path):
    """
    Sets log files when base API is used buy end users.
    """
    # the strategy on how to set up the actually logger is up to you.
    # here I should you the one I use for my projects.
    
    # I like to have info and debug files separated.
    # users can use info for their work and debug serves to communicate
    # between users and devels
    info_filename = os.fspath(Path(path, Logger.info_filename).resolve())
    debug_filename = os.fspath(Path(path, Logger.info_filename).resolve())
    
    with contextlib.suppress(FileNotFoundError):
        os.remove(debug_filename)
        os.remove(info_filename)
    
    debug_fmt = logging.Formatter(
        "%(message)s"
        "`%(levelname)s - "
        "%(filename)s:%(name)s:%(funcName)s:%(lineno)d`\n"
        )
    debughandler = logging.handlers.RotatingFileHandler(
        debug_filename,
        maxBytes=10485760,
        mode="a",
        )
    debughandler.setFormatter(debug_fmt)
    debughandler.setLevel(logging.DEBUG)
    
    debugmemhandler = logging.handlers.MemoryHandler(
        1000,
        target=debughandler,
        flushLevel=logging.ERROR,
        )
    
    debugmemhandler.setFormatter(debug_fmt)
    debugmemhandler.setLevel(logging.DEBUG)
    
    infohandler = logging.handlers.RotatingFileHandler(
        info_filename,
        maxBytes=10485760,
        mode="a",
        )
    
    infohandler.setLevel(logging.INFO)
    infohandler.setFormatter(Logger.info_fmt)
    
    infomemhandler = logging.handlers.MemoryHandler(
        1000,
        target=infohandler,
        flushLevel=logging.ERROR,
        )
    
    infomemhandler.setLevel(logging.INFO)
    infomemhandler.setFormatter(Logger.info_fmt)
    
    # add handlers
    logger.addHandler(debugmemhandler)
    logger.addHandler(infomemhandler)
    
def silence(logger):
    logger.setLevel(logging.WARNING)

def unsilence(logger):
    logger.setLevel(logging.DEBUG)
```

Now, in the `__init__.py` of the project's base folder we will set up the logger singleton :) singletons in Python are simply variables in a module

I do it this way:

**__init__.py**

```python
import MyApp.logger as LOGGER
log = LOGGER.init_logger(__name__)
```

Now from every other module MyApp.log is the log handler. In the base.py API (aimed to be used by end users and not by lib users) the .info and .debug files are setup at the `API.__init__` by calling

**base.py**

```python
from MyApp import log
from MyApp.logger import set_enduser_logger
class MyApp:
    def __init__(self):
        set_enduser_logger(log, self.resultsp)
```

Likewise in the other app's libs we can use the same approach but without `.set_enduser_logger`.

```python
from MyApp import log
from MyApp.logger import silence, unsilence
class MyPackage1:
    def __init__(self):
        self.log = log  # this is optional, you can just ust log directly
        
        # you can silence the log easily:
        silence(log)
        log.info('this will be silenced')
        
        unsilence(log)
        log.info('back again')
```

The reason why I am not using a Logger class to encapsulate the log is because funcName will trace the method of the Logger class instead of the function (code position) of interest to record; [see here an example](https://stackoverflow.com/questions/19615876/showing-the-right-funcname-when-wrapping-logger-functionality-in-a-custom-class).

I hope you find this entry useful,
Best,

# 4: Factory Method Creational Pattern
_31.08.2019_
[download the code snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-4.py)

Some things are not trivial to learn, other are even difficult, but some are not about learning, but actually incorporating behaviours and methodologies in our own practices, and make their use a natural process.

It was definitively not one nor twice that I read descriptions and explanations about the Factory Method as a Creational Pattern. What are those concepts? Look around for Object-Oriented Programming concepts and the associated (most standard) Design Patterns. There are many (yet maybe not that many) websites and books about these topics, here's one website I have as reference ([Source Making](https://sourcemaking.com/design_patterns/factory_method)).

But, despite all my cheerful readings on the matter and struggles with my code, just the other day I came to realize how to properly build this pattern and in which situations apply it.

**EDIT (05.09.2019):**
_ There's a very interesting fault in the design shown in the original example of this post, I came to understand it when I was actually using the pattern. Can you spot it before hand? Bellow is the original example from this post, and at the end, an updated explanation._

Here's an example of the Factory Method on Python. Honestly, I am naming my example and example of Factory Method because is as I see it from what I can interpret as a Factory Method from my readings. Please be gentle and let me know if I am mistaken on the naming of this strategy. Regardless of the name, the aim of the approach is "straight forward": to allow a common public interface for subclasses that manage different environments but that have to communicate with the rest of the application (or users) in the same way, hence, providing the same formatted output. I see the [Python pathlib module](https://docs.python.org/3/library/pathlib.html) is a clear example of this strategy of implementation, where PosixPath or WindowsPath are returned upon initiating Path.

Here's the code example, as always with this series, let me know your thoughts.

```python
# Python Thoughts Snippet #4 - Factory Method - Creational Pattern
# Python 3.7
# 2019/08/31
# blog post: https://viviendomochileros.com/pythonic-thoughts-snippets-4/
# THIS CODE IS NOT MEANT TO BE FUNCTIONAL OR EXECUTABLE,
# IT IS A REPRESENTATION OF AN IDEA AND AN EXAMPLE TO RAISE DISCUSSION.

# As a common practice in my Pythonic Thoughts Snippets,
# I am not going into the details of the minor implementations,
# on doubts please search elsewhere on the web, there are countless of
# amazing explanations; here, we focus on the broader concept

from abc import ABC, abstractmethod


class Interface(ABC):
    
    def __new__(cls, arg1):
        
        match_d = {
            # evalutes arguments based on conditions to decide which
            # subclass to instantiate
            is_for_subclass1(arg1): SubClass1,
            is_for_subclass2(arg1): SubClass2,
            # etc...
            }
    
        try:
            # this actually calls object.__new__(match_d[True])
            # dont know if it is better to super(Interface) or object.
            return super(Interface, cls).__new__(match_d[True])
        except KeyError:
            raise NotImplementedError from None

    @abstractmethod
    def run(self):
        return


class SubClass1(Interface):
    
    def __init__(self, arg1):
        # do init magic
        
    def run(self):
        #run magic
        
        self._private_meth_1()
        self._private_meth_2()
        
        return
    
    def _private_meth_1(self):
        return    
    
    def _private_meth_2(self):
        return


class SubClass2(Interface):
    
    def __init__(self, arg1):
        # do init magic
        
    def run(self):
        #run magic
        
        self._private_meth_1()
        self._private_meth_2()
        
        return
    
    def _private_meth_1(self):
        return    
    
    def _private_meth_2(self):
        return
    
    def _private_meth_3(self):
        return
```

**EDIT (2019 Sep 08 continued):**
_The problem with the example above is that you can never instantiate SubClass1 or SubClass2 directly. For example, SubClass1(arg1) with an argument specific for SubClass2 will return a SubClass2 instance.
This happens because the abstract interface is coupled with the Factory Method, that is, SubClass1 and SubClass2 inherit from Interface. It is easy to solve this by decoupling the ABC interface from the Factory. Easy, right? Yet so difficult to spot._

```python
# An updated discussion:
from abc import ABC, abstractmethod


class Interface:
    
    def __new__(cls, arg1):
        
        match_d = {
            # evalutes arguments based on conditions to decide which
            # subclass to instantiate
            is_for_subclass1(arg1): SubClass1,
            is_for_subclass2(arg1): SubClass2,
            # etc...
            }
    
        try:
            # this actually calls object.__new__(match_d[True])
            # dont know if it is better to super(Interface) or object.
            return super(Interface, cls).__new__(match_d[True])
        except KeyError:
            raise NotImplementedError from None


class ClassBase(ABC):
    
    @abstractmethod
    def run(self):
        return


class SubClass1(ClassBase):
    
    def __init__(self, arg1):
        # do init magic
        
    def run(self):
        #run magic
        
        self._private_meth_1()
        self._private_meth_2()
        
        return
    
    def _private_meth_1(self):
        return    
    
    def _private_meth_2(self):
        return


class SubClass2(ClassBase):
    
    def __init__(self, arg1):
        # do init magic
        
    def run(self):
        #run magic
        
        self._private_meth_1()
        self._private_meth_2()
        
        return
    
    def _private_meth_1(self):
        return    
    
    def _private_meth_2(self):
        return
    
    def _private_meth_3(self):
        return
```

Best