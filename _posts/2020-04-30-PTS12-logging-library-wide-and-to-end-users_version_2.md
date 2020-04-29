---
title: "PTS 12: Logging library-wide, reviewed"
tags: [log, logging, Python, Python Programming]
author: "Joao MC Teixeira"
---
_29.04.2020_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-12.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

I started using the [Python logging library](https://docs.python.org/3/library/logging.html) a couple of years ago, it is definitively a powerful lib, and I encourage you to embrace it in your code developments.

There are countless web pages with *HowTo's* and tutorials explaining how to set a logging system for your Python one-file script. However, it is difficult to find places that explain how to set up the Python logging library application-wide, and how to properly integrate and share the `logging` in all your application modules comfortably. You can still find some examples on the web and in the [official documentation cookbook](https://docs.python.org/3/howto/logging-cookbook.html). The referred lack of tutorials is especially true if your project focuses on both library development and non-developer interface distribution. In these cases, it is necessary to set up the logging such that it provides a proper output when users are utilizing your application as a library and when users use it as stand-alone software. In the latter case, end-users will expect `.log` and/or `.debug` files with run records. On the other hand, users of your libraries won't need such files, and almost certainly a message printed to the `sys.stdout` or `sys.stderr` suffices.
So, the question remains on how to set up such difference interfaces properly. You will see that once configured, the set up looks quite apparent. But, honestly, it takes time to decompose the `logging` functionalities to a minimum set of operations that meets these requirements. For this logging configuration, we will make use of two main concepts: 1) global variables and 2) the fact that we can configure the `logging` dynamically. I will present the solution I currently using. Please share your thoughts if you have a different or better approach.

## Configuring the logger widely

Let's go for it! Consider the following project structure:

```
SampleProject/ # <- main repository folder
	src/
		sampleproject/
    		__init__.py
    		base.py (here is where your main API interface goes)
    		logger.py
            package1/
                __init__.py
                pkg1_module.py
            (... etc ...)
    docs/
    (... other files...)
```

**What we want** is to setup our `logger` object **before** any of project's module initiates, so that they can make use of `log` right from the beginning.

Therefore, directly in the `sampleproject` root `__init__.py`, I start defining the main logger with logging level set to `DEBUG`. In the configuration I am presenting here, which is the one I am using nowadays, I define **a single** `logger` that serves the whole application/library/package.

```python
# in src/sampleproject/__init__.py
import logging

log = logging.getLogger(__name__)
log.setLevel(logging.DEBUG)
```
`log` is now instantiated and is a global variable set on the main project's `__init__.py` file.

Secondly, and immediately after in the `__init__.py`, I define the logging handler responsible for writing, by default, to the `sys.stderr`, read more about the [StreamHandler](https://docs.python.org/3/library/logging.handlers.html#streamhandler). In elementary words, this is the logger handler that replaces the `print` function. I am not explaining `logging Handlers` here, for that, as said in the beginning, there are countless high quality explanations out there.

```python
# defines the handler
_ch = logging.StreamHandler()  # creates the handler
_ch.setLevel(logging.INFO)  # sets the handler info
_ch.setFormatter(logging.Formatter(INFOFORMATTER))  # sets the handler formatting

# adds the handler to the global variable: log
log.addHandler(_ch)
```

Do you notice the variable `INFOFORMATTER`? I import it from `logger.py` module in `sampleproject`, where I define additional help variables or functions related to the logging, for example, the formatters:

```python
# in src/sampleproject/logger.py
DEBUGFORMATTER = '%(filename)s:%(name)s:%(funcName)s:%(lineno)d: %(message)s'
"""Debug file formatter."""

INFOFORMATTER = '%(message)s'
"""Log file and stream output formatter."""
```

Generally, I define a highly verbose debug formatter that identifies the file and line where the message is logged and, on the other hand, I use a simple and user friendly formatter for the `INFO` log level, which is just the message the program needs to print. 

Because the logger is now defined at the root level of the `sampleproject`, we can import it in any module.

For example, in `pkg1_module.py`:

```python
from sampleproject import log

# and then just use it
log.info('message')
```

You can use it the same whether it is inside classes or functions.

```python
def myfunct(args):
    # do something
    log.info('worked, this is the result {}'.format(result))
    return result
```

Put this way, it sounds effortless.

### Other considerations in the `__init__` file

However, in case we need to import any of our project functionalities to the main *namespace* and because we want to set up the logging before loading all other modules in our project, we have to break a PEP8 rule on [import statements](https://www.python.org/dev/peps/pep-0008/#imports), that says:

>  "Imports are always put at the top of the file, just after any module comments and docstrings, and before module globals and constants."

The `__init__.py` layout would look like this:

```python
"""
PROJECT MAIN DOCSTRING
"""
# import ... # Python standard library imports here
import logging

from sampleproject.logger import (
    DEBUGFILE,
    DEBUGFORMATTER,
    INFOFILE,
    INFOFORMATTER,
	)

# this is exactly what we explained before
log = logging.getLogger(__name__)
log.setLevel(logging.DEBUG)
_ch = logging.StreamHandler()
_ch.setLevel(logging.INFO)
_ch.setFormatter(logging.Formatter(INFOFORMATTER))
log.addHandler(_ch)

# finally you would import from your project what you need to bring to the root NameSpace,
# if you need to do so.
from sampleproject.package1.module1 import SuperClass, megafunction
```

This breaks, in my understanding, the referred PEP8 rule, because we import only after the log has been configured. If you are using a lint checker, for example `flake8` and `isort`, you may need to add the ignore statement that at the end of the import line.

```python
from sampleproject.package1.module1 import SuperClass, megafunction  # noqa: F401 isort:skip
```

### is this a circular import?

Although `SuperClass` and `megafunction` use the `log` defined in the main `__init___.py` file and this imports the former two objects, I have found no problems with circular imports. Do you find it otherwise?

## Configuring logging files for users

As said, if you are developing a project that serves both library purpose and stand-alone application, you need to create log files of the program executions. I usually create at least two files: a `info.log` and a `debug.log`. As expected, the `info.log` registers all messages with `INFO` level while the `debug.log` registers all the `DEBUG` minimum level messages. How to separate the creation of the log files from the log instantiation itself, so that the former only operates when the program runs as stand-alone software, for example, in a command-line application? Simply, add the `handlers` that configure both log files at the execution `entry point`. How?

In the main `logger.py` file, define a `init_log_files` function, which is called from the `entry point` when `sampleproject` is executed as a stand-alone program. If `sampleproject` is used just as a library, these files will not be created.

```python
def init_log_files(log, mode='w'):
    """
    Initiate log files.

    Two files are initiated:
        
    1. :py:attr:`myapp.logger.DEBUGFILE`
    2. :py:attr:`myapp.logger.INFOFILE`

    Adds the two files as log Handlers to :py:attr:`log`.

    Parameters
    ----------
    mode : str, (``'w'``, ``'a'``)
        The writing mode to the log files.
        Defaults to ``'w'``, overwrites previous files.    """
    # here I show a very simple configuration
    # but you can extend it to how many handlers
    # and tweaks you need.
    db = logging.FileHandler(DEBUGFILE, mode=mode)
    db.setLevel(logging.DEBUG)
    db.setFormatter(logging.Formatter(DEBUGFORMATTER))

    info = logging.FileHandler(INFOFILE, mode=mode)
    info.setLevel(logging.INFO)
    info.setFormatter(logging.Formatter(INFOFORMATTER))

    log.addHandler(db)
    log.addHandler(info)
```

You can also extend this configuration to which better fit your needs. Lastly, in your `CLI` file, simply call `init_log_file`.

```python
from sampleproject import log
from sampleproject.logger import init_log_files

# some where down the line
def main(*args, **kwargs):
    init_log_files(log)
    # continue operating
```

Any additional configurations, like dedicated `CLI` log filenames, can be setup as parameters in `init_log_files`.

## Summary

In this post, I have shared with you how:

* I currently configure the Python `logging` library to serve a project as a whole, with multiple packages and modules
* to separate the logging setup and the creation of file handlers, this allows the log to serve for both library purpose and stand-alone applications
* activate log file handlers on `CLI` request

I hope you find this article of interest, and I look forward to your comments.

