
==================
Database Interface
==================

The `db.js`_ file provides a convenient object-oriented interface to
the database. Facilitation of database management is the goal of this
module.

.. _db.js: http://www.akshell.com/apps/ak/code/0.2/db.js

.. data:: rv

   The ``rv`` object maps relation variable names to :class:`RelVar`
   objects representing correspondent relation variables. It's an
   entry point of all database interactions.

   
RelVar
======
   
.. class:: RelVar

   A ``RelVar`` object represents a relation variable. It cannot be
   constructed manually, but should be obtained as a property of the
   :data:`rv` object.

   .. attribute:: name

      The name of the relation variable.

   .. method:: exists()

      Return ``true`` if this relation variable exists, ``false``
      otherwise.

   .. method:: create(header[, constrs...])

      Create this relation variable. The *header* object should map
      attribute names to ``string`` attribute descriptions, *constrs*
      should be ``string`` constraint descriptions.

      An attribute description is a ``string`` consisted of a type
      name and (optionally) type constraints separated by spaces (the
      order is unimportant).
       
      The type names are:
       
      * ``number``;
      * ``string``;
      * ``bool``;
      * ``date``.
       
      The type constraints are:
       
      * ``integer``;
      * ``serial``;
      * ``unique``;
      * :samp:`-> {referencedRelVarName}.{referencedAttrName}`;
      * :samp:`check ({expr})`;
      * :samp:`default {value}`.
       
      If the ``integer`` or ``serial`` constraint is specified, the
      type name can be omitted -- it defaults to ``number``.
      
      A constraint description is a ``string`` of one of the forms:

      * :samp:`check ({expr})`;
      * :samp:`unique [{attrNames}...]`;
      * :samp:`[{referencingAttrNames}...] -> {referencedRelVarName}[{referencedAttrNames}...]`.

      A data model of an application for hotel management could look
      like this::

         rv.Room.create(
           {
             floor: 'integer',
             number: 'integer',
             price: 'number check (price > 0)'
           },
           'unique [floor, number]');
                   
         rv.Client.create(
           {
             id: 'unique serial',
             name: 'string',
             discount: 'number check (discount >= 0 && discount < 1)'
           });
                   
         rv.Book.create(
           {
             floor: 'integer',
             number: 'integer',
             client: 'integer -> Client.id',
             arrival: 'date',
             departure: 'date'
           },
           '[floor, number] -> Room[floor, number]',
           'check (arrival < departure)');
             
      .. note::

         In real-world applications :term:`surrogate key` should be
         preferred to multiattribute foreign key.
   
   .. method:: drop()

      Drop the relation variable; fail if there are references to it.

   .. method:: insert(values)

      Insert a tuple into the relation variable; return the inserted
      tuple. *values* must be an object mapping attribute names to
      attribute values. ::

         >>> rv.X.create({s: 'serial', d: 'number default 42'})
         >>> repr(rv.X.insert({s: 0, d: 0}))
         {d: 0, s: 0}
         >>> repr(rv.X.insert({d: 1}))
         {d: 1, s: 0}
         >>> repr(rv.X.insert({}))
         {d: 42, s: 1}

   .. method:: where(expr[, params...])
               where(values)

      Return a :class:`Selection` of tuples of the relation variable
      matching *expr* with *params*. In the second form *values* must
      be an object mapping attribute names to required attribute
      values, an expression is generated from this object.

      .. note::

         ``where()`` call does not perform any database interaction.

      ::

         >>> rv.X.create({n: 'number', b: 'bool', s: 'string'})
         >>> rv.X.insert({n: 0, b: false, s: 'zero'})
         >>> rv.X.insert({n: 42, b: true, s: 'the answer'})
         >>> repr(rv.X.where('n == $1 && b == $2', 42, true).get({attr: 's'}))
         ["the answer"]
         >>> repr(rv.X.where({n: 42, b: true}).get({attr: 's'})) // the same
         ["the answer"]
         
   .. method:: all()

      Return a :class:`Selection` of all tuples of the relation
      variable. It's equivalent to ``where('true')``.

   .. method:: getHeader()

      Return the header of the relation variable represented by an
      object mapping the attribute names to the attribute type
      names. ::

         >>> rv.X.create({n: 'number', s: 'string', b: 'bool', d: 'date'})
         >>> repr(rv.X.getHeader())
         {b: "bool", d: "date", n: "number", s: "string"}

   .. method:: getInteger()

      Return an array of the integer attribute names. ::

         >>> rv.X.create({i: 'integer', s: 'serial'})
         >>> repr(rv.X.getInteger())
         ["i", "s"]

   .. method:: getSerial()

      Return an array of the serial attribute names. ::

         >>> rv.X.create({i: 'integer', s: 'serial'})
         >>> repr(rv.X.getSerial())
         ["s"]

   .. method:: getUnique()

      Return an array of the unique keys represented by name
      arrays. ::

         >>> rv.X.create({a: 'unique number', b: 'number', c: 'number'},
                         'unique [b, c]')
         >>> repr(rv.X.getUnique())
         [["a"], ["b", "c"]]

   .. method:: getForeign()

      Return an array of the foreign keys represented by three-item
      arrays: the first item of such array is itself an array of
      referencing attribute names, the second is a name of a
      referenced relation variable, the third is an array of
      referenced attribute names. ::

         >>> rv.X.create({a: 'number', b: 'number'})
         >>> rv.Y.create({c: 'number', d: 'number'},
                         '[c, d] -> X[a, b]')
         >>> repr(rv.Y.getForeign())
         [[["c", "d"], "X", ["a", "b"]]]

   .. method:: getDefault()

      Return an object mapping the names of the attributes with
      default values to these values. ::

         >>> rv.X.create({n: 'number default 42', s: 'string default ""'})
         >>> repr(rv.X.getDefault())
         {n: 42, s: ""}

   .. method:: addAttrs(attrs)

      Add new attributes to the relation variable. Each attribute is
      described by a ``string`` of the form ``'type value'`` where
      ``type`` is ``number``, ``string``, ``bool``, ``date``, or
      ``integer`` and ``value`` is used to extend existing tuples. ::

         >>> rv.X.create({n: 'number'})
         >>> rv.X.insert({n: 0})
         >>> rv.X.insert({n: 1})
         >>> rv.X.addAttrs({i: 'integer 42', s: 'string "the answer"'})
         >>> repr(rv.X.all().get())
         [{n: 0, i: 42, s: "the answer"}, {n: 1, i: 42, s: "the answer"}]

   .. method:: dropAttrs(names...)

      Drop some attributes of the relation variable. ::

         >>> rv.X.create({n: 'number', s: 'string', b: 'bool'})
         >>> rv.X.dropAttrs('s', 'b')
         >>> repr(rv.X.getHeader())
         {n: "number"}
         >>> rv.X.dropAttrs('n')
         >>> repr(rv.X.getHeader())
         {}

   .. exception:: DoesNotExist

      Tuple was not found. The :meth:`~Selection.getOne`
      :class:`Selection` method throws this exception if a query
      returns an empty relation.
      
   .. exception:: IsAmbiguous

      Tuple is ambiguous. The :meth:`~Selection.getOne`
      :class:`Selection` method throws this exception if a query
      returns more than one tuple.
      
.. exception:: TupleDoesNotExist

   A base class of all ``DoesNotExist`` exceptions of ``RelVar``
   instances.

.. exception:: TupleIsAmbiguous

   A base class of all ``IsAmbiguous`` exceptions of
   ``RelVar`` instances.

   
Selection
=========
      
.. class:: Selection

   A ``Selection`` object represents a subset of relation variable
   tuples and provides methods for managing them.

   .. attribute:: name

      The name of the relation variable

   .. attribute:: expr

      The expression the selection tuples match to.

   .. attribute:: params

      The parameters of the expression.

   .. attribute:: relVar

      The :class:`RelVar` object of the selection.

   .. method:: get(options={} [, byParams...])

      Return an array of the tuples represented by objects mapping
      attribute names to attribute values. The *options* object can
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

         >>> rv.X.create({n: 'number', b: 'bool', s: 'string'})
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

   .. method:: getOne(options={} [, byParams...])

      Run the :meth:`~Selection.get` method with the given arguments
      and return the only tuple found. If there are no tuples, throw a
      :attr:`~Selection.relVar`.\ :exc:`~RelVar.DoesNotExist`
      exception; if there is more than one tuple, throw a
      :attr:`~Selection.relVar`.\ :exc:`IsAmbiguous` exception. ::
      
         >>> rv.X.create({n: 'number'})
         >>> rv.X.insert({n: 0})
         >>> rv.X.insert({n: 15})
         >>> rv.X.insert({n: 42})
         >>> repr(rv.X.where('n % 2 == 1').getOne())
         {n: 15}
         >>> rv.X.where('n % 2 == 0').getOne()
         rv.X.IsAmbiguous: ...
         >>> rv.X.where('n < 0').getOne()
         rv.X.DoesNotExist: ...
         
   .. method:: count()

      Return the number of the selection tuples not loading them from
      the database. Useful for big selections. ::

         >>> rv.X.create({n: 'number'})
         >>> for (var i = 0; i < 1000; ++i) rv.X.insert({n: i})
         >>> rv.X.where('n % $ == 0', 2).count()
         500

   .. method:: del()

      Delete the selection tuples from the relation variable; return
      the number of the deleted tuples. ::

         >>> rv.X.create({n: 'number'})
         >>> for (var i = 0; i < 10; ++i) rv.X.insert({n: i})
         >>> rv.X.where('n % $ == 0', 2).del()
         5
         >>> repr(rv.X.all().get({attr: 'n', by: 'n'}))
         [1, 3, 5, 7, 9]

   .. method:: update(exprs[, exprParams...])

      Update the selection tuples calculating new attribute values
      using *exprs*; return the number of the updated tuples. *exprs*
      is an object mapping attribute names to expressions;
      *exprParams* are parameters of these expressions. ::

         >>> rv.X.create({n: 'number', s: 'string'})
         >>> rv.X.insert({n: 0, s: 'zero'})
         >>> rv.X.insert({n: 1, s: 'one'})
         >>> rv.X.insert({n: 42, s: 'the answer'})
         >>> rv.X.where('n != 0').update({s: 's + $'}, '!')
         2
         >>> repr(rv.X.all().get({attr: 's', by: 's'}))
         ["one!", "the answer!", "zero"]

   .. method:: set(values)

      Set the selection tuple attributes to *values*; return the
      number of the changed tuples. *values* is an object mapping
      attribute names to attribute values. ::

         >>> rv.X.create({n: 'number', s: 'string'})
         >>> rv.X.insert({n: 0, s: 'zero'})
         >>> rv.X.insert({n: 1, s: 'one'})
         >>> rv.X.insert({n: 42, s: 'the answer'})
         >>> rv.X.where('n != 0').set({s: 's + $'})
         2
         >>> repr(rv.X.all().get({attr: 's', by: 's'}))
         ["s + $", "zero"]
