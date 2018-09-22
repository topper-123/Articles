# optioneer - make in-program options

These days, Python has a **lot** of choices for creating CLI options:
* ``argparse``, ``getopt`` and ``optparse`` (deprecated) from the standard library
*  [click](http://click.pocoo.org/), [docopt](http://docopt.org/) and
   [many others](http://docs.pyinvoke.org/en/1.2/#the-invoke-cli-tool)

It is safe to say Python got creating CLI options covered.

But I felt for some time that in many cases CLI options is not really what you want to offer your users.
In particular, if your code is a libray that other programmers will interact with interactively or programatically
quite often you want to expose some package-wide options that they can set in python.

For example, you want to have some default options set in your library, and if the users wants
to, they could change those options.

So, for example, consider a package that allows you to interactively download data from some source
and manipulate the data. Now, you *could* make all http and other machinery options
available as function/method parameters, but that could become overwhelming and make
your API large and ugly. So, in this package, the focus should be on the data and function
parameters should reflect this focus. So, it would be much better to have centralized
and package-wide options for these needed, but not central options. 

So, you'd want to expose something like this to your users:

```python
import mylib

mylib.options.security.require_https = False
mylib.options.connections.max_retry = 3
mylib.options.connections._404_raises = True

# do something here 
```

Now, 80 % of the code needed for the above options to work could be mashed together very quickly,
but have you thought of all the use cases? Will you need to refactor everything later? For example:

* Do you want to have a description for each option?
* Should similar options be grouped (i.e. group ``max_retry`` and ``_404_raises`` above)?
* should you preemptively guard against the user setting wrong values?
* How do you handle deprecation of a option?
* If you've already made a library with options set in various locations, how do
  you collect them in one coherent object?
* Should changing an option trigger a callback to something?
* should the library user get a warning if they an unsafe change that is wrong 99 %
  of the time, but the correct choice 1 % of the time?

The above issues look like stuff you don't *really* want to think about, and you should
be able to just pip install a package to deal with. Well, you're in luck, because that's
what [optioneer](https://pypi.org/project/optioneer/) does!

## Introducing ``optioneer``

``optioneer`` let's you create in-program options, while solving the above issues.
Typically, in your package, you'd have a ``config.py`` file, where you set up your options:

```python
    from optioneer import Optioneer
    options_maker = Optioneer()
    options_maker.register_option('api_key', 'abcdefg', doc='The API key to our service')
    options_maker.register_option('display.width', 200, doc='Width of our display')
    options_maker.register_option('display.height', 200, doc='Height of our display')
    options_maker.register_option('color', 'red', validator=options_maker.is_str)

    options = options_maker.options
```

Then, in the relevant location of your library, just do
``from .config import options`` and you're got your options set up.

Users of your library can now access the options from the chosen location
in your package. For example, if you've made it available in the top-level
``__init__.py`` of a package called ``mylib``:

```python
 >>> import mylib
 >>> import mylib.options
 Options(
   api_key: The API key to our service.
       [default: abcdefg] [currently: abcdefg]
   color: No description available.
       [default: red] [currently: red]
   display.height: Height of our display
       [default: 200] [currently: 200]
   display.width: Width of our display
       [default: 200] [currently: 200]
   )
```

Notice how the repr output shows the relevant options and their descriptions.

The relevant options are discoverable using tabs in the REPL:

```python
 >>> mylib.options.<TAB>
 option.api_key options.color options.display
 >>> mylib.options.display.<TAB>
 options.display.height options.display.width
```

You can also easily see the options and their values and docs for subgroups in
the repr string:

```python
 >>> mylib.options.display
 Options(
   display.height: Height of our display
       [default: 200] [currently: 200]
   display.width: Width of our display
       [default: 200] [currently: 200]
   )
```

### Callbacks
By providing a callback when registering options, changed options may trigger
a desired actions. For example, if you in your ``config.py`` do:

```python
 options_maker.register_option('shout', True, callback=lambda x: print("YEAH!"))
 ```

Then the user, when changing that option will trigger the callback:

```python
 >>> mylib.options.shout = False
 YEAH!
```

Of course, the callback can be more realistic than above, e.g. logging or
setting some internal option or something else.

### Deprecating options

If you need to deprecate an option, ``optioneer`` allows you to do that:

```python
 options_maker.deprecate_option('api_key', msg='An api key is no longer needed')
```

Now your users get a deprecation warning, if they access this option:

```python
 >>> mylib.options.api_key
 An api key is no longer needed
 C:\Users\TP\Documents\Python\optioneer\optioneer\lib.py:677: FutureWarning: An api key is no longer needed
   warnings.warn(deprecated_option.msg, FutureWarning)
 Out[20]: 'abcdefg'
```

If an options should be renamed and/or a marker should be set for when the
option will be removed, that is also possible:

```python
 options_maker.register_option('display.length', 300, doc='Length of our display')
 options_maker.deprecate_option('display.height', redirect_key='display.length',
                                removal_version='v1.3')
```

Then accessing the option will show

```python
 >>> mylib.options.display.height
 C:\Users\TP\Documents\Python\optioneer\optioneer\lib.py:689: FutureWarning: 'display.height' is deprecated and will be removed in v1.3, please use 'display.length' instead.
   warnings.warn(msg, FutureWarning)
 Out[24]: 300
```

Deprecated options will not show up in the repr output or when tab-completing.
