
===============
Getting Started
===============

New to Akshell? Or to web development in general? You came to the
right place: read this material to quickly get up and running.

If you’re new to JavaScript, you might want to start by getting an
idea of what the language is like. I recommend the `introduction from
Mozilla`__ for this purpose. To learn the basics of HTML, you could
look through the `five-minute tutorial from W3C`_.

__ https://developer.mozilla.org/en/A_re-introduction_to_JavaScript
.. _five-minute tutorial from W3C: http://www.w3.org/MarkUp/Guide/


Overview
========

Akshell is a web application network. Just like a social network is a
place for people to communicate smoothly, Akshell is a place for
programs to work effectively via interaction.

Akshell is a tool for developing and an environment for hosting web
applications at the same time. It's designed to facilitate the whole
process of web development from creating a prototype to supporting a
production version.

Creation of a modern web application is complicated: besides its main
task, it has to perform a lot of routine tasks common for all web
applications. By contrast, divide-and-conquer is the main philosophy
of Akshell: each program should do one thing well; auxiliary task
should be handled by other programs; interaction between programs is
Akshell's business. This approach has been employed in UNIX-like
operating systems for forty years -- why not to use it for web
applications?

For example, to provide your application with a robust logging system,
you could use the log_ application. Only one line is necessary for
it::

   use('log');

Your logs will be available at http://log.akshell.com/.

.. _log: /apps/log/

Besides the unique environment, Akshell provides applications with
traditional facilities: a relational database and a file storage. Here
convenience of a developer is the main point too: a special query
language was created for database access. It was designed to be
naturally embedded into JavaScript combining the power of SQL and the
easiness of object-relational mappings.

And has I mentioned that a web browser is everything you need for
development in Akshell? If you're reading this text, you already have
one; so you can start your first application right now! For those who
prefer his favorite editor or IDE to the browser, there is a
:ref:`tool <tool>` for access to Akshell.

Currently Akshell is in beta state; so only creation of free
non-commercial applications is available.  Akshell makes their code
publicly accessible; it *must* be provided under the terms of the `BSD
License`_. Development and hosting of such applications are
*free*. Support of commercial applications is a work in progress.

.. _BSD License: /about/bsd/


Example
=======

Let me explain the basics of Akshell by an example. I'll guide you
through the creation of a simple yet fully functional blog
application.

I named it ``simple-blog``. Immediately after the creation each
application receives an address on the ``akshell.com`` domain; so you
can test the example at http://simple-blog.akshell.com/.  You could
also `browse its code`__ as well as code of other free applications.

__ /apps/simple-blog/code/


Skeleton Code
-------------

Akshell generates a skeleton code for every new application. Its
layout is the recommended one:

``__main__.js``
   The main code file of the application.

``README``
   Contains a brief introduction to Akshell; should be filled by the
   application description.

``static/``
   The directory containing static files (stylesheets, icons, etc.).

``static/main.css``
   The base stylesheet of the application.

``templates/``
   The directory containing HTML templates.

``templates/base.html``
   The base HTML template of the application. It should define
   blocks to be filled out by child templates.

``templates/index.html``
   The template of the index page.

``templates/error.html``
   The template of the error page.


Spots
-----

Each application can have a few developers working on it. To prevent
developers from hindering each other and to isolate serving
application users from development, Akshell provides :dfn:`spots` --
independent application versions owned by developers. When the next
stable version is ready, the spot containing it could be "released":
it instantly becomes available to users (this operation is atomic).

By default, every developer has a *debug* spot. I employed it for
writing code of ``simple-blog``.

   
Database
--------

A blog consists of posts and comments to these posts. To store them, I
created two :term:`relation variables <relation variable>`: ``Post``
and ``Comment``. You may imagine them as two tables: each row of
``Post`` represents a single post, each row of ``Comment`` represents
a single comment.

I put the function creating ``Post`` and ``Comment`` into the
``__main__.js`` file::

   function init() {
     db.create('Post',
               {
                 id: 'unique serial',
                 author: 'string',
                 date: 'date',
                 title: 'string',
                 text: 'string'
               });
     db.create('Comment',
               {
                 post: 'integer -> Post.id',
                 author: 'string',
                 date: 'date',
                 text: 'string'
               });
   }
   
The :func:`db.create` function accepts a name of a relation variable
to be created and an object mapping its attributes (columns) to their
types.

The ``id`` attribute of ``Post`` is :ref:`serial <serial>` and
:ref:`unique <unique>`, i.e., its values come from a sequence 0, 1, 2,
etc., and two posts cannot have the same ``id``. The ``post``
attribute of ``Comment`` is a :ref:`foreign key <foreign_key>` to the
``id`` attribute of ``Post``; it represents a many-to-one
relationship: every comment has a post it was added to.

OK, then I needed to call the ``init()`` function. I went to the
:ref:`evaluate` tab, typed ``init()``, and pressed ``Enter``. The
function returned ``undefined`` -- ``Post`` and ``Comment`` were
created.


Libraries
---------

Akshell is not a web framework; so it offers rather low-level means of
web development. However, applications can :func:`use <use>` other
applications as libraries. This feature really frees your creativity:
use libraries, create new ones, set up your own environment --
everything is open!

The :doc:`basic Akshell library <ref/ak/index>`, called ak_, provides
general JavaScript goodies and a :term:`Model-View-Controller <MVC>`
framework facilitating web development. This library is enabled in the
application skeleton; the code of the rest of this document actively
uses it.

.. _ak: /apps/ak/


Index Handler
-------------

Every web application worth its salt should have an index page. The
index page of ``simple-blog`` displays a list of blog authors.

Creating a new "kind" of page for your application via the MVC
framework is a two-step process:

* first, you create a handler -- JavaScript code responsible for
  performing a request and returning a response;

* then, you create a template -- a document to be transformed into
  HTML code representing a page.

A handler is usually a subclass of the :class:`Handler` class. Yes,
subclass -- the ``ak`` library provides a lightweight yet powerful
implementation of class hierarchies for JavaScript through the
:meth:`~Function.subclass` method of ``Function``.

Here is the handler of the ``simple-blog`` index page::

   var IndexHandler = Handler.subclass(
     {
       get: function (request) {
         return render(
           'index.html',
           {
             request: request,
             authors: rv.Post.all().get({attr: 'author', by: 'author'})
           });
       }
     }).decorated(obtainingSession);

The ``get()`` method is defined to handle GET HTTP requests; it should
return an instance of the :class:`Response` class.

The :func:`render` function renders the ``index.html`` template into
HTML code and returns a response containing this code. The object
passed to ``render()`` is used as a :dfn:`context` for the template
rendering (see below).

The ``Post`` relation variable is accessed through the :data:`rv`
object. The ``authors`` context property is set to the sorted array of
all post authors.

Akshell provides applications with a centralized authentication
system. A user has to create only one account to use all Akshell
applications. Without this feature productive application interaction
would be impossible. Users authenticate themselves to applications via
a :dfn:`session cookie`. The :func:`obtainingSession`
:term:`decorator` instructs the handler to redirect the users who
don't have a session cookie to the special Akshell page which sets
this cookie and redirects back. Use this decorator if your application
needs to know an identity of a user.


URL Mapping
-----------

Besides the index page, ``simple-blog`` should be able to show a list
of posts of a particular author and a particular post with its
comments. To support these two "kinds" of pages I needed to create two
handlers: ``BlogHandler`` and ``PostHandler``.

Before rushing to creating new handlers, it's usually reasonable to
think which URLs will their pages have. Clean URL scheme is vital for
high-quality web application: it improves usability and promotes
robust design. See `this article`__, by World Wide Web creator Tim
Berners-Lee, for excellent arguments for this.

__ http://www.w3.org/Provider/Style/URI

``simple-blog`` has a very natural URL scheme:

* ``/`` displays the index page;
* :samp:`/{authorName}/` displays a list of posts by a particular
  author;
* :samp:`/{authorName}/{postId}/` displays a particular post with its
  comments.

The Akshell MVC framework offers the :class:`URLMap` class for mapping
URLs to handlers. A mapping is a tree-like structure where each node
is a pattern of a path part; this approach encourages clean and robust
design. See :ref:`url_mapping` for details.


``simple-blog`` has this URL mapping::

	var __root__ = new URLMap(
	  IndexHandler, 'index',
	  ['', BlogHandler, 'blog',
	   [/(\d+)\//, PostHandler, 'post']]);

The ``''`` pattern designates the default pattern
``([^/]+)/``. Matches of parenthesized substrings are passed to a
corresponding handler.

Each pattern has a name (``'index'``, ``'blog'``, and ``'post'``);
it's used for :dfn:`path reversing`, i.e., determining a path of a
particular page.


Base Template
-------------

Templates are text documents intended for rendering into HTML
code. The :doc:`template language <guide/template>` is borrowed from
the `Django web framework`__; it encourages clear separation of
presentation and program logic.

__ http://www.djangoproject.com/

:ref:`Template inheritance <template_inheritance>` is the most
powerful feature of the language. It allows you to build a base
"skeleton" template that contains all the common elements of your site
and defines :dfn:`blocks` that child templates can override.

The ``simple-blog`` application employs the following base template
located in the ``base.html`` file:

.. code-block:: html+django

   <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
             "http://www.w3.org/TR/html4/strict.dtd">
   <html>
     <head>
       <link rel="stylesheet" type="text/css" href="{% code 'static/base.css' %}">
       <title>{% block title %}{% endblock %}</title>
     </head>
     <body>
       <a href="{% url 'index' %}">Blogs</a>
       {% if request.user %}
         <a href="{% url 'blog' request.user %}">Your Blog</a>
       {% else %}
         <a href="{% url 'login' request.fullPath %}">Login</a>
       {% endif %}
       {% block content %}
       {% endblock %}
     </body>
   </html>

It's a common HTML document with a number of :ref:`tags <tags>`, which
perform various actions during template rendering. Every tag is
surrounded by ``{%`` and ``%}``.

The ``{% code %}`` tag outputs an absolute link to the ``base.css``
file.

The ``{% if %}`` tag displays one content for registered users, the
other for anonymous ones (the ``request.user`` property is a
``string``; it's empty for anonymous users). A value of ``request`` is
retrieved from a context object passed to the :func:`render` function.

The ``{% url %}`` tags output links to the index page, the page of the
visiting user's blog, and the login page. The ``'index'`` and
``'blog'`` names were defined by the URL mapping; the ``'login'`` name
is a predefined one (it corresponds to the Akshell login page).


Index Template
--------------

Here is ``index.html``, the template of the index page:

.. code-block:: html+django

   {% extends 'base.html' %}
   
   {% block title %}Simple Blog{% endblock %}
   
   {% block content %}
     <h1>Blogs</h1>
     <ul>
       {% for author in authors %}
         <li><a href="{% url 'blog' author %}">{{ author }}</a></li>
       {% endfor %}
     </ul>
   {% endblock %}

It extends ``base.html`` and defines the ``title`` and ``content``
blocks. The ``{% for %}`` tag iterates over the ``authors`` array
creating an unordered list of links to the blogs of the given authors.

The :ref:`template expression <variables>` ``{{ author }}`` outputs a
value of the ``author`` variable. Every template expression is
surrounded by ``{{`` and ``}}``.


Blog Handler
------------

``BlogHandler`` is more complex than ``IndexHandler``; it handles both
GET and POST requests. The ``get()`` method renders a page with posts;
the ``post()`` method inserts a new post into the database and
redirects to the page of this post. Both methods receive the
``author`` argument from the URL mapping. ::

   var BlogHandler = IndexHandler.subclass(
     {
       get: function (request, author) {
         var posts = rv.Post.where({author: author}).get({by: '-date'});
         if (!posts.length && request.user != author)
           throw NotFound(author + ' doesn\'t have a blog');
         return render('blog.html',
                       {
                         request: request,
                         author: author,
                         posts: posts
                       });
       },
   
       post: function (request, author) {
         if (request.user != author)
           throw HttpError('Forbidden', http.FORBIDDEN);
         if (!request.post.title)
           throw HttpError('Empty title');
         var post = rv.Post.insert(
           {
             author: author,
             date: new Date(),
             title: request.post.title,
             text: request.post.text
           });
         return redirect(reverse('post', author, post.id));
       }
     });

Note the exceptions the methods throw. Akshell converts them into the
appropriate HTTP responses: ``NotFound`` results in a 404 response,
``HttpError`` by default results in a 400 response.

Every ``post()`` method should always redirect after a successful
request processing. This tip isn't specific to Akshell -- it's a
common web development practice.
     

Blog Template
-------------

The ``blog.html`` template is also more complex than ``index.html``:

.. code-block:: html+django

   {% extends 'base.html' %}
   
   {% block title %}{{ author }}'s Blog{% endblock %}
   
   {% block content %}
     <h1>{{ author }}'s Blog</h1>
     {% if request.user == author %}
       <h2>Add Post</h2>
       <form action="." method="post">
         {% csrfToken %}
         <p>Title:</p>
         <p><input type="text" name="title"></p>
         <p>Text:</p>
         <p><textarea name="text" cols="80" rows="10"></textarea></p>
         <input type="submit" value="Add">
       </form>
       {% if posts.length %}<h2>Posts</h2>{% endif %}
     {% endif %}
     <ul>
       {% for post in posts %}
         <li>
           <a href="{% url 'post' author post.id %}">{{ post.title }}</a>
         </li>
       {% endfor %}
     </ul>
   {% endblock %}

The main point of interest here is the ``{% csrfToken %}`` tag. It
protects applications from so-called :term:`cross-site request forgery
<CSRF>` attacks. You should place it in every post form; the display
of the form is unaffected.


Post Handler
------------

The last handler of ``simple-blog``, ``PostHandler``, employs
constructor to initialize the ``_post`` property used in ``get()`` and
``post()`` methods. Both methods are rather straightforward: ``get()``
returns an HTML page; ``post()`` inserts a new comment into the
database and redirects.

::

   var PostHandler = BlogHandler.subclass(
     function (request, author, postId) {
       this._post = rv.Post.where({id: postId}).get()[0];
       if (!this._post || this._post.author != author)
         throw NotFound('No such post');
     },
     {
       get: function (request, author, postId) {
         return render(
           'post.html',
           {
             request: request,
             post: this._post,
             comments: rv.Comment.where({post: this._post.id}).get({by: 'date'})
           });
       },
   
       post: function (request, author, postId) {
         if (!request.user)
           throw HttpError('Login first', http.FORBIDDEN);
         if (!request.post.text)
           throw HttpError('Empty comment text');
         rv.Comment.insert(
           {
             post: this._post.id,
             author: request.user,
             date: new Date(),
             text: request.post.text
           });
         return redirect('.');
       }
     });
     

Post Template
-------------

Here is the ``post.html`` template:

.. code-block:: html+django

	{% extends 'base.html' %}
 
	{% block title %}{{ post.title }}{% endblock %}
	 
	{% block content %}
	  <a href="{% url 'blog' post.author %}">{{ post.author }}'s Blog</a>
	  <h1>{{ post.title }}</h1>
	  {{ post.text|paragraph }}
	  <em>Posted {{ post.date|timeSince }} ago by {{ post.author }}</em>
	  {% if comments.length %}
	    <h2>Comments</h2>
	    {% for comment in comments %}
	      {{ comment.text|paragraph }}
	      <em>Added {{ comment.date|timeSince }} ago by {{ comment.author }}</em>
	      <br><br>
	    {% endfor %}
	  {% endif %}
	  {% if request.user %}
	    <h2>Add Comment</h2>
	    <form action="." method="post">
	      {% csrfToken %}
	      <p><textarea name="text" cols="80" rows="10"></textarea></p>
	      <input type="submit" value="Add">
	    </form>
	  {% endif %}
	{% endblock %}

The template actively uses :ref:`filters <filters>`, namely
``paragraph`` and ``timeSince``. Filters transform values of variables
before outputting them. The ``paragraph`` filter converts newlines in
plain text into the ``<p>`` and ``<br>`` HTML tags; the ``timeSince``
filter presents a date as a time passed since that date. Filters can
be chained; they can take arguments.


Entry Point
-----------

The ``simple-blog`` application was created via the means of the MVC
framework. This line tells Akshell to use the framework for request
handling::

   var __main__ = defaultServe;

It's the last line of the application skeleton. The ``__main__``
function is the entry point of request handling; so this assignment
delegates request handling to the framework.

After writing the code, I tested it in the ``debug`` spot, released
this spot, and called the ``init()`` function in the
release. ``simple-blog`` immediately became publicly available at
http://simple-blog.akshell.com/.


What's Next?
============

So you've read the introduction to Akshell. I've only just scratched
the surface with it (it's less than 10% of overall documentation), but
at this point you should know enough to create an application and
start fooling around. As you need to learn new tricks, come back to
the documentation. To form a deeper understanding of Akshell, read the
:doc:`guide/index`; to look up a description of a particular function
or class, consult the :doc:`ref/index`.

Enjoy.
