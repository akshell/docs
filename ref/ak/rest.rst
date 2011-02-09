==============
REST Framework
==============

The rest_ module defines tools for convenient, robust, and
:term:`RESTful <REST>` request handling.

.. _rest: https://github.com/akshell/ak/blob/0.3/rest.js


``Request``
===========

.. class:: Request

   A ``Request`` object represents an HTTP request sent to your app.

   .. attribute:: method

      The request HTTP method in uppercase.

   .. attribute:: path

      The request path.

   .. attribute:: fullPath

      The request path with GET parameters.

   .. attribute:: uri

      The request URI.

   .. attribute:: get

      An object mapping GET parameter names to their values.

   .. attribute:: post

      An object mapping POST parameter names to their values.

   .. attribute:: headers

      An object mapping HTTP header names to their values.

   .. attribute:: cookies

      An object mapping cookie names to their values.


``Response``
============

.. class:: Response(content='', status=http.OK[, headers])

   A ``Response`` object represents an app's response. *content* is a
   :class:`Binary` or a ``string`` representing the response content;
   *status* is an HTTP status code (the ``ak`` library defines
   :ref:`constants for status codes <status_codes>`); *headers* are
   HTTP headers, which default to::

      {'Content-Type': 'text/html; charset=utf-8'}

   the ``main()`` function exported by the ``main`` module should
   return a Response object.

   .. method:: setCookie(name, value='', options={})

      Set the cookie *name* to *value*. The following *options* are
      available:

      ``domain``
         The cookie domain; defaults to the current host.

      ``path``
         The cookie path; defaults to ``/``.

      ``expires``
         The cookie expiry date; if unset, the cookie only lasts for
         the duration of users using the app.

      ``httpOnly``
         If set, the cookie value won't be available to a client side
         script.


``Handler``
===========

A :dfn:`handler` is a controller of a particular resource. For each
request the :doc:`URL mapping <url>` determines which handler to use,
after that the handler is responsible for processing the request and
producing the output to a user.

A handler receives a :class:`Request` object and positional arguments
retrieved from the request path. It should return a :class:`Response`
object.

A plain JavaScript function can be used as a handler, but the library
provides a class facilitating development of a robust handler:

.. class:: Handler(request, args...)

   To create your own handler class you should subclass ``Handler``
   and define the ``get()``, ``post()``, ``head()``, ``put()``, or
   ``del()`` methods responsible for handling of corresponding HTTP
   requests. You could also define the ``perform()`` method which will
   be used for requests not handled by the previous methods.  Each
   method receives a :class:`Request` object and positional arguments.
   It should return a :class:`Response` object.

   For each request your subclass is instantiated and then the
   appropriate method is called. In the constructor your could perform
   initialization common for all methods.

   You could subclass your ``Handler`` subclass to define a
   "subhandler", i.e., a handler responsible for some part of the
   resource of the parent handler. The handling methods of the parent
   class will not be used when the child class handles a request.

   For example, a blog application could employ the following handler
   classes::

      var UserHandler = Handler.subclass(
        function (request, userName) {
          this._user = ... // Retrieve the user info
        },
        {
          get: function () { /* Return user info */ }
        });

      var PostsHandler = UserHandler.subclass(
        {
          // this._user is available in the methods
          get:  function () { /* Return a post list */ },
          post: function () { /* Create a new post  */ }
        });

      var PostHandler = PostsHandler.subclass(
        function (request, userName, postName) {
          UserHandler.call(this, request, userName);
          this._post = ...
        },
        {
          get: function () { /* Return a post representation */ }
        });

      exports.root = new URLMap(
        ['users/',
         ['', UserHandler,
          ['posts/', PostsHandler,
           ['', PostHandler]
          ]
         ]
        ]);


Shortcut Functions
==================

.. function:: redirect(location)

   Return a :class:`Response` object with the :data:`http.FOUND`
   status code redirecting to the *location* URL.

.. function:: render(name, context={}, status=http.OK[, headers])

   Load a template via the :func:`getTemplate` function, render it via
   the :meth:`~Template.render` :class:`Template` method, and return a
   :class:`Response` object containing the rendered template.


Serve Functions
===============

The Akshell core initiates a request handling by calling
``require.main.exports.main(request)`` (the ``main()`` function
exported by ``main.js``). The library provides ``main()``
implementations handling a request via the high-level framework
abstraction.

.. function:: serve(request)

   :func:`Resolve <resolve>` a handler to use; determine a handler
   method to use; return its result. It is the "naked" serve function;
   it's designed to be extended by the decorators described below.

.. function:: defaultServe(request)

   The ``defaultServe()`` function is :func:`serve()` extended by all
   the decorators described below. ``require.main.exports.main()``
   defaults to this function.


Middleware
==========

:func:`serve` decorators are :dfn:`application middleware`, i.e.,
logic common for all application handlers. You can write your own
serve decorators or import them from other libraries.

When using custom serve decorators, remember two things:

* the entire application is affected;
* the order of decorators **does** matter.


.. _default_middleware:

Default Middleware
------------------

The library provides these middleware:

.. function:: serve.protectingFromCSRF(func)

   Protect the application from :term:`CSRF` attacks.

.. function:: serve.catchingFailure(func)

   Catch a :exc:`Failure` thrown by a handler; render the
   ``error.html`` template for the error; return a :class:`Response`
   object with the appropriate status code.

.. function:: serve.servingStaticFiles(func)

   Serve static files from the ``/static/`` path.

.. function:: serve.catchingTupleDoesNotExist(func)

   Catch a :exc:`TupleDoesNotExist` error; throw a :exc:`NotFound`
   error instead (to be processed by ``serve.catchingFailure``).

.. function:: serve.appendingSlash(func)

   Catch a :exc:`ResolveError`; if the request path with a slash added
   resolves successfully, redirect a user to the path with the
   slash.

.. function:: serve.rollbacking(func)

   :func:`Roll back <rollback>` the current transaction if the
   handler has thrown an exception.
