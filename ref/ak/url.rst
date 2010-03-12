
===========
URL Mapping
===========

The `url.js`_ file supplies the main tool for handling URLs in your
application, the ``URLMap`` class, along with the corresponding
functions and error classes.

.. _url.js: http://www.akshell.com/apps/ak/code/url.js


URLMap
======

.. class:: URLMap([handler, [name,]] children...)

   An ``URLMap`` object defines a mapping from URL patterns to
   handlers responsible these URLs. An URL mapping of an application
   is a tree-like structure with nodes represented by ``URLMap``
   objects and the root node stored in the ``__root__`` global
   variable.

   The *handler* argument is a function associated with the root of
   this map; *name* is a ``string`` name of the map; *children* are
   definitions of the maps included by this map. Each child definition
   consist of a ``string`` or ``RegExp`` pattern and an ``URLMap``
   object, which could be defined by the arguments of the ``URLMap``
   constructor (this way is both concise and expressive).

   ``URLMap`` considers ``string`` patterns as constant parts of URLs
   and ``RegExp`` patterns as templates for variable parts of
   URLs. For example, the ``'search/'`` patterns defines one page; the
   ``/day\/(\d\d)\//`` pattern defines pages with addresses
   ``day/01/``, ``day/02/``, ... , ``day/99/``. The value of the match
   group (``'01'``, ``'02'``, ... , ``'99'`` in the example) is passed
   to the handler as an argument.

   The empty pattern ``''`` designates a common variable pattern
   ``/([^\/]+)\//``. It's the most straightforward way of defining
   variable parts of URLs.

   .. _urlmap_example:

   Example::

      var __root__ = new URLMap(
        IndexHandler, 'index'
        ['users/',
         ['',
          ['profile/', ProfileHandler, 'profile'],
          ['posts/', PostsHandler, 'posts',
           ['add/', AddPostHandler, 'add-post'],
           [/(\d+)\//, PostHandler, 'post']
          ]
         ]
        ]);

   It maps:

   * ``/`` to ``IndexHandler``;
   * :samp:`/users/{userName}/profile/` to ``ProfileHandler``;
   * :samp:`/users/{userName}/posts/` to ``PostsHandler``;
   * :samp:`/users/{userName}/posts/add/` to ``AddPostHandler``;
   * :samp:`/users/{userName}/posts/{postId}/` to ``PostHandler``.

   The pattern tree is the following:

   .. graphviz::

      digraph {
         rankdir = LR;
         node [
            shape = box,
            fontname = monospace,
            fontsize = 10,
            height = .25,
            width = .8,
            fixedsize = true
         ];

         "/" -> "users/" -> "([^/]*)/" -> "profile/";
         "([^/]*)/" -> "posts/" -> "add/";
         "posts/" -> "(\\d+)/";
      }

   It is common for complex applications to be separated into modules,
   each module being responsible for a particular functionality. In
   such cases modules could define their own URL mappings which are
   incorporated into the ``__root__`` mapping in the ``__main__.js``
   file.

   In the example, post handling could be moved to the ``post.js``
   module::

      var postMap = new URLMap(
        PostsHandler, 'posts',
        ['add/', AddPostHandler, 'add-post'],
        [/(\d+)\//, PostHandler, 'post']);

   In the ``__main__.js`` file::

      include('post.js');
      ...
      var __root__ = new URLMap(
        IndexHandler, 'index'
        ['users/',
         ['',
          ['profile/', ProfileHandler, 'profile'],
          ['posts/', postMap]
         ]
        ]);
   

Functions
=========

.. function:: resolve(path)

   Resolve the absolute path *path* against the application URL
   mapping; return the ``[handler, args]`` pair where ``args`` is an
   array of positional arguments retrieved from the ``RegExp`` pattern
   match groups. Throw a :exc:`ResolveError` on failure.

   Example usage (for the :ref:`example mapping <urlmap_example>`)::

      >>> repr(resolve('/'))
      [<function IndexHandler>, []]
      >>> repr(resolve('/users/Anton/posts/42/'))
      [<function PostHandler>, ["Anton", "42"]]
      >>> resolve('/invalid/path/')
      ak.ResolveError: ...
      >>> resolve('relative/path/')
      ak.ValueError: ak.resolve() requires absolute path 

.. function:: reverse(name, args...)

   Return a path which would resolve to the URL map node with the name
   *name* and the positional arguments *args*. Throw a
   :exc:`ReverseError` if a node with this name does not exist or has
   a different number of arguments.

   Four names are treated specially: ``login``, ``logout``,
   ``signup``, and ``session``. For these names ``reverse()`` returns
   an absolute URL of a corresponding Akshell page. If the user visits
   the page and performs the necessary action, he will be redirected
   back to your application. The path of the redirection could be
   specified by the second ``reverse()`` argument (to redirect the
   user back to the current page, use ``request.fullPath``).

   Example usage (for the :ref:`example mapping <urlmap_example>`)::

      >>> reverse('index')
      /
      >>> reverse('post', 'Anton', 42)
      /users/Anton/posts/42/
      >>> reverse('no-such-name')
      ak.ReverseError: ...
      >>> reverse('post', 'too', 'many', 'arguments')
      ak.ReverseError: ...
      >>> reverse('login')
      http://www.akshell.com/login/?domain=example&path=%2F
      >>> reverse('session', '/some/path/')
      http://www.akshell.com/session/?domain=example&path=%2Fsome%2Fpath%2F
      
   
Error Classes
=============

.. exception:: ResolveError

   Failed to find a handler of a resource with the given
   path. Subclass of :exc:`NotFoundError`. Thrown by the
   :func:`resolve` function.

.. exception:: ReverseError

   Failed to reconstruct a request path. Subclass of
   :exc:`BaseError`. Thrown by the :func:`reverse` function.
