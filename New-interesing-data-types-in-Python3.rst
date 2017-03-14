New interesting data Structures in Python3
------------------------------------

Python 3 is no longer a new version of Python. In fact, recently, it was celebrated that it's 3000 days old :). Pyrhon 3's uptake is also dramatically rising these days. I think it is therefore time to take a look at some data structures that Python 3 offers that are not available in Python 2. 

I will take a look at ``types.MappingProxyType``, ``types.SimpleNamespace`` and ``typing.NamedTuple``.

``types.MappingProxyType``
-------------------------

``types.MappingProxyType``[https://docs.python.org/3/library/types.html#types.MappingProxyType] is used as a read-only dict and was added in Python 3.3.

That ``types.MappingProxyType`` is read-only means that it can't be directly manipulated. This is perfect it you're handing a dict over to a consumer, and you want to assure that the consumer is not unintentionally changing the original data. This in in practical use often very useful, as consumers changing passed-in data structures are often the most difficult to debug code.

An example:

.. code-block :: python

    >>> from  types MappingProxyType
    >>> data = {'a': 1, 'b':2}
    >>> read_only = MappingProxyType(a)
    >>> del read_only['a']
    TypeError: 'mappingproxy' object does not support item deletion
    >>> read_only['a'] = 3
    TypeError: 'mappingproxy' object does not support item assignment
      
So, if you want to deliver dict data to different functions or threads and want to ensure that a single function is not changing data for another function, you just deliver a mappingproxy object, rather than the original ``dict``:


.. code-block :: python
    
    >>> def my_threaded_func(in_dict):
    >>>    ...
    >>>    in_dict['a'] *= 10  # this will change the dict for other usages, leading to subtle bugs
    
    ...
    # in some thread:
    >>> my_threaded_func(read_only)
    TypeError: 'mappingproxy' object does not support item deletion
    
Great! Using ``MappingProxyType`` ensures that ``my_threaded_func`` fails, if it tries to alter ``in_dict``, and we therefore have to change it to first copy ``in_dict`` using ``in_dict.copy()`` before altering it.

Note that while ``read_only`` is read-only, it is not immutable, so if you change ``data``, ``read_only`` will change too:
 
.. code-block :: python
    
    >>> data['a'] = 3
    >>> data['c'] = 4
    >>> read_only  # changed!
    mappingproxy({'a': 3, 'b': 2, 'c': 4})

``types.SimpleNamespace``
-------------------------
 
``types.SimpleNamespace``(https://docs.python.org/3/library/types.html#types.SimpleNamespace) is a simple class that provides attribute access to its namespace, as well as a meaningful repr. It was added in Python 3.3.

.. code-block :: python
    
    >>> from types import SimpleNamespace
    >>> data = SimpleNamespace(a=1, b=2)
    >>> data
    namespace(a=1, b=2)
    data.c = 3
    >>> data
    namespace(a=1, b=2, c=3)

In short, ``types.SimpleNamespace`` is just a ultrasimple class, allowing setting, changing and deleting attributes and providing a nice repr output string. I sometimes use it as an easier-to-read-and-write alternative to ``dict``.

``typing.NamedTuple``
---------------------

``typing.NamedTuple`` (https://docs.python.org/3/library/typing.html#typing.NamedTuple) is a more readable and a typed version of the venerable ``collections.namedtuple`` and while it was added in Python 3.5, its syntax became really nice in Python 3.6.

.. code-block :: python
    
    >>> from typings import NamedTuple
    >>> class Student(NamedTuple):
    >>>    name: str
    >>>    address: str
    >>>    age: int
    
    >>> tommy = Student(name='Tommy Johnson', address='Main street', age=22)
    Student(name='Tommy Johnson', address='Main street', age=22)

I like the subclassing syntax compared to the old namedtuple syntax, and find it very readable. 
