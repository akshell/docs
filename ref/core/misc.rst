
==================
Miscellaneous APIs
==================

Miscellaneous low-level tools.

.. function:: readCode([appName,] path)

   Return contents of a code file as a ``string``. If *appName* is a
   name of the executing application search for a file in the
   executing code base; otherwise search for it in the release code
   base of the specified application. If *appName* is omitted it
   defaults to the name of the executing application. *path* separator
   is the slash (``'/'``).
   
.. function:: include([lib,] path)

   Include a code file; return a code evaluation result. Do not
   include the same file twice: return a cached result instead. *lib*
   must be an application name optionally followed by a path within
   its release code. If *lib* is specified cross-application include
   takes place; otherwise a current application code is searched for a
   file. A :dfn:`current application` is an executing application or
   an application of the latest incomplete cross-application
   include. A current application include could be relative or
   absolute. A *relative include* happens if *path* does not begin
   with the slash; search is performed based on the directory of the
   including file. An *absolute include* happens if *path* begins with
   the slash; search is performed based on the root directory or the
   directory specified in the latest incomplete cross-application
   include.

   For example, if application ``app1`` has this lines in its
   :file:`__main__.js` file::

      include('app2/0.1', 'feature/__init__.js')
      include('utils/useful.js')

   ... they will include file :file:`utils/useful.js` of ``app1``
   application and file :file:`0.1/feature/__init__.js` of ``app2``
   application. Suppose :file:`utils/useful.js` has the lines::

      include('very-useful.js')
      include('/base.js')

   ... these lines will include files :file:`utils/very-useful.js` and
   :file:`base.js` of ``app1`` application because the first include
   was relative and the second one was absolute. Finally if
   :file:`0.1/feature/__init__.js` file of ``app2`` application has
   the lines::

      include('impl.js')
      include('/base.js')

   ... they will include files :file:`0.1/feature/impl.js` and
   :file:`0.1/base.js` of ``app2`` application because a base
   directory for absolute includes was set by the cross-application
   include to :file:`0.1`.
   
.. function:: use(lib)

   Include :file:`__init__.js` file from a library; it's a common
   library interface file. This function is equivalent to::

      include(lib, '__init__.js')

.. function:: set(object, name, attributes, value)

   Set the property *name* of *object* to *value*; if the property was
   not defined before it's created with *attributes*. There are four
   property attributes available:

   .. data:: COMMON
   
      Common: no special treatment.

   .. data:: READONLY
   
      Read-only: values of ``READONLY`` properties could not be
      changed.

   .. data:: HIDDEN
   
      Non-enumerable: ``HIDDEN`` properties do not appear in
      ``for..in`` loops.

   .. data:: PERMANENT
   
      Non-deletable: ``PERMANENT`` properties could not be deleted.

   Several attributes could be combined by the "bitwise or" operator
   ``|``::

      >>> var object = {}
      >>> setObjectProp(object, 'x', READONLY | HIDDEN | PERMANENT, 42)
      >>> object.x
      42
      >>> object.x = 0
      0
      >>> object.x
      42
      >>> repr(keys(object))
      []
      >>> delete object.x
      false
      >>> object.x
      42
      
.. function:: hash(value)

   Return an identity hash of an object if ``typeof(value)`` is either
   ``'object'`` or ``'function'``; return 0 otherwise. An :dfn:`object
   identity hash` is a non-zero integer; it's **not** guaranteed to be
   unique.

.. function:: construct(constructor, args)

   Instantiate *constructor* with *args*; *args* must be a list.

.. function:: isList(value)

   Check if *value* is an object with non-negative integer ``length``
   property.

.. class:: Script(source[, origin])

   A compiled JavaScript code representation. *origin* is displayed in
   exception backtraces.

   .. method:: run()

      Run the script; return the evaluation value.
