============
File Storage
============

The ``fs`` module provides read-only access to the app's work tree and
the basic Akshell lib.

.. class:: FileStorage

   A ``FileStorage`` object represents a file tree.

   .. method:: exists(path)

      Test whether an entry exists.

   .. method:: isFile(path)

      Test whether an entry exists and is a file.

   .. method:: isFolder(path)

      Test whether an entry exists and is a folder

   .. method:: read(path)

      Return a :class:`Binary` object with the file's content; throw a
      :exc:`NoSuchEntryError` if there's no such entry or an
      :exc:`EntryIsFolderError` if it's a folder.

   .. method:: list(path)

      Return a sorted array of names of folder's subentries; throw a
      :exc:`NoSuchEntryError` if there's no such entry or an
      :exc:`EntryIsFileError` if it's a file.

.. data:: code

   A :class:`FileStorage` object for accessing the app's work tree.

.. data:: lib

   A :class:`FileStorage` object for accessing the basic Akshell lib.


Exceptions
==========

.. exception:: FSError

   A base class of all file storage exceptions.

.. exception:: EntryExistsError

   Entry already exists

.. exception:: NoSuchEntryError

   Entry doesn't exist.

.. exception:: EntryIsFolderError

   Entry is a folder.

.. exception:: EntryIsFileError

   Entry is a file.
