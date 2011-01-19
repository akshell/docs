====
Base
====

In the `base.js`_ file the library extends standard JavaScript by some
features borrowed mainly from Python_ making it more suitable for
creating modular server-side applications. The code was inspired by
the MochiKit_ client-side JavaScript library, by Bob Ippolito.

.. _base.js: http://www.akshell.com/apps/ak/code/0.2/base.js
.. _Python: http://python.org/
.. _MochiKit: http://mochikit.com/


Object Functions
================

.. function:: update(self[, attributes], objects...)

   Mutate the object *self* by setting its key:value pairs to those
   from *objects* (key:value pairs from later objects will overwrite
   those from earlier ones); return *self*. Property setting is done
   via the :func:`set` function called with *attributes*; if
   *attributes* are not specified they default to :data:`COMMON`.

.. function:: items(object)

   Return an array of ``[propertyName, propertyValue]`` pairs for
   *object* in the order determined by the ``for..in`` loop.

.. function:: keys(object)

   Return an array of the *object* property names in the order
   determined by the ``for..in`` loop.

.. function:: values(object)

   Return an array of the *object* property values in the order
   determined by the ``for..in`` loop.


Function Methods
================

.. class:: Function

   Added ``Function`` methods.

   .. method:: decorated(decorators...)

      Return a :term:`decorated<decorator>` function; *decorators* are
      applied in reverse order.

   .. method:: wraps(func)

      Borrow the ``prototype``, ``__proto__``, and ``__name__``
      properties from *func*; return ``this``. Useful for writing
      :term:`decorators<decorator>`.

   .. method:: subclass([constructor,] prototype={})

      Return a :term:`subclass` of this :term:`class`. ``subclass()``
      is a heart of Akshell object-oriented system; it brings classes
      to the prototype-oriented world of JavaScript in a natural
      way. Each function can be a class, i.e., could be used for
      creating objects by the operator ``new``. A class can be
      subclassed via the ``subclass()`` method; a subclass inherits
      methods and attributes of its parent class, which are specified
      in the ``prototype`` property of the parent class; so objects of
      the subclass can be used wherever objects of the parent class
      are required.

      *constructor* defaults to a function simply calling this
      function if this function is not ``Object``; otherwise it
      defaults to an empty function. ::

         var Figure = Object.subclass(
           {
             getArea: abstract
           });

         var Rectangle = Figure.subclass(
           function (a, b) {
             this._a = a; // leading underscore marks private attributes
             this._b = b;
           },
           {
             getArea: function () { return this._a * this._b; }
           });

         var Square = Rectangle.subclass(
           function (a) {
             Rectangle.call(this, a, a);
           });

      ``Function`` is a class of classes, a so-called
      :term:`metaclass`. By subclassing ``Function`` one could produce
      other metaclasses; the ``subclass()`` method could be redefined
      in them to alter the behavior of the class machinery. This is
      for advanced use only; do **not** use metaclasses unless you
      understand what you are doing and failed to find a simpler
      approach!

   .. method:: subclassOf(base)

      Test if this class is a subclass of the class *base*.


Value Representation
====================

.. function:: repr(value)

   Return a *value* representation. This function is targeted on
   debugging. One could add ``repr()`` support to his own class by
   adding the ``__repr__()`` method to it. ::

      >>> repr(42)
      42
      >>> repr(true)
      true
      >>> repr("Some\" tricky\n\t'string'")
      "Some\" tricky\n\t'string'"
      >>> repr({n: 42, s: "string"})
      {n: 42, s: "string"}
      >>> repr({__repr__: function () { return 'My own repr!'; }})
      My own repr!


Value Comparison
================

The JavaScript comparison operators are practically limited to numbers
and strings, and it's impossible to extend their scope. To overcome
this shortcoming Akshell provides these comparison functions.

.. function:: cmp(lhs, rhs)

   Return -1 if *lhs* is less than *rhs*, 0 if they are equal, +1 if
   *lhs* is greater than *rhs*; throw a :exc:`CmpError` if these
   values are incomparable. The comparison algorithm is:

   1. if the values are equivalent (``lhs === rhs``) return ``0``;

   2. if *lhs* has a ``__cmp__`` method return ``lhs.__cmp__(rhs)``;

   3. if *rhs* has a ``__cmp__`` method return ``-rhs.__cmp__(lhs)``;

   4. the values are incomparable -- throw ``CmpError(lhs, rhs)``.

   Your own types can support ``cmp`` by providing a method
   ``__cmp__(other)``; it should

   * return -1 if *this* is less than *other*;
   * return  0 if *this* is equal to *other*;
   * return +1 if *this* is greater than *other*;
   * throw ``CmpError(this, other)`` if *this* and *other* are
     incomparable.

   ::

      (function ()
      {
        assertSame(cmp(null, null), 0);
        var C = Object.subclass(
          function (n) {
            this._n = n;
          },
          {
            __cmp__: function (other) {
              if (!(other instanceof C))
                throw CmpError(this, other);
              return cmp(this._n, other._n);
            }
          });
        assertSame(cmp(new C(0), new C(0)), 0);
        assertSame(cmp(new C(1), new C(0)), 1);
        assertSame(cmp(new C(0), new C(1)), -1);
        assertThrow(CmpError, cmp, new C(0), 42);
      })()

   The ``__cmp__(other)`` method of ``Number``, ``String``,
   ``Boolean``, and ``Date`` throws a :exc:`CmpError` if *other* is
   not a value/object of the same type/class; if follows common
   comparison semantics otherwise::

      >>> cmp(true, false)
      1
      >>> cmp('abc', 'def')
      -1
      >>> cmp(42, new Number(42))
      0
      >>> cmp(new Date('Feb 1 2010'), new Date('Sep 13 2010'))
      -1
      >>> cmp(42, '42')
      CmpError: ...
      >>> cmp(0, false)
      CmpError: ...
      >>> cmp(false, null)
      CmpError: ...

   The ``__cmp__(other)`` method of ``Array`` perform a lexicographic
   comparison of array-like objects: it iterates over the objects and
   returns

   * ``cmp(this[i], other[i])`` where ``i`` is the smallest index less
     than ``this.length`` and ``other.length`` such that
     ``cmp(this[i], other[i]) != 0``;

   * ``cmp(this.length, other.length)`` if such ``i`` does not exist.

.. exception:: CmpError(lhs, rhs)

   Values *lhs* and *rhs* are incomparable.

.. function:: equal(lhs, rhs)

   Return ``true`` is *lhs* and *rhs* are equal, ``false``
   otherwise. This is achieved by the following algorithm:

   1. if the values are equivalent (``lhs === rhs``) return ``true``;

   2. if *lhs* has an ``__eq__`` method return ``lhs.__eq__(rhs)``;

   3. if *rhs* has an ``__eq__`` method return ``rhs.__eq__(lhs)``;

   4. return ``true`` if ``cmp(lhs, rhs) == 0``, ``false`` if it's
      non-zero or a :exc:`CmpError` was thrown.

   If your class has a ``__cmp__`` method it already supports
   ``equal``. Classes which have equality semantics but don't have
   order semantics should define a ``__eq__(other)`` method returning
   ``true`` if *this* and *other* are equal and ``false``
   otherwise. The ``__eq__`` method could also be added for
   optimization reasons. ::

      >>> equal(42, 42)
      true
      >>> equal(42, new Number(42))
      true
      >>> equal(42, '42')
      false
      >>> equal([1,2, [3, 4]], [1, 2, [3, 4]])
      true
      >>> equal({}, {})
      false
      >>> equal({__eq__: function () { return true; }}, null)
      true


.. _debug_tools:

Debug Tools
===========

.. exception:: AssertionError

   Assertion failed. Subclass of :exc:`BaseError`.

.. function:: assert(value[, message])

   Throw an :exc:`AssertionError` if ``!value``.

.. function:: assertSame(lhs, rhs[, message])

   Throw an :exc:`AssertionError` if ``lhs !== rhs``.

.. function:: assertEqual(lhs, rhs[, message])

   Throw an :exc:`AssertionError` if ``!equal(lhs, rhs)``.

.. function:: assertThrow(errorClass, func[, args...])

   Evaluate ``func.apply(global, args)``; throw an
   :exc:`AssertionError` if exception wasn't thrown or if the thrown
   exception was not instance of *errorClass*.


Array Functions
===============

Akshell makes the following ``Array`` methods available as ``Array``
properties to be used as generic functions on array-like objects:

* ``every``
* ``filter``
* ``indexOf``
* ``forEach``
* ``join``
* ``lastIndexOf``
* ``map``
* ``pop``
* ``push``
* ``reverse``
* ``shift``
* ``slice``
* ``some``
* ``sort``
* ``splice``
* ``unshift``

::

   >>> (function () { return Array.shift(arguments); })(1, 2, 3, 4)
   1
   >>> repr((function () { return Array.splice(arguments, 1, 2); })(1, 2, 3, 4))
   [2, 3]
   >>> Array.indexOf({0: 'a', 1: 'b', 2: 'c', length: 3}, 'b')
   1


String Methods
==============

.. class:: String

   Akshell adds two useful methods to the ``String`` class.

   .. method:: startsWith(prefix)

      Test if the string starts with *prefix*.

   .. method:: endsWith(suffix)

      Test if the string ends with *suffix*.


RegExp Escaping
===============

.. function:: RegExp.escape(string)

   Return a string escaped for embedding into a regular expression.

   >>> RegExp.escape('.*')
   \.\*
   >>> RegExp('^' + RegExp.escape('.*') + '$').test('some string')
   false
   >>> RegExp('^' + RegExp.escape('.*') + '$').test('.*')
   true
