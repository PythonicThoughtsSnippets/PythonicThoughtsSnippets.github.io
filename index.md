# Motivation
I created the Pythonic Thoughts Snippets series out of the need to share my connection with the code I write, how my vision and style evolve, how my mindset for code writing shapes with time in a continuous quest towards writing more maintainable, easy to read, robust and beautiful code. I hope you find these little articles of interest and also share your comments. These posts are the result of many online and book readings, discussions with others and hours spent staring at the code. Please consider I am not a purely computational person, I am a biochemist passionate about coding that have learned my way through this beautiful discipline. - Joao M.C. Teixeira, 2019.

# [Parameter Validation on Class Creation](#Parameter-Validation-on-Class-Creation)

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

# Dictionaries for Control Flow

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


