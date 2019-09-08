# 4: Factory Method Creational Pattern
_31.08.2019_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-4.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

Some things are not trivial to learn, other are even difficult, but some are not about learning, but actually incorporating behaviours and methodologies in our own practices, and make their use a natural process.

It was definitively not one nor twice that I read descriptions and explanations about the Factory Method as a Creational Pattern. What are those concepts? Look around for Object-Oriented Programming concepts and the associated (most standard) Design Patterns. There are many (yet maybe not that many) websites and books about these topics, here's one website I have as reference ([Source Making](https://sourcemaking.com/design_patterns/factory_method)).

But, despite all my cheerful readings on the matter and struggles with my code, just the other day I came to realize how to properly build this pattern and in which situations apply it.

**EDIT (05.09.2019):**  
_There's a very interesting fault in the design shown in the original example of this post, I came to understand it when I was actually using the pattern. Can you spot it before hand? Bellow is the original example from this post, and at the end, an updated explanation._

Here's an example of the Factory Method on Python. Honestly, I am naming my example and example of Factory Method because is as I see it from what I can interpret as a Factory Method from my readings. Please be gentle and let me know if I am mistaken on the naming of this strategy. Regardless of the name, the aim of the approach is "straight forward": to allow a common public interface for subclasses that manage different environments but that have to communicate with the rest of the application (or users) in the same way, hence, providing the same formatted output. I see the [Python pathlib module](https://docs.python.org/3/library/pathlib.html) is a clear example of this strategy of implementation, where PosixPath or WindowsPath are returned upon initiating Path.

Here's the code example, as always with this series, let me know your thoughts.

```python
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
