========
Database
========

This document describes Akshell database model, an API for managing an
app's database, and a language for querying it.


Relational Model
================

Akshell database management is based on the :term:`relational
model` which provides robust time-proved foundation for storing
structured data.


Concepts
--------

A basic database unit is a :dfn:`type`, i.e., a collection of
values. An :dfn:`attribute` is a pair of a name and a type; an
:dfn:`attribute value` is a specific valid value for the type of the
attribute. A :dfn:`tuple` is a set of uniquely named attributes with
their values. A :dfn:`header` of a tuple is a set of its attributes. A
:dfn:`relation` is a pair of a header and a body. A :dfn:`relation
header` is a tuple header; a :dfn:`relation body` is a set of tuples
with a header matching the relation header. A :dfn:`relation variable`
is a named relation. A :dfn:`relational database` is a set of relation
variables with unique names.


Example
-------

Let me sweeten this definition soup by an example. Suppose you are
writing a blog application; you'll need to define a data model for
it. As the users of your application will write posts and comment
posts of each other, you'll need to have two relation variables:
``Post`` and ``Comment``. The header of the relation of the ``Post``
variable will consist of the ``id:'serial'``, ``author:'string'``, and
``text:'string'`` attributes (``id`` is a unique identifier of a post
also known as a :term:`surrogate key`). The header of the relation of
the ``Comment`` variable will consist of the ``post:'integer'``,
``author:'string'``, and ``text:'string'`` attributes.

.. note::

   ``Post`` and ``Comment`` are relation variables, their values are
   relations. For brevity I'll mean by relation variable header and
   body the header and the body of the value of the relation variable.

Initially both relation variables will have an empty body. When the
user Bob writes his first post, the tuple will be added to the body of
``Post``::

   {id: 0, author: 'Bob', text: 'Hello world!'}

When Ann replies him, the tuple will be added to the body of
``Comment``::

   {post: 0, author: 'Ann', text: 'Hi Bob!'}


Relations as Tables
-------------------

A relation can be imagined as a table with a header matching the
relation header and rows containing the tuple attribute values. If Bob
replies Ann: "Hi Ann!" and she writes in her blog: "Hey, Bob is
onboard," the ``Post`` relation variable will have the following
representation:

+----+--------+---------------------+
| id | author | text                |
+====+========+=====================+
|  0 | Bob    | Hello world!        |
+----+--------+---------------------+
|  1 | Ann    | Hey, Bob is onboard |
+----+--------+---------------------+

And ``Comment`` will have the following:

+------+--------+---------+
| post | author | text    |
+======+========+=========+
|    0 | Ann    | Hi Bob! |
+------+--------+---------+
|    0 | Bob    | Hi Ann! |
+------+--------+---------+

The two main caveats of representing relations as tables are:

* table rows are ordered but relation tuples are not; and
* a table can have duplicate rows but relation tuples are unique.

A relation should be considered as a set of statements about some
subject. In the example these statements are "Bob's written 'Hello
world' in post 0," "Ann's replied 'Hi Bob!' to post 0," etc. The order
of statements doesn't matter; repeating the same statement twice
doesn't add truth to it.

This approach forbids Ann to reply "Hi Bob!" to post 0 again which is
unnatural for blogs. Addition of a unique serial attribute to
``Comment`` will solve the problem:

+----+------+--------+-----------------------+
| id | post | author | text                  |
+====+======+========+=======================+
|  0 |    0 | Ann    | Hi Bob!               |
+----+------+--------+-----------------------+
|  1 |    0 | Bob    | Hi Ann!               |
+----+------+--------+-----------------------+
|  2 |    0 | Ann    | Hi Bob!               |
+----+------+--------+-----------------------+
|  3 |    0 | Ann    | Sorry for double post |
+----+------+--------+-----------------------+


.. _constraints:

Constraints
-----------

As a relation is a set of statements, a database is a knowledge about
a domain of an application. Knowledge makes sense only if it's
consistent; in a relational database consistency is maintained using
:dfn:`constraints`. Akshell supports three types of constraints:

Unique constraint
   A set of attributes which must have unique values across all tuples
   of the relation variable body. The ``[id]`` or ``[author, text]``
   attributes can be unique constraints of the ``Post`` variable.

Foreign key constraint
   A reference from the relation variable to a unique key in another or
   the same relation variable. Formally a foreign key is a referencing
   variable, a subset of its attributes, a referenced variable, and
   its unique key such that for all tuples in the referencing variable
   body projected over the referencing attributes, there must exist an
   equal tuple in the referenced variable body projected over the
   referenced attributes. ``post`` is a reference from the ``Comment``
   relation variable to the ``id`` key of the ``Post`` relation
   variable.

Check constraint
   An expression which must evaluate to ``true`` for each tuple of the
   relation variable body. ``(text != '')`` can be a check constraint of
   the ``Post`` variable.


Database Management
===================

Types
-----

Akshell database supports eight types:

``'number'``
   The double precision number type.

``'integer'``
   The integer number type.

``'serial'``
   The integer number type with values generated from a sequence 0, 1,
   2, etc.

``'string'``
   The string type.

``'boolean'``
   The boolean type.

``'date'``
   The date type; represented by ``Date`` objects.

``'json'``
   The JSON type; can hold any serializable JavaScript object.

``'binary'``
   The binary type; represented by :class:`Binary` objects.


Functions
---------

These functions constitute the low-level database interface. The
``ak`` library offers a :doc:`convenient object-oriented wrapper
<../ak/rv>` of it.

.. function:: create(name, header, [uniqueKeys, foreignKeys, checks])

   Create a relation variable. *header* is an object mapping attribute
   names to their types; attributes with default values are defined by
   ``[type, value]`` pairs. *uniqueKeys*, *foreignKeys*, and *checks*
   are arrays with constraint definitions. ::

      >>> create(
            'Post',
            {
              id: 'serial',
              author: 'string',
              text: 'string'
            },
            [['id'], ['author', 'text']])
      >>> create(
            'Comment',
            {
              id: 'serial',
              post: 'integer',
              author: 'string',
              text: 'string',
            },
            [['id']],
            [[['post'], 'Post', ['id']]],
            ['text != "+1"'])

.. function:: drop(names)

   Drop relation variables. Drop fails if any of them is referenced by
   a variable not being dropped. ::

      >>> create('X', {u: 'number'})
      >>> create('Y', {f: 'number'}, [], [[['f'], 'X', ['u']]])
      >>> drop(['X'])
      RelVarDependencyError: ...
      >>> drop(['X', 'Y'])
      undefined

.. function:: dropAll()

   Drop all relation variables.

.. function:: list()

   Return a sorted array of relation variable names. ::

      >>> create('X', {})
      >>> create('Y', {})
      >>> repr(list())
      ["X", "Y"]
      >>> dropAll()
      >>> repr(list())
      []

.. function:: query(query, queryParams=[], by=[], byParams=[], start=0[, length])

   Perform a database query; return a relation represented by an array
   of tuples, each tuple being an object mapping attribute names to
   their values. The function accepts the following arguments:

   *query*
      A query string; see :ref:`Query Language <query_language>` for
      details.

   *queryParams*
      An array of query parameters.

   *by*
      An expression or an array of expressions to order resulting
      tuples. Order is ascending, to get descending order use
      ``-expr``.

   *byParams*
      An array of *by* expression parameters.


   *start*
      A number of tuples to skip before starting to return tuples.

   *length*
      A maximum number of tuples to return.

   .. warning::

      If *by* option is not specified, the order of returned tuples is
      undefined. Using *start* or *length* without *by* is discouraged
      because there is no guarantee which tuples would be returned.

   ::

      >>> create('X', {n: 'number'})
      >>> for (var i = 0; i < 6; ++i) insert('X', {n: i});
      >>> repr(query('X', [], '-n'))
      [{n: 5}, {n: 4}, {n: 3}, {n: 2}, {n: 1}, {n: 0}]
      >>> repr(query('X', [], 'n', [], 2, 3))
      [{n: 2}, {n: 3}, {n: 4}]
      >>> repr(query('X where n < $', [4], 'n'))
      [{n: 0}, {n: 1}, {n: 2}, {n: 3}]
      >>> repr(query('X', [], ['n % $', 'n'], [3]))
      [{n: 0}, {n: 3}, {n: 1}, {n: 4}, {n: 2}, {n: 5}]

.. function:: count(query, params=[])

   Return a number of tuples matching *query* not loading them. Useful
   for big relations. ::

      >>> create('X', {n: 'number'})
      >>> for (var i = 0; i < 1000; ++i) insert('X', {n: i});
      >>> count('X')
      1000
      >>> count('X where n % $1 == $2', [4, 1])
      250

.. function:: rollback()

   Roll back the :term:`transaction` of the current request. See
   :doc:`../ak/request` for details.


Exceptions
----------

.. exception:: DBError

   A base class of all database exceptions.

.. exception:: RelVarExistsError

   Relation variable already exists.

.. exception:: NoSuchRelVarError

   Relation variable doesn't exist.

.. exception:: ConstraintError

   Database constraint violation.

.. exception:: QueryError

   Incorrect database query.

.. exception:: AttrExistsError

   Attribute already exists.

.. exception:: NoSuchAttrError

   Relation variable attribute doesn't exist.

.. exception:: DependencyError

   Relation variable cannot be dropped because other variable depends
   on it.


.. _query_language:

Query Language
==============

The query language is an implementation of the `tuple relational
calculus`__; it's designed to be a simple yet powerful database
querying tool naturally embedded into JavaScript.

__ http://en.wikipedia.org/wiki/Tuple_relational_calculus


Example
-------

Let me define a familiar blog database. All example queries could be
performed on it using the :func:`query` function. ::

   >>> create(
         'Post',
         {
           id: 'serial',
           author: 'string',
           text: 'string'
          },
          [['id']])
   >>> create(
         'Comment',
         {
           id: 'serial',
           post: 'integer',
           author: 'string',
           text: 'string'
         },
         [['id']],
         [[['post'], 'Post', ['id']]])
   >>> insert('Post', {author: 'Bob', text: 'Hello, world!'})
   >>> insert('Comment', {post: 0, author: 'Ann', text: 'Hi, Bob!'})
   >>> insert('Comment', {post: 0, author: 'Bob', text: 'Hi, Ann!'})
   >>> insert('Post', {author: 'Ann', text: 'Hey, Bob is onboard'})


Range Variables
---------------

The basic concept of the language is :dfn:`range variable`. A range
variable is a named variable ranging over a relation, i.e., its values
are the tuples of the relation. Range variables are declared by the
``for`` construction and by the ``forsome`` and ``forall``
expressions.

The relations of Bob's posts and posts commented by Bob could be
retrieved by the queries::

   for (p in Post) p where p.author == "Bob"

   for (p in Post)
     p where forsome (c in Comment)
       c.post == p.id && c.author == "Bob"

Range variables can also be specified implicitly: an undeclared range
variable with a name of an existing relation variable ranges over the
value of this relation variable. Using implicit declarations the
previous examples could be rewritten::

   Post where Post.author == "Bob"

   Post where forsome (Comment)
     Comment.post == Post.id && Comment.author == "Bob"


Prototype Tuples
----------------

Range variables form a :dfn:`prototype tuple` describing resulting
relation tuples as a whole. In the previous example ``Post`` was a
prototype tuple, i.e., all ``Post`` attributes were included in the
result. There are two classes of prototype tuples: simple and complex.

* :dfn:`Simple` prototype tuples are formed by one range variable.

  - To include all range variable attributes just use the name of the
    range variable::

       Post

  - To include specific attributes use a square bracket notation::

       Post[author, text]

  - To include one attribute use a dot notation::

       Post.id

* :dfn:`Complex` prototype tuples are formed by any number of range
  variables and represented by a prototype list enclosed by the curly
  brackets. Each prototype is either a simple prototype or a named
  expression. The latter has the form ``name: expr``. The following
  query returns the relation of name pairs such that for each pair
  there exists a post written by ``author`` and commented by
  ``commenter``::

     {Post.author, commenter: Comment.author}
       where Comment.post == Post.id

  Complex prototype can have no range variables at all. For example,
  this query returns a single tuple::

     {n: 42, s: "the answer"}

Relations with the same header can form a :dfn:`union` relation, i.e.,
a relation consisting of all their tuples. In the query language this
operation is performed by the ``union`` construction. All post and
comment texts could be retrieved by::

   union(Post.text, Comment.text)


.. _expressions:

Expressions
-----------

Tuple selection expression is specified in the optional ``where``
construction after a prototype tuple. The expression syntax and
semantics mimics JavaScript whenever possible but the static nature of
the database makes the strict correspondence impossible: query
language operator return type must depend only on operand types, not
their values. Supported operators are:

* The :dfn:`quantifier operators` ``forsome`` and ``forall`` have
  boolean values based on results of subqueries: ``forsome`` returns
  ``true`` if and only if a tuple matching its expression exists in
  its relation; ``forall`` returns true if and only if all tuples of
  its relation match its expression. The following queries return the
  posts commented by the post author and the posts without empty
  comments respectively::

     Post where forsome (Comment)
       Comment.post == Post.id && Comment.author == Post.author

     Post where forall (Comment)
       Comment.post != Post.id || Comment.text

* The :dfn:`attribute operator` ``.`` returns a value of a range
  variable attribute. If there is only one range variable in the
  prototype, its name and a dot could be omitted. These queries are
  equivalent::

     Post where Post.author == "Bob"

     Post where author == "Bob"

  Subexpressions of one-relation ``forsome`` and ``forall``
  expressions also have a default range variable. Their example could
  be rewritten::

     Post where forsome (Comment)
       post == Post.id && author == Post.author

     Post where forall (Comment)
       post != Post.id || text

* The :dfn:`reference operator` ``->`` provides a convenient access to
  attributes of a referenced relation variable. It can be used only on
  referencing attribute(s) of a relation variable. Multiple attributes
  are specified using square bracket syntax. The following queries
  return the comments of Bob's posts and the post and comment text
  pairs respectively::

     Comment where post->author == "Bob"

     {postText: Comment.post->text, commentText: Comment.text}

* The :dfn:`parameter operator` ``$n`` returns a value of *n*\ -th
  parameter where *n* is a positive integer. If *n* is not specified,
  it defaults to 1.

* The :dfn:`conditional operator` ``?:`` return type depends on types
  of the second and the third operands, it is

  - the type of the second and the third operands if they are the same,
  - ``string`` if the second or the third operand is a ``string``,
  - ``number`` otherwise.

* The :dfn:`logical operators` ``||``, ``&&``, and ``!`` always return
  ``boolean``.

* The :dfn:`comparison operators` ``==``, ``!=``, ``<=``, ``>=``,
  ``<``, and ``>`` always return ``boolean``; operands are coerced to
  ``number`` if they have different types.

* The :dfn:`addition operator` ``+`` performs the string concatenation
  if at least one of its arguments is a ``string`` and the numerical
  addition otherwise.

* The :dfn:`arithmetic operators` ``*``, ``/``, ``%``, binary ``-``,
  unary ``-``, and unary ``+`` always return ``number``; their
  semantics mimics JavaScript.


Operator Precedence
-------------------

The operator precedence in ascending order:

1.  ``forsome forall``
2.  ``?:``
3.  ``||``
4.  ``&&``
5.  ``== !=``
6.  ``<= >= < >``
7.  binary ``+ -``
8.  ``* / %``
9.  unary ``+ - !``
10. ``$ . ->``


Full Grammar
------------

The full grammar of the query language in the `EBNF`__ form:

__ http://en.wikipedia.org/wiki/Ebnf

.. productionlist::
   relation: 'for' rangevars relation | union | select
   rangevars: '(' NAME  (',' NAME)* 'in' relation ')'
   union: 'union' '(' relation (',' relation)* ')'
   select: ('{' [prototype (',' prototype)*] '}' |
         :  field_expr | NAME)
         : ['where' expression]
   prototype: NAME ':' expr | field_expr | NAME
   expr: (('forsome' | 'forall')
       :  (rangevars | '(' NAME (',' NAME)* ')')
       :  expr) |
       : cond_expr
   cond_expr: log_or_expr ['?' expr ':' cond_expr]
   log_or_expr: log_and_expr ('||' log_and_expr)*
   log_and_expr: eq_expr ('&&' eq_expr)*
   eq_expr: rel_expr (('==' | '!=') rel_expr)*
   rel_expr: add_expr (('<=' | '>=' | '<' | '>') add_expr)*
   add_expr: mul_expr (('+' | '-') mul_expr)*
   mul_expr: unary_expr (('*' | '/' | '%') unary_expr)*
   unary_expr: ['+' | '-' | '!'] prim_expr
   prim_expr: NUMBER | STRING | BOOLEAN |
            : '(' expr ')' | '$' DIGIT* | field_expr | NAME
   field_expr: NAME ('.' NAME | '[' NAME (',' NAME)* ']')
             : ('->' (NAME | '[' NAME (',' NAME)* ']'))*
