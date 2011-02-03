===========
HTTP Parser
===========

.. class:: HttpParser(type, handler)

   An ``HttpParser`` object parses an HTTP message and calls the
   corresponding callbacks. *type* is either ``'request'`` or
   ``'response'``. The *handler* object holds the following optional
   callback properties::

      {
        onMessageBegin: function () {
          // Parsing has begun
        },

        onPath: function (part) {
          // URI path
        },

        onQueryString: function (part) {
          // URI query string
        },

        onFragment: function (part) {
          // URI fragment
        },

        onURI: function (part) {
          // URI
        },

        onHeaderField: function (part) {
          // HTTP header field
        },

        onHeaderValue: function (part) {
          // HTTP header value
        },

        onHeadersComplete: function (info) {
          // The info object has the versionMajor, versionMinor,
          // shouldKeepAlive, and upgrade properties;
          // request parsers set the method property;
          // response parsers set the status property
        },

        onBody: function (part) {
          // HTTP message body
        },

        onMessageComplete: function () {
          // Parsing has finished
        }
      }

   .. method:: exec(chunk)

      Parse the next *chunk*; throw a :exc:`ValueError` on failure.
