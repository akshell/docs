====
HTTP
====

In the http_ module HTTP exception classes and status codes are
defined.

.. _http: https://github.com/akshell/ak/blob/0.3/http.js


Exceptions
==========

.. exception:: Failure(message='Bad request', status=http.BAD_REQUEST)

   A base class of errors which should be reported to a user by a HTTP
   response with the status code *status* and *message* in the
   content. Whenever your application faces a problem it has to report
   to a user, it should throw a ``Failure``. The
   :func:`serve.catchingFailure` decorator will catch the error and
   convert it to a :class:`Response` object.

.. exception:: NotFound(message='Not found')

   A resource was not found. Reported to a user by a 404 HTTP
   response. Subclass of :exc:`Failure`.

.. exception:: Forbidden(message='Forbidden')

   A request is forbidden. Reported to a user by a 403 HTTP
   response. Subclass of :exc:`Failure`.


.. _status_codes:

Status Codes
============

.. module:: http

HTTP status code constants are properties of the ``http`` module. The
most popular constants a described here. You are unlikely to need
other constants, but if you do, see the `source code`_ and the `List
of HTTP status codes`__ Wikipedia page.

.. _source code: http_
__ http://en.wikipedia.org/wiki/List_of_HTTP_status_codes


Success
-------

This class of status codes indicates the action requested by the
client was received, understood, accepted, and processed successfully.

.. data:: OK

   Standard response for successful HTTP requests.

.. data:: CREATED

   The request has been fulfilled and resulted in a new resource being
   created.


Redirection
-----------

This class of status codes indicates that further action needs to be
taken by the user agent in order to fulfill the request.

.. data:: MOVED_PERMANENTLY

   This and all future requests should be directed to the URI
   specified in the ``Location`` response header.

.. data:: FOUND

   The response to the request can be found under the URI specified in
   the Location response header. This status code is used by the
   :func:`redirect` function to redirect the user agent after a
   successful fulfillment of a POST request.

.. data:: NOT_MODIFIED

   The resource has not been modified since last requested. Typically,
   the HTTP client provides a header like *If-Modified-Since* or
   *If-None-Match* to identify the state of the resource possessed by
   the client.


Client Error
------------

This class of status codes is intended for cases in which the client
seems to have erred.

.. data:: BAD_REQUEST

   The request contains bad syntax or cannot be fulfilled.

.. data:: FORBIDDEN

   The application understood the request, but is refusing to fulfill
   it. The reason should be described in the content of the response.

.. data:: NOT_FOUND

   The requested resource could not be found.

.. data:: METHOD_NOT_ALLOWED

   A request was made of a resource using a request method not
   supported by that resource. For example, using GET on a form which
   requires data to be presented via POST, or using PUT on a read-only
   resource.


Server Error
------------

These status codes indicate cases in which the application is aware
that it has encountered an error or is otherwise incapable of
performing the request.

.. data:: INTERNAL_SERVER_ERROR

   The application has erred.

.. data:: NOT_IMPLEMENTED

   The application does not support the functionality required to
   fulfill the request.

.. data:: SERVICE_UNAVAILABLE

   The application is currently unavailable (because it is overloaded
   or down for maintenance).
