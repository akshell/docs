
==================
Miscellaneous APIs
==================

Miscellaneous low-level tools.

.. function:: readCode([appName,] path)

   Return contents of a code file as a ``string``. If *appName* is a
   name of the executing application, search for a file in the
   executing code base; otherwise search for it in the release code
   base of the specified application. If *appName* is omitted, it
   defaults to the name of the executing application. *path* separator
   is the slash (``'/'``).
   
.. function:: include([appName,] path)

   Include a code file; return a code evaluation result. Do not
   include the same file twice -- return a cached result instead. If
   *appName* is specified, a cross-application include takes place;
   otherwise code of a current application is searched for a file. A
   :dfn:`current application` is an executing application or an
   application of the latest incomplete cross-application include. A
   current application include can be relative or absolute. A
   *relative include* happens if *path* doesn't begin with a slash;
   search is performed based on the directory of the including
   file. An *absolute include* happens if *path* begins with the
   slash; search is performed based on the root directory.

   For example, if the application ``app1`` has this lines in its
   :file:`__main__.js` file::

      include('utils/useful.js')
      include('app2', 'feature/__init__.js')

   ... they will include the file :file:`utils/useful.js` of the
   ``app1`` application and the file :file:`feature/__init__.js` of
   the ``app2`` application. Suppose :file:`utils/useful.js` has the
   lines::

      include('very-useful.js')
      include('/base.js')

   ... these lines will include the files :file:`utils/very-useful.js`
   and :file:`base.js` of the ``app1`` application because the first
   include is relative and the second one was absolute. Finally, if
   the :file:`feature/__init__.js` file of the ``app2`` application
   has the lines::

      include('impl.js')
      include('/base.js')

   ... they will include the files :file:`feature/impl.js` and
   :file:`base.js` of the ``app2`` application.
   
.. function:: use(appName [,path])

   Include the :file:`__init__.js` file from a library; it's a common
   library interface file. This function is equivalent to::

      include(appName, (path ? path + '/' : '') + '__init__.js')

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

   Instantiate *constructor* with *args*; *args* must be a list.

.. function:: isList(value)

   Check if *value* is an object with non-negative integer ``length``
   property.

.. function:: getAppDescription(appName)

   Return an object describing the given application. The object has
   the following properties:

   ``name``
      The application name.

   ``admin``
      The name of the application admin.

   ``developers``
      An ``Array`` of the names of the application developers
      (``developers[0]`` is ``admin``).

   ``summary``
      The application summary.

   ``description``
      The application description.

   ``labels``
      An ``Array`` of the application labels.

.. function:: getAdminedApps(userName)

   Return names of applications admined by the given user.

.. function:: getDevelopedApps(userName)

   Return names of applications developed by the given user.

.. class:: Script(source[, resourceName, [lineOffset, [columnOffset]]])

   A ``Script`` object represents a compiled JavaScript
   code. *resourceName*, *lineOffset*, and *columnOffset* are used in
   exception backtraces.

   .. method:: run()

      Run the script; return the evaluation value.

.. data:: app

   An object describing the application being executed.

   .. data:: app.name

      The name of the application.

   .. data:: app.spot

      An object describing the current spot. In the release version of
      the application this attribute does not exist.

      .. data:: app.spot.name

         The name of the current spot.

      .. data:: app.spot.owner

         The name of the owner of the current spot.
