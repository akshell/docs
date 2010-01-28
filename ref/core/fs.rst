
================
File Storage API
================

.. module:: ak.fs

.. todo:: Add an appropriate link to "Expires header"

Akshell provides applications with a simple file storage. It's
designed for common web static files like CSS, client JavaScript, etc;
for huge media collections you should use an external service like
`Amazon S3`_ or Nirvanix_. Files are served by a web server from
http://media.akshell.com/ with a far future ``Expires`` header; anyone
could access them for browsing and reading. An application manages its
storage using :mod:`ak.fs` module functions described here.

.. _Amazon S3: http://aws.amazon.com/s3/
.. _Nirvanix: http://www.nirvanix.com/

Concepts
========

Akshell :dfn:`storage entries` are files and directories organized
into a traditional treelike structure; entry names could contain any
Unicode symbols except ``'\0'``, ``'\n'``, and ``'/'``. An application
identify its storage entries by :dfn:`paths`; name delimiting
character is the slash (``'/'``). Application clients retrieve entries
from URLs of the form :file:`http://media.akshell.com/{prefix}/{path}`
where :file:`{prefix}` is :file:`release/{appName}` for the release
version of the application and
:file:`spots/{appName}/{ownerName}/{spotName}` for a spot one
(:file:`{ownerName}` is lower cased with spaces replaced by hyphens).

For example, file :file:`hello.txt` in subdirectory :file:`subdir` of
directory :file:`dir` has a path :file:`'dir/subdir/hello.txt'`. In
the release version of ``example`` application it has an URL
http://media.akshell.com/release/example/dir/subdir/hello.txt, in
Anton Korenyushkin's (mine) spot ``debug`` it has an URL
http://media.akshell.com/spots/example/anton-korenyushkin/debug/dir/subdir/hello.txt.

:dfn:`Temporary files` are another kind of files: they come from
requests (see :doc:`request` for details), have no path, and disappear
after the request was processed. :class:`~ak.TempFile` class objects
representing temporary files are accepted by all :mod:`ak.fs`
functions expecting file path parameter.

Functions
=========

.. function:: read(path)

   Return file contents represented by a :class:`~ak.Data` object.

.. function:: write(path, data)

   Coerce *data* to ``string`` and write it into a file; if the file
   already exists overwrite it.

.. function:: list(path)

   Return an array of names of directory subentries in arbitrary order.

.. function:: exists(path)

   Test whether an entry exists.

.. function:: isDir(path)

   Test whether an entry exists and is a directory.

.. function:: isFile(path)

   Test whether an entry exists and is a file.

.. function:: makeDir(path)

   Create a directory; fail if the parent directory does not exist.

.. function:: remove(path)

   Remove an entry; if it's a directory remove it recursively.

.. function:: rename(oldPath, newPath)

   Change a path of an entry; fail if *newPath* already exists or its
   parent directory does not exist.

.. function:: copyFile(origPath, copyPath)

   Create a file with path *copyPath* and contents of the original file.

Classes
=======

.. currentmodule:: ak

.. class:: Data

   A file contents representation.

   .. method:: toString(encoding='UTF-8')

      Return a ``string`` decoded from the data using *encoding*.

.. class:: TempFile

   A temporary file representation. Values of ``files`` request
   property are instances of this class (see :doc:`request` for
   details).

   .. method:: read()

      Return the file contents represented by a :class:`Data` object.
