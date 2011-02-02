===
Git
===

Classes for reading app's Git repositories.

.. class:: Repo(owner, app)

   A ``Repo`` object represents a Git repository of the *owner's*
   *app*. The constructor throws a :exc:`ValueError` if the app
   doesn't exist or isn't public.

   .. attribute:: refs

      An object mapping reference names to the correspondent Git
      object IDs or strings of the form :file:`'ref: {otherRef}'`.

   .. method:: deref(ref)

      Return the Git object ID referenced by *ref*; throw a
      :exc:`ValueError` if not found.

   .. method:: readObject(id)

      Read the Git object with *id* and return a JavaScript object
      with the ``type`` and ``data`` properties; throw a
      :exc:`ValueError` if not found. ``type`` is ``'commit'``,
      ``'tree'``, ``'tag'``, or ``'blob'``; ``data`` is a
      :class:`Binary` representing the Git object.

   .. method:: getStorage(ref)

      Return ``new GitStorage(this, ref)``.

.. class:: GitStorage(repo, ref)

   A ``GitStorage`` object provides a :class:`Storage`-like interface
   to the *repo's* commit tree referenced by *ref*. The constructor
   throws a :exc:`ValueError` if such commit doesn't exist.

   .. attribute:: repo

      The repository.

   .. attribute:: ref

      The original reference.

   .. attribute:: commit

      The commit ID.

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
