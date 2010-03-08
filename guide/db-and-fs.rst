
=========================
Database and File Storage
=========================

An application has two places to store persistent information: a
database and a file storage. This document describes their interfaces.


Database
========

A database is intended for storing structured data and maintaining
their integrity.


Overview
--------

The Akshell database system is based on the :term:`relational
model`. A database consists of named :dfn:`relation variables`. Each
relation variable store data about a particular subject.

For example, a ``Post`` relation variable could store information
about blog posts:

==  ======  ===========  ==============
id  author  title        text
==  ======  ===========  ==============
0   Bob     Greeting     Hello, world!
1   Alice   Declaration  I love Akshell
2   Bob     The Answer   42
==  ======  ===========  ==============

A value of a relation variable is a :dfn:`relation`, which in turn is
a set of :dfn:`tuples`. A relation should be considered as a set of
statements about some subject where each tuple is a single
statement. Relations could be imagined as tables, each tuple being a
row, but there are a few caveats:

* a relation couldn't have duplicate tuples;
* relation tuples are unordered;
* relation attributes are unordered.

Relation variables of a database form together an application's
knowledge about its domain. Akshell ensures integrity of this
knowledge on a number of levels:

* :dfn:`types` of individual attributes (columns) guarantee their
  sense;

* :dfn:`constraints` maintain logical consistency of the data;

* :dfn:`transactions` prevent database changes occurring only
  partially: a series of database operations performed during a
  request handling either all occur, or nothing occurs.


Relation Variables
------------------

To create a new relation variable, call the :func:`db.create`
function, for example::

   >>> db.create('Post',
                 {
                   id: 'serial unique',
                   author: 'string',
                   title: 'string',
                   text: 'string'
                 });

Properties of the :data:`rv` object provide your program with an
interface to the relation variables. Each property is an instance of
the :class:`RelVar` class.

For example, the ``Post`` relation variable could be managed as
follows::

   >>> rv.Post.insert(
         {author: 'Bob', title: 'Greeting', text: 'Hello, world!'})
   >>> rv.Post.insert(
         {author: 'Alice', title: 'Declaration', text: 'I love Akshell'})
   >>> rv.Post.insert(
         {author: 'Bob', title: 'The Answer', text: '42'})
   >>> rv.Post.all().get({by: 'id'}).map(repr).join('\n')
   {author: "Bob", id: 0, text: "Hello, world!", title: "Greeting"}
   {author: "Alice", id: 1, text: "I love Akshell", title: "Declaration"}
   {author: "Bob", id: 2, text: "42", title: "The Answer"}

Each tuple is represented by a plain ``Object`` instance; a relation
retrieved from the database is represented by an ``Array`` of such
objects.

To permanently remove a relation variable from the database use the
:meth:`~RelVar.drop` method; the corresponding :data:`rv` property
will disappear after that::

   >>> rv.Post.drop()
   >>> 'Post' in rv
   false
   

Types
-----

Akshell provides four database types: ``number``, ``string``,
``bool``, and ``date``. Yes, they are borrowed from JavaScript -- the
database system was specifically designed to integrate seamlessly into
this language.


Integer
~~~~~~~

In the majority of cases when you store numbers in the database, these
numbers could have only integer values: a blog post could not have
3.14 comments, neither could a group have 2.72 members. To designate
this restriction, use the ``'integer number'`` type description or
just ``'integer'`` -- Akshell is rather shrewd.

For example, this relation variable could be used in a hotel
management application::

   >>> db.create('Room', {number: 'integer', capacity: 'integer'})

Fractional numbers will be rounded when inserted as values of an
integer attribute::

   >>> repr(rv.Room.insert({number: 3.001, capacity: 1.5}))
   {capacity: 2, number: 3}

Some numbers could not be converted to integer::

   >>> rv.Room.insert({number: 1, capacity: Infinity})
   ak.ConstraintError: ...
   >>> rv.Room.insert({number: 1, capacity: NaN})
   ak.ConstraintError: ...
   >>> rv.Room.insert({number: 1, capacity: 1e10})
   ak.ConstraintError: ...

   
Default
~~~~~~~

Some relation variable attributes have the same value in the majority
of cases. In order to avoid repetitions, you could attach a
:dfn:`default value` to such attribute: whenever a value of the
attribute is not specified, the default will be used.

Suppose you store data about the users of your application in the
``Profile`` relation variable. Then the ``description`` and ``banned``
attributes will naturally have default values::

   >>> db.create('Profile',
                 {
                   name: 'unique string',
                   description: 'string default ""',
                   banned: 'bool default false'
                 })
   >>> repr(rv.Profile.insert({name: 'Bob'}))
   {banned: false, description: "", name: "Bob"}
   >>> repr(rv.Profile.insert({name: 'Anton', description: "That's me"}))
   {banned: false, description: "That's me", name: "Anton"}


.. _serial:

Serial
~~~~~~

To maintain uniqueness of tuples and to have a way of referencing
them, it's often necessary to add a :dfn:`serial` attribute counting
tuples: 0, 1, 2, etc. Use ``'number serial'`` or just ``'serial'``
type description for this purpose.

Whenever a value of such attribute is omitted, the next number of a
sequence is used. Serial attributes could have only integer values. ::

   >>> db.create('Counter', {s: 'serial'})
   >>> repr(rv.Counter.insert({}))
   {s: 0}
   >>> repr(rv.Counter.insert({}))
   {s: 1}
   >>> repr(rv.Counter.insert({s: 42}))
   {s: 42}
   >>> repr(rv.Counter.insert({}))
   {s: 2}
   
   
Constraints
-----------

Constraints are the main tool for maintaining logical consistency in a
database. Akshell provides the :dfn:`unique`, :dfn:`foreign key`, and
:dfn:`check` constraints.


.. _unique:

Unique
~~~~~~

If a set of relation variable attributes has a unique constraint,
these attributes must have unique sets of values across all tuples of
the relation.

To attach a unique constraint to one attribute, add the ``unique``
word to its type description; to declare more than one attribute
unique, use a separate unique declaration.

For example::

   >>> db.create('Post',
                 {
                   id: 'unique serial',
                   author: 'string',
                   title: 'string',
                   text: 'string'
                 },
                 'unique [author, title]');

All posts will have a unique ``id`` attribute; posts of the same
author will never have the same title::

   >>> rv.Post.insert(
         {author: 'Bob', title: 'Greeting', text: 'Hello, world!'}).id
   0
   >>> rv.Post.insert(
         {id: 0, author: 'Alice', title: 'Declaration', text: 'I love Akshell'})
   ak.ConstraintError: ...
   >>> rv.Post.insert(
         {author: 'Bob', title: 'Greeting', text: 'Hello again!'})
   ak.ConstraintError: ...


.. _foreign_key:
         
Foreign Key
~~~~~~~~~~~

It's very common relation variables to be interconnected. A foreign
key is a reference from one relation variable to another, i.e., a
many-to-one relationship between them.

For example, the ``Comment`` relation variable could reference
``Post`` defined before through its ``id`` attribute::

   >>> db.create('Comment',
                 {
                   id: 'unique serial',
                   post: 'integer -> Post.id',
                   author: 'string',
                   text: 'string'
                 });

A referenced attribute of a foreign key must be unique (otherwise the
key would be meaningless). Akshell ensures that for each referencing
tuple referenced tuple exists, i.e., a reference makes sense. If you
try to break this rule, an error will be thrown::

   >>> rv.Post.insert(
         {author: 'Bob', title: 'Greeting', text: 'Hello, world!'}).id
   0
   >>> rv.Comment.insert(
         {post: 0, author: 'Alice', text: 'Hi, Bob'})
   >>> rv.Comment.insert(
         {post: 42, author: 'Bob', text: 'Bump!'})
   ak.ConstraintError: ...
   >>> rv.Post.where({id: 0}).del()
   ak.ConstraintError: ...
         

Check
~~~~~

A check constraint simply checks than a given expression holds
``true`` for each tuple of a relation variable. You could add a check
inside a type declaration or in a separate check declaration.

For example, an airline company could employ the following relation
variable (note the required parenthesis)::

   >>> db.create('Flight',
                 {
                   departure: 'date',
                   arrival: 'date',
                   passengers: 'integer check (passengers > 0)'
                 },
                 'check (arrival > departure)');

If you try to break a check, an error will be thrown::

   >>> rv.Flight.insert({
                          departure: 'Jan 1 2010',
                          arrival: 'Dec 31 2009',
                          passengers: 100
                        })
   ak.ConstraintError: ...
   >>> rv.Flight.insert({
                          departure: 'Jan 1 2010',
                          arrival: 'Jan 2 2010',
                          passengers: -1
                        })
   ak.ConstraintError: ...


Transactions
------------

Akshell wraps each request handling by a transaction. If an
application fails to handle a request and throws an exception, the
changes it has made to the database are :dfn:`rolled back`. If the
handling succeeds, the changes are stored permanently. You could also
roll back changes of the current transaction manually via the
:func:`db.rollback` function.


Querying
--------

The main point of a database is handy retrieving information from
it. Akshell provides a sophisticated yet simple tool for this purpose
-- the query language. It was designed on the :term:`relational model`
foundation with the JavaScript integration in mind. This section
covers the basics of the query language; if you are interested in
details, consult :ref:`its reference <query_language>`.

.. admonition:: Note for SQL users

   The query language provides the capabilities of the SQL SELECT
   statement in more consistent and simple way.

Performing a database query is a two-step process:

* first, you call the :func:`~RelVar.where` method of a
  :class:`RelVar` object defining *what* tuples you'd like to
  retrieve;

* then, you call the :func:`~Selection.get` method of the
  :class:`Selection` object returned on the previous step defining
  *how* you'd like to retrieve these tuples.

  
where()
~~~~~~~
  
:func:`~RelVar.where` accepts an expression the resulting tuples
should match and positional arguments of this expression. They are
substituted for ``$1``, ``$2``, etc. placeholders in the expression.

For example, this query returns posts of the given author with the
given title::

   rv.Post.where('author == $1 && title == $2', 'Bob', 'Greeting').get()
   
For brevity a single ``$`` could be used instead of ``$1``, for
example::

   rv.Post.where('author == $', 'Bob').get()

.. warning::

   **Never** construct a query string manually: a malicious user could
   give a tricky input to form an illegal query and read data he is
   forbidden to read. **Always** use positional arguments.
     
Besides JavaScript operators Akshell also supports some additional
operators in query expressions, including the reference operator
"``->``" which provides a convenient access to attributes of a
referenced relation variable.

For example, this query returns comments to Bob's posts::

   rv.Comment.where('post->author == $', 'Bob').get()

Query expressions mimic JavaScript syntax and semantics as close as
possible. See :ref:`their reference <expressions>` for details.


There is a shortcut for retrieving all tuples of a relation variable
-- the :meth:`~RelVar.all` method; it's equivalent to
``where('true')``::

   rv.Post.all().get()


get()
~~~~~

:meth:`~Selection.get` accepts an optional object describing the way
to retrieve tuples. It could define:

* an ordering of tuples;
* attributes to retrieve;
* a number of tuples to skip before starting to return tuples;
* a maximum number of tuples to return.

The following query returns authors and titles of all posts. Resulting
tuples are ordered by author; tuples with the same author are ordered
by title::

   rv.Post.all().get({only: ['author', 'title'], by: ['author', 'title']})

This query returns at most 42 posts in descending order by id omitting
the first 15 posts::

   rv.Post.all().get({by: '-id', start: 15, length: 42})

   
getOne()
~~~~~~~~

Some queries should return one and only one tuple. For such cases the
:meth:`getOne` method is useful: it performs a query and returns an
object representing the resulting tuple. If the query has returned no
tuples, a ``DoesNotExist`` error is thrown; if the query has returned
more than one tuple, a ``MultipleTuplesReturned`` error is
thrown. These errors are properties of the corresponding
:class:`RelVar` object.

For example::

   try {
     var post = rv.Post.where('id == $', id).getOne();
   } catch (error) {
     if (!(error instanceof rv.Post.DoesNotExist)) throw error;
     // There is no such post
   }

.. note::

   If you haven't caught a ``DoesNotExist`` error, Akshell will
   display a 404 error page to the user, which is the desired behavior
   for the majority of cases.

   
Updating
--------

Changing values of already existing tuples of a relation variable is
called :dfn:`updating`. As well as querying, it's a two-step process:

* first, you define what tuples you'd like to update via the already
  familiar :meth:`~RelVar.where` method of a :class:`RelVar` object;

* then, via the :meth:`~Selection.update` method you define
  expressions for calculation of new attribute values.

For example, the following code adds a signature to all Bob's posts::

   rv.Post.where('author == $', 'Bob').update({text: 'text + $'}, '\n--\nBob')

This code changes posts with empty texts::

   rv.Post.where('!text').update({
                                   title: 'title + $1',
                                   text: '$2'
                                 },
                                 ' (empty)',
                                 'subj')

Sometimes it's necessary to set attributes of some tuples to given
values, i.e., perform an update with constant expressions. To
facilitate this task, the :meth:`~Selection.set` shortcut method is
provided; it accepts an object mapping attribute names to their
values.

For example, this sets texts of all Bob's comments::

   rv.Comment.where('author == $', 'Bob').set({text: '<censored>'})
                        
   
Deleting
--------

I doubt you'll be surprised to know that deleting of tuples is also a
two-step process:

* first, you define what tuples to delete;
* then, you call the :meth:`~Selection.del` method.

For example, this code deletes all Bob's posts::

   rv.Post.where('author == $', 'Bob').del()

It will throw a :exc:`ConstraintError` if at least one of Bob's posts
has comments; so it'd be reasonable to delete these comments
beforehand::

   rv.Comment.where('post->author == $', 'Bob').del();
   rv.Post.where('author == $', 'Bob').del();


File Storage
============

A file storage is intended for storing unstructured data in files
grouped under directories. It has the same semantics as a common file
system on your local hard drive; so in this section you don't have to
learn anything.

File and directory names could contain any Unicode symbol except
``'\0'``, ``'\n'``, and ``'/'``; path separator is the slash
(``'/'``).

Here is an example of file storage usage:

   >>> fs.list('').forEach(fs.remove)
   >>> fs.write('greeting', 'Hello, world!')
   >>> fs.createDir('dir')
   >>> fs.createDir('dir/subdir')
   >>> fs.write('dir/subdir/answer', 42)
   >>> repr(fs.list(''))
   ["dir", "greeting"]
   >>> repr(fs.list('dir/subdir'))
   ["answer"]
   >>> fs.read('greeting')
   Hello, world!
   >>> fs.isFile('greeting')
   true
   >>> fs.isFile('dir/subdir')
   false
   >>> fs.isDir('dir/subdir')
   true
   >>> fs.exists('dir/subdir/answer')
   true
   >>> fs.exists('no-such-entry')
   false
   >>> fs.isFile('no-such-entry')
   false
   >>> fs.read('no-such-entry')
   ak.NoSuchEntryError: ...

See the :doc:`file storage API reference </ref/core/fs>` for details.
   

Quotas
======

Akshell sets quotas on code storage, database, and file storage sizes:

===================  ============  ========  ============
Application Version  Code storage  Database  File storage
===================  ============  ========  ============
Release              10M           32M       32M
Spot                 10M           2M        2M
===================  ============  ========  ============

If your application needs more, just write me to support@akshell.com.
