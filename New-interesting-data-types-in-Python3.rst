New interesting data structures in Python 3
=============================================

Python 3 is no longer new.

In fact, recently, its 3000 day birthday_ was celebrated :). After quite some wait, Python 3's uptake is dramatically on the rise now, and I think it is therefore time to take a look at some data structures that Python 3 offers, but that are not available in Python 2. 

We will take a look at ``types.MappingProxyType``, ``typing.NamedTuple`` and ``types.SimpleNamespace``, all of which are new to Python 3.

``types.MappingProxyType``
-------------------------

``types.MappingProxyType`` is used as a read-only dict and was added in Python 3.3. See docs_ for details.

That ``types.MappingProxyType`` is read-only means that it can't be directly manipulated and if users want to make changes, they have to deliberately make a copy, and make changes to that copy. This is perfect if you're handing a ``dict`` -like structure over to a data consumer, and you want to ensure that the data consumer is not unintentionally changing the original data. In practical use this is often extremely useful, as cases of data consumers changing passed-in data structures leads to very obscure bugs in your code that are difficult to track down.

A ``types.MappingProxyType`` example:

.. code-block :: python

    >>> from  types import MappingProxyType
    >>> data = {'a': 1, 'b':2}
    >>> read_only = MappingProxyType(data)
    >>> del read_only['a']
    TypeError: 'mappingproxy' object does not support item deletion
    >>> read_only['a'] = 3
    TypeError: 'mappingproxy' object does not support item assignment
      
Note that the example shows that the ``read_only`` object cannot be directly changed. 

So, if you want to deliver data dicts to different functions or threads and want to ensure that a function is not changing data that is also used by another function, you can just deliver a ``MappingProxyType`` object to all functions, rather than the original ``dict``, and the data dict now cannot be changed unintentionally. An example illustrates this usage of ``MappingProxyType``:

.. code-block :: python
    
    >>> def my_threaded_func(in_dict):
    >>>    ...
    >>>    in_dict['a'] *= 10  # oops, a bug, this will change the sent-in dict
    
    ...
    # in some function/thread:
    >>> my_threaded_func(data)
    >>> data
    data = {'a': 10, 'b':2}  # note that data['a'] has changed as an side-effect of calling my_threaded_func

If you send in a ``mappingproxy`` to ``my_threaded_func`` instead, however, attempts to change the dict will result in an error:

.. code-block :: python

    >>> my_threaded_func(MappingProxyType(data))
    TypeError: 'mappingproxy' object does not support item deletion
    
We now see that we have to correct the code in ``my_threaded_func`` to first copy ``in_dict`` and then alter the copied dict to avoid this error. This feature of ``mappingproxy`` is great, as it helps us avoid a whole class of difficult-to-find bugs.

Note though that while ``read_only`` is read-only, it is not immutable, so if you change ``data``, ``read_only`` will change too:
 
.. code-block :: python
    
    >>> data['a'] = 3
    >>> data['c'] = 4
    >>> read_only  # changed!
    mappingproxy({'a': 3, 'b': 2, 'c': 4})

This is something to be aware of.

``typing.NamedTuple``
---------------------

``typing.NamedTuple`` is a supercharged version of the venerable ``collections.namedtuple`` and while it was added in Python 3.5, it really came into its own in Python 3.6.

In comparions to ``collections.namedtuple``, ``typing.NamedTuple`` gives you (Python >= 3.6):

- nicer syntax
- inheritance
- type annotations
- default values (python >= 3.6.1)

See a ``typing.NamedTuple`` example below:

.. code-block :: python
    
    >>> from typing import NamedTuple
    >>> class Student(NamedTuple):
    >>>    name: str
    >>>    address: str
    >>>    age: int
    >>>    sex: str
    
    >>> tommy = Student(name='Tommy Johnson', address='Main street', age=22, sex='M')
    >>> tommy
    Student(name='Tommy Johnson', address='Main street', age=22, sex='M')


I like the class-based syntax compared to the old function-based syntax, and find this much more readable.

The ``Student`` class is a subclass of ``tuple``, so it can be handled like any normal ``tuple``:

.. code-block :: python
    
    >>> isinstance(tommy, tuple)
    True
    >>> tommy[0]
    'Tommy Johnson' 

A more advanced example, subclassing ``Student`` and using default values (note: default values require Python >= **3.6.1**):

.. code-block :: python
    
    >>> class MaleStudent(Student):
    >>>    sex: str = 'M'  # default value, requires Python >= 3.6.1 
    
    >>> MaleStudent(name='Tommy Johnson', address='Main street', age=22)
    MaleStudent(name='Tommy Johnson', address='Main street', age=22, sex='M')  # note that sex defaults to 'M'

In short, this modern version of namedtuples is just super-nice, and will no doubt become the standard namedtuple variation in the future.

``types.SimpleNamespace``
-------------------------
 
``types.SimpleNamespace`` is a simple class that provides attribute access to its namespace, as well as a meaningful repr. It was added in Python 3.3.

.. code-block :: python
    
    >>> from types import SimpleNamespace
    >>> data = SimpleNamespace(a=1, b=2)
    >>> data
    namespace(a=1, b=2)
    >>> data.c = 3
    >>> data
    namespace(a=1, b=2, c=3)

In short, ``types.SimpleNamespace`` is just a ultra-simple class, allowing you to set, change and delete attributes while  it also provides a nice repr output string. I sometimes use this as an easier-to-read-and-write alternative to ``dict`` or I subclass it to get the flexible instantiation and repr output for free:


.. code-block :: python
    
    >>> import random
    >>> class DataBag(SimpleNamespace):
    >>>    def choice(self):
    >>>        items = self.__dict__.items()
    >>>        return random.choice(tuple(items))
  
    >>> data_bag = DataBag(a=1, b=2)
    >>> data_bag
    DataBag(a=1, b=2)
    >>> data_bag.choice()
    (b, 2)
    
This subclassing of  ``types.SimpleNamespace`` is probably not really revolutionary, but it can save on a few lines of text in some common cases, which is nice.

Conclusion
------------

I hope you enjoyed this little walkthrough of some new data structures in Python 3.

Translations
-------------

A `Korean translation of this article <https://mnpk.github.io/2017/03/16/python3-data-structure.html>`_ has been made, courtesy of `mnpk <https://mnpk.github.io/about.html>`_. Thanks!

.. _birthday: https://www.reddit.com/r/Python/comments/5v0tt6/python_3_created_via_pep_3000_is_exactly_3000/
.. _docs: https://docs.python.org/3/library/types.html#types.MappingProxyType
.. _typingNamedTuple: https://docs.python.org/3/library/typing.html#typing.NamedTuple
