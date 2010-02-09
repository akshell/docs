
============
Database API
============

This document describes Akshell database model, an API for managing an
application database, and a language for querying it.


Relational Model
================

Akshell database management is based on the :term:`relational
model` which provides robust time-proved foundation for storing
structured data in applications.


Concepts
--------

A basic database unit is a type. Akshell database types are borrowed
from JavaScript: they are :data:`number`, :data:`string`,
:data:`bool`, and :data:`date`. An :dfn:`attribute` is a pair of a
name and a type; an :dfn:`attribute value` is a specific valid value
for the type of the attribute. A :dfn:`tuple` is a set of uniquely
named attributes with their values. A :dfn:`header` of a tuple is a
set of its attributes. A :dfn:`relation` is a pair of a header and a
body. A :dfn:`relation header` is a tuple header; a :dfn:`relation
body` is a set of tuples with a header matching the relation header. A
:dfn:`relation variable` is a named relation. A :dfn:`relational
database` is a set of relation variables with unique names.


Example
-------

Let me sweeten this definition soup by an example. Suppose you are
writing a blog application; you'll need to define a data model for
it. As users of your application will write posts and comment posts of
each other, you'll need to have two relation variables: ``Post`` and
``Comment``. The header of the relation of the ``Post`` variable will
consist of the ``<"id", number>``, ``<"author", string>``, and
``<"text", string>`` attributes (``id`` is a unique identifier of a
post also known as a :term:`surrogate key`). The header of the
relation of the ``Comment`` variable will consist of the ``<"post",
number>``, ``<"author", string>``, and ``<"text", string>``
attributes.

.. note::

   ``Post`` and ``Comment`` are relation variables, their values are
   relations. For brevity I'll mean by relation variable header and
   body the header and the body of the value of the relation variable.

Initially both relation variables will have an empty body. When user
Bob writes his first post, the tuple will be added to the body of
``Post``::

   <"id", number, 0>, <"author", string, "Bob">, <"text", string, "Hello, world!">

When Ann replies him, the tuple will be added to the body of
``Comment``::

   <"post", number, 0>, <"author", string, "Ann">, <"text", string, "Hi, Bob!">

   
Relations as Tables
-------------------
   
A relation can be imagined as a table with a header matching the
relation header and rows containing the tuple attribute values. If Bob
replies Ann: "Hi, Ann!" and she writes in her blog: "Hey, Bob is
onboard," the ``Post`` relation variable will have the following
representation:

+----+--------+---------------------+
| id | author | text                |
+====+========+=====================+
|  0 | Bob    | Hello, world!       |
+----+--------+---------------------+
|  1 | Ann    | Hey, Bob is onboard |
+----+--------+---------------------+

And ``Comment`` will have the following:

+------+--------+----------+
| post | author | text     |
+======+========+==========+
|    0 | Ann    | Hi, Bob! |
+------+--------+----------+
|    0 | Bob    | Hi, Ann! |
+------+--------+----------+

The two main caveats of representing relations as tables are:

* table rows are ordered but relation tuples are not; and
* a table could have duplicate rows but relation tuples are unique.

A relation should be considered as a set of statements about some
subject. In the example these statements are "Bob's written 'Hello,
world' in post 0," "Ann's replied 'Hi, Bob!' to post 0," etc. The
order of statements doesn't matter; repeating the same statement twice
doesn't add truth to it.

This approach forbids Ann to reply "Hi, Bob!" to post 0 again which is
unnatural for blogs. Addition of a unique identifier attribute to
``Comment`` will solve the problem:

+----+------+--------+-----------------------+
| id | post | author | text                  |
+====+======+========+=======================+
|  0 |    0 | Ann    | Hi, Bob!              |
+----+------+--------+-----------------------+
|  1 |    0 | Bob    | Hi, Ann!              |
+----+------+--------+-----------------------+
|  2 |    0 | Ann    | Hi, Bob!              |
+----+------+--------+-----------------------+
|  3 |    0 | Ann    | Sorry for double post |
+----+------+--------+-----------------------+


Constraints
-----------

As a relation is a set of statements, a database is a knowledge about
a domain of an application. Knowledge makes sense only if it's
consistent; in a relational database consistency is maintained using
:dfn:`constraints`. Akshell supports three types of constraints:

Unique constraint
   a set of attributes which must have unique values across all tuples
   of the relation variable body. The ``[id]`` or ``[author, text]``
   attributes could be unique constraints of the ``Post`` variable.

Foreign key constraint
   a reference from the relation variable to a unique key in another or
   the same relation variable. Formally a foreign key is a referencing
   variable, a subset of its attributes, a referenced variable, and
   its unique key such that for all tuples in the referencing variable
   body projected over the referencing attributes, there must exist an
   equal tuple in the referenced variable body projected over the
   referenced attributes. ``post`` is a reference from the ``Comment``
   relation variable to the ``id`` key of the ``Post`` relation
   variable.

Check constraint
   an expression which must evaluate to ``true`` for each tuple of the
   relation variable body. ``text`` (``text`` is not empty) or ``id
   % 1 == 0`` (``id`` is integer) could be check constraints of
   the ``Post`` variable.

   
Database Management
===================

Types
-----

Four Akshell database types are represented by properties of the
``ak`` module; they are instances of the :class:`Type` class.

.. data:: number

   The double precision number type. Could be constrained to
   :meth:`integer<Type.integer>` values or made
   :meth:`serial<Type.serial>`.

.. data:: string

   The string type.

.. data:: bool

   The boolean type.

.. data:: date

   The date type storing JavaScript ``Date`` objects.

.. class:: Type

   The class of type objects. Provides methods returning modified
   type. Could not be instantiated manually, use predefined type
   objects instead.

   .. method:: integer()

      Return a type constrained to integer values; applicable only to
      :data:`number`.

   .. method:: serial()

      Return a serial-generated type constrained to integer values;
      applicable only to :data:`number`. ::

         >>> db.create('X', {s: number.serial()})
         >>> repr(rv.X.insert({}))
         {s: 0}
         >>> repr(rv.X.insert({}))
         {s: 1}
         >>> repr(rv.X.insert({s: 42}))
         {s: 42}
         >>> repr(rv.X.insert({}))
         {s: 2}

   .. method:: unique()

      Return a type with a unique constraint. ::

         >>> db.create('X', {n: number.unique(), s: string})
         >>> rv.X.insert({n: 42, s: 'the answer'})
         >>> rv.X.insert({n: 42, s: 'forty two'})
         ak.ConstraintError: ...

   .. method:: foreign(refRelVarName, refAttrName)

      Return a type with a foreign key constraint, i.e., a reference
      to a unique attribute in another or the same relation variable.

         >>> db.create('X', {u: number}) // u is unique because it's lonely
         >>> db.create('Y', {f: number.foreign('X', 'u')})
         >>> rv.X.insert({u: 0})
         >>> rv.Y.insert({f: 0})
         >>> rv.Y.insert({f: 42})
         ak.ConstraintError: ...

   .. method:: check(expr)

      Return a type with a check constraint, i.e., *expr* must
      evaluate to ``true`` for each tuple of a relation with an
      attribute of this type. ::

         >>> db.create('X', {n: number.check('n > 0')})
         >>> rv.X.insert({n: -1})
         ak.ConstraintError: ...
   
   .. method:: default_(value)

      Return a type with a default value attached.

         >>> db.create('X', {n: number.default_(42)})
         >>> repr(rv.X.insert({}))
         {n: 42}


:mod:`db` Module
-------------------

.. module:: db

Low-level database management functions are properties of the
:mod:`db` module.

.. function:: create(name, header, constrs={})

   Create a relation variable. *header* must be an object mapping
   attribute names to their types; The *constrs* object could have the
   fields:

   unique
      a list of unique keys represented by lists of attribute names;

   foreign
      a list of foreign keys represented by three-element lists: the
      first item of such list is itself a list of referencing
      attribute names, the second is a referenced relation variable
      name, and the third is a list of referenced attribute names;

   check
      a list of check expressions.

   .. note::

      One-attribute constraints could be specified by a
      :class:`~ak.Type` method or by *constr* argument. The first way
      should be preferred because it's more expressive.

   ::

      >>> db.create('Post',
                    {
                      id: number.serial(),
                      author: string,
                      text: string
                    },
                    {unique: [['author', 'text'], ['id']]})
      >>> db.create('Comment',
                    {
                      id: number.serial().unique(),
                      post: number.integer(),
                      author: string,
                      text: string,
                    },
                    {
                      foreign: [[['post'], 'Post', ['id']]],
                      check: ['text != "+1"']
                    })
                    
.. function:: drop(names...)

   Drop relation variables. Drop fails if any of them is referenced by
   a variable not being dropped.

      >>> db.create('X', {u: number})
      >>> db.create('Y', {f: number.foreign('X', 'u')})
      >>> db.drop('X')
      ak.RelVarDependencyError: ...
      >>> db.drop('X', 'Y')
      undefined
      

.. function:: query(query, options={})

   Perform a database query; return a relation represented by an array
   of tuples, each tuple being an object mapping attribute names to
   their values. *query* is a query string, see :ref:`Query Language
   <query_language>` for details; The *options* object could have the
   properties:

   params
      a list of query parameters;

   by
      an expression or a list of expressions to order resulting
      tuples (order is ascending, to get descending order use
      *-expr*);

   byParams
      a list of *by* expression parameters;

   start
      a number of tuples to skip before starting to return tuples; and

   length
      a maximum number of tuples to return.

   .. note::

      If *by* option is not specified, the order of returned tuples is
      undefined. Using *start* or *length* without *by* is discouraged
      because there is no guarantee which tuples would be returned.

   ::

      >>> db.create('X', {n: number})
      >>> for (var i = 0; i < 6; ++i) rv.X.insert({n: i});
      >>> repr(db.query('X', {by: '-n'}))
      [{n: 5}, {n: 4}, {n: 3}, {n: 2}, {n: 1}, {n: 0}]
      >>> repr(db.query('X', {by: 'n', start: 2, length: 3}))
      [{n: 2}, {n: 3}, {n: 4}]
      >>> repr(db.query('X where n < $', {params: [4], by: 'n'}))
      [{n: 0}, {n: 1}, {n: 2}, {n: 3}]
      >>> repr(db.query('X', {by: ['n % $', 'n'], byParams: [3]}))
      [{n: 0}, {n: 3}, {n: 1}, {n: 4}, {n: 2}, {n: 5}]

.. function:: count(query[, params... ])

   Return a number of tuples matching *query* not loading them. Useful
   for big relations. ::

      >>> db.create('X', {n: number})
      >>> for (var i = 0; i < 1000; ++i) rv.X.insert({n: i});
      >>> db.count('X')
      1000
      >>> db.count('X where n % $1 == $2', 4, 1)
      250

.. function:: rollback()

   Roll back the :term:`transaction` of the current request. See
   :doc:`request` for details.

   
Relation Variables
------------------

.. currentmodule:: None

The :class:`RelVar` and :class:`Selection` classes provide a
convenient access to the database management. You should prefer this
interface over the :mod:`db` module whenever possible because it's
more expressive.

.. data:: rv

   An object mapping relation variable names to :class:`RelVar`
   objects representing correspondent relation variables.

.. class:: RelVar

   A ``RelVar`` object represents a relation variable. It could not be
   constructed manually, but should be obtained as a property of the
   :data:`rv` object.

   .. attribute:: name

      The name of the relation variable.

   .. attribute:: header

      The header of the relation variable represented by an object mapping
      the attribute names to the attribute type names. ::

         >>> db.create('X', {n: number, s: string, b: bool, d: date})
         >>> repr(rv.X.header)
         {b: "bool", d: "date", n: "number", s: "string"}

   .. attribute:: integer

      A sorted array of the integer attribute names. ::

         >>> db.create('X', {i: number.integer(), s: number.serial()})
         >>> repr(rv.X.integer)
         ["i", "s"]

   .. attribute:: serial

      A sorted array of the serial attribute names. ::

         >>> db.create('X', {i: number.integer(), s: number.serial()})
         >>> repr(rv.X.serial)
         ["s"]

   .. attribute:: unique

      A sorted array of the unique keys represented by name arrays. A
      set of all attributes is always a unique key. ::

         >>> db.create('X',
                       {a: number.unique(), b: number, c: number},
                       {unique: [['b', 'c']]})
         >>> repr(rv.X.unique)
         [["a"], ["a", "b", "c"], ["b", "c"]]

   .. attribute:: foreign

      A sorted array of the foreign keys represented by three-item
      arrays: the first item of such array is itself an array of
      referencing attribute names, the second is a name of a
      referenced relation variable, the third is an array of
      referenced attribute names. ::

         >>> db.create('X', {a: number, b: number})
         >>> db.create('Y',
                       {c: number, d: number},
                       {foreign: [[['c', 'd'], 'X', ['a', 'b']]]})
         >>> repr(rv.Y.foreign)
         [[["c", "d"], "X", ["a", "b"]]]

   .. attribute:: default_

      An object mapping the names of the attributes with default
      values to these values. ::

         >>> db.create('X', {n: number.default_(42), s: string.default_('')})
         >>> repr(rv.X.default_)
         {n: 42, s: ""}
      
   .. method:: drop()

      Drop the relation variable; fail if there are references to it.

   .. method:: insert(values)

      Insert a tuple into the relation variable; return the inserted
      tuple. *values* must be an object mapping attribute names to
      attribute values. ::

         >>> db.create('X', {s: number.serial(), d: number.default_(42)})
         >>> repr(rv.X.insert({s: 0, d: 0}))
         {d: 0, s: 0}
         >>> repr(rv.X.insert({d: 1}))
         {d: 1, s: 0}
         >>> repr(rv.X.insert({}))
         {d: 42, s: 1}

   .. method:: where(expr[, params... ])
               where(values)

      Return a :class:`Selection` of tuples of the relation variable
      matching *expr* with *params*. In the second form *values* must
      be an object mapping attribute names to required attribute
      values, an expression is generated from this object.

      .. note::

         ``where()`` call does not perform any database interaction.

      ::

         >>> db.create('X', {n: number, b: bool, s: string})
         >>> rv.X.insert({n: 0, b: false, s: 'zero'})
         >>> rv.X.insert({n: 42, b: true, s: 'the answer'})
         >>> repr(rv.X.where('n == $1 && b == $2', 42, true).get({attr: 's'}))
         ["the answer"]
         >>> repr(rv.X.where({n: 42, b: true}).get({attr: 's'})) // the same
         ["the answer"]
         
   .. method:: all()

      Return a :class:`Selection` of all tuples of the relation
      variable. It's equivalent to ``where(true)``.

.. class:: Selection

   A ``Selection`` object represents a subset of relation variable
   tuples and provides methods for managing them.

   .. attribute:: name

      The name of the relation variable

   .. attribute:: expr

      The expression the selection tuples match to.

   .. attribute:: params

      The parameters of the expression.

   .. attribute:: rv

      The :class:`RelVar` object of the selection.

   .. method:: get(options={} [, byParams... ])

      Return an array of the tuples represented by objects mapping
      attribute names to attribute values. The *options* object could
      have the properties:

      only
         a list of attribute names to fetch;

      attr
         a name of an attribute to fetch, if *attr* option is used,
         ``get()`` returns an array of attribute values;
         
      by
         an expression or a list of expressions to order resulting
         tuples;
    
      start
         a number of tuples to skip before starting to return tuples; and
    
      length
         a maximum number of tuples to return.
         
      *byParams* is a list of *by* expression parameters. See the
      corresponding :func:`db.query` options for details. Unless *by*
      option is specified the order of the returned tuples is
      undefined. ::

         >>> db.create('X', {n: number, b: bool, s: string})
         >>> rv.X.insert({n: 0, b: false, s: 'zero'})
         >>> rv.X.insert({n: 1, b: false, s: 'one'})
         >>> rv.X.insert({n: 42, b: true, s: 'the answer'})
         >>> repr(rv.X.all().get({by: 'n', start: 1, length: 1}))
         [{b: false, n: 1, s: "one"}]
         >>> repr(rv.X.all().get({attr: 'n', by: 'n * $'}, -1))
         [42, 1, 0]
         >>> repr(rv.X.where('!b').get({only: ['n', 's']})) // undefined order
         [{n: 0, s: "zero"}, {n: 1, s: "one"}]
         >>> repr(rv.X.all().get({attr: 'b', by: 'b'})) // tuples are unique
         [false, true]
         
   .. method:: count()

      Return a number of the selection tuples not loading them from the
      database. Useful for big selections. ::

         >>> db.create('X', {n: number})
         >>> for (var i = 0; i < 1000; ++i) rv.X.insert({n: i})
         >>> rv.X.where('n % $ == 0', 2).count()
         500

   .. method:: del()

      Delete the selection tuples from the relation variable; return a
      number of the deleted tuples. ::

         >>> db.create('X', {n: number})
         >>> for (var i = 0; i < 10; ++i) rv.X.insert({n: i})
         >>> rv.X.where('n % $ == 0', 2).del()
         5
         >>> repr(rv.X.all().get({attr: 'n', by: 'n'}))
         [1, 3, 5, 7, 9]

   .. method:: update(exprs[, exprParams... ])

      Update the selection tuples calculating new attribute values
      using *exprs*; return a number of the updated tuples. *exprs* is
      an object mapping attribute names to expressions; *exprParams*
      are parameters of these expressions. ::

         >>> db.create('X', {n: number, s: string})
         >>> rv.X.insert({n: 0, s: 'zero'})
         >>> rv.X.insert({n: 1, s: 'one'})
         >>> rv.X.insert({n: 42, s: 'the answer'})
         >>> rv.X.where('n != 0').update({s: 's + $'}, '!')
         2
         >>> repr(rv.X.all().get({attr: 's', by: 's'}))
         ["one!", "the answer!", "zero"]

   .. method:: set(values)

      Set the selection tuple attributes to *values*; return a number
      of the changed tuples. *values* is an object mapping attribute
      names to attribute values. ::

         >>> db.create('X', {n: number, s: string})
         >>> rv.X.insert({n: 0, s: 'zero'})
         >>> rv.X.insert({n: 1, s: 'one'})
         >>> rv.X.insert({n: 42, s: 'the answer'})
         >>> rv.X.where('n != 0').set({s: 's + $'})
         2
         >>> repr(rv.X.all().get({attr: 's', by: 's'}))
         ["s + $", "zero"]

         
.. _query_language:

Query Language
==============

The query language is an implementation of the :term:`tuple relational
calculus`; it's designed to be a simple yet powerful database querying
tool naturally embedded into JavaScript.


Example
-------

Let me define a familiar blog database. All example queries could be
performed on it using the :func:`db.query` function. ::

   >>> db.create('Post',
                 {
                   id: number.serial().unique(),
                   author: string,
                   text: string
                 })
   >>> db.create('Comment',
                 {
                   id: number.serial().unique(),
                   post: number.integer().foreign('Post', 'id'),
                   author: string,
                   text: string,
                 })
   >>> rv.Post.insert({author: 'Bob', text: 'Hello, world!'})
   >>> rv.Comment.insert({post: 0, author: 'Ann', text: 'Hi, Bob!'})
   >>> rv.Comment.insert({post: 0, author: 'Bob', text: 'Hi, Ann!'})
   >>> rv.Post.insert({author: 'Ann', text: 'Hey, Bob is onboard'})

   
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
     p where forsome (c in Comment) c.post == p.id && c.author == "Bob"

Range variables could also be specified implicitly: an undeclared range
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
  brackets. Each prototype could be a simple prototype or a named
  expression. The latter has the form ``name: expr``. The following
  query returns the relation of name pairs such that for each pair
  there exists a post written by ``author`` and commented by
  ``commenter``::

     {Post.author, commenter: Comment.author}
       where Comment.post == Post.id

  Complex prototype could have no range variables at all. For example,
  this query returns a single tuple::

     {n: 42, s: "the answer"}

Relations with the same header could form a :dfn:`union` relation,
i.e., a relation consisting of all their tuples. In the query language
this operation is performed by the ``union`` construction. All post
and comment texts could be retrieved by::

   union(Post.text, Comment.text)

   
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
  attributes of a referenced relation variable. It could be used only
  on referencing attribute(s) of a relation variable. Multiple
  attributes are specified using square bracket syntax. The following
  queries return the comments of Bob's posts and the post and comment
  text pairs respectively::

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
  ``bool``.

* The :dfn:`comparison operators` ``==``, ``!=``, ``<=``, ``>=``,
  ``<``, and ``>`` always return ``bool``; operands are coerced to
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
   select: ('{' [prototype (',' prototype)*] '}' | field_expr | NAME)
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
