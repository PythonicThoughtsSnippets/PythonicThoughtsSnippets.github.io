# 7: TDD - pytest fixtures
_09.12.2019_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-7.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

As code goes by my hands, my `spider sense` gets refined and `tingles` sooner than before on situations where *there must be a better way to do it!*.

Today insight has been on [pytest](https://docs.pytest.org) fixtures. I am already used to [pytest parametrization](https://pythonicthoughtssnippets.github.io/#6-test-driven-development) but today I was looking to my test functions with more than one `assert` statement - *spider sense tingles*..., *there must be a better way!*.

Multiple `assert` statements get wrapped around the same test umbrella and are not highlighted as test counts and, more importantly, multiple `assert` statements inside the same `test_function` sum to [coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming)) instead of [cohesion](https://en.wikipedia.org/wiki/Cohesion_(computer_science)).

So, looking for a proper method to separate those multiple `assert`s inside my `parametrize`d `test_functions`, I found the [fixture implementation in pytest](https://docs.pytest.org/en/latest/fixture.html#pytest-fixtures-explicit-modular-scalable).

Descriptions to this approach are found when one searches for ways to reuse variables from test function to test function. However, as far as my current readings, this is a *bad* practice because tests should be completely independent from each other. Therefore, the best to way to reuse a variable (or treatment) in several tests is to recreate it for every test; and the best way (I found so far) to achieve this with [pytest](https://docs.pytest.org) is using its *fixture* implementation.

Bellow is a simple transformation of a parametrized test split into two *fixtures*:

```python
# pseudo code

@pytest.mark.parametrize(
    'fname',
    [
        (Path('file1')),
        (Path('file2')),
        ],
    )
def test_my_file_parser(self, fname):
    parser = MyFileParser(fname)
    data1 = parser.data_foo()
    assert isinstance(data1, list)
    assert len(data1) == 50


# bellow the fixture approach
@pytest.fixture(
    params=[
        Path('file1'),
        Path('file2'),
        ],
    )
def parser_data1(request):
    parser = MyFileParser(request.param)
    data1 = parser.data_foo()
    return data1


def test_parser_data1_type(parser_data1):
    assert isinstance(parser_data1, list)


def test_parser_data1_len(parser_data1):
    assert len(parse_data1) == 50
```

This setup will created a product of four tests: two for each of the two `params` in the `fixture` decorator.

Pytest fixtures provide much more advanced options that I don't fully understand at this point but which I will be willing to explore!

Is there something that could be improved on this entry?i Share your thoughts, [comment on this post here](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/issues/8).

## Further readings

* [pytest docs fixture](https://docs.pytest.org/en/latest/fixture.html)
* [fixtures and parameters at Medium](https://medium.com/ideas-at-igenius/fixtures-and-parameters-testing-code-with-pytest-d8603abb390a)
* [StackOverFlow 47402435](https://stackoverflow.com/questions/47402435)

## tags

* python
* pytest
* test-driven-development
* pytest-fixtures
* pytest-parametrize
