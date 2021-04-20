# please-import-modules

A quick demonstration of why I recommend people import Python modules and not functions/classes/etc.

Clone this repository and open up the Python REPL using `python` from this root directory. (This should work with either Python 2 or 3).

```python
>>> import bad, dynamic, good
>>> id(bad.Dynamic)
140194524186640
>>> id(dynamic.Dynamic)
140194524186640
>>> id(good.dynamic.Dynamic)
140194524186640
>>> bad.get_dynamic().value
'default'
>>> good.get_dynamic().value
'default'
```

Without closing the REPL session, open up the dynamic.py file using your favorite text editor. Change the default value to something different.

```python
# dynamic.py
class Dynamic(object):
    def __init__(self, value="something different"):
        self.value = value
```

Save your changes.

Back in the same python REPL session:

```python
>>> import importlib
>>> try:
...     reload = reload or importlib.reload  # In Python2, reload is a built-in
... except NameError:
...     reload = importlib.reload  # In Python3, reload was moved to importlib
...
>>> reload(dynamic)
<module 'dynamic' from '.../github/creyes17/please-import-modules/dynamic.py'>
>>> id(bad.Dynamic)
140194524186640
>>> id(dynamic.Dynamic)
140194523277536
>>> id(good.dynamic.Dynamic)
140194523277536
>>> bad.get_dynamic().value
'default'
>>> good.get_dynamic().value
'something different'
```

What's happening here is that when the `dynamic` module is reloaded, the value for the `Dynamic` class is changed.

In `good.py`, we imported the `dynamic` module itself. The `dynamic` module has a property `Dynamic` which is a reference to the class. When we reloaded the `dynamic` module, it recompiled the `Dynamic` class and stored the newly compiled class in a different location in memory. Then the `dynamic.Dynamic` property is updated to point to the new location in memory for that updated class. Because reloading the module mutates the `dynamic` module itself, our `good.dynamic` pointer is still valid and still points to the `dynamic` module which now has the updated `Dynamic` class.

By contrast, in `bad.py` we imported the class `Dynamic` directly. This means `bad.Dynamic` is a variable which references the particular implementation of the class at the time that it was imported. When we reload the `dynamic` module, the `dynamic.Dynamic` property is changed, but our `bad.Dynamic` variable is unchanged - it is still a reference to the originally compiled class.

This is one of two reasons why I recommend that Python developers always import modules and never import functions/class/objects directly.

(The other reason is that I believe that it is more clear and readable when imported functions are always fully qualified. If I'm reading through some code in a large file, I don't need to scroll all the way back to the top to determine if a function/class/whatever is defined locally or imported from elsewhere.)

## Appendix: dynamic.py

```python
class Dynamic(object):
    def __init__(self, value="default"):
        self.value = value
```

## Appendix: good.py

```python
import dynamic


def get_dynamic():
    return dynamic.Dynamic()
```

## Appendix: bad.py

```python
from dynamic import Dynamic


def get_dynamic():
    return Dynamic()
```
