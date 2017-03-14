New interesting data structures in Python 3
=============================================

Python 3 is no longer new.

In fact, recently, it's 3000 days birthday was celebrated :). After quite some wait, Python 3's uptake is dramatically on the rise, and I think it is therefore time to take a look at some data structures that Python 3 offers, but that are not available in Python 2. 

We will take a look at ``types.MappingProxyType``, ``typing.NamedTuple`` and ``types.SimpleNamespace``, all of which are new to Python 3.

``types.MappingProxyType``
-------------------------

``types.MappingProxyType`` is used as a read-only dict and was added in Python 3.3. See docs_ for details.

That ``types.MappingProxyType`` is read-only means that it can't be directly manipulated and if the user wants to make changes, he has to deliberately make a copy, and make changes to that copy. This is perfect if you're handing a ``dict`` -liek structure over to a data consumer, and you want to ensure that the data consumer is not unintentionally changing the original data. In practical use this is often extremely useful, as cases of data consumers changing passed-in data structures leads to very obscure bugs in your code that are difficult to track down.

A ``types.MappingProxyType`` example:

.. code-block :: python

    >>> from  types import MappingProxyType
    >>> data = {'a': 1, 'b':2}
    >>> read_only = MappingProxyType(a)
    >>> del read_only['a']
    TypeError: 'mappingproxy' object does not support item deletion
    >>> read_only['a'] = 3
    TypeError: 'mappingproxy' object does not support item assignment
      
Note that ``read_only`` cannot be directly changed. So, if you want to deliver data dicts to different functions or threads and want to ensure that a function is not changing data needed for another function, you can just deliver a ``MappingProxyType`` object to all functions, rather than the original ``dict``, and the data dict cannot be changed unintentionally:

.. code-block :: python
    
    >>> def my_threaded_func(in_dict):
    >>>    ...
    >>>    in_dict['a'] *= 10  # oops, a bug, this will change the sent-in dict
    
    ...
    # in some function/thread:
    >>> my_threaded_func(data)
    >>> data
    data = {'a': 10, 'b':2}  # note that data['a'] has changed as an side-effect of calling my_threaded_func

if you send in a ``mappingproxy`` to ``my_threaded_func`` instead, however, attempts to change the dict will result in an error:

.. code-block :: python

    >>> my_threaded_func(MappingProxyType(data))
    TypeError: 'mappingproxy' object does not support item deletion
    
We now see that we have to correct ``my_threaded_func`` to first copy ``in_dict`` and alter the copied dict to avoid this error. This feature is great, as it helps us avoid a whole class of difficult-to-find bugs.

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

In comparions to ``collections.namedtuple``, ``typing.NamedTuple`` gives you:

- nicer syntax
- inheritance
- type hints
- default values (python >= 3.6.1)

See an example below:

.. code-block :: python
    
    >>> from typings import NamedTuple
    >>> class Student(NamedTuple):
    >>>    name: str
    >>>    address: str
    >>>    age: int
    >>>    sex: str
    
    >>> tommy = Student(name='Tommy Johnson', address='Main street', age=22, sex='M')
    >>> tommy
    Student(name='Tommy Johnson', address='Main street', age=22, sex='M')


I like the subclassing syntax compared to the old function-based syntax, and find this much more readable.

Note that we're really having a tuple here, not a normal class instance:

.. code-block :: python
    
    >>> isinstance(tommy, tuple)
    True
    >>> tommy[0]
    'Tommy Johnson' 

A more advanced example, subclassing ``Student`` and using default values (note: default values require Python >= **3.6.1**):

.. code-block :: python
    
    >>> class MaleStudent(Student):
    >>>    sex: str = 'M'  # default value, requires Python >= 3.6.1 
    
    >>> Student(name='Tommy Johnson', address='Main street', age=22)
    Student(name='Tommy Johnson', address='Main street', age=22, sex='M')  # note that sex has a defaults to 'M'

In short, this modern version of namedtuples is just super-nice, and will no doubt become the standard namedtuple variation in the future.

``types.SimpleNamespace``
-------------------------
 
``types.SimpleNamespace`` is a simple class that provides attribute access to its namespace, as well as a meaningful repr. It was added in Python 3.3.

.. code-block :: python
    
    >>> from types import SimpleNamespace
    >>> data = SimpleNamespace(a=1, b=2)
    >>> data
    namespace(a=1, b=2)
    data.c = 3
    >>> data
    namespace(a=1, b=2, c=3)

In short, ``types.SimpleNamespace`` is just a ultra-simple class, allowing you to set, change and delete attributes while also provides a nice repr output string. I sometimes use it as an easier-to-read-and-write alternative to ``dict`` or I subclass it to get the flexible instantiation and repr output for free.

I hope you enjoyed this little walkthrough of some new data structures in Python 3.

.. _docs: https://docs.python.org/3/library/types.html#types.MappingProxyType
.. _typingNamedTuple: https://docs.python.org/3/library/typing.html#typing.NamedTuple
