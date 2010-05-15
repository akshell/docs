
================
Database Goodies
================

The `db.js`_ file alters the :func:`db.create` function to support a
human-friendly syntax and adds the :meth:`getOne` :class:`Selection`
method. Facilitation of database management is the goal of this
module.


.. _human_friendly_db_create:

Human-Friendly db.create()
==========================

The human-friendly :func:`db.create` interface accepts string
definitions of relation variable attribute :ref:`types<types>` and
:ref:`constraints<constraints>`. This interface should be preferred
over the low-level one for defining relation variables by hand because
it's more brief and expressive. The low-level interface is more
convenient for machine generation of relation variables.

.. _db.js: http://www.akshell.com/apps/ak/code/0.1/db.js

The human-friendly syntax is:

.. function:: db.create(header[, constrs...])
   :noindex:

   The *header* object should map attribute names to ``string``
   attribute descriptions, *constrs* should be ``string`` constraint
   descriptions. The syntax of these descriptions is described below.


Attribute Description
---------------------

An attribute description is a ``string`` consisted of a type name and
(optionally) type constraints separated by spaces (the order is
unimportant).

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

If the ``integer`` or ``serial`` constraint is specified, the type
name can be omitted -- it defaults to ``number``.


Constraint Description
----------------------

A constraint description is a ``string`` of one of the forms:

* :samp:`check ({expr})`;
* :samp:`unique [{attrNames}...]`;
* :samp:`[{referencingAttrNames}...] -> {referencedRelVarName}[{referencedAttrNames}...]`.


Example
-------

A data model of an application for hotel management could look like this::

   db.create('Room',
             {
               floor: 'integer',
               number: 'integer',
               price: 'number check (price > 0)'
             },
             'unique [floor, number]');
             
   db.create('Client',
             {
               id: 'unique serial',
               name: 'string',
               discount: 'number check (discount >= 0 && discount < 1)'
             });
             
   db.create('Book',
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

   In real-world applications :term:`surrogate key` should be used
   instead of multiattribute foreign key.


getOne() Selection Method
=========================

.. method:: getOne(options={})

   Return the only tuple of this :class:`Selection`.

   If there are more than one tuple, ``getOne()`` throws a
   ``MultipleTuplesReturned`` exception. The
   ``MultipleTuplesReturned`` class is a property of the
   :class:`RelVar` instance.

   If there are no tuples, ``getOne()`` throws a ``DoesNotExist``
   exception. The ``DoesNotExist`` class is a property of the
   :class:`RelVar` instance. ::

      >>> db.create('X', {n: number})
      >>> rv.X.insert({n: 0})
      >>> rv.X.insert({n: 15})
      >>> rv.X.insert({n: 42})
      >>> repr(rv.X.where('n % 2 == 1').getOne())
      {n: 15}
      >>> rv.X.where('n % 2 == 0').getOne()
      ak.rv.X.MultipleTuplesReturned: ...
      >>> rv.X.where('n < 0').getOne()
      ak.rv.X.DoesNotExist: ...

.. exception:: TupleDoesNotExist

   A base class of all ``DoesNotExist`` exceptions of ``RelVar``
   instances (see above).

.. exception:: MultipleTuplesReturned

   A base class of all ``MultipleTuplesReturned`` exceptions of
   ``RelVar`` instances (see above).
