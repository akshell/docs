===============
Getting Started
===============

In this tutorial I'll guide you through the creation of a simple
guestbook app in Akshell. Read this material to quickly get up and
running.

If you’re new to JavaScript, you might want to start by getting an
idea of what the language is like. I recommend the `introduction from
Mozilla`__ for this purpose. To learn the basics of HTML you could
look through the `five-minute tutorial from W3C`_.

__ https://developer.mozilla.org/en/A_re-introduction_to_JavaScript
.. _five-minute tutorial from W3C: http://www.w3.org/MarkUp/Guide/

Let's get started. I create the ``guestbook`` app using the
:menuselection:`File --> New App` command.


Skeleton Code
=============

Akshell generates a skeleton code for a new app. Its layout is the
recommended one:

``main.js``
   The main code file.

``manifest.json``
   The metadata file.

``static/``
   The directory containing static files (stylesheets, icons, etc.).

``static/main.css``
   The base stylesheet.

``templates/``
   The directory containing HTML templates.

``templates/base.html``
   The base HTML template; it should define blocks to be filled out by
   child templates.

``templates/index.html``
   The template of the index page.

``templates/failure.html``
   The template of the failure page.


Database
========

To store guestbook data I create the ``Entry`` :term:`relation
variable`. You may imagine it as a table with rows representing
guestbook entries. I put the code into the ``init()`` function to be
reusable::

   init = function () {
     rv.Entry.create(
       {
         id: 'unique serial',
         author: 'string',
         title: 'string',
         message: 'string'
       });
   };

The :data:`rv` object maps relation variable names to their
:class:`RelVar` representations. The :meth:`~RelVar.create` method
accepts an object mapping attribute (column) names to their types. The
``id`` attribute is unique and serial, i.e., its values are unique and
come from a sequence 0, 1, 2, etc.

OK, then I need to call the ``init()`` function. I switch to the Eval
mode, type ``init()``, and press ``Enter``. The function returned
``undefined`` -- ``Entry`` was created.


Index Handler
=============

Every web app worth its salt should have an index page. The index page
of ``guestbook`` displays a list of entries and a form for entering
new ones.

Creating a new "kind" of page for your app via the Akshell MVC
framework is a two-step process:

* first, you create a handler -- JavaScript code responsible for
  performing a request and returning a response;

* then, you create a template -- a document to be transformed into
  HTML code representing a page.

A handler is usually a subclass of the :class:`Handler` class. Yes,
subclass -- the ``ak`` library provides a lightweight yet powerful
implementation of class hierarchies for JavaScript through the
:meth:`~Function.subclass` ``Function`` method.

Here is the handler of the ``guestbook`` index page::

   var IndexHandler = Handler.subclass(
     {
       get: function (request) {
         return render(
           'index.html', {entries: rv.Entry.all().get({by: '-id'})});
       },

       post: function (request) {
         if (!request.post.author ||
             !request.post.title ||
             !request.post.message)
           throw Failure('All fields are required');
         rv.Entry.insert(
           {
             author: request.post.author,
             title: request.post.title,
             message: request.post.message
           });
         return redirect('/');
       }
     });

The ``get()`` and ``post()`` methods handle GET and POST requests
respectively; they should return a :class:`Request` object. The
:func:`render` function renders the ``index.html`` template into HTML
code and returns a response containing this code. The object passed to
``render()`` is used as a :dfn:`context` for the template rendering
(see below). The ``entries`` context property is set to the array of
all entries sorted from recent to older. The ``post()`` method saves a
new entry and redirects a user to the index page.


URL Mapping
===========

Besides the index page, ``guestbook`` should be able to show pages of
particular entries. To support this new “kind” of page I need to
create ``EntryHandler``. But before rushing to creating new handlers,
it’s usually reasonable to think which URLs will their pages
have. Clean URL scheme is vital for high-quality web app: it improves
usability and promotes robust design.

The Akshell MVC framework offers the :class:`URLMap` class for mapping
URLs to handlers. A mapping is a tree-like structure where each node
is a pattern of a path part; this approach encourages clean and
:term:`RESTful <REST>` design. See :ref:`url_mapping` for details.

Entry pages will have their identifiers as paths; so the URL mapping
is quite simple::

   exports.root = new URLMap(
     IndexHandler, 'index',
     [/\d+/, EntryHandler, 'entry']);


Entry Handler
=============

``EntryHandler`` simply renders the ``entry.html`` template with the
given entry::

   var EntryHandler = Handler.subclass(
     {
       get: function (request, id) {
         return render(
           'entry.html', {entry: rv.Entry.where({id: id}).getOne()});
       }
     });


Index Template
==============

Templates are text documents intended for rendering into HTML
code. The :doc:`template language <guide/template>` is borrowed from
the `Django web framework`__; it encourages clear separation of
presentation and program logic.

__ http://www.djangoproject.com/

:ref:`Template inheritance <template_inheritance>` is the most
powerful feature of the language. It allows you to build a base
template that contains all the common elements of your site and
defines :dfn:`blocks` that child templates can override.

Fortunately, the skeleton code already contains the base template with
two blocks: ``title`` and ``content``. To create ``index.html`` I just
have to extend ``base.html`` and fill these blocks:

.. code-block:: html+django

   {% extends 'base.html' %}

   {% block title %}Guestbook{% endblock %}

   {% block content %}
     <h1>Guestbook</h1>
     <form method="post" action=".">
       Name:<br><input type="text" name="author"><br>
       Title:<br><input type="text" name="title"><br>
       Message:<br>
       <textarea name="message" rows="5" cols="80"></textarea><br>
       <input type="submit" value="Add">
     </form>
     <ul>
       {% for entry in entries %}
         <li><a href="{{ entry.id }}">{{ entry.title }}</a></li>
       {% endfor %}
     </ul>
   {% endblock %}

As you can see, templates consist of HTML code, :ref:`tags <tags>`,
and :ref:`expressions <variables>`.

Template tags are surrounded by ``{%`` and ``%}``; they perform
various actions during template rendering. The ``{% extends %}`` tag
defines template inheritance; the ``{% block %}`` tag defines and
fills blocks; the ``{% for %}`` tag iterates over arrays.

Template expressions are surrounded by ``{{`` and ``}}``; they are
substituted by the values from a context during template rendering.


Entry Template
==============

The ``entry.html`` file has nothing surprising:

.. code-block:: html+django

   {% extends 'base.html' %}

   {% block title %}{{ entry.title }}{% endblock %}

   {% block content %}
     <h1>{{ entry.title }}</h1>
     <em>{{ entry.author }}</em>
     <p>{{ entry.message }}</p>
   {% endblock %}


Deploying
=========

After writing the code I click the Preview button and test the
app. Everything is OK; so I switch to the Commit mode, enter a commit
message, and press the Amend button to overwrite the commit of the
skeleton code. Then I click the ``release`` :ref:`environment
<environments>` and run the ``init()`` function in it.

To make the app available to the world I should bind it to a domain. I
open the :menuselection:`App --> Manage Domains` dialog and use the
free ``guestbook.akshell.com`` domain. Done! You can test_ the example
and browse its `source code`_.

.. _test: http://guestbook.akshell.com/
.. _source code: https://github.com/akshell/guestbook


What's Next?
============

So you've read the introduction to Akshell. I've only just scratched
the surface with it (it's less than 10% of overall documentation), but
at this point you should know enough to create an app and start
fooling around. As you need to learn new tricks, come back to the
documentation. To form a deeper understanding of Akshell, read the
:doc:`guide/index`; to look up a description of a particular function
or class, consult the :doc:`ref/index`.
