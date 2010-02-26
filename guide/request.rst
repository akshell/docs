
================
Request Handling
================

There are only two ways of communicating with your application:
evaluate an expression in it and send a request to it. The first way
is available only to the developers; the second -- to all users and to
other application. This document describes the means Akshell offer for
request handling.


Serve Function
==============

Each request handling starts from the ``__main__()`` function, which
should be defined in the ``__main__.js`` file of your
application. This function is an analog of the ``main()`` function in
C programs. ``__main__()`` receives a :class:`Request` object as an
argument; it should handle the request and return a :class:`Response`
object.

Here is the full code of the hello-world_ application::

   ak.use('ak');
   this.update(ak);

   function __main__(request) {
     return new Response('Hello, world!');
   }

It returns the same text for all requests. `Test it`_!

.. _hello-world: http://www.akshell.com/apps/hello-world/
.. _Test it: http://hello-world.akshell.com/

   
MVC Framework
=============

You could handle requests via a big ``if`` and ``switch`` mess in the
``__main__()`` function, but this approach is rather fragile for
nontrivial applications. The Akshell Model-View-Controller framework
provides a simple and robust way of request handling. The
:func:`defaultServe` function is a framework entry point; set
``__main__`` to it to enable the framework (this is already done in
the application skeleton)::

   __main__ = defaultServe;

The MVC framework splits an application into three parts:

Models
   are the representation of the data upon which the application
   operates. In Akshell the database and the files store the data;
   they are described in the :doc:`next chapter <db>`.

Views
   render models into a form suitable for interaction; for web
   applications this form is HTML code. Akshell provides the
   :doc:`template language <template>` for this purpose.

Controllers
   receive requests and produce responses by operating models and
   views. Akshell provides the :class:`Handler` base class to
   facilitate creation of controllers.

The MVC approach isolates domain logic (models) from input
(controllers) and presentation (views). This permits independent
development of each part, so the resulting application is more robust
and maintainable.

The rest of this document describes controllers in Akshell; the next
two documents describe models and views.


RESTful Design
==============

REST is an acronym for Representational State Transfer. It was
introduced by Roy Fielding in his `doctoral
dissertation`__. Fortunately, you don't have to read this document to
understand the REST principles employed in Akshell.

__ http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm

In RESTful applications requests and responses are built around the
transfer of :dfn:`representations` of :dfn:`resources`. A resource can
be any meaningful concept that may be addressed. A representation of a
resource is typically a document that captures the current state of
the resource. A limited set of :dfn:`verbs` enables clients to express
operations to be performed on resources.

In web applications resources are addressed by URLs; HTTP methods
(GET, POST, HEAD, PUT, DELETE) constitute the set of verbs.

For example, a request such as::

   DELETE /posts/42/

... should be understood by a RESTful application to refer to the post
number 42 and to indicate the desired action -- delete this post.

RESTful design is natural for web applications; it makes them more
structured and comprehensible. Akshell encourages it in the MVC
framework by providing the :class:`URLMap` class for mapping addresses
to resource controllers and the :class:`Handler` base class for
creating controllers whose operation is determined by a request verb
(an HTTP method).


URL Mapping
===========

An URL mapping tie together resources and their URLs. It serves as a
front end of request handling: an URL dispatching determines a
controller responsible for the requested resource by the value of the
``request.path`` property. The mapping is also capable of the reverse
task: determine the path of the given resource by its name and
arguments.

An URL mapping of an application is defined by the value of the
``__root__`` variable. It should be an instance of the :class:`URLMap`
class. The mapping is a tree-like structure where each node could be
either a constant part of a path (a ``string`` value) or a variable
part of it (a ``RegExp`` object or an empty string ``''`` for the
default pattern ``([^/]*)/``).


Dispatching
-----------

After a request has arrived and has been processed by the middleware
(see :ref:`below <middleware>`), an URL dispatching takes place. It
starts from the root of the mapping tree and applies the patterns of
the tree nodes one after another until the path matches one of them.

Example::

   __root__ = new URLMap(
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
   
It maps path patterns to controllers:

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
         width = .9,
         fixedsize = true
      ];

      "/" -> "users/" -> "([^/]*)/" -> "profile/";
      "([^/]*)/" -> "posts/" -> "add/";
      "posts/" -> "(\\d+)/";
   }
   
The controller found by the URL dispatching is responsible for the
rest of the request handling. If the dispatching has failed, a 404
"Not found" response is returned.


Reversing
---------

Note that each pattern in the example has a string name (``'index'``,
``'profile'``, etc.); it's optional and intended for reconstructing a
path of a particular resource. This task is reverse to dispatching; it
could arise, for example, when you need to redirect the user to this
resource.

A resource path could be reconstructed in-place, but this approach is
ugly because it violates the :term:`DRY` principle: an URL mapping
should be defined only once in one place to be maintainable, not
scattered about the whole application. Akshell provides a clear
solution to the problem -- the :func:`reverse` function. It accepts a
name of a map pattern and positional arguments for variable parts of
the pattern.

In the previous example reversing will work as follows::

   >>> reverse('index')
   /
   >>> reverse('profile', 'Anton')
   /users/Anton/profile/
   >>> reverse('post', 'Anton', 42)
   /users/Anton/posts/42/


Handlers
========

Handlers are responsible for retrieving data from a database and a
file storage, making the necessary changes to them, forming a response
content (usually via :doc:`templates <template>`), and returning a
response.

A handler receives a request object and positional arguments obtained
by an URL dispatching. A plain JavaScript function could serve as a
handler, but the :class:`Handler` base class offers a more
:term:`RESTful <REST>` solution. To create a resource controller,
subclass this class and define ``get()``, ``post()``, ``head()``,
``put()``, or ``del()`` methods performing the required actions. The
subclass constructor should perform initialization common for all
methods.

For example::

   var PostsHandler = Handler.subclass(
     function (request, userName) {
       this._user = ... // Retrieve the user info
     },
     {
       get:  function (request, userName) { /* Return a post list */ },
       post: function (request, userName) { /* Create a new post  */ }
     });

Note that the constructor and the methods receive the same
arguments. This redundancy is deliberate: it's convenient to have
request arguments at hand.

The zest of handler classes is that you can subclass them. In a
subclass you could use properties set by the parent class constructor
and methods of its prototype.

For example, a handler of single post could be written as follows::

   var PostHandler = PostsHandler.subclass(
     function (request, userName, postId) {
       PostsHandler.call(this, request, userName);
       this._post = ... // Retrieve the post info
     },
     {
       get: function () { /* Return a post representation */ }
     });

Note that you can use the ``post()`` method of the parent class in
``PostHandler``, but it won't be called if ``PostHandler`` receives a
POST request -- that's exactly what you need.


.. _middleware:

Middleware
==========

Some logic is common for the whole application, so it would be
inconvenient to add it to every handler class. Akshell offers a
:term:`DRY` way of implementing such logic -- :dfn:`application
middleware`. A piece of middleware is a :term:`decorator` of the
:func:`serve` function; it adds the desired behavior to the very entry
point of request handling. The above-mentioned :func:`defaultServe`
function is simply ``serve()`` with the :ref:`default middleware
<default_middleware>` applied.

For example, the :func:`serve.catchingHttpError` middleware catches
:class:`HttpError` exceptions thrown by your handlers and returns
error responses rendered from the ``error.html`` template. This way of
error handling is extremely handy because you don't have to worry
about returning an appropriate error response whenever you encounter
an error situation -- you just throw an ``HttpError`` and forget about
it.

You could borrow middleware from third-party libraries or even write
you own -- it's easy. For example, this middleware adds a custom HTTP
header to all responses::

   function addingUselessHeader(func) {
     return function (request) {
       var response = func(request);
       response.headers['X-Useless'] = '42';
       return response;
     };
   }

To enable it in your application decorate the ``defaultServe()``
function::

   __main__ = defaultServe.decorated(addingUselessHeader);
