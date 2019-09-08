# 3: logging library wide and to end-users
_26.08.2019_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-3.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

I have start using the [Python logging library](https://docs.python.org/3/library/logging.html) about one year ago it is definitively a powerful lib that you (maybe) should embrace on your code development.

There are countless of web pages with howtos and tutorials on how to set a  logging system for your script. However, there are not so many on how to set a logging system library-wide, thought you can still find some, and the official documentation has examples.

However, I always felt that a precise explanation on how to properly set a logging system library wide is not easy to find. Specially, if your project focus on both library development and nondev end-user distribution.

In this cases it is necessary to set up the logging such that it provides proper output when using the libraries (your project) and when using the software by end-users. Normally, end-users will expect, and you as a developer also need such, some .log file and .debug file with which the communication between users and developers is facilitated. On the other hand, users of your libraries won't need such files and almost certainly a messaged printed to the sys.stdout suffices.

So the question remains on how to properly set up such difference interfaces. At the end, everything looks quite obvious but, honestly, it took me quite a bit of time to figure it out and have the desired behaviour.

It is definitively not straightforward because we will be making use of advanced concepts like Python singletons (that are different from other languages), and the realization that the logging can be set dynamically. :-)

This is the solution I came by. Please share your thoughts if you have a different or better approach. I am far from being a full expert on the Python logging library.

```
> Consider the following project structure:
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
