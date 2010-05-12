
================
File Storage API
================

.. module:: fs

Akshell provides applications with a simple file storage. Files are
served by a web server from http://static.akshell.com/media/ with a
far future ``Expires`` header; anyone can access them for browsing and
reading. An application manages its storage via the :mod:`fs` module
functions described here.


Concepts
========

Akshell :dfn:`storage entries` are files and directories organized
into a traditional treelike structure; entry names can contain any
Unicode symbols except ``'\0'``, ``'\n'``, and ``'/'``. An application
identify its storage entries by :dfn:`paths`; name delimiting
character is the slash (``'/'``). Application clients retrieve entries
from URLs of the form
:file:`http://static.akshell.com/media/{prefix}/{path}` where
:file:`{prefix}` is :file:`release/{appName}` for the release version
of the application and :file:`spots/{appName}/{ownerName}/{spotName}`
for a spot one (:file:`{ownerName}` is lower cased with spaces
replaced by hyphens).

For example, the file :file:`hello.txt` in the subdirectory
:file:`subdir` of the directory :file:`dir` has a path
:file:`'dir/subdir/hello.txt'`. In the release version of the
``example`` application it has an URL
http://static.akshell.com/media/release/example/dir/subdir/hello.txt,
in Anton Korenyushkin's (mine) spot ``debug`` it has an URL
http://static.akshell.com/media/spots/example/anton-korenyushkin/debug/dir/subdir/hello.txt.

:dfn:`Temporary files` are another kind of files: they come from
requests (see :doc:`request` for details), have no path, and disappear
after the request was processed. :class:`~ak.TempFile` objects
representing temporary files are accepted by all :mod:`fs` functions
expecting file path parameter.


Functions
=========

.. function:: read(path)

   Return file contents represented by a :class:`~ak.Data` object.

.. function:: write(path, data)

   Coerce *data* to ``string`` and write it into a file; if the file
   already exists, overwrite it.

.. function:: list(path)

   Return an array of names of directory subentries in arbitrary order.

.. function:: exists(path)

   Test whether an entry exists.

.. function:: isDir(path)

   Test whether an entry exists and is a directory.

.. function:: isFile(path)

   Test whether an entry exists and is a file.

.. function:: createDir(path)

   Create a directory; fail if the parent directory does not exist.

.. function:: remove(path)

   Remove an entry; if it's a directory, remove it recursively.

.. function:: rename(oldPath, newPath)

   Change a path of an entry; fail if *newPath* already exists or its
   parent directory does not exist.

   
Classes
=======

.. currentmodule:: None

.. class:: Data

   A ``Data`` object represents file contents.

   .. method:: toString(encoding='UTF-8')

      Return a ``string`` decoded from the data using *encoding*.

.. class:: TempFile

   A ``TempFile`` object represents a temporary file passed to the
   application through the values of the ``files`` request property
   (see :doc:`request` for details).

   .. method:: read()

      Return the file contents represented by a :class:`Data` object.
