---
title: "PTS 6: Test-driven development"
tags: [TDD, test-driven development]
author: "Joao MC Teixeira"
---
_02.12.2019_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-6.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

In the beginning I didn't understand the need for them, then I was smashed by their debugging power, then I struggled to make their use a daily work practice, now I cannot conceive writing code without compulsive test driven development. Now, I simply can't work without using three _screens_, one for the logic, other for the tests and other to run the tests; and I jump continuously between these three _windows_, _terminals_, _spliscreens_.

I dare to even say that I actually stopped using Jupyter Notebooks to test code outputs and, instead, I directly write tests, and explore my code in that way (*some very visual exceptions apply*).

Last week I found my self writing these tests for this simple function, and it was when I realized that something actually had changed a lot in my daily practices.


```python
def random_fragment(iterable, fragsize=None):
    """
    Generate a slice object that reflects a random fragment from iterable.
    
    (iterable, int -> slice object)
    
    Parameters
    ----------
    iterable : iterable-type
        An interable: string, list, tuple, etc.

    fragsize : int or NoneType
        The size of the fragment to generate.

    Returns
    -------
    slice object
        A slice object reflecting a random fragment of the `iterable`
        of size `fragsize`.
        If `fragsize` is ``None``, returns a full range slice object.
    """
    # this is separate from the try: block to account input of for types
    # that are now iterable and have not to do with the usage
    # of fragsize=None
    len_ = len(iterable)

    try:
        start = random.randint(0, len_ - fragsize)
    except TypeError:  # fragsize is None
        return slice(None, None, None)
    else:
        return slice(start, start + fragsize, None)
```

And these are the tests I wrote.

```python
"""Test libutil."""
import pytest

# I know libutil is a very bad name. What does that mean? That other
# libs are not *util* ? :-P

from project.libs import libutil as UTIL


def test_random_fragment_return():
    """Test return type is slice object."""
    result = UTIL.random_fragment([1, 2])
    assert isinstance(result, slice)


@pytest.mark.parametrize(
    'in1,fragsize,expected',
    [
        (list(range(1000)), 7, 7),
        (list(range(1000)), 0, 0),
        (list(range(1000)), None, 1000),
        ([], None, 0),
        ([], 0, 0),
        ('abcdefgh', 3, 3),
        ((1,2,3,4), 1, 1),
        ],
    )
def test_random_fragment(in1, fragsize, expected):
    """
    Test fragment has expected length.

    Parametrize
    -----------
    1: list with fragsize < len(list)
    2: list with fragsize == 0, should return empty list
    3: list with fragsize is None, should return whole list
    4: empty list with fragsize is None, should return empty list
    5: empty list with fragsize is None, should return empty list
    6: functionality for strings
    7: functionality for tuples
    """
    result = UTIL.random_fragment(in1, fragsize)
    assert len(in1[result]) == expected


@pytest.mark.parametrize(
    'in1,fragsize,error',
    [
        ([], 7, ValueError),
        (list(range(100)), 700, ValueError),
        (9, 2, TypeError),
        (9.0, 2, TypeError),
        ],
    )
def test_random_fragment_errors(in1, fragsize, error):
    """
    Test errors raised with input.

    Parametrize
    -----------
    1: Empty list with fragsize > 0
    2: list with fragsize > len(list)
    3: int
    4: float
    """
    with pytest.raises(error):
        UTIL.random_fragment(in1, fragsize)
```

Comments on a better way to do it? :-) [please raise an issue](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/issues).
