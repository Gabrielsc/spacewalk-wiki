## WARNING

All python code must be compatible with Python 2 ( including 2.4 on RHEL5) and Python 3.
For additional information see articles:
  * [Cheat Sheet: Writing Python 2-3 compatible code](http://python-future.org/compatible_idioms.html)
  * [Writing code that runs under both Python2 and 3](https://wiki.python.org/moin/PortingToPy3k/BilingualQuickRef)


This is a rough guideline about coding styles, comments, variable names and other stuff in the Python
code for the Spacewalk backend, for full information, see [PEP 8 - Style Guide for Python Code](http://www.python.org/dev/peps/pep-0008/)

1. Line wrapping

   One shall not wrap up lines just because we need to make the text
   fit to 60 columns, or 70 columns, or 80 columns. Most code written
   today has a hard time fitting within 80 columns. Let's be
   reasonable here. I am using xterms that are 100 cols x 80 rows. I
   am trying to make everything fit to the 80 columns mark, but if for
   some reason a line expands to be 83 columns, don't break it
   needlessly.

2. Variable and function Names

   Please use variable names that are easy to write and understand
   what they are for. Please try to match the field names most likely
   found in the SQL queries that make use of those variables. So
   please use system_id, not systemId, org_id, not orgId and so forth.
   Regarding function names, studly caps *suck*. I personally favor
   `some_other_function` instead of `someOtherFunction`. It is easier to
   read/spot/grep/parse whatever. Yeah, it is C style and again, there
   is a reason why C coding style has been around for so long...

3. Comments

   Comments should be written as [docstring](http://www.python.org/dev/peps/pep-0257/). This way programmers documentation can be created automatically.

   So please use something like this:

   ```python
   def auth(system_cert):
       """ Authenticate the server certificate """
   ```

   Instead of:

   ```python
   def auth(system_cert):
       # Authenticate the server cert
   ```
   Or instead of:

   ```python
   # Authenticate the server cert
   def auth(system_cert):
   ```

4. Classes and inheritance

   Please don't build up classes and inherit from them where you think
   that it may produce more readable code without considering the
   consequences of overloading classes with members that really have
   nothing to do with the base class. A classic example is having
   methods in a server object designed to solve a dependency, or
   extract a package path. Neither of these functions need anything
   from the base server object to function completely. Even though it
   is easier to be able to call `server.solve_dep` in some piece of code
   when we already have the server object, this thoroughly confuses the
   meaning of a server object and makes updating the base class a whole
   lot harder because of all these hidden dependencies and reliance on
   unspecified behavior of the base class. Think it through, sometimes
   it makes a whole lot more sense to have an extra module that deals
   only with those aspects.

5. Internal functions

   Internal functions in the code (usually denoted by a `_` or by a `__` prefix) are a cool thing when they are not abused. Having an
   internal function that is two lines long and is called from a single place in the whole code base does nothing but complicate the whole code base and makes it harder to follow.

6. The % operator on long lines

   The % operator (and, if required, the opening paren for the tuple)
   should follow immediately the string on the same line.
   Use this:
    
   ```python

        "long string" % (
	    arg1, args2)
   ```

   Do not do this:
   
   ```python
        "long string"
	    % (arg1, arg2)
   ```
   
7. Translations

   Be careful how you translate strings. Translation is done usually
   by using the _() function on a string of text. Make sure you use
   only the string of text and not include the arguments to string
   formatters too.
   
   Use this:
   ```python
       print _("Foo %s: %s") % ("bar", 1)
   ```

   Do not do this:
   ```python
       print _("Foo %s: %s" % ("bar", 1))
   ```

   There is a BIG difference.


8. Consistency (Grandfather clause)

   If you are editing an existing file that follows a different style
   than that suggested in these guidelines, please conform to
   to the foreign style for your changes in that particular file (when
   in Rome...).  This is better than changing the entire file to meet
   the official guidelines (which can introduce regression), and *much*
   better than using multiple styles in a single file (difficult to
   read).  In general, we want to maintain consistency and readability
   within files. 


