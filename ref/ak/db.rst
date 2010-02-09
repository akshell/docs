
================
Database Goodies
================

The `db.js`_ file alters the :func:`db.create` function to accept
human-friendly string definitions of relation variable attribute
:ref:`types<types>` and :ref:`constraints<constraints>`. This
interface should be preferred over the low-level one for defining
relation variables by hand because it's more brief and expressive. The
low-level interface is more convenient for machine generation of
relation variables.

.. _db.js: http://www.akshell.com/apps/ak/code/db.js

The human-friendly syntax is:

.. function:: db.create(header[, constrs...])
   :noindex:

   The *header* object should map attribute names to ``string``
   attribute descriptions, *constrs* should be ``string`` constraint
   descriptions. The syntax of these descriptions is described below.

   
Attribute Description
=====================

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
name could be omitted -- it defaults to ``number``.


Constraint Description
======================

A constraint description is a ``string`` of one of the forms:

* :samp:`check ({expr})`;
* :samp:`unique [{attrNames}...]`;
* :samp:`[{referencingAttrNames}...] -> {referencedRelVarName}[{referencedAttrNames}...]`.


Example
=======

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
