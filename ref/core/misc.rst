
==================
Miscellaneous APIs
==================

Miscellaneous low-level tools.


require()
=========

.. function:: require(id)
              require(lib, version, id='index')

   Return an exported API of a module.

   The first form exports a module of the current application. *id* is
   a module identifier; The ``'.js'`` extension is appended to it to
   form a file path. *id* is relative if it starts with ``'./'`` or
   ``'../'``; otherwise *id* is top-level.

   The second form exports a module of the application *lib* from the
   subdirectory *version*.

   A foreign module may not have finished executing at the time it is
   required by one of its transitive dependencies; in this case, the
   object returned by ``require()`` contains the exports that the
   foreign module has prepared before the call to ``require()`` that
   led to the current module's execution.

   ``require()`` has the ``main`` property which is a reference to the
   ``module`` object of the ``main`` module of the application being
   executed.

   This function conforms to the `CommonJS Modules 1.1.1 proposal`__.

__ http://wiki.commonjs.org/wiki/Modules/1.1.1

.. data:: exports

   A module-scope object that the module may add its API to.
   
.. data:: module

   A module-scope object describing the current module. Has the
   following properties.

   ``exports``
      The module :data:`exports` object.

   ``id``
      The module identifier.

   ``version``
      The package version, i.e., the base directory of top-level
      identifiers.

   ``app``
      The module application.

   ``owner``
      The spot owner. Does not exist in release versions.

   ``spot``
      The spot name. Does not exist in release versions.


global
======
      
.. data:: global

   The global object reference; an analog of the ``window`` property
   in client-side JavaScript.

.. class:: Global

   The global object class.


Stuff
=====
        
.. function:: readCode([appName,] path)

   Return contents of a code file as a ``string``. If *appName* is a
   name of the executing application, search for a file in the
   executing code base; otherwise search for it in the release code
   base of the specified application. If *appName* is omitted, it
   defaults to the name of the executing application. *path* separator
   is the slash (``'/'``).
   
.. function:: getCodeModDate([appName,] path)

   Return a modification ``Date`` of a code entry. 
   
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
   
.. class:: Script(source[, resourceName, [lineOffset, [columnOffset]]])

   A ``Script`` object represents a compiled JavaScript
   code. *resourceName*, *lineOffset*, and *columnOffset* are used in
   exception backtraces.

   .. method:: run()

      Run the script; return the evaluation value.


Exceptions
==========
        
.. exception:: ValueError

   Inappropriate argument value (of correct type).

.. exception:: UsageError

   Function was used in a wrong way.

.. exception:: NotImplementedError

   Function hasn't been implemented yet.


Binary
======
      
.. class:: Binary()

   A ``Binary`` object represents raw binary data. ``Binary`` is a
   mutable fixed-length numeric byte storage type. It can be
   instantiated in a few ways:

   ``new Binary(length, byte=0)``
      Create a new ``Binary`` with the given *length* and fill it with
      the given *byte*.

   ``new Binary(string, charset='utf-8')``
      Convert the *string* to ``Binary`` using the given *charset*.

   ``new Binary(array)``
      Initialize ``Binary`` bytes from the *array* values.

   ``new Binary(binary, toCharset, fromCharset='utf-8')``
      Transcode *binary* from *fromCharset* to *toCharset*.
   
   ``new Binary(binary[, binary1...])``
      Create new ``Binary`` concatenating the given binaries.

   The index operator ``[]`` can be used to get and set byte values.

   .. attribute:: length

      The length of the byte sequence. Cannot be changed.
   
   .. method:: toString(charset='utf-8')

      Convert to ``string`` using the given *charset*.

   .. method:: range(start=0, stop=length)

      Return a new ``Binary`` that views the given range of this
      ``Binary``.

   .. method:: fill(byte=0)

      Fill the ``Bynary`` by the given *byte*.

   .. method:: indexOf(value, start=0)

      Return the index of the first occurence of *value*, starting
      search at *start*; return ``-1`` if *value* is not
      found. *value* can be ``Binary`` or ``string``.
   
   .. method:: lastIndexOf(value, start=length)

      Return the index of the last occurence of *value*, starting
      search at *start*; return ``-1`` if *value* is not
      found. *value* can be ``Binary`` or ``string``.

   .. method:: md5()

      Calculate the MD5 hash and return it as a ``string`` hex dump.
      
   .. method:: sha1()

      Calculate the SHA1 hash and return it as a ``string`` hex dump.

.. exception:: ConversionError

   Failed to encode, decode, or transcode data.
      

Proxy
=====
      
.. class:: Proxy(handler)

   A ``Proxy`` object intercepts property access on it. The *handler*
   object must have five attributes::

      new Proxy(
        {
          get: function (name) {
            // Return the property value or undefined if not found
          },

          set: function (name, value) {
            // Set the property value
          },

          del: function (name) {
            // Delete the property and return true;
            // if the property cannot be deleted, return false
          },

          query: function (name) {
            // Return true if the proxy has the property;
            // otherwise return false
          },

          list: function () {
            // Return an Array of all property names
          }
        })

        
Metadata
========
   
.. function:: getAppDescription(appName)

   Return an object describing the given application. The object has
   the following properties:

   ``name``
      The application name.

   ``admin``
      The name of the application admin.

   ``developers``
      A sorted ``Array`` of the names of the application developers.

   ``summary``
      The application summary.

   ``description``
      The application description.

   ``labels``
      A sorted ``Array`` of the application labels.

.. function:: getAdminedApps(userName)

   Return names of applications admined by the given user.

.. function:: getDevelopedApps(userName)

   Return names of applications developed by the given user.
