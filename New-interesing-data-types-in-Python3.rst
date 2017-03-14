New interesting data Structures in Python 3
=============================================

Python 3 is no longer new. In fact, recently, it was celebrated that it's 3000 days old :). After quite some wait, Python 3's uptake is dramatically on the rise, and I think it is therefore time to take a look at some data structures that Python 3 offers, but that are not available in Python 2. 

I will take a look at ``types.MappingProxyType``, ``typing.NamedTuple`` and ``types.SimpleNamespace``.

``types.MappingProxyType``
-------------------------

``types.MappingProxyType`` is used as a read-only dict and was added in Python 3.3. See docs_ for details.

That ``types.MappingProxyType`` is read-only means that it can't be directly manipulated an dif the user wants to make changes, he has to very delibirately make a copy, and change that. This is perfect it you're handing a ``dict`` over to a consumer, and you want to ensure that the data consumer is not unintentionally changing the original data. This in in practical use often extremely useful, as cases of data consumers changing passed-in data structures leads to very obscure bugs that are difficult to track down.

An ``types.MappingProxyType`` example:

.. code-block :: python

    >>> from  types import MappingProxyType
    >>> data = {'a': 1, 'b':2}
    >>> read_only = MappingProxyType(a)
    >>> del read_only['a']
    TypeError: 'mappingproxy' object does not support item deletion
    >>> read_only['a'] = 3
    TypeError: 'mappingproxy' object does not support item assignment
      
So, if you want to deliver data dicts to different functions or threads and want to ensure that a function is not changing data for another function, you can just deliver a mappingproxy object, rather than the original ``dict``, and the datadict cannot be changed unintentionally:

.. code-block :: python
    
    >>> def my_threaded_func(in_dict):
    >>>    ...
    >>>    in_dict['a'] *= 10  # oops, this will change the sent-in dict, leading to subtle bugs
    
    ...
    # in some thread:
    >>> my_threaded_func(data)
    >>> data
    data = {'a': 10, 'b':2}  # note that data['a'] has changed

if you send in a mappingproxy to ``my_threaded_func``, attempts to change the dict will result in errors:

.. code-block :: python

    >>> my_threaded_func(MappingProxyType(data))
    TypeError: 'mappingproxy' object does not support item deletion
    
This is great, as it helps avoid a whole class of difficult bugs. We therefore have to change ``in_dict`` using ``in_dict.copy()`` before altering it to avoid this error.

Note that while ``read_only`` is read-only, it is not immutable, so if you change ``data``, ``read_only`` will change too:
 
.. code-block :: python
    
    >>> data['a'] = 3
    >>> data['c'] = 4
    >>> read_only  # changed!
    mappingproxy({'a': 3, 'b': 2, 'c': 4})

``typing.NamedTuple``
---------------------

``typing.NamedTuple`` (https://docs.python.org/3/library/typing.html#typing.NamedTuple) is a supercharged version of the venerable ``collections.namedtuple`` and while it was added in Python 3.5, it really came into its own in Python 3.6.

In comparions to ``collections.namedtuple``, ``typing.NamedTuple`` gives you:
- nicer syntax compared to the cluncy function-based syntax
- inheritance
- default values
- type hints

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


I like the subclassing syntax compared to the old namedtuple syntax, and find this very readable.

Note that we're really having a tuple here, not a normal class instance:

.. code-block :: python
    
    >>> isinstance(tommy, tuple)
    True
    >>> tommy[0]
    'Tommy Johnson' 

A more advanced example, subclassing ``Student`` and using default values (note: default values require Python 3.6.**1**):

.. code-block :: python
    
    >>> class MaleStudent(Student):
    >>>    sex: str = 'M'  # default value, requires Python >= 3.6.1 
    
    >>> Student(name='Tommy Johnson', address='Main street', age=22)
    Student(name='Tommy Johnson', address='Main street', age=22, sex='M')

In short, this modern version of namedtuples is just super-nice, and will no doubt become the standard namedtuple variations in the future.

``types.SimpleNamespace``
-------------------------
 
``types.SimpleNamespace`` (https://docs.python.org/3/library/types.html#types.SimpleNamespace) is a simple class that provides attribute access to its namespace, as well as a meaningful repr. It was added in Python 3.3.

.. code-block :: python
    
    >>> from types import SimpleNamespace
    >>> data = SimpleNamespace(a=1, b=2)
    >>> data
    namespace(a=1, b=2)
    data.c = 3
    >>> data
    namespace(a=1, b=2, c=3)

In short, ``types.SimpleNamespace`` is just a ultrasimple class, allowing setting, changing and deleting attributes and providing a nice repr output string. I sometimes use it as an easier-to-read-and-write alternative to ``dict``.


.. _docs: https://docs.python.org/3/library/types.html#types.MappingProxyType
