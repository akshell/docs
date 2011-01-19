=============
Using Akshell
=============

This document explains the ideas behind Akshell, their implementation,
and the way you could employ them in your applications.


Overview
========

Akshell is a :dfn:`web application network`. I've recently invented
this term; so I'd better explain it.

Akshell is a site where you can develop web applications in JavaScript
and host them in the cloud. The development is easy and the deployment
is instant. You don't have to buy and administer a server or deal with
low-level details; you even don't have to install anything on your
computer.

But what makes Akshell a "network" is the environment it provides
applications with. In Akshell applications make requests to each
other; they interact just like programs in UNIX-like operation
systems. This creates the whole new world: a web application is no
longer an all-in-one bundle of code and data, but a part of a larger
ecosystem where every member has an access to the information
collected by others.

It should change the way you think about web application architecture:
instead of writing a big strong coupled piece of code handling all
tasks, you can create a number of applications, each one performing
one small task. The applications will interact with each other and
with third-party applications to perform larger tasks. This approach
makes applications more universal and user experience more consistent,
so developers will have less work and users will be happier.


Design
======

.. epigraph::

   Perfection is achieved, not when there is nothing more to add, but
   when there is nothing left to take away.

   -- Antoine de Saint-Exupéry


JavaScript
----------

JavaScript is the language of Akshell. Modern web applications usually
have a rich AJAX interface created using one of the popular
client-side JavaScript libraries; in Akshell you employ it for the
server side too. This makes your application more integral and robust;
you can even share some code between the client and server sides
(e.g., the code of form validation).


Applications
------------

Application is the basic unit of the Akshell environment. Each
application has a unique name and serves from the
:samp:`{appName}.akshell.com` domain. The name can contain lowercase
Latin letters, numbers, and single hyphens (in order to be a valid
domain name).

Every Akshell user can create applications. Currently only the
creation of free non-commercial applications is available (code of
such applications *must* be provided under the terms of the `BSD
License`_). After the creation you become an application admin, i.e.,
the user with full access to the application (you could even
completely delete it).

.. _BSD License: /about/bsd/

By default the maximum number of administrated applications per user
is 10; this limit is set to prevent name squatting. If you're
productive and need more, just write a letter to support@akshell.com.


Authentication
--------------

To enable application interoperability Akshell provides centralized
authentication system: all applications use the Akshell login. User
identity is preserved during cross-application requests; this is
essential for application interaction.

Akshell guarantees the security of the authentication. It is
implemented via cookies restricted to application domains; so a
malicious application can neither guess a user's password nor steal an
authentication cookie of other application.


Developers
----------

Akshell encourages collaboration: you can give other users a
development access to your applications. There are two kinds of
developers: those with an access to the release code and those without
it. If you do not trust a person enough, you can provide him with the
second, limited access. He won't be able to affect the release code
(you will have to release his work by yourself); so the users of your
applications will be out of danger. When the guy shows he can be
relied upon, you will give him the release access.


Spots
-----

A spot is a version of an application completely independent from the
release version and owned by one of application developers. Spots are
identified by the name and the owner. The spot application version
handles requests from the
:samp:`{spotName}.{ownerName}.{appName}.akshell.com` domain (click the
*Show* spot menu item to view the corresponding page in your browser).

The purpose of spots is to separate development and debugging from
serving application users. When the next stable version is ready, the
spot containing it could be "released": it will instantly become
available to everyone (this operation is atomic).

Spots also facilitate collaboration: developers have independent spots
and don't hinder each other. If you want to experiment with a spot
of other developer, you could clone it into your spot.

.. note::

   The spot system is **not** a substitution for a version control
   system. You should employ version control for your application
   because it significantly facilitates development. There are plenty
   of sites providing free VCS hostings for free projects,
   e.g., Bitbucket_, GitHub_, `Google Code`_.

.. _Bitbucket: http://bitbucket.org
.. _GitHub: http://github.com
.. _Google Code: http://code.google.com


Akshell Core
------------

The Akshell core executes applications via the Google V8 JavaScript
engine. The engine compiles JavaScript code into the native code; so
the execution is utterly fast. The core_ library exposes the
:doc:`core APIs </ref/core/index>` to applications.

.. _core: http://www.akshell.com/apps/core/

To handle a request the Akshell core evaluates the ``main.js`` file of
your application and runs the ``app()`` function exported by it,
passing a request object to the function. This architecture conforms
to the :term:`JSGI` specification. Note that the application state is
not guaranteed to persist across requests, i.e., variables set during
a request handling may not be available after the handling is
completed.  The application should maintain persistence through the
:doc:`database </ref/core/db>` and the :doc:`file storage
</ref/core/fs>`.


Basic Library
-------------

The core APIs are rather low-level, just like system calls in common
operation systems; a web framework is needed for comfortable
development. The ak_ library provides general JavaScript goodies and a
:term:`Model-View-Controller <MVC>` framework. For convenience, the
library exports the core APIs along with its own. Because these
exports will be used quite often in your program, it's recommended to
put them to the global object (it's already done in the application
skeleton)::

   require('ak', '0.2').setup()

.. _ak: http://www.akshell.com/apps/ak/

During a request handling the ak_ library converts a :term:`JSGI`
request object into a more convenient :class:`Request` object and
passes it to the ``main()`` function exported by ``main.js``, which
defaults to :func:`defaultServe`.


Interaction
===========

Akshell states that a browser should be the only required tool for web
development; so all application management can be done from a browser
window.

The major part of your interaction with Akshell will take place in the
*Code* tab of the application section. Here you write code, create,
clone, and delete spots, view code of other developers (these actions
are covered above).


.. _evaluate:

Evaluate
--------

In the *Evaluate* tab you could evaluate JavaScript expressions in
your spots and in the release code if you have an access to it. Be
careful with the release code: all changes will immediately affect
users.

Remember that Akshell does **not** guarantee the persistence of the
application state across requests and evaluations. I.e., two
subsequent evaluations can easily give the following::

   >>> x = 42
   >>> x
   ReferenceError: x is not defined

The state persists during evaluation::

   >>> x = 42; x
   42

If you want to evaluate a complex piece of code, wrap it by an
anonymous function (this technique is broadly used in the
documentation)::

   >>> (function ()
       {
         var x = 42;
         assertSame(x, 42);
       })()


Administer
----------

In the *Administer* tab you could set the contact email (displayed
protected by reCAPTCHA_), the summary and the labels (used for the
application search), the description (displayed on the application
home page). You could also manage the developers here and completely
delete the application.

.. _reCAPTCHA: http://recaptcha.net/


.. _tool:

Tool
====

Akshell has a RESTful API for external access to the application
management. The API is not documented yet, but it is used in the
Akshell tool -- the utility for those of us who prefer his favorite
editor to the browser's one. The tool enables you to:

* download the application code to your computer;
* upload the code back to Akshell;
* evaluate expressions just like in the *Evaluate* tab.

The tool has a command-line interface and suites for manual usage,
scripting, and IDE integration. Type this in the command line to
access its built-in help::

   akshell help


Installation
------------

To install the tool follow the steps specific to your OS.


Mac OS X
~~~~~~~~

Type::

   sudo easy_install akshell


Windows
~~~~~~~

Download the installer_, launch it, click Next, Next, Next...

.. _installer: http://bitbucket.org/akshell/tool/downloads/setup.exe


Linux
~~~~~

Type::

   sudo easy_install akshell

If ``easy_install`` is not found, type the following on Ubuntu or
Debian::

   sudo apt-get install python-setuptools

... and the following on Fedora, SUSE, RHEL, or CentOS::

   sudo yum install python-setuptools python-setuptools-devel


IDE Integration
---------------

The tool is a command-line Python script with a simple interface; so
it should be easy to integrate it into your favorite IDE. For advanced
usage the ``akshell`` Python module is provided; you could employ it
in your scripts.

If you've created an Akshell plugin for you favorite IDE, `let me
know`_ -- I'd be glad to describe it here.

.. _let me know: anton@akshell.com


Emacs
~~~~~

If you are an Emacs user, add these lines to your configuration file
(if you are not, don't look at this code -- it may cause a brain
injury)::

   (eval-after-load "compile"
     '(progn
        (add-to-list
         'compilation-error-regexp-alist-alist
         '(akshell
           "^    at \\(?:[^\n]*(\\)?\\([^:\n]*\\):\\([0-9]+\\))?" 1 2 3 nil 0))
        (add-to-list 'compilation-error-regexp-alist 'akshell t)))

Then you can use ``M-x compile`` to launch the tool and enjoy nice
error backtraces.
