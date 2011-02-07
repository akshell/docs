====
JSGI
====

JSGI stands for JavaScript Web Server Gateway Interface. Akshell
conforms to the `CommonJS JSGI 0.3`__ standard.

__ http://wiki.commonjs.org/wiki/JSGI/Level0/A/Draft2

The JSGI application entry point is the ``app()`` function exported by
the ``main`` module. It takes exactly one argument, a request object,
and returns a response object. Here is an example of a request
object::

   {
     version: [1, 1], // HTTP version
     method: 'GET', // HTTP method
     host: 'yourapp.akshell.com', // Request host
     pathInfo: '/some/path/', // Request path
     queryString: 'x=42', // GET parameters
     headers: { // Request headers with lowercase names
       host: 'yourapp.akshell.com',
       'x-header': ['multiple', 'headers']
     },
     input: '', // POST data
     jsgi: {
       version: [0, 3], // JSGI version
     },
     env: {} // A place for middleware to put keys
   }

A response object must have three properties: ``status``, ``headers``,
and ``body``. Here is an example::

   {
     status: 200,
     headers: {
       'Content-Type': 'text/plain',
       'Content-Length': '11',
       'X-Header': ['multiple', 'headers']
     },
     body: ['Hello', ' ', 'world']
   }
