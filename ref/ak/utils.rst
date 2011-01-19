=========
Utilities
=========

In the `utils.js`_ file various utility functions and classes are
defined.

.. _utils.js: http://www.akshell.com/apps/ak/code/0.2/utils.js


Function Utilities
==================

.. function:: bind(func, self)

   Return a :dfn:`bound function`, i.e., a function receiving *self*
   as ``this``. If *func* is a string, ``self[func]`` will be used as
   a function. ::

      >>> bind('join', [1, 2, 3])('+')
      1+2+3

.. function:: partial(func[, args...])

   Return a :dfn:`partial function`, i.e., a function which when
   called will behave like *func* called with *args*. If more
   arguments are supplied to the call, they are appended to *args*. ::

      >>> partial(function () { return Array.join(arguments, ', '); },
                  1, 2)(3, 4)
      1, 2, 3, 4


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


MemTextStream
=============

.. class:: MemTextStream

   A console emulator. Targeted at debugging.

   .. method:: write(value)

      Coerce *value* to ``string`` and append it to the stream buffer.

   .. method:: get()

      Return the contents of the stream buffer as a ``string``.

   .. method:: reset()

      Reset the contents of the stream buffer.

   ::

      (function ()
      {
        var s = new MemTextStream();
        s.write('Hello world!');
        s.write(42);
        assertSame(s.get(), s.get());
        assertSame(s.get(), 'Hello world!42');
        s.reset();
        assertSame(s.get(), '');
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
                d.set(42, 'number');
                d.set('42', 'string');
                return repr(d);
              })()
         {42: "number", "42": "string"}


Miscellaneous Utilities
=======================

.. function:: abstract()

   Throw a :exc:`NotImplementedError`. Useful for declaring methods
   which should be defined by subclasses.

.. function:: timeSince(date, now=new Date())

   Format *date* as the time since that date, e.g., ``'4 days, 6
   hours'``.  *now* is the date to use as the comparison point
   (defaults to now). Minutes is the smallest unit used, and ``'0
   minutes'`` will be returned for any date that is in the future
   relative to the comparison point.

.. function:: timeUntil(date, now=new Date())

   Format *date* as the time from *now* until that date. *now* is the
   date to use as the comparison point (defaults to now). Minutes is
   the smallest unit used, and ``'0 minutes'`` will be returned for
   any date that is in the past relative to the comparison point.

.. function:: escapeHTML(string)

   Escape *string's* HTML. Specifically, make these replacements:

   * ``<`` is converted to ``&lt;``
   * ``>`` is converted to ``&gt;``
   * ``'`` (single quote) is converted to ``&#39;``
   * ``"`` (double quote) is converted to ``&quot;``
   * ``&`` is converted to ``&amp;``
