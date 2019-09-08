# 0: Parameter Validation on Class Creation
_19.08.2019_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-0.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

This post serves as tutorial, question to the audience and challenge. I will explain how I solved the problem but I am yet far to be sure about the most correct and Pythonic way to address this question.

For the project I currently have at hands, I have to design a series of classes (I call it modules because of the project's scope, nothing to do with python modules) that can be initiated with a bunch of keyword parameters. I am assigning those parameters to @property attributes inside the class because I want to control the getter and setter methods when accessing those attributes. So far so good and nothing special.

The setter method implement the logic that should accept or reject the values to be assigned to the attributes. This is critical to the project because my classes are bound to a very precise biological and physical meaning. When a wrong value is introduced an error raises.

I want to extend the implementation of the classes to the possibility to force its instantiation even with non-valid values. For that I've introduced a "build" parameter that bypasses the errors risen and assigns the value to its corresponding dunder attribute.

Additionally, I implemented a "testit" option that tests the class instantiation for a given set of parameters and reports on possible value errors by returning Report class object instead of the class itself.  Here's an example:

```python
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
