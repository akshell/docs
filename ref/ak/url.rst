
===========
URL Mapping
===========

The `url.js`_ file supplies the main tool for handling URLs in your
application, the ``URLMap`` class, along with the corresponding
functions and error classes.

.. _url.js: http://www.akshell.com/apps/ak/code/url.js


URLMap
======

.. class:: URLMap([controller, [name,]] children...)

   An ``URLMap`` object defines a mapping from URL patterns to
   controllers handling these URLs. An URL mapping of an application
   is a tree-like structure with nodes represented by ``URLMap``
   objects and the root node stored in the ``__root__`` global
   variable.

   The *controller* argument is a controller constructor associated
   with the root of this map; *name* is a ``string`` name of the map;
   *children* are definitions of the maps included by this map. Each
   child definition consist of a ``string`` or ``RegExp`` pattern and
   an ``URLMap`` object, which could be defined by the arguments of
   the ``URLMap`` constructor (this way is both concise and
   expressive).

   ``URLMap`` considers ``string`` patterns as constant parts of URLs
   and ``RegExp`` patterns as templates for variable parts of
   URLs. For example, the ``'search/'`` patterns defines one page; the
   ``/day\/(\d\d)\//`` pattern defines pages with addresses
   ``day/01/``, ``day/02/``, ... , ``day/99/``. The value of the match
   group (``'01'``, ``'02'``, ... , ``'99'`` in the example) is passed
   to the controller as an argument.

   The empty pattern ``''`` designates a common variable pattern
   ``/([^\/]*)\//``. It's the most straightforward way of defining
   variable parts of URLs.

   .. _urlmap_example:

   Example::

      __root__ = new URLMap(
        MainController, 'home'
        ['users/',
         ['', UserController, 'user',
          ['posts/', PostsController, 'posts',
           ['', PostController, 'post']
          ]
         ]
        ]);

   This maps:

   * ``/`` to ``MainController``;
   * :samp:`/users/{userName}/` to ``UserController``;
   * :samp:`/users/{userName}/posts/` to ``PostsController``;
   * :samp:`/users/{userName}/posts/{postName}/` to
     ``PostController``.

   It is common for complex applications to be separated into modules,
   each module being responsible for a particular functionality. In
   such cases modules could define their own URL mappings which are
   incorporated into the ``__root__`` mapping in the ``__main__.js``
   file.

   In the example, post handling could be moved to the ``post.js``
   module::

      var postMap = new URLMap(
        PostsController, 'posts',
        ['', PostController, 'post']);

   In the ``__main__.js`` file::

      include('post.js');
      ...
      __root__ = new URLMap(
        MainController, 'home'
        ['users/',
         ['', UserController, 'user',
          ['posts/', postMap]
         ]
        ]);
   

Functions
=========

.. function:: resolve(path)

   Resolve the absolute path *path* against the application URL
   mapping; return the ``[controller, args]`` pair where ``args`` is
   an array of positional arguments retrieved from the ``RegExp``
   pattern match groups. Throw a :exc:`ResolveError` on failure.

   Example usage (for :ref:`this<urlmap_example>` URL mapping)::

      >>> repr(resolve('/'))
      [<function MainController>, []]
      >>> repr(resolve('/users/Anton/posts/first'))
      [<function PostController>, ["Anton", "first"]]
      >>> resolve('/invalid/path/')
      ak.ResolveError: ...
      >>> resolve('relative/path/')
      ak.ValueError: ak.resolve() requires absolute path 

.. function:: reverse(name, args...)

   Return a path which would resolve to the URL map node with the name
   *name* and the positional arguments *args*. Throw a
   :exc:`ReverseError` if a node with this name does not exist or has
   a different number of arguments.

   Example usage (for :ref:`this<urlmap_example>` URL mapping)::

      >>> reverse('home')
      /
      >>> reverse('post', 'Anton', 'first')
      /users/Anton/posts/first/
      >>> reverse('no-such-name')
      ak.ReverseError: ...
      >>> reverse('post', 'too', 'many', 'arguments')
      ak.ReverseError: ...
      
   
Error Classes
=============

.. exception:: ResolveError

   Failed to find a controller of a resource with the given
   path. Subclass of :exc:`NotFoundError`. Thrown by the
   :func:`resolve` function.

.. exception:: ReverseError

   Failed to reconstruct a request path. Subclass of
   :exc:`BaseError`. Thrown by the :func:`reverse` function.
