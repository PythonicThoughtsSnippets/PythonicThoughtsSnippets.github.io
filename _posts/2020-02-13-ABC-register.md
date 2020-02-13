---
title: "PTS 10: ABC register"
tags: [Python, programming, ABC, Abstract Class, ABC.register]
author: "Joao MC Teixeira"
---

Some months ago [Raymond Hettinger](https://twitter.com/raymondh/status/1150410766241030152?s=20) tweeted about the usage of [ABC.register](https://docs.python.org/3/library/abc.html#abc.ABCMeta.register) to register subclassing as an alternative to inheritance. Well, I found a place today to apply this strategy in my code (I was doing other nice things meanwhile :-P).

I am writing different config file readers with some enhanced features needed for the project, which all _ducktype_ the same interface. In general I don't like inheriting anything (except for `Exceptions`), and I assert for the same interface in tests, but I needed all my `ConfigReader` classes to be identified as an unique family. Well I though that I could register them; here's the example:

```python
# in the __init__.py file of the config_readers_pkg:

from myverycoolproject.libs.config_readers_pkg.reader_json import CReaderJson
from myverycoolproject.libs.config_readers_pkg.reader_param import CReaderParam

_readers = {
    CReaderJson,
    CReaderParam,
    }

config_readers = {r.ext: r for r in _readers}
```

Then on the readers `.py` files I define the readers as follow:

```python
from myverycoolproject.libs.config_readers.base import ConfigReader


@ConfigReader.register
class CReaderJson:
    ext = '.json'
    # ...
```

Same goes for `CReaderParam` and others.

The `base.py` is actually:

```python
from abc import ABC


class ConfigReader(ABC):
    pass
```

Interestingly, one can also use the `register` feature as part of the test suite:

```python
# some where in tests/test_config_readers.py
from myverycoolproject.libs.config_readers import config_readers
from myverycoolproject.libs.config_readers_pkg.base import ConfigReader

def test_config_readers_dict():
    """Testing config_readers dict is functional."""
    assert config_readers
    assert all(k.startswith('.') for k in config_readers.keys())
    assert all(issubclass(v, ConfigReader) for v in config_readers.values())
```

With this setup it is possible to test that all members of the `config_readers` dictionary are actually part of the `ConfigReader` family. This, asserts there it no contamination by any other object in the dictionary and slightly enforces that `ConfigReaders` were designed with thought. Also, and more importantly, having all `ConfigReader` classes registered as such allows control-flow tuning if needed.

Why not using inheritance directly? Because, **explicitly** I don't want to allow methods to be designed in the base class and consequently injected in the subclasses; and, I want to make clear to developers that inheritance is not an option here `;-P`.

Any comments? Is there a better way to do it? Let me know! [Share your thoughts](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/issues/10).
