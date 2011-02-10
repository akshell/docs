=============
Using Akshell
=============

This document explains the ideas behind Akshell, their implementation,
and the way you can employ them in your apps.


Overview
========

Akshell has two major parts: the infrastructure executing apps and the
online IDE for developing them.

The infrastructure frees you from deployment problems and routine
administrative tasks. Your code is automatically deployed when you
commit it to the ``master`` branch. Running app doesn't require any
kind of maintenance: Akshell recovers it from failures, supplies with
necessary resources, and keeps up its database.

The IDE provides a desktop-like experience in a browser window. You
don't have to install anything on your computer; now you can start
coding after a single click, from anywhere. The tight integration
between the IDE and the Akshell guts ensures the best web-development
experience.


JavaScript
==========

JavaScript is the language of Akshell. Modern web apps usually have a
rich AJAX interface created using one of the popular client-side
JavaScript libraries; in Akshell you employ it for the server side
too. This makes your app more integral and robust; you can even share
some code between the client and server sides, e.g., the code of form
validation.


Apps
====

App is the basic development and execution unit of Akshell. First, you
create an app using the IDE and develop it committing your changes
into the app's Git repository. Then, you bind the app to domains,
e.g., ``yourapp.akshell.com`` or ``yourapp.com`` (in the latter case
you should also buy a domain from a domain registrar and point DNS
servers to ``akshell.com`` via a CNAME record). When Akshell receives
a request to to one this domains, it loads the ``main`` module from
the ``master`` branch of your app's repository and launches the
``main()`` function exported by it with a :class:`Request` object as
an argument.

Apps can be published to share their code with others, i.e., serve as
libraries. They are identified by an owner name, an app name, and a
version which is just a Git reference. ``akshell`` is the official
Akshell developer, ``akshell/ak:0.3`` is the identifier of the ``0.3``
version of his ``ak`` library.


Environments
============

To separate serving users from development, debugging, and testing
Akshell provides isolated execution environments. They are completely
independent and cannot affect each other. The ``release`` environment
is intended for serving users; it executes code from the ``master``
branch of the app's repository. Other environments are intended for
development; they execute code from the work tree. A common setup is
to use the ``debug`` environment for manual testing and the ``test``
environment for unit testing.


Engine
======

The Akshell engine is based on Google V8. It compiles JavaScript code
into native code; so execution is utterly fast. The engine exposes the
:doc:`basic API </ref/basic/index>` to apps.

The engine will be open sourced soon to eliminate the vendor lock-in:
you'll be able to launch Akshell apps on your own server.


``ak`` Library
==============

The basic API is rather low-level, just like system calls in an
operation system; a web framework is needed for comfortable
development. The :doc:`ak library </ref/ak/index>` provides general
JavaScript goodies and a :term:`Model-View-Controller <MVC>`
framework. Because the library will be used quite often in your code,
it's recommended to put its exports on the global object. This is
already done in the app skeleton::

   require('ak').setup();
