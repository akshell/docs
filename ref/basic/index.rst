=========
Basic API
=========

The basic Akshell API is built into the JavaScript engine running
application code; it provides low-level means of environment
interaction.

.. toctree::
   :maxdepth: 1

   modules
   core
   binary
   db
   fs
   proxy
   script
   socket
   git
   http-parser
   jsgi

.. note::

   The ``core``, ``binary``, and ``jsgi`` modules are loaded by the
   ``ak`` library; the ``db``, ``fs``, ``proxy``, ``script``,
   ``socket``, ``git``, and ``http-parser`` modules must be
   :func:`required <require>` explicitly.
