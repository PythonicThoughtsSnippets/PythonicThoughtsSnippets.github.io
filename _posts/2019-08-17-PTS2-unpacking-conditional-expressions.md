_15.08.2019_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-2.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

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
