=======
Modules
=======

App's code is distributed among modules. Each module is represented by
a file with the ``.js`` extension and identified by its path without
the extension. Akshell initializes an app by loading its ``main``
module, i.e., executing the ``main.js`` file.

Public apps share their modules with others for reuse. You can publish
any of your apps, i.e., turn it into a library. To use a library
provided by other developer, you need to give an alias to a desired
version of it in the ``manifest.json`` file. A version is just a Git
reference (a branch or tag name) of a library repository.

The default ``manifest.json`` gives the alias ``ak`` to the ``0.3``
version of the ``ak`` library provided by the ``akshell`` developer::

   {
     "libs": {
       "ak": "akshell/ak:0.3"
     }
   }

Modules load each other via the :func:`require` function, export
interfaces via the :data:`exports` object, and access their metadata
via the :data:`module` object. This API conforms to the `CommonJS
Modules 1.1.1 Proposal`__.

__ http://wiki.commonjs.org/wiki/Modules/1.1.1

.. function:: require(id or alias)
              require(alias, id)

   Return an exported API of a module.

   The two-argument form loads a module with *id* from a library with
   *alias*. By specifying ``'default'`` as an alias you can load
   built-in modules.

   The one-argument form has more tricky semantics:

   1. if the argument starts with ``'./'`` or ``'../'``, it's a module
      identifier relative to the current path;

   2. otherwise it's interpreted as a top-level module identifier;

   3. if a top-level module with this *id* doesn't exist, Akshell
      tries to load the ``index`` module from the library with
      *alias*;

   4. if this fails, the argument is interpreted as a built-in module
      identifier;

   5. if a built-in module with *id* doesn't exist, a
      :exc:`RequireError` is thrown.

   A foreign module may not have finished executing at the time it is
   required by one of its transitive dependencies; in this case, the
   object returned by ``require()`` contains the exports that the
   foreign module has prepared before the call to ``require()`` that
   led to the current module's execution.

   ``require()`` has the ``main`` property which is a reference to the
   :data:`module` object of the ``main`` module of the app being
   executed.

.. data:: exports

   A module-scope object that the module may add its API to.

.. data:: module

   A module-scope object describing the current module. Has the
   following properties:

   ``id``
      The module identifier.

   ``exports``
      The module :data:`exports` object.

   ``storage``
      A :class:`FileStorage` or a :class:`GitStorage` the module was
      loaded from.
