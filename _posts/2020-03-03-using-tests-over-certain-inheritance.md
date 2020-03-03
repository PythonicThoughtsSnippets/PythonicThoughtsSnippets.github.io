---
title: "PTS 11: Using tests over certain inheritance.
author: "Joao MC Teixeira"
tags: [Python, Programming, OOP, Object-oriented Programming, Test-driven development, testing, tests, pytest, design patterns]
---

I have recently engaged friends in profound conversations about **inheritance** and **aggregation**: which are the best approaches, practices and patterns. Deciding on whether or not to use inheritance is a dilemma we programmers encounter on the daily basis. Though tasty at first glance, inheritance may be a poisoned candy. On the other hand, the full potential of aggregation (sometimes lightly called also composition) is harder to grasp than one can think of initially, yet it offers several advantages. I won't go deeper into *aggregation* in this post, but I would rather, for now, discuss a specific example that uses *inheritance* and how can we avoid it. *Okay... this is really taking the form of a series of posts. Stay tuned :-)*

**A very experienced friend of mine**, a pure *informatician*, always says, "favour *composition* over *inheritance*"; and, this is actually a very common sentence in many design pattern books and online readings. But, when he says "*favour*", how often is that? His answer is straightforward: **always**. He usually continues:

>  "If you have a flu, then a flu is composed/aggregated with you and, eventually, you can get rid of it. While if you have an inherited disease... well, that's forever."

However, one may argue, **there are situations where inheritance comes handy**. For example, defining the `__init__` method in the base class is a way to avoid retyping the default attributes for subclasses; in this way, your code is [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and you are happy. If you are reading this post, I am sure you have faced this situation already. Another example are Abstract Base (ABCs) classes; lets focus on ABCs for now.

*ABCs are so cool, right?*

> Yes!

*Why?*

> They enforce the implementation of certain methods in the respective subclasses!

*To whom are you enforcing those rules?*

> To other developers.

In conclusion, abstract base classes serve, mostly, to enforce other developers some rules. *(If you are reading this and you strongly disagree with me, please [raise an issue](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/issues/11) and lets discuss, I am not attached to my opinions at all, only to accurateness, please enlighten me!)*

Repeating and extending the previous thought, we are using *inheritance* in our code through ABCs just to enforce other developers to implement certain methods in their subclass and, maybe, provide some shared functionality *via* other methods, such as the `__init__` example.

One of the drawbacks I found in reusing parent methods in subclasses is that the parent code is never available in the working file. That's simply boring! More importantly than the code itself, is that **the interface for that code is not in the subclass**. Therefore, if I am inspecting (working on) a file with a subclass that inherits from a parent class in other module, the complete list of methods available in that class interface is never (only if you bridge it) available in from of the eyes. This *hidden* interfaces greatly increase the cognitive load. I know that when you are reading this you are thinking about one single method:

> *How can you not remember a single method?* 
>
> What is your alternative, to break the DRY and to repeat code everywhere?

Let's pause on the first question. What would happen if your subclass inherits from several parents in a diamond shaped pattern, or is the last child of a chain of inheritances? How many *hidden* interfaces would you have to load on your *cognitum* to grasp the whole functionality of that class? Does this really escalates properly?

Let's pause on the second question. You can avoid inheritance while not breaking the DRY rules by using **aggregation/composition**.

*(Sorry for the mixed used of aggregation and composition, but I've seen this words being used as synonyms in many readings, and my background is not vast enough to rightfully decide which concept is more accurate. I do recognise, though, that composition is a kind of aggregation where parts can't exist without the whole, for example, "files in folders", "leaves in tree nodes"; while aggregation is using one object as part of another, and both can exist separately.)*

**Recapitulating our discussion**, we have identified two situations where inheritance can be used: 1) to share methods (functionality) between classes/subclasses and to enforce rules to developers. Let's focus on the latter and leave the former to another post, just because the latter is related with the previous posts about **testing**!

**How can we enforce certain implementations without using Abstract Base classes and `@abstractmethods` ?**

Yes, Python is a dynamically typed language and that means that polymorphism and duck-typing are always lurking in the dark, for the good and for the bad. Therefore, instead of enforcing implementation requirements at the *implementation* stage, enforce them at the *run time* stage through tests. What is the difference? Which are the added benefits?

1. Enforcing interface implementations at the code level adds a lot of code complexity that needs to be understood and parsed during development. 
2. Moreover, by using ABCs to enforce method implementation, you are also opening the door for new developers to use that ABC to actually implemented methods destined to be shared among subclasses. Maybe your initial idea was only to enforce some rules, but the door is open for more.
   1. I really believe that our code should correctly state our intentions.
3. On the other hand, you can enforce a certain interface by calling it in the tests. If the new implemented class lacks such interface, tests won't pass. By doing this, you fulfil two requirements with one shot:
   1. enforcing the implementation
   2. and actually writing the tests to it; which is always required.
4. Contrarily to 1., tests nicely tell you how the code has to behave. For me, this has much less cognitive load.
5. How would you test ABCs? [*Mocking*](https://docs.python.org/3/library/unittest.mock.html) things around is an option. Yet, is it *the* option?
6. Implementation on ABCs have to be tested twice: on the ABC itself and on the subclass.

In my opinion, it is safe and desirable to just write a test that tests for those initial argument default values upon a class instantiation, rather than sharing that `__init__` around and testing it awkwardly in a base class (and subclasses also). Yes, you do have to write that `__init__` more than once, I don't think that is that bad; but, surely, if that gets written too often maybe it is an indicator that some refactor is needed towards a **aggregation** pattern. Please, understand that I am using the `__init__` example here because is a very common example where inheritance is used. Processes more complex than this one, should go straight to aggregation/composition, in my opinion; but that is for another post.

**How to implement such testing?**

[pytest.fixtures](https://www.tutorialspoint.com/pytest/pytest_fixtures.htm) come handy in this case. For example:

```python
those_cool_classes = # a list of your cool classes to test that share the same default args

@pytest.fixture(params=those_cool_classes)
def parser_class(request):
    return request.param


def test_parser_class_default_args(parser_class):
    pc = parser_class()
    assert pc.attr1 == 1
    assert pc.attr2 is None
    # etc...
```



There are definitively much more to this matter than just this discussion. I have to say that when I learned about inheritance I started using and overused it everywhere it looked so obvious to use it, just to have headaches later on when I completely lost track of my code. I am adopting now these strategies in my new projects, so far with very good results.

I hope you have enjoyed this reading. And let me know your comments: [raise and issue]() or tweet me @vvcientificos.

Cheers and stay Pythonic,

Joao MC Teixeira