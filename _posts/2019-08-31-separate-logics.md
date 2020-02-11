# 5: Separate Logics
_31.08.2019_ - [Download Code Snippet](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/blob/master/pts-5.py) - [Back to top](https://pythonicthoughtssnippets.github.io)

A quick view on business and administrative logic separation. I found myself coding this way as of today. I will write more on this in the near future.

A dummy example where we use decorators to check if path exists before assigning it to the private class attribute. Same could be accomplished by contextmanagers.

```python
def check_folder_exists(func):
    """
    Decorator for class property setters
    """
    @wraps(func)
    def wrapper(self, folder):
        
        if not Path(folder).exists():
            raise FileNotFoundError(f'Path not found: {folder}')
        
        else:
            return func(self, folder)
    return wrapper


class Parameters:
    def __init__(self, **kwargs):
        self.kwargs = kwargs
    
    @property
    def db_folder(self):
        return self._db_folder
    
    @db_folder.setter
    @check_folder_exists
    def db_folder(self, folder):
        self._db_folder = folder
```

Another snippet from my code where I use a combination of three contexts to handle exceptions hierarchically.

```python
for step in config_reader.steps:
                
    with GCTXT.abort_if_exception(), \
            GCTXT.query_upon_error(XCPTNS.ConfigReadingException), \
            GCTXT.abort_if_critical_exceptions(CONFREADER.critical_errors):
            
        step()
```

Is it clear and Pythonic? [Share your thoughts](https://github.com/PythonicThoughtsSnippets/PTS-Code-Snippets/issues).