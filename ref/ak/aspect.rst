
===========================
Aspect-Oriented Programming
===========================

The `aspect.js`_ file provides simple means of the `Aspect-Oriented
Programming`__ (AOP). They are designed to modify a behavior of an
application on the fly by weaving :dfn:`advices` to functions and
methods. An advice is a piece of code altering a function by adding
some secondary or supporting logic to it. An advice weaved to a
function form an :dfn:`aspect`, which can be temporary disabled or
completely unweaved. When a function is advised several times, an
:dfn:`aspect chain` is formed: the first advice is weaved to the
original function and each following advice is weaved to the previous
aspect. Each aspect of the chain can be disabled or unweaved.

__ http://en.wikipedia.org/wiki/Aspect-Oriented_Programming
.. _aspect.js: http://www.akshell.com/apps/ak/code/0.1/aspect.js

AOP aims to increase modularity by the separation of accessory logic
from business logic of an application. Aspects form an agile web of
advices stretching through the whole program.

.. warning::

   Aspects should not be used for making permanent changes of a
   control flow of an application. :term:`Decorators<decorator>`
   specially target this task.


Aspect
======

.. class:: Aspect(holder, name, advice)

   An aspect base class, subclass of ``Function``. Concrete aspect
   classes should subclass it and implement an ``_apply(self, args)``
   method. *holder* is an object holding a function being advised;
   *name* is a name of this function; *advice* is an advice function.

   .. warning::

      Don't advise functions with custom properties!  ``Aspect``
      handles only plain JavaScript functions.

   The aspect interface is:
      
   .. method:: apply(self, args)

      If the aspect is enabled, run the advised function; otherwise
      run the original function.
   
   .. attribute:: enabled

      A boolean aspect switch. Use it to enable/disable the aspect and
      to read the state of the aspect. Aspects are created with
      ``enabled`` set to ``true``.

   .. method:: unweave()

      Unweave the aspect. If this is the only aspect of the function,
      restore the function and return it; otherwise remove the aspect
      from the aspect chain and return the top aspect of the chain.
      
      
AspectArray
===========

.. class:: AspectArray

   An ``Array`` of :class:`Aspect` objects.

   .. method:: unweave()

      Unweave all aspects of the array and return an ``Array`` of
      :meth:`~Aspect.unweave` results.

   .. method:: enable()

      Enable all aspects of the array.
   
   .. method:: disable()
   
      Disable all aspects of the array.
   

weave()
=======

.. function:: weave(aspectClass, holder, names, advice, directly=false)

   Weave *advice* to the functions of the *holder* object (if
   *directly* is ``false`` and *holder* is a ``function``, use the
   ``holder.prototype`` object instead). The behavior of ``weave()``
   depends on the type of *names* argument; it can be:

   ``string``
      Interpret *names* argument as a name of the only function to be
      weaved; return an :class:`Aspect` object.
      
   ``Array``
      Interpret *names* as a list of names of the functions to be weaved;
      return an :class:`AspectArray` object.

   ``RegExp``
      Interpret *names* as a pattern which the function properties of
      the holder object should match to to be weaved; return an
      :class:`AspectArray` object.

   ::

      (function ()
      {
        var func = function () { return 'original'; };
        var holder = {foo: func, bar: func, baz: func};
        var append = function (suffix) {
          return function (result) { return result + ', ' + suffix; }
        };
        var fooAspect = weave(After, holder, 'foo', append('foo'));
        assertSame(holder.foo(), 'original, foo');
        fooAspect.enabled = false;
        assertSame(holder.foo(), 'original');
        weave(After, holder, /^b.*/, append('b.*'));
        assertSame(holder.bar(), 'original, b.*');
        assertSame(holder.baz(), 'original, b.*');
        var foobarAspects = weave(After, holder, ['foo', 'bar'],
                                  append('foobar'));
        assertSame(holder.bar(), 'original, b.*, foobar');
        assertSame(holder.foo(), 'original, foobar');
        fooAspect.enabled = true;
        assertSame(holder.foo(), 'original, foo, foobar');
        assertSame(foobarAspects.unweave()[0], fooAspect);
        assertSame(fooAspect.unweave(), func);
        assertSame(holder.foo, func);
      })()
      

Aspect Subclasses
=================

Concrete :class:`Aspect` subclasses implement various kinds of
aspects. They should be instantiated only via the :func:`weave`
function.
   
.. class:: Before

   An aspect executing the advice before the function. The advice can
   not prevent the execution of the function. It receives call
   arguments and the function name. ::

      (function ()
      {
        var object = {func: function () {}};
        weave(Before, object, 'func',
              function (args, name) {
                assertSame(this, object);
                assertEqual(args, [1, 2, 3]);
                assertSame(name, 'func');
              });
        object.func(1, 2, 3);
      })()

.. class:: After

   An aspect executing the advice after the function has completed an
   execution successfully. The advice receives the result of the
   execution, call arguments, and the function name; its return value
   is the result of the aspect execution. ::

      (function ()
      {
        var object = {func: function () { return 0; }};
        weave(After, object, 'func',
              function (result, args, name) {
                assertSame(this, object);
                assertSame(result, 0);
                assertEqual(args, [1, 2, 3]);
                assertSame(name, 'func');
                return 42;
              });
        assertSame(object.func(1, 2, 3), 42);
      })()

.. class:: AfterCatch

   An aspect executing the advice after the function has thrown an
   exception. The advice receives the exception, call arguments, and
   the function name; its return value is the result of the aspect
   execution. ::

      (function ()
      {
        var object = {func: function () { throw 'error'; }};
        weave(AfterCatch, object, 'func',
              function (error, args, name) {
                assertSame(this, object);
                assertSame(error, 'error');
                assertEqual(args, [1, 2, 3]);
                assertSame(name, 'func');
                return 42;
              });
        assertSame(object.func(1, 2, 3), 42);
      })()

.. class:: AfterFinally

   An aspect executing the advice after the function, whether an
   exception has occurred or not. The advice receives call arguments
   and the function name. ::

      (function ()
      {
        var object = {
          foo: function () {},
          bar: function () { throw 'error'; }
        };
        weave(AfterFinally, object, /./,
              function (args, name) {
                assertSame(this, object);
                assertSame(args[0], name);
              });
        object.foo('foo');
        assertThrow(String, function () { object.bar('bar'); });
      })()

.. class:: Around

   An aspect executing the advice around the function so that the
   advice has a full control over the function execution. It receives
   the function, call arguments, and the function name. ::

      (function ()
      {
        var original = function (arg) { return arg; };
        var object = {func: original};
        weave(Around, object, 'func',
              function (func, args, name) {
                assertSame(this, object);
                assertSame(func, original);
                assertSame(name, 'func');
                return args[0] == 42 ? 0 : func.apply(this, args);
              });
        assertSame(object.func(15), 15);
        assertSame(object.func(42), 0);
      })()

.. class:: InsteadOf

   An aspect executing the advice instead of the function. The advice
   is applied with call arguments. ::

      (function ()
      {
        var object = {func: function () { throw Error(); }};
        weave(InsteadOf, object, 'func',
              function () {
                assertSame(this, object);
                assertEqual(arguments, [1, 2, 3]);
                return 42;
              });
        assertSame(object.func(1, 2, 3), 42);
      })()
