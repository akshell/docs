====
Core
====

Core Akshell functions and exceptions.


Functions
=========

.. function:: set(object, name, attributes, value)

   Set the property *name* of *object* to *value*; if the property was
   not defined before, it's created with *attributes*. There are four
   property attributes available:

   .. data:: COMMON

      Common: no special treatment.

   .. data:: READONLY

      Read-only: values of ``READONLY`` properties cannot be changed.

   .. data:: HIDDEN

      Non-enumerable: ``HIDDEN`` properties do not appear in
      ``for..in`` loops.

   .. data:: PERMANENT

      Non-deletable: ``PERMANENT`` properties cannot be deleted.

   Several attributes can be combined by the "bitwise or" operator
   ``|``::

      (function ()
      {
        var object = {};
        set(object, 'x', READONLY | HIDDEN | PERMANENT, 42);
        assertSame(object.x, 42);
        object.x = 0;
        assertSame(object.x, 42);
        assertEqual(keys(object), []);
        assert(!delete object.x);
        assertSame(object.x, 42);
      })()

.. function:: hash(value)

   Return an identity hash of an object if ``typeof(value)`` is either
   ``'object'`` or ``'function'``; return 0 otherwise. An :dfn:`object
   identity hash` is a non-zero integer; it's **not** guaranteed to be
   unique.

.. function:: construct(constructor, args)

   Instantiate *constructor* with *args*; *args* must be an ``Array``.


Exceptions
==========

.. exception:: ValueError

   Inappropriate argument value (of correct type).

.. exception:: NotImplementedError

   Function hasn't been implemented yet.

.. exception:: RequireError

   Failed to :func:`require` a module.

.. exception:: QuotaError

   A quota has been exceeded.
