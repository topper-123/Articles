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
