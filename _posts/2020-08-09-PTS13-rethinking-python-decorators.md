---
title: "PTS 13: Rethinking Python Decorators"
tags: [Python, Python 3.8, Python Programming, decorators, closures, high-order functions, multiprocessing, pickling]
author: "Joao MC Teixeira"
---
_09.08.2020_ - [Back to top](https://pythonicthoughtssnippets.github.io)

In this article, I will discuss the implications of Python **decorators** in software architecture, the drawbacks that emerge from the common recommendations and, finally, discuss an alternative way to construct and think about *decorators*.

In detail, I will go through four topics:

1. realize that the process of `@decoration` is actually a `@mutation`, and the implications thereof
1. avoid *mutation* using the `decorator()` syntax, instead of the `@` syntax - these are not synonymous!
1. understand the limitations of the `decorator()` construction in pickling and multiprocessing
1. rethink and rewrite decorators as high-order functions, instead of closures, to solve all previous problems

In this article, I do not alter Python syntax or use code outside the standard library; instead, I present and propose a change in the nomenclature and conceptualization of decorators in the contexts of software development and of how they are discussed in books and tutorials.

I assume the reader is fluent in the concept of Python decorators, normal and parametrizable decorators, high-order functions, and closures. If not, there are countless descriptions online and in books, I invite the reader to investigate those first.

Having said this, let us proceed `:-)`:

# 1) Raising awareness: decorators are actually mutating!

What drove me to write this post was the understanding that *decorating functions* is, in fact, a process which carries the burden of absolute coupling between the function and the decorator and that such burden is amplified when the construction of decorators is abused by the incepted need to use the *sweet* `@decorator` syntax -- which everybody seems to be looking for an opportunity to use (me the first).

Let us uncover the intrinsic coupling created by the `@decorator` syntax, we will go through some examples:


```python
# some relevant imports for this post
from functools import wraps
from time import sleep, time


# follows the simplest decorator form:
def timeme(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time()
        result = func(*args, **kwargs)
        print(f'elapsed time for {func.__name__!r}: {time() - start:.3f}')
        return result
    return wrapper
```


```python
# decorating the `calculation` function with `timeme` decorator
@timeme
def calculation(a, b, c=4):
    sleep(1)
    return a + b * c
```


```python
calculation(1, 2)
```

    elapsed time for 'calculation': 1.001
    9


`calculation(1, 2)` performed as expected.

Though using the `@timeme` syntax suggested so fiercely in tutorials, I found by myself (Pain-Driven development as forged by others) that this construction hinders the development of **complex packages or applications**.

**The reason:** it is impossible (to my knowledge) to recover the original function using the decorator public interface. In other words, forever during runtime, `calculation` is now *decorated*, and such decoration cannot be undone. `calculation` is no longer the original `calculation` but is, instead, the decorated version.

Consequently, the `@decorator` syntax is not *decorating* a function; **it is, instead, mutating the function**, as the original function ceases to exist.

This realization had a profound change in the way I craft software.

Just as a note, the original `calculation` function can still be recovered by the non-public interface of the decorator:


```python
calculation.__dict__['__wrapped__']
```
    <function __main__.calculation(a, b, c=4)>


But, you would agree, accessing the original with dunders is not the way to go. Also here, I am representing a function with a single decorator, how complex would be to retrieve the original function from several layers of decoration?


```python
def reportme(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print('Reporting to somewhere...')
        result = func(*args, **kwargs)
        print('Job Done')
        return result
    return wrapper
```


```python
@timeme
@reportme
def calculation_2(a, b, c=4):
    sleep(1)
    return a + b * c
```


```python
calculation_2.__dict__['__wrapped__']
```
    <function __main__.calculation_2(a, b, c=4)>

Well, thanks to `@functools.wraps`, obtaining the original function from a parametrizable decorator is equal as for a basic decorator. But still, it goes through a dunder interface.

The same hold true for parametrizable decorators:


```python
def param_decor(msg='I am a mutator'):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(msg)
            return func(*args, **kwargs)
        return wrapper
    return decorator

@param_decor('I am mutated!')
def calculation_3(a, b):
    return a + b

calculation_3(1, 2)
```

    I am mutated!
    3

```python
calculation_3.__dict__['__wrapped__'](1, 2)
```
    3

## Conclusion of 1)

Despite the `@decorator` syntax is well-favored in books and tutorials as the main, and almost only, way to create decorators, it does carry the bit burden of coupling. Why do we care about such coupling? Honestly, this coupling might be irrelevant if the end product is a one-page script. But, if the end product if a complex software with several packages full of modules with multiple workflows, accessing the original function at a certain moment of the run-time may be crucial -- actually, it is for my projects.

# 2) Avoid mutation and do actually decorate!

Having understood that the `@decorator` syntax actually `@mutates`, **how can we actually decorate a function and avoid the referred coupling?** The answer is simple:

**Apply the decorator at run-time and not at the definition time.**

Please note here the difference between *run-time* and *definition time*.

Let's review everything in detail. Before continue, let us just define a calculation function, free from any decorators, it will be useful in future examples as well:


```python
def free_calculation(a, b, c=4):
    sleep(1)
    return a + b * c
```

How to decorate the function using decorators?

As you know, as taught in so many tutorials and books (which I respect to most), the `@decorator` constructor is a syntax-sugar for:

```python
myfunc = decorator(myfunc)
```

**Wrong!** `:-)`.

The difference between these two syntaxes goes well beyond a *syntax-sugar*. The first one, `@decorator`, is evaluated at the function's definition time; while the second, highlighted above, is evaluated after `myfunc` is defined. Yet, taking the exact example above, both return the same object.

Understanding and realizing the differences between these two methods of construction has huge implications on software architecture and design.

Despite both syntaxes are presented widely in books and forums, **the second one is generally (if not always) discouraged** and, for most of the times just presented briefly in the last sentence of the decorator explanation; likely because the `@decorator` syntax just looks prettier. Here, I want to emphasize the fact that despite looking uglier *(does it?)*, the `myfunc = decorator(myfunc)` expression opens the door to several understandings and possibilities when building software.

First of all, note that the above extended-expression still does not solve our coupling problem. The original `myfunc` is still lost forever, *it is still mutated*. *Why?* It's so simple that it is hard to spot.

*`myfunc` variable name is reassigned to the decorated object, and the reference to the original function is lost.*

As simple as that. And despite the `myfunc = decorator(myfunc)` is discouraged in mainstream readings, here I am to encourage it. Combining everything we have discussed so far, we have that the solution to the decorator coupling problem is **to assign the decorated function to another variable**:

```python
# if you agree with my rationale in this post
# we now know that the following is actually a `mutator`
myfunc = decorator(myfunc)

# So we can safely rename the previous line to the following convention
myfunc = mutator(myfunc)

# contrarily to the above that shows a mutation process,
# the line bellow sincerely reflects a decoration process:
decorated_myfuc = decorator(myfunc)
```

**By assigning the decorated function to another variable, both `myfunc` and `decorated_myfunc` can be accessed, or created, independently at any time in the run-time.** This completely removes the coupling described before. As consequence, our `decorator` does work as a *decorator*, and not as a *mutator*, because the original function is still accessible. Also, `myfunc` can be redecorated with another decorator at a given moment if needed!

Gaël Varoquaux wrote a [post](http://gael-varoquaux.info/programming/decoration-in-python-done-right-decorating-and-pickling.html) highlighting the harm provoked by overusing the `@decorator` syntax, and he noticed also the need to assign decorated functions to another variable. He focused mostly on *pickling* issues related to multiprocessing, while here I am focusing mostly on architectural concepts and drawbacks of the different approaches. I have to say however, it was the pickling issue with decorators in multiprocessing that brought me to write this post in the first place and from which the argument has evolved.

## Conclusions from 2)

Having reached this far, we have now conceptualized that the `@decorator` syntax **mutates** a function because that code is executed at the function definition moment, while `decorated_myfuc = decorator(myfunc)` **decorates** the function because this code executes at run-time after the function is defined and, moreover, it assigns the decorated function to another variable.

To avoid the absolute coupling originated by the `@decorator` syntax, I suggest the developer not be afraid of defining decorated functions at run-time, both at the module level or when the decorated function is needed. For example, right before it is executed or, in case the function is to be used at several places in the package, just define the decorated function at the module level **after** the function definition using the `decorated_myfuc = decorator(myfunc)` syntax. In this way, both the decorated and the original function will always be available.

# 3) We have solved the coupling, now we have pickling problems!

The whole conversation in this post makes sense if we are developing complex software; it has little significance when creating single page scripts. Therefore, which complex software does not make use of multiprocessing techniques? And, Multiprocessing uses pickle; so, **how does the argument of this post relates with pickling?** Here, I will show that pickling is not compatible with the uncoupled `myfunc = decorator(func)` syntax, sadly. But we will solve that on topic `4)` `:-)`.

Let's define some needs first:


```python
import pickle

# try_pickle will help us on the argument later on
def try_pickle(f):
    """
    Thanks to Gaël Varoquaux for this example.
    
    http://gael-varoquaux.info/programming/decoration-in-python-done-right-decorating-and-pickling.html
    """
    try:
        pickle.dumps(f)
        return 'Pickling successful'
    except pickle.PicklingError as e:
        print(e)
        return "Oops, can't pickle"
```

To simplify the demonstration of our point, lets try to pickle (simullate multiprocessing) several functions decorated differently with both the `@decorator` and the `decorator()` syntaxes, also making using of several layers of decoration:

| Function  | Decorated with |
| ------------- | ------------- |
| `free_calculation`  | none  |
| `calculation`  | `@`  |
| `calculation_2` | `@@` |
| `calculation_3` | `@()` |
| `calculation_4` | `@@()` |
| `calculation_5` | `d()` |
| `calculation_6` | `d(d())` |
| `calculation_7` | `d()(d())` |
| `calculation_8` | `d()(d(d()))` |


We now define functions `4` to `7`:


```python
@param_decor('I am mutated!')
@timeme
def calculation_4(a, b):
    return a + b
```


```python
calculation_4(1, 2)
```

    I am mutated!
    elapsed time for 'calculation_4': 0.000
    3




```python
calculation_5 = timeme(free_calculation)
calculation_6 = reportme(calculation_5)
calculation_7 = param_decor('decorated')(calculation_5)
calculation_8 = param_decor('decorated')(calculation_6)
```

Prove everything is working:


```python
print(calculation_5(1, 2))
print(calculation_6(1, 2))
print(calculation_7(1, 2))
print(calculation_8(1, 2))
```

    elapsed time for 'free_calculation': 1.001
    9
    Reporting to somewhere...
    elapsed time for 'free_calculation': 1.001
    Job Done
    9
    decorated
    elapsed time for 'free_calculation': 1.001
    9
    decorated
    Reporting to somewhere...
    elapsed time for 'free_calculation': 1.001
    Job Done
    9


Let us defined a decorator with the `functools.wraps` to inspect its behaviour:


```python
# follows the simplest decorator form:
def timeme2(func):
    def wrapper(*args, **kwargs):
        start = time()
        result = func(*args, **kwargs)
        print(f'elapsed time for {func.__name__!r}: {time() - start:.3f}')
        return result
    return wrapper

calculation_9 = timeme2(free_calculation)
```

**Okay**. Everything is defined, it is time for pickling:


```python
funcs = [
    free_calculation,
    calculation,
    calculation_2,
    calculation_3,
    calculation_4,
    calculation_5,
    calculation_6,
    calculation_7,
    calculation_8,
    calculation_9,
]

for f in funcs:
    print(f'{f.__name__} ** {try_pickle(f)}')
    print()
```

    free_calculation ** Pickling successful
    
    calculation ** Pickling successful
    
    calculation_2 ** Pickling successful
    
    calculation_3 ** Pickling successful
    
    calculation_4 ** Pickling successful
    
    Can't pickle <function free_calculation at 0x7f6f84066ca0>: it's not the same object as __main__.free_calculation
    free_calculation ** Oops, can't pickle
    
    Can't pickle <function free_calculation at 0x7f6f84066e50>: it's not the same object as __main__.free_calculation
    free_calculation ** Oops, can't pickle
    
    Can't pickle <function free_calculation at 0x7f6f84066af0>: it's not the same object as __main__.free_calculation
    free_calculation ** Oops, can't pickle
    
    Can't pickle <function free_calculation at 0x7f6f840663a0>: it's not the same object as __main__.free_calculation
    free_calculation ** Oops, can't pickle
    



    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-72-e982093c6264> in <module>
         13 
         14 for f in funcs:
    ---> 15     print(f'{f.__name__} ** {try_pickle(f)}')
         16     print()


    <ipython-input-58-edf8cac06b99> in try_pickle(f)
          9     """
         10     try:
    ---> 11         pickle.dumps(f)
         12         return 'Pickling successful'
         13     except pickle.PicklingError as e:


    AttributeError: Can't pickle local object 'timeme2.<locals>.wrapper'


## Conclusion from 3)

The conclusion of this section is straightforward: the syntax `func2 = decorator(func)` that beautifully avoids the coupling between the original function and its decorated version is not pickable and, therefore, not compatible with multiprocessing.

So, can we rewrite decorators in a different way? Can we rethink and rewrite decorators in a way that fulfills the non-coupling and pickling requirements?

# 4) Writing decorators as high-order functions

Having realized the problems inherited with the most recommended usage of decorators we ask the question:

How can we rewrite all decorators such that the end result is pickable, and so it can work with [multiprocessing](https://docs.python.org/3.8/library/multiprocessing.html) library, and the original function is always available, regardless of how many decorator layers we have placed upon the function.

Again, bear in mind that all this effort might seem useless if the end product is a one page Python script. But, in my experience, all these considerations become necessary when one is writing a complex package or software that requires long-term maintainability and modularity.

The solution I came to, is to write all decorators as **high-order functions** instead of **closures** (the natural nature of decorators). To quickly state the difference between HO functions and closures, in my words (wiser words can be found elsewhere):

> high-order functions are functions that receive other functions but return non-function (callable) values, while closures, may not receive a function as an argument, but return other functions.

I understood that conceptualizing decorators as high-order functions favor an architecture that remains modular to the core, has no cyclomatic complexity added, and provides a pickable solution in the present and, most likely, in the long-term.

**How can we write decorators as high order functions?**

Using [partials](https://docs.python.org/3/library/functools.html#functools.partial)!

Let's see some examples:


```python
# rewriting the timeme decorator as an high-order function
def timeme_decorator(func, *args, **kwargs):
    start = time()
    result = func(*args, **kwargs)
    print(f'elapsed time for {func.__name__!r}: {time() - start:.3f}')
    return result
```

Then, somewhere in the program (remember, at run-time), you would write the following to create the decorated function:


```python
## somewhere in your program at run-time
## maybe in another module

# at the import level
from functools import partial

# somewhere down the line...
# decorate the function using functools.partial!
timed_calculation = partial(timeme_decorator, free_calculation)
timed_calculation(0, 2, c=10)
```

    elapsed time for 'free_calculation': 1.001
    20

```python
# lets try another example:
def reports(func, *args, **kwargs):
    print('Reporting to somewhere...')
    result = func(*args, **kwargs)
    print('Job Done')
    return result

# constructing the function decoration at run time using partials
reports_calc = partial(reports, timed_calculation)
reports_calc(1, 2)
```

    Reporting to somewhere...
    elapsed time for 'free_calculation': 1.001
    Job Done
    9


The above construction behaves exactly like the example before with stacked `@decorators`. Note that the work with `partial` can be done at the moment the decorated function is needed or right after the function is defined in case it needs to be used widely in your application in different contexts. Either way, the original function is always available: the `free_calculation` -- there is not coupling associated as we have seen before for the `f2 = d(f1)` syntax.

**Are the decorated functions pickable?** That is, can they be used in `multiprocessing` processes? Yes, because using `partial` and `high-order` functions avoids the use of `closures` where pickling fails in the cases identified before.


```python
print(try_pickle(timed_calculation))
print(try_pickle(reports_calc))
```

    Pickling successful
    Pickling successful


What about parametrizable decorators?


```python
def other_param_decor(pos1, func, *args, msg='I am a decorator', **kwargs):
    print(pos1)
    print(msg)
    return func(*args, **kwargs)

decor_calc = partial(other_param_decor, 'decorator param', free_calculation, msg='I am decorated')
decor_calc(1, 2)
```

    decorator param
    I am decorated
    9


```python
# let's twist it a bit more:
do_all_calculation = partial(
    other_param_decor,
    'decorator positional parameter',
    reports_calc,
    msg='I am decorated')

do_all_calculation(5, 5)
```

    decorator positional parameter
    I am decorated
    Reporting to somewhere...
    elapsed time for 'free_calculation': 1.001
    Job Done
    25



Does it pickle?


```python
try_pickle(do_all_calculation)
```
    'Pickling successful'


**Yes!**

The **only** problem with parametrizable decorators using `partial` is that the decorator's named parameters need to be named differently from the functions `kwargs`. Personally, I don't consider that a problem at all.

**Note** that the order in which partials have to be defined must follow the same order as the `@decorator` syntax - from inward to outward.

```python
@evaluates_3rd
@evaluates_2nd
@evaluates_1st
def func...
```

## Conclusions from 4)

Using partials provides a way to write and think of decorators as high-order functions instead of closures. The end results is always pickable and its construction is not hard.

# General Conclusions

Generally speaking, the act of decorating refers to adding a feature to something, a feature which can later be removed from the decorated entity at the demand and without altering the true nature of the original - for example, wearing an earring.

In my opinion, Python **decorators**, as presented in generally in literature and in the [Python Grammar](https://docs.python.org/3/reference/grammar.html?highlight=decorators) (`@decorator` syntax), do not decorate functions, they **mutate** them. A function decorated **at definition time** is decorated forever during the program execution and the original function cannot be reused through a public interface, therefore, in my opinion, the original function is not *decorated*, it is, instead, *mutated*.

By renaming the convention name from **decorator** to **mutator**, it hopefully becomes more clear to the developers the fact that the original function is lost. Most importantly, renaming the convention highlights the unavoidable coupling burden between the decorator constructed through the `@decorator` notation and the function being decorated. The impact of this understanding is more noticeable if you're developing more complex systems rather than single-logic scripts.

Distinguishing a *decoration* from *mutation* operation can be achieved by utilizing the different syntaxes:

```python
# mutating...
@mutator
def func...

# decorating...
# note the different variable name
decorated_function = decorator(func)
```

As a consequence, the term **decoration** can be applied nicely in Python using the second expression during run-time. To this point, the new understanding is to break with the convention of teaching the above expressions as equivalent when they are, actually, inherently different.

Investigating further, we realize that the `decorator(func)` expression fails upon pickling from its simplest form to the most complex parametrizable decorators:

```python
# all these fail while pickling
myfunc = decorator(func)

myfunc = decorator2(decorator(func))

myfunc = decorator(some_parameter)(decorator(func))
```

This forces us to maintain coupling if we want to use multiprocessing techniques. To avoid this, we can rethink decorators as *high-order* functions instead of *closures* and use `functools.partial` to define decorated functions. In a similar line, [Gael Varoquaux suggested defining decorators as scopes in classes.
](http://gael-varoquaux.info/programming/decoration-in-python-done-right-decorating-and-pickling.html)

Here, by using `partial`, different decorators can be applied to the function at different moments of the run-time and when required by the execution flow. In case a *decoration* is required system-wide, it can always be defined at the function module level, right after the function definition, as exemplified before.

To my knowledge, the topics discussed in this post are obscure and not discussed much online. I have not encountered elsewhere a discussion on the architectural implications of and the conceptualization that *decorators* being actually *mutators*. And, Gael Varoquaux post from 2009 is the most complete discussion raising awareness of creating decorators as object scopes instead of closures. Here, I have used partials. Which do you think is more readable? It can be that `partials` have a more verbose creation process, but on the other hand, the decorator itself has a much cleaner footprint than classes.

I hope this proposal and conclusions foster discussion within the Python community.

* Is it a revelation for you, as it was for me, that decorators are mutators?
* How do you feel about using `partial` to define decorators?
* Did you encountered similar problems while crafting your packages?


I look forward to any comment from you, please [comment here for that](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/issues/13). And I apologize if I have missed some crucial literature on this, and I would love it if you would point me to that, but you would agree that these topics do not emerge when reading the most common and well-known literature in Python. Here, we are already going deep :-)

Follow the twitter thread [here](https://twitter.com/vvcientificos/status/1292435637744934912) and [there](https://twitter.com/vvcientificos/status/1292435639766585345).

Cheers
