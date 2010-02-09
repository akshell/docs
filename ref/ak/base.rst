
====
Base
====

In the `base.js`_ file the library extends standard JavaScript by some
features borrowed mainly from Python_ making it more suitable for
creating modular server-side applications. The code was inspired by
the MochiKit_ client-side JavaScript library, by Bob Ippolito.

.. _base.js: http://www.akshell.com/apps/ak/code/base.js
.. _Python: http://python.org/
.. _MochiKit: http://mochikit.com/

The only module-level object defined here is

.. data:: global

   The global object reference; an analog of the ``window`` property
   in client-side JavaScript.
   

Object Functions and Methods
============================

``Object`` handling tools are provided as module-level functions and
as respective ``Object`` methods. The formers should be used when
dealing with objects with arbitrary properties; otherwise the latters
should be preferred because of their expressiveness.
   
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

.. class:: Object

   Added ``Object`` methods.

   .. method:: set(name, attributes, value)

      A shortcut for :func:`set`.
   
   .. method:: setHidden(name, value)

      Set a hidden property; equivalent to ``set(name, HIDDEN,
      value)``.
   
   .. method:: instances(constructor)

      Set the class of the object to *constructor*; return ``this``.

   .. method:: update([attributes,] objects...)

      A shortcut for :func:`update`.

   .. method:: items()

      A shortcut for :func:`items`.

   .. method:: keys()
   
      A shortcut for :func:`keys`.

   .. method:: values()

      A shortcut for :func:`values`.
      
   
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
      way. Each function could be a class, i.e., could be used for
      creating objects by the operator ``new``. A class could be
      subclassed via the ``subclass()`` method; a subclass inherits
      methods and attributes of its parent class, which are specified
      in the ``prototype`` property of the parent class; so objects of
      the subclass could be used wherever objects of the parent class
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


Error Subclasses
================

.. class:: ErrorMeta

   The standard JavaScript exception classes could be instantiated
   without the operator ``new``. To guarantee that all user defined
   exception classes follow this rule Akshell sets the
   :term:`metaclass` of the base exception class ``Error`` to
   ``ErrorMeta``. ``ErrorMeta`` redefines the
   :meth:`~Function.subclass` method to provide exception classes with
   the instantiation without ``new``, the initialization of stack
   trace, and a sensible ``name`` property.

   .. note::

      In Akshell the preferred style of instantiation of error classes
      is **without** ``new``.

The following subclasses of :class:`BaseError` are broadly used by the
rest of the ``ak`` library; you should also employ them in your code.
      
.. class:: ValueError

   Inappropriate argument value (of correct type).

.. class:: NotImplementedError

   Method or function hasn't been implemented yet.

   
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
      >>> repr(db)
      <module ak.db>
      >>> repr(db.create)
      <function ak.db.create>
      >>> repr({__repr__: function () { return 'My own repr!'; }})
      My own repr!


Value Comparison
================

The JavaScript comparison operators are practically limited to numbers
and strings, and it's impossible to extend their scope. To overcome
this shortcoming Akshell provides these comparison functions.

.. function:: cmp(lhs, rhs)

   Return -1 if *lhs* is less than *rhs*, 0 if they are equal, +1 if
   *lhs* is greater than *rhs*; throw a :class:`CmpError` if these
   values are incomparable. The comparison algorithm is:

   1. if the values are equivalent (``lhs === rhs``) return ``0``;

   2. if *lhs* has a ``__cmp__`` method return ``lhs.__cmp__(rhs)``;
      
   3. if *rhs* has a ``__cmp__`` method return ``-rhs.__cmp__(lhs)``;

   4. the values are incomparable -- throw ``CmpError(lhs, rhs)``.

   You can see that your own types could support ``cmp`` by providing
   a method ``__cmp__(other)``; it should

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
   ``Boolean``, and ``Date`` throws a :class:`CmpError` if *other* is
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
      ak.CmpError: ...
      >>> cmp(0, false)
      ak.CmpError: ...
      >>> cmp(false, null)
      ak.CmpError: ...
      
   The ``__cmp__(other)`` method of ``Array`` perform a lexicographic
   comparison of array-like objects: it iterates over the objects and
   returns

   * ``cmp(this[i], other[i])`` where ``i`` is the smallest index less
     than ``this.length`` and ``other.length`` such that
     ``cmp(this[i], other[i]) != 0``;

   * ``cmp(this.length, other.length)`` if such ``i`` does not exist.

.. function:: equal(lhs, rhs)

   Return ``true`` is *lhs* and *rhs* are equal, ``false``
   otherwise. This is achieved by the following algorithm:

   1. if the values are equivalent (``lhs === rhs``) return ``true``;

   2. if *lhs* has an ``__eq__`` method return ``lhs.__eq__(rhs)``;
      
   3. if *rhs* has an ``__eq__`` method return ``rhs.__eq__(lhs)``;
   
   4. return ``true`` if ``cmp(lhs, rhs) == 0``, ``false`` if it's
      non-zero or a :class:`CmpError` was thrown.

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
   
.. class:: CmpError(lhs, rhs)

   Values *lhs* and *rhs* are incomparable.


Module
======

.. class:: Module([name[, version]])

   A module representation. *name* and *version* should be strings.

   .. attribute:: __name__

      The name of the module defined by *name* constructor argument.

   .. attribute:: __version__

      The version of the module defined by *version* constructor
      argument.

   .. method:: __repr__()

      Return a string :samp:`'<module {name} {version}>'`, or
      :samp:`'<module {name}>'` for modules without a version, or
      ``'<anonymous module>'`` for modules without a name.


.. _debug_tools:
      
Debug Tools
===========

.. class:: AssertionError

   Assertion failed. Subclass of :class:`BaseError`.

.. function:: assert(value[, message])

   Throw an :class:`AssertionError` if ``!value``.

.. function:: assertSame(lhs, rhs[, message])

   Throw an :class:`AssertionError` if ``lhs !== rhs``.

.. function:: assertEqual(lhs, rhs[, message])

   Throw an :class:`AssertionError` if ``!equal(lhs, rhs)``.

.. function:: assertThrow(errorClass, func[, args...])

   Evaluate ``func.apply(global, args)``; throw an
   :class:`AssertionError` if exception wasn't thrown or if the thrown
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
