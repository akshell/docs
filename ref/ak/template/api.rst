
============
Template API
============

This document explains the Akshell template system from a technical
perspective -- how it works and how to extend it. If you're just
looking for reference on the language syntax, see
:doc:`/guide/template`.


Concepts
========

A :dfn:`template` is a JavaScript string that is marked-up using the
Akshell template language. A template can contain :dfn:`block tags`
and :dfn:`template expressions`.

A :dfn:`block tag` is a symbol within a template that does something.

This definition is deliberately vague. For example, a block tag can
output content, serve as a control structure (an "if" statement or
"for" loop), or grab content from a database.

Block tags are surrounded by ``'{%'`` and ``'%}'``.

Example template with block tags:

.. code-block:: html+django

   {% if isLoggedIn %}Thanks for logging in!{% else %}Please log in.{% endif %}

A :dfn:`template expression` is a symbol within a template that
outputs a value.

Template expressions are surrounded by ``'{{'`` and ``'}}'``.

Example template with expressions:

.. code-block:: html+django

    My first name is {{ firstName }}. My last name is {{ lastName }}.

A :dfn:`context` is a plain JavaScript object mapping variable names
to variable values.

A template :dfn:`renders` a context by replacing the variable "holes"
with values from the context and executing all block tags.


Usage
=====

Usage of the template system is a two-step process:

* first, you compile the raw template code into a ``Template`` object;

* then, you call the ``render()`` method of the ``Template`` object
  with a given context.

.. class:: Template(string, env=template.env)

   A ``Template`` object is a compiled template
   representation. *string* is a raw template; *env* is a
   :ref:`template environment<template_environment>` object.

   The raw template is parsed only once -- when you instantiate
   ``Template``. From then on, it's stored internally as a "node"
   structure for performance. Even the parsing itself is quite fast:
   most of the parsing happens via a single call to a single short
   regular expression.

   .. method:: render(context={})

      Render the template in *context*; return the resulting
      ``string``.

   ::
      
      >>> (new Template('My name is {{ name }}.')).render({name: 'Anton'})
      My name is Anton.

.. exception:: TemplateSyntaxError

   A ``TemplateSyntaxError`` is thrown when a template has invalid
   syntax. ::

      >>> new Template('{{ }}')
      ak.TemplateSyntaxError: ...

      
Template Rendering
------------------

Context variable names must consist of any letter (A-Z, a-z), any
digit (0-9), an underscore, or a dot.

Dots have a special meaning in template rendering. A dot in a variable
name signifies :dfn:`lookup`: the engine gets the specified attribute
of the object and, if this attribute is a function, performs a method
call::

   >>> (new Template('{{ person.name.toUpperCase }}')).render(
         {person: {name: 'Bob'}})
   BOB

When lookup fails, an empty string is returned::

   >>> repr((new Template('{{ variable }}')).render())
   ""

Lookup calls methods without arguments. Exceptions thrown by methods
are propagated::

   >>> (new Template('{{ func }}')).render({func: function () { throw 42; }})
   Line 1, column 57
   Uncaught 42
   
.. warning::

   Do **not** use complex methods and methods with side effects in
   template expressions. Mixing presentation and application logic
   could produce tricky bugs.


Template Loading
----------------

Generally, you'll store templates in code files of your application
rather than using the low-level :class:`Template` API. In specific
situations templates could be stored in the database or in the file
storage. Akshell provides a single entry-point to the template
loading:

.. function:: getTemplate(name, env=template.env)

   Load a ``Template`` object from the location specified by the
   *name* argument. By default, load from the directory ``/templates``
   of the application code (*name* specifies a path relative to this
   directory). The default behavior could be overridden by a template
   environment.

The most common template use case is rendering HTML and returning it
to the user in a :class:`Response` object. The ``render()`` shortcut
function specifically targets this use case:
   
.. function:: render(name, context={}, status=http.OK[, headers])

   Load a template via the :func:`getTemplate` function; render it via
   the :meth:`~Template.render` ``Template`` method; return a
   :class:`Response` object containing the rendered template.


.. _template_customization:
   
Customization
=============

.. module:: template

The default behavior of the template system, default template
:doc:`tags<tags>` and :doc:`filters<filters>` should suite most use
cases. But if you need more, you can easily customize and extend it
using the ``template`` module.

.. _template_environment:

Template Environment
--------------------

A :dfn:`template environment` is an object defining a configuration of
the template engine. It must have three properties:

``filters``
   An object mapping template filter names to :class:`Filter` objects.

``tags``
   An object mapping template tag names to compilation functions.

``load``
   A template loader function; it should accept a ``string``
   template name and return a :class:`Template` object.

   For example, templates could be loaded from a relation variable::

      db.create('Template', {name: 'unique string', value: 'string'});

      ...

      template.env.load = function (name) {
        return rv.Template.where({name: name}).getOne().value;
      }
      
The default template engine configuration is represented by:
   
.. data:: env

   The default template environment object. Used by the
   :class:`Template` constructor and the :func:`getTemplate` function
   if the *env* argument is omitted.


.. _custom_filters:

Custom Filters
--------------

To create a custom filter you should write a filter function and
instantiate the :class:`Filter` class with it. A filter function
receives an input and (optionally) a filter argument; it should return
the output.


Wrap
~~~~

In order to perform proper :ref:`HTML escaping<html_escaping>`, safety
indicator must be transferred through filters along with the value. To
achieve this, the engine wraps each value to be inserted into a
template by the ``Wrap`` class.

.. class:: Wrap(raw, safe=false)

   A ``Wrap`` object represents a value to be inserted into a
   template.

   .. attribute:: raw

      The raw JavaScript value.

   .. attribute:: safe

      A flag indicating whether the value needs HTML escaping.

   .. method:: prepare(accept)

      Prepare the value for a filter. Return:

      * ``this`` if *accept* is ``'wrap'``;
      * ``raw + ''`` if *accept* is ``'string'``;
      * ``raw`` otherwise.

   .. method:: toString()

      Return a string representation of the value performing HTML
      escaping if needed. If the value is ``undefined``, return an
      empty string.

   ::

      >>> new template.Wrap('<>')
      &lt;&gt;
      >>> new template.Wrap('<>', true)
      <>
      >>> repr(new template.Wrap(undefined) + '')
      ""

      
Filter
~~~~~~
      
To create a custom filter instantiate the ``Filter`` class:
      
.. class:: Filter(func, traits={})

   *func* is a filter function; the *traits* object could have the
   following properties:

   ``accept``
      An argument for the :meth:`~Wrap.prepare` ``Wrap`` method
      to prepare a value and an argument for the filter function.

   ``safety``
      A determinant of return value safety. If the filter function
      returns a raw value, a :class:`Wrap` result object is created
      for it; its *safe* argument is:

      * ``true`` if ``safety`` is ``'always'``;
      * ``value.safe`` if ``safety`` is ``'value'``;
      * ``!arg || arg.safe`` if ``safety`` is ``'arg'``;
      * ``value.safe && (!arg || arg.safe)`` if ``safety`` is
        ``'both'``;
      * ``false`` otherwise.

      If the filter function returns a :class:`Wrap` instance,
      ``safety`` is irrelevant.

   .. method:: filter(value[, arg])

      Prepare *value* and *arg* using the ``accept`` trait; pass them
      to the filter function; return the result wrapping it, if
      needed, using the ``safety`` trait.
      
Filter functions should never throw an exception -- they should fail
silently returning an empty string or the original value, whichever is
more appropriate.


Examples
~~~~~~~~

A filter multiplying the value by the argument could look like this::

   var mulFilter = new template.Filter(
     function (value, arg) {
       var result = value.raw * arg.raw;
       return isNaN(result) ? value : result;
     },
     {safety: 'always', accept: 'wrap'});

If the multiplication succeeds, the result (number) is marked as safe;
otherwise the original value wrap is returned.

The :filter:`last` filter is implemented as::

    var lastFilter = new template.Filter(
      function (value) {
        return (typeof(value) == 'string' || ak.isList(value)
                ? value[value.length - 1]
                : value);
      },
      {safety: 'value'})

It accepts a raw value and returns the last item of the list; if the
value is not a list or a string, it returns the value itself. Value
safety is preserved by this filter.

You could make your filter available either by creating a new template
environment or by adding it to the catalog of default filters. For
example, the multiplication filter could be published as::

   template.env.filters.mul = mulFilter;

... and used:

   >>> (new Template('{{ x|mul:y }}')).render({x: 3, y: 5})
   15


.. _custom_tags:
      
Custom Tags
-----------

Tags are more complex than filters, because tags can do anything.

Above, this document explained that the template system works in a
two-step process: compiling and rendering. To define a custom template
tag, you should specify how the compilation works and how the
rendering works.

When the engine compiles a template, it splits the raw template text
into "nodes". Each node has a ``render()`` method. A compiled template
is, simply, a list of node objects. When you call ``render()`` on a
compiled template object, the template calls ``render()`` on each node
in its node list with the given context.  The results are all
concatenated together to form the output of the template.

Thus, to define a custom template tag, you should specify how the raw
template tag is converted into a node (the compilation function) and
what the node's ``render()`` method does.


Parser
~~~~~~

For each template tag the template parser encounters, it calls a
JavaScript function passing itself as an argument. This function is
responsible for returning a node object based on the contents of the
tag.

.. class:: Parser(string, store, env=template.env)

   The following parser attributes and methods provide compilation
   functions with an access to the template machinery.

   .. attribute:: env

      The template environment.

   .. attribute:: content

      The content of the current tag.
   
   .. method:: parse(until=[])

      Parse the template until one of the block tags specified in the
      *until* argument is encountered.

   .. method:: makeExpr(string)

      Parse a template expression and return an expression object with
      the ``resolve(context)`` method.

   .. method:: makeExprs(strings)

      Parse template expressions and return an ``Array`` of expression
      objects.

      
Node Objects
~~~~~~~~~~~~
      
A compilation function could create node objects as:

* a plain ``Object`` instance with a ``render`` property (arguments
  are passed through a closure)::

     return {render: function (context) { ... }};

* an instance of a specific node class (arguments are passed through
  the constructor)::

     return new SomeNode(...);

The second way is recommended for complex tags because it explicitly
separates parsing and rendering logic. For trivial cases, like the
following example, the first way is appropriate.

Let me write a template tag, ``{% context %}``, that displays the
rendering context (such tag could be useful for debugging)::

   template.env.tags.context = function () {
     return {
       render: function (context) {
         return ('<pre>\n' +
                 escapeHTML(JSON.stringify(context, null, 2)) +
                 '\n</pre>');
       }
     };
   }
   
``{% context %}`` output perfectly suites for embedding into HTML::

   >>> (new Template('{% context %}')).render({s: 'string', n: 42})
   <pre>
   {
     &quot;s&quot;: &quot;string&quot;,
     &quot;n&quot;: 42
   }
   </pre>


Arguments
~~~~~~~~~
   
Template tags could accept arguments. They could be cut from the
:attr:`~Parser.content` parser property via the function:
   
.. function:: smartSplit(text)

   Split text by white spaces regarding string constants and template
   expressions. ::

      >>> repr(template.smartSplit('1 2\t "a b c" value|some|filters'))
      ["1", "2", "\"a b c\"", "value|some|filters"]

Let me illustrate this function by a more complex example. The
following template tag, ``{% ul %}``, takes arbitrary number of
template expressions and renders itself into an unordered list,
``<ul>``. Note, that for this tag I create a node class because the
``{render: function () { ... }}`` approach would look cluttered here::

   var ULNode = Object.subclass(
     function (exprs) {
       this._exprs = exprs;
     },
     {
       render: function (context) {
         return ('<ul>\n' +
                 this._exprs.map(
                   function (expr) {
                     return '<li>' + expr.resolve(context) + '</li>';
                   }).join('\n') +
                 '\n</ul>');
       }
     });

   template.env.tags.ul = function (parser) {
     return new ULNode(
       parser.makeExprs(
         template.smartSplit(parser.content).slice(1)));
   }

Example usage::

   >>> (new Template('{% ul 42 "string constant" variable %}')).render(
         {variable: 'variable value'})
   <ul>
   <li>42</li>
   <li>string constant</li>
   <li>variable value</li>
   </ul>
