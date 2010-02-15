
=====================
Default Template Tags
=====================

.. highlightlang:: html+django

The default template tags are represented by the ``template.env.tags``
properties.


.. tag:: block

block
=====

Define a block that can be overridden by child templates. See
:ref:`template_inheritance` for more information.


.. tag:: csrfToken

csrfToken
=========

.. todo::

   Describe it somewhere and add a link here.

   
.. tag:: cycle

cycle
=====

Cycle among the given template expressions each time this tag is
encountered::

   {% for item in someList %}
     <tr class="{% cycle 'row1' 'row2' %}">
       ...
     </tr>
   {% endfor %}

Arbitrary template expressions could be used as ``cycle``
arguments. For example, if you have two template variables,
``rowvalue1`` and ``rowvalue2``, you can cycle between their
uppercased values like this::

   {% for item in someList %}
     <tr class="{% cycle rowvalue1|toUpperCase rowvalue2|toUpperCase %}">
       ...
     </tr>
   {% endfor %}

In some cases you might want to refer to the next value of a cycle
from outside of a loop. To do this, just give the ``{% cycle %}`` tag
a name, using "as", like this::

   {% cycle 'row1' 'row2' as rowcolors %}
   
From then on, you can insert the current value of the cycle wherever
you'd like in your template::

   <tr class="{% cycle rowcolors %}">...</tr>
   <tr class="{% cycle rowcolors %}">...</tr>

   
.. tag:: extends

extends
=======

Signal that this template extends a parent template, whose name is
passed as the only argument. The ``{% extends %}`` tag must be the
first tag of a template. See :ref:`template_inheritance` for details.


.. tag:: filter

filter
======

Filter contents of a tag through expression filters. Filters can be
piped through each other, and they can have arguments -- just like in
the expression syntax. ::

   {% filter forceEscape|toLowerCase %}
     This text will be HTML-escaped, and will appear in all lowercase.
   {% endfilter %}


.. tag:: firstOf
   
firstOf
=======

Output the first expression passed that is not ``false``. Output
nothing if all the passed expressions are ``false``.

Sample usage::

   {% firstOf var1 var2 var3 %}

This is equivalent to::

   {% if var1 %}
     {{ var1 }}
   {% else %}{% if var2 %}
     {{ var2 }}
   {% else %}{% if var3 %}
     {{ var3 }}
   {% endif %}{% endif %}{% endif %}


.. tag:: for

for
===

Loop over each item in an array-like object.  For example, to display
a list of athletes provided in ``athleteList``::

   <ul>
     {% for athlete in athleteList %}
       <li>{{ athlete.name }}</li>
     {% endfor %}
   </ul>

You can loop over a list in reverse by using ``{% for item in list
reversed %}``.

The for loop sets a number of variables available within the loop:

==========================  ================================================
Variable                    Description
==========================  ================================================
``forloop.counter``         The current iteration of the loop (1-indexed)
``forloop.counter0``        The current iteration of the loop (0-indexed)
``forloop.revcounter``      The number of iterations from the end of the
                            loop (1-indexed)
``forloop.revcounter0``     The number of iterations from the end of the
                            loop (0-indexed)
``forloop.first``           True if this is the first time through the loop
``forloop.last``            True if this is the last time through the loop
``forloop.parentloop``      For nested loops, this is the loop "above" the
                            current one
==========================  ================================================


for ... empty
-------------

The ``{% for %}`` tag can take an optional ``{% empty %}`` clause that
will be displayed if the given list is empty or could not be found::

   <ul>
     {% for athlete in athleteList %}
       <li>{{ athlete.name }}</li>
     {% empty %}
       <li>Sorry, no athlete in this list!</li>
     {% endfor %}
   </ul>

The above is equivalent to -- but shorter and cleaner than -- the
following::

   <ul>
     {% if athleteList %}
       {% for athlete in athleteList %}
         <li>{{ athlete.name }}</li>
       {% endfor %}
     {% else %}
       <li>Sorry, no athletes in this list.</li>
     {% endif %}
   </ul>


.. tag:: if

if
==

The ``{% if %}`` tag evaluates a condition, and if it is ``true``, the
contents of the block are output::

   {% if athleteList.length %}
     Number of athletes: {{ athleteList.length }}
   {% else %}
     No athletes.
   {% endif %}

In the above, if ``athleteList`` is not empty, the number of athletes
will be displayed by the ``{{ athleteList.length }}`` expression.

As you can see, the ``{% if %}`` tag can take an optional ``{% else
%}`` clause that will be displayed if the condition is ``false``.
   
A :dfn:`condition` could consist of constants and variables combined
by these JavaScript operators::

   || && == != === !== !

JavaScript operator precedence rules apply. The parentheses ``()``
could be used to explicitly define grouping. For example, the
following complex ``{% if %}`` tag::

   {% if a == b || c == d && e %}

... is equivalent to::

   {% if (a == b) || ((c == d) && e) %}


.. tag:: ifchanged
   
ifchanged
=========

Check if a value has changed from the last iteration of a loop.

The ``{% ifchanged %}`` block tag is used within a loop. It has two
possible uses:

1. Check own rendered content against the previous state and only
   display the content if it has changed. For example, this displays a
   list of days, only displaying the month if it changes::

      {% for date in days %}
        {% ifchanged %}<h3>{{ date.getMonth }}</h3>{% endifchanged %}
        <a href="{{ date.getMonth }}/{{ data.getDay }}/">{{ date.getDay }}</a>
      {% endfor %}
      
2. If given an expression, check whether that expression has
   changed. For example, the following shows the date every time it
   changes, but only shows the hour if both the hour and the date have
   changed::
   
      {% for date in days %}
        {% ifchanged date.getDate %} {{ date.getDate }} {% endifchanged %}
        {% ifchanged date.getHour date.getDate %}
          {{ date.getHour }}
        {% endifchanged %}
      {% endfor %}

The ``{% ifchanged %}`` tag can also take an optional ``{% else %}``
clause that will be displayed if the value has not changed::

   {% for match in matches %}
     <div style="background-color:
       {% ifchanged match.ballot_id %}
         {% cycle "red" "blue" %}
       {% else %}
         gray
       {% endifchanged %}
     ">{{ match }}</div>
   {% endfor %}


.. tag:: include
   
include
=======

Load a template and render it with the current context. This is a way
of "including" other templates within a template. The template name
could be an arbitrary expression.

This example includes the contents of the template ``"foo/bar.html"``::

   {% include "foo/bar.html" %}

This example includes the contents of the template whose name is
contained in the variable ``templateName``::

   {% include templateName %}

An included template is rendered with the context of the template
that's including it. This example produces the output ``"Hello,
John"``:

* Context: variable ``person`` is set to ``"John"``.
* Template::

     {% include "name-snippet.html" %}

* The ``name-snippet.html`` template::

     Hello, {{ person }}


.. tag:: spaceless
     
spaceless
=========

Remove white space between HTML tags. This includes tab characters and
newlines.

Example usage::

   {% spaceless %}
     <p>
       <a href="foo/">Foo</a>
     </p>
   {% endspaceless %}

This example would return the HTML::

   <p><a href="foo/">Foo</a></p>

Only space between *tags* is removed -- not space between tags and
text. In this example the space around ``Hello`` won't be stripped::

   {% spaceless %}
     <strong>
       Hello
     </strong>
   {% endspaceless %}


.. tag:: templateTag
   
templateTag
===========

Output one of the syntax characters used to compose template tags.

Since the template system has no concept of "escaping," to display one
of the bits used in template tags you must use the ``{% templateTag
%}`` tag.

The argument tells which template bit to output:

==================  =======
Argument            Outputs
==================  =======
``openBlock``       ``{%``
``closeBlock``      ``%}``
``openExpr``        ``{{``
``closeExpr``       ``}}``
``openBrace``       ``{``
``closeBrace``      ``}``
``openComment``     ``{#``
``closeComment``    ``#}``
==================  =======


.. tag:: url

url
===

.. todo::

   Refactor url.js and rewrite this section.

Returns an absolute URL (i.e., an URL without the domain name)
matching a given controller and optional parameters. This is a way to
output links without violating the DRY principle by having to
hard-code URLs in your templates::

   {% url someContoller#SomePage arg1 arg2 "string arg3" %}

The first argument is a controller name optionally followed by a page
name. Additional arguments will be used as positional arguments for
the URL dispatcher. See :doc:`../url` and :doc:`../rest` for details.

Suppose you have a ``ClientController`` class with an ``Orders`` page
and the following url routes mapping :samp:`/clients/{id}/` to the
main client page and :samp:`/clients/{id}/orders/` to the ``Orders``
page::

   ['clients/',
     [
       ClientController,
       ['orders/', ClientController.page('Orders')]
     ]
   ]

In a template you can create links to these pages like this::

   {% url ClientController client.id %}
   {% url ClientController#Orders client.id %}

If ``client.id`` is ``123``, they will output::

   /clients/123/
   /clients/123/orders/

Note that if the URL you're reversing doesn't exist, you'll get an
:exc:`ReverseError` exception raised, which will cause your site to
display an error page.
   
If you'd like to retrieve a URL without displaying it, you can use a
slightly different call::

   {% url someController arg1 arg2 as theURL %}

   <a href="{{ theURL }}">I'm linking to {{ theURL }}</a>

This ``{% url ... as variable %}`` syntax will *not* cause an error if
the view is missing. In practice you'll use this to link to views that
are optional::

   {% url someController as theURL %}
   {% if theURL %}
     <a href="{{ theURL }}">Link to optional stuff</a>
   {% endif %}


.. tag:: widthRatio
   
widthRatio
==========

For creating bar charts and such, this tag calculates the ratio of a
given value to a maximum value and then applies that ratio to an
expression.

For example::

   <img src="bar.gif" height="10" width="{% widthRatio value max 100 %}">

Above, if ``value`` is 175 and ``max`` is 200, the image in the above
example will be 88 pixels wide (because 175/200 = .875; .875 * 100 =
87.5 which is rounded up to 88).


.. tag:: with

with
====

Caches a complex variable under a simpler name.

For example::

   {% with company.department.employees.count as total %}
     {{ total }} employee{{ total|pluralize }}
   {% endwith %}

The populated variable (in the example above, ``total``) is only
available between the ``{% with %}`` and ``{% endwith %}`` tags.
