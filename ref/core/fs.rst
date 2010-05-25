
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
for a spot one (:file:`{ownerName}` has spaces replaced by hyphens).

For example, the file :file:`hello.txt` in the subdirectory
:file:`subdir` of the directory :file:`dir` has a path
:file:`'dir/subdir/hello.txt'`. In the release version of the
``example`` application it has an URL
http://static.akshell.com/media/release/example/dir/subdir/hello.txt,
in Anton Korenyushkin's (mine) spot ``debug`` it has an URL
http://static.akshell.com/media/spots/example/Anton-Korenyushkin/debug/dir/subdir/hello.txt.


Functions
=========

.. function:: open([appName,] path)

   Open a file and return a :class:`fs.File` object.

.. function:: read([appName,] path)

   Return file contents represented by a :class:`Binary` object.

.. function:: write(path, data)

   Coerce *data* to :class:`Binary` and write it into a file; if the
   file already exists, overwrite it.

.. function:: list([appName,] path)

   Return an array of names of directory subentries in arbitrary order.

.. function:: exists([appName,] path)

   Test whether an entry exists.

.. function:: isDir([appName,] path)

   Test whether an entry exists and is a directory.

.. function:: isFile([appName,] path)

   Test whether an entry exists and is a file.

.. function:: getModDate([appName,] path)

   Return a modification ``Date`` of an entry.

.. function:: createDir(path)

   Create a directory; fail if the parent directory does not exist.

.. function:: remove(path)

   Remove an entry; if it's a directory, remove it recursively.

.. function:: rename(oldPath, newPath)

   Change a path of an entry; fail if *newPath* already exists or its
   parent directory does not exist.

   
File
====

.. class:: File

   A ``File`` object provides means of reading and writing a file. It
   can be obtained via the :func:`fs.open` function or the ``files``
   :class:`Request` property. In the latter case file is read-only.

   .. attribute:: closed

      ``true`` if the file is closed; ``false`` otherwise.
      
   .. attribute:: writable

      ``true`` if the file is writable; ``false`` otherwise.
   
   .. attribute:: length

      The length of the file, in bytes. If the file is writable,
      ``length`` is assignable.
   
   .. attribute:: position

      The position within the file at which the next read or write
      operation occurs, in bytes. Setting ``position`` to a new value
      moves the stream's position to the given byte offset.

   .. method:: read([max])

      Read *max* bytes from the stream, or until the end of the file
      has been reached, and return a :class:`Binary` object. If the
      argument is omitted, read until the end of the file.
   
   .. method:: write(data)

      Coerce *data* to :class:`Binary` and write it to the file.
      Return ``this``.
   
   .. method:: flush()

      Flush the file to the disc. Return ``this``.
   
   .. method:: close()

      Close the file, freeing the resources it is holding. ``close()``
      can be called multiple times; all other operations on a closed
      file result in a :exc:`ValueError` exception.
   

Exceptions
==========

.. currentmodule:: None

.. exception:: FSError

   A base class of all file storage exceptions.
   
.. exception:: PathError

   Incorrect path.

.. exception:: EntryExistsError

   Entry already exists

.. exception:: NoSuchEntryError

   Entry doesn't exist.

.. exception:: EntryIsDirError

   Entry is a directory.
   
.. exception:: EntryIsNotDirError

   Entry is not a directory.
   
.. exception:: FileIsReadOnly

   Attempt to write into a read-only file.
   
.. exception:: FSQuotaError

   File storage quota exceeded.
