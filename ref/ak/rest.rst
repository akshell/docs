
==============
REST Framework
==============

The `rest.js`_ file defines tools for convenient, robust, and
:term:`RESTful <REST>` request handling.

.. _rest.js: http://www.akshell.com/apps/ak/code/rest.js


Handler
=======

Handler is the C in :term:`MVC`. For each request the :doc:`URL
mapping <url>` determines which handler to use, after that the handler
is responsible for processing the request and producing the output to
the user.

A handler receives a :class:`Request` object and positional arguments
retrieved from the request path. It should return a :class:`Response`
object.

A plain JavaScript function could be used as a handler, but the
library provides a class facilitating development of a robust handler:

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
          // PostsHandler.prototype.post won't interfere with PostHandler
        });

      var __root__ = new URLMap(
        ['users/',
         ['', UserHandler,
          ['posts/', PostsHandler,
           ['', PostHandler]
          ]
         ]
        ]);


Utilities
=========

The following functions are useful for writing handlers:

.. function:: redirect(location)

   Return a :class:`Response` object with the :data:`http.FOUND`
   status code redirecting to the *location* URL.

.. function:: getLoginURL(path)

   Return a login page URL. After the login on this URL Akshell
   redirects users to the page with the path *path* of your
   application.

.. function:: getSignUpURL(path)

   Return a sign up page URL. After the registration on this URL
   Akshell redirects users to the page with the path *path* of your
   application.

.. function:: getSessionURL(path)

   Return a session generator page URL. On this URL Akshell sets a
   session cookie and redirects users to the page with the path *path*
   of your application.
   
   
Handler Decorators
------------------

Handler decorators could be used to decorate :class:`Handler`
subclasses, RESTful methods of these subclasses, or plain JavaScript
functions used as controllers.
   
.. function:: loggingIn(handler)

   Decorate *handler* to redirect anonymous users to the Akshell login
   page. After the login Akshell redirects the user back to the page
   he came from.

.. function:: obtainingSession(handler)

   Decorate *handler* to redirect visitors who doesn't have a
   :term:`session` to the session generator URL. Akshell will set a
   session cookie and redirect the user back to the page he came
   from. If cookies are disabled in the user's browser, he'll get an
   error message asking to enable them.


Serve Functions
===============

The Akshell core initiates a request handling by the
``__main__(request)`` call. The library provides ``__main__``
implementations handling a request via the high-level framework
abstraction.

.. function:: serve(request)

   :func:`Resolve <resolve>` a handler to use; determine a handler
   method to use; return its result. It is the "naked" serve function;
   it's designed to be extended by the decorators described below.

.. function:: defaultServe(request)

   The ``defaultServe()`` function is ``serve()`` extended by all the
   decorators described below. It should suite most use cases.


Middleware
==========

:func:`serve` decorators are :dfn:`application middleware`, i.e.,
logic common for all application handlers. You could write your own
serve decorators or import them from other libraries.

When using custom serve decorators, remember two things:

* the entire application is affected;
* the order of decorators **does** matter.


.. _default_middleware:

Default Middleware
------------------

The library provides these middleware:

.. function:: serve.protectingFromICAR(func)

   Protect the application from illegal cross-application
   requests. This decorator blocks all cross-application request which
   were not marked as legal, i.e., have a false ``request.legal``
   property.

.. function:: serve.protectingFromCSRF(func)

   Protect the application from CSRF attacks. See :ref:`csrf` for
   details.

.. function:: serve.catchingHttpError(func)

   Catch a :exc:`HttpError` thrown by a handler; render the
   ``error.html`` template for the error; return a :class:`Response`
   object with the appropriate status code.

.. function:: serve.catchingTupleDoesNotExist(func)

   Catch a :exc:`TupleDoesNotExist` error; throw a
   :exc:`NotFoundError` instead (to be processed by
   ``serve.catchingHttpError``).

.. function:: serve.appendingSlash(func)

   Catch a :exc:`ResolveError`; if the request path with a slash added
   resolves successfully, redirect the user to the path with the
   slash.
   
.. function:: serve.rollbacking(func)

   :func:`Roll back <db.rollback>` the current transaction if the
   handler has thrown an exception.
