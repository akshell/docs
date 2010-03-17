
================
Request Handling
================

Akshell application model is centered in request handling. During its
life cycle an application waits for a request delivery, handles a
request, or requests other application and waits for the request to be
handled. Besides that, application developers can also evaluate
arbitrary expressions in their spots (and in the release if they have
a permission of an application admin to do so), but this feature is
intended only for debug purposes.

Akshell protects application database by wrapping each request
handling and expression evaluation by a :term:`transaction`: database
changes either all occur or nothing occurs. Cancellation of changes
happens if request/expression processing throws an exception or calls
the :func:`db.rollback` function.

The request handling entry point of an application is the
``__main__()`` function which must be defined by the
:file:`__main__.js` file. It should accept a :class:`Request` object
as an argument and return a :class:`Response` object.


Request
=======

.. class:: Request

   A ``Request`` object stores data passed with an application
   request. The ``__main__()`` function receives a ``Request`` object
   whenever the application is requested.

   .. attribute:: method

      A lowercase string representing the method of the request.
   
   .. attribute:: user

      A name of the user who initiated the request; an empty string if the
      user is not logged in.
      
   .. attribute:: issuer

      A name of the application which initiated the request; an empty
      string if the request was initiated by a client browser.
      
   .. attribute:: path

      A string representing the path of the requested resource; always
      starts with the slash.
      
   .. attribute:: fullPath

      The path of the requested resource plus an appended query
      string if applicable.
      
   .. attribute:: uri

      The full requested URI.
      
   .. attribute:: get

      An object mapping GET parameter names to their values.
   
   .. attribute:: post
   
      An object mapping POST parameter names to their values; does
      **not** include file uploads, see :attr:`files`.
      
   .. attribute:: headers

      An object mapping the request header names to their values.
   
   .. attribute:: files

      An object mapping the uploaded file names to their
      :class:`TempFile` representations.
   
   .. attribute:: data

      The raw POST data represented by a :class:`Data` object;
      ``null`` if not present.

      
Response
========

.. class:: Response(content='', status=http.OK[, headers])

   A ``Response`` object represents an application response; it has
   ``content``, ``status``, and ``headers`` properties. *content* is a
   text of the response; *status* is a HTTP status code (the ``ak``
   library defines :ref:`constants for status codes<status_codes>`);
   *headers* are HTTP headers, which default to::

      {'Content-Type': 'text/html; charset=utf-8'}
      
   ``__main__()`` function should return a ``Response`` object.


requestApp
==========

.. function:: requestApp(name, request)

   Perform an application request; return a :class:`Response`
   object. *name* is a name of the application being requested. The
   *request* object can contain the following fields:

      method
         the request method;

      path
         the path of the requested resource;

      get
         an object mapping GET parameter names to their values;

      post
         an object mapping POST parameter names to their values;

      headers
         an object mapping the request header names to their values; and

      files
         an object mapping the request file names to their paths or
         :class:`TempFile` objects.
