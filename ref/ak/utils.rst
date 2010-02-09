
=========
Utilities
=========

In the `utils.js`_ file various utility functions and classes are
defined.

.. _utils.js: http://www.akshell.com/apps/ak/code/utils.js


Function Utilities
==================

.. function:: bind(func, self)

   Return a :dfn:`bound function`, i.e., a function receiving *self*
   as ``this``. If *func* is a string, ``self[func]`` will be used as
   a function. ::

      >>> bind('join', [1, 2, 3])('+')
      1+2+3

.. function:: partial(func[, args... ])

   Return a :dfn:`partial function`, i.e., a function which when
   called will behave like *func* called with *args*. If more
   arguments are supplied to the call, they are appended to *args*. ::

      >>> partial(function () { return Array.join(arguments, ', '); },
                  1, 2)(3, 4)
      1, 2, 3, 4

.. function:: giveNames(namespace)

   Recursively set the ``__name__`` property of all functions and
   modules of the *namespace* object. ::

      (function ()
      {
        global.abc = new Module('abc');
        abc.foo = function () {};
        abc.submodule = new Module();
        abc.submodule.bar = function () {};
        giveNames(abc);
        assertSame(repr(abc.foo), '<function abc.foo>');
        assertSame(repr(abc.submodule), '<module abc.submodule>');
        assertSame(repr(abc.submodule.bar), '<function abc.submodule.bar>');
      })()

.. function:: abstract()

   Throw a :class:`NotImplementedError`. Useful for declaring methods
   which should be defined by subclasses.


List Utilities
==============

.. function:: sum(list, start=0)

   Return the sum of the *list* items plus the value of parameter
   *start*. When the list is empty, return start.

.. function:: range([start,] stop[, step])

   Return an array containing an arithmetic progression of integers.
   ``range(i, j)`` returns ``[i, i+1, i+2, ..., j-1]``; *start* (!)
   defaults to 0; the end point is omitted. When *step* is given, it
   specifies the increment (or decrement). ::

      >>> repr(range(5))
      [0, 1, 2, 3, 4]
      >>> repr(range(2, 5))
      [2, 3, 4]
      >>> repr(range(10, 0, -2))
      [10, 8, 6, 4, 2]

.. function:: zip(list1[, list2 [...]])

   Return an array of arrays where each one contains the i-th elements
   from each of the argument lists.  The returned array is truncated
   in length to the length of the shortest argument list. ::

      >>> repr(zip([1, 2, 3], [4, 5, 6], [7, 8, 9, 10]))
      [[1, 4, 7], [2, 5, 8], [3, 6, 9]]


Parsing Utility
===============

.. function:: nextMatch(re, string, errorClass=SyntaxError)

   Try to match *string* against the regular expression *re*; return a
   match object if parsing succeeded or ``null`` if the whole *string*
   was parsed (``re.lastIndex == string.length``). Throw an error of
   *errorClass* on parse failure. This function is extremely useful
   for creating parsers of domain-specific languages; see `db.js`_ and
   `template.js`_ for examples.

   .. _db.js: http://www.akshell.com/apps/ak/code/db.js
   .. _template.js: http://www.akshell.com/apps/ak/code/template.js


Time Utilities
==============

.. function:: timeSince(date, now=new Date())

   Format *date* as the time since that date, e.g., ``'4 days, 6
   hours'``.  *now* is the date to use as the comparison point
   (defaults to now). Seconds is the smallest unit used, and ``'0
   seconds'`` will be returned for any date that is in the future
   relative to the comparison point.

.. function:: timeUntil(date, now=new Date())

   Format *date* as the time from *now* until that date. *now* is the
   date to use as the comparison point (defaults to now). Seconds is
   the smallest unit used, and ``'0 seconds'`` will be returned for
   any date that is in the past relative to the comparison point.


Stream
======

.. class:: Stream

   A console emulator. Targeted at debugging. 

   .. method:: write(values...)

      Coerce *values* to strings and store them in the stream buffer.

   .. method:: read()

      Return the contents of the stream buffer as a ``string`` and
      empty the buffer.

   ::

      (function ()
      {
        var s = new Stream();
        s.write(1, 2, 3, '\n');
        s.write('Hello', ', ', 'world!');
        assertSame(s.read(), '123\nHello, world!');
        assertSame(s.read(), '');
        s.write('Buy!');
        assertSame(s.read(), 'Buy!');
      })()

.. data:: out

   The standard debug output stream.
   
.. function:: dump(values...)

   Dump representations of *values* to the stream :data:`out`
   separated by ``'\n'``.


Dict
====

.. class:: Dict

   A dictionary designed for mapping objects to arbitrary
   values. Dictionary keys are distinguished by identity (the operator
   ``===``) . Implemented as a hash map via the :func:`hash`
   function. Should not be used for storing non-objects because their
   handling by ``Dict`` is ineffective -- use plain ``Object``
   instances instead.

   .. method:: clear()

      Remove all items from the dictionary.
   
   .. method:: set(key, value)

      Map *key* to *value*.

   .. method:: get(key, default_=undefined)

      Return the value of *key*; if *key* is not found, return
      *default_*.

   .. method:: has(key)

      Test if the dictionary has *key*.

   .. method:: setDefault(key, default_=undefined)

      Return the value of *key*; if *key* is not found, map it to
      *default_* and return *default_*.

   .. method:: pop(key, default_=undefined)

      Remove *key* and return its value; if *key* is not found, return
      *default_*.

   .. method:: popItem()

      Remove and return some ``[key, value]`` pair; return
      ``undefined`` if the dictionary is empty.

   .. method:: map(func, self=global)

      Return an array of the results of applying *func* to the items
      of the dictionary; pass *self* to *func* as ``this``. ::

         (function ()
         {
           var d = new Dict();
           d.set({x: 0}, 'zero');
           d.set({x: 1}, 'one');
           var f = function (key, value) { return key.x + ':' + value; };
           assertEqual(d.map(f).sort(), ['0:zero', '1:one']);
         })()
         
   .. method:: items()

      Return ``[key, value]`` pairs of the dictionary in arbitrary
      order.
   
   .. method:: keys()
   
      Return the dictionary keys in arbitrary order.
      
   .. method:: values()
   
      Return the dictionary values in arbitrary order.
   
   .. method:: __eq__(other)

      Test if the *other* dictionary equals ``this``; called by
      :func:`equal`.

   .. method:: __repr__()

      Return the representation of the dictionary; called by
      :func:`repr`. ::

         >>> (function () {
                var d = new Dict();
                d.set(ak, 42);
                d.set(ak.Dict, 'Dict class!');
                return repr(d);
              })()
         {<module ak 0.1>: 42, <function ak.Dict>: "Dict class!"}
