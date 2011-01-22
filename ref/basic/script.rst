======
Script
======

.. class:: Script(source[, resourceName, [lineOffset, [columnOffset]]])

   A ``Script`` object represents a compiled JavaScript
   code. *resourceName*, *lineOffset*, and *columnOffset* are used in
   exception backtraces.

   .. method:: run()

      Run the script; return the evaluation value.
