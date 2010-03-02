
=================
Template Language
=================

This document explains the language syntax of the Akshell template
engine. If you're looking for a more technical perspective on how it
works and how to extend it, see :doc:`/ref/ak/template/api`.

The engine is a JavaScript port of the `Django template
engine`__. This text is mainly borrowed from the Django documentation.

__ http://docs.djangoproject.com/en/dev/topics/templates/

Akshell template language is designed to strike a balance between
power and ease. It's intended to feel comfortable to those used to
working with HTML.

If you have a background in programming, or if you're used to
languages like PHP which mix programming code directly into HTML,
you'll want to bear in mind that the Akshell template system is not
simply JavaScript embedded into HTML. This is by design: the template
system is meant to express presentation, not program logic.

The Akshell template system provides tags which function similarly to
some programming constructs -- an :tag:`if` tag for boolean tests, a
:tag:`for` tag for looping, etc. -- but these are not simply executed
as the corresponding JavaScript code, and the template system will not
execute arbitrary JavaScript expressions. Only the tags, filters, and
syntax listed below are supported by default (although you can add
:ref:`your own extensions <template_customization>` to the template
language as needed).


Templates
=========

.. highlightlang:: html+django

A template is simply a text file. It can generate any text-based
format (HTML, XML, CSV, etc.).

A template contains :dfn:`template expressions`, which get replaced
with values when the template is rendered, and :dfn:`tags`, which
control the logic of the template.

Below is a minimal template that illustrates a few basics. Each
element will be explained later in this document.::

   {% extends "base_generic.html" %}

   {% block title %}{{ section.title }}{% endblock %}

   {% block content %}
     <h1>{{ section.title }}</h1>
    
     {% for story in stories %}
       <h2>
         <a href="{{ story.href }}">
           {{ story.headline|toUpperCase }}
         </a>
       </h2>
       <p>{{ story.tease|truncateWords:100 }}</p>
     {% endfor %}
   {% endblock %}


Variables
=========

:dfn:`Variable` is the simplest form of template expression. Variables
look like this: ``{{ variable }}``. When the template engine
encounters a variable, it evaluates that variable and replaces it with
the result.

Use a dot (``.``) to access properties of a variable. If a property is
a function, it is called as a method without arguments.

In the above example, ``{{ section.title }}`` will be replaced with
the ``title`` attribute of the ``section`` object.

If you use a variable that doesn't exist, the template system will
output an empty string ``''``.


Filters
=======

You can modify variables for display using :dfn:`filters`.

Filters look like this: ``{{ name|toTitleCase }}``. This displays the
value of the ``name`` variable after being filtered through the
``toTitleCase`` filter, which converts text to title-case. Use a pipe
(``|``) to apply a filter.

Filters can be "chained." The output of one filter is applied to the
next. For example, ``{{ names|last|toUpperCase }}`` outputs the last
name in uppercase.

Some filters take arguments. A filter argument looks like this: ``{{
bio|truncateWords:30 }}``. This will display the first 30 words of the
``bio`` variable.

Filter arguments that contain spaces must be quoted; for example, to
join a list with commas and spaces you'd use ``{{ list|join:', ' }}``.

Akshell provides about forty default template filters. You can read
all about them in the :doc:`default filter reference
</ref/ak/template/filters>`. To give you a taste of what's available,
here are some of the more commonly used template filters:

:filter:`default`
   If a variable evaluates to ``false`` in boolean context, use the
   given default. Otherwise, use the value of the variable.

   For example::

      {{ value|default:'nothing' }}

   If ``value`` isn't provided or is an empty string ``''``, the above
   will display ``'nothing'``.

:filter:`slice`
   Return a slice of the string or the list. The argument could have
   either ``begin`` or ``begin,end`` form.

   Example::

      {{ someList|slice:'2,4' }}

   If ``someList`` is ``['a', 'b', 'c', 'd', 'e']``, the output will
   be ``['c', 'd']``.
   
:filter:`stripTags`
   Strips all HTML tags. For example::

      {{ value|stripTags }}

   If ``value`` is ``'<b>Joel</b> <button>is</button> a
   <span>slug</span>'``, the output will be ``'Joel is a slug'``.

Again, these are just a few examples; see the :doc:`default filter
reference </ref/ak/template/filters>` for the complete list.

You can also create your own custom template filters; see
:ref:`custom_filters`.


Tags
====

Tags look like this: ``{% tag %}``. Tags are more complex than
template expressions: some create text in the output, some control
flow by performing loops or logic, and some load external information
into the template.

Some tags require beginning and ending tags (i.e., ``{% tag %} ... {%
endtag %}``).

Akshell has about twenty default template tags. You can read all about
them in the :doc:`default tag reference </ref/ak/template/tags>`. To
give you a taste of what's available, here are some of the more
commonly used tags:

:tag:`for`
   Loop over each item in a list. For example, to display an array of
   athletes provided in ``athletes``::

      <ul>
        {% for athlete in athletes %}
          <li>{{ athlete.name }}</li>
        {% endfor %}
      </ul>

:tag:`if` and ``else``
   Evaluate a comparison expression, and if it's "true," the contents
   of the block are displayed::

      {% if athletes.length %}
        Number of athletes: {{ athletes.length }}
      {% else %}
        No athletes.
      {% endif %}

   In the above, if the ``athletes`` variable is not empty, the number
   of athletes will be displayed.

   You can also use comparison operators in the ``if`` tag::

      {% if athletes.length > 1 %}
        Team: {% for athlete in athletes %} ... {% endfor %}
      {% else %}
        Athlete: {{ athletes.0.name }}
      {% endif %}

:tag:`block` and :tag:`extends`
   Set up :ref:`template inheritance <template_inheritance>` (see
   below), a powerful way of cutting down on "boilerplate" in
   templates.

Again, the above is only a selection of the whole list; see the
:doc:`default tag reference </ref/ak/template/tags>` for the complete
list.

You can also create your own custom template tags; see
:ref:`custom_tags`.


Comments
========

To comment-out part of a line in a template, use the comment syntax:
``{# #}``.

For example, this template would render as ``'hello'``::

   {# greeting #}hello

A comment can contain any template code, valid or not. For example::

   {# {% if foo %}bar{% else %} #}

This syntax can only be used for single-line comments (no newlines are
permitted between the ``{#`` and ``#}`` delimiters). If you need to
comment out a multiline portion of the template, see the
:tag:`comment` tag.


.. _template_inheritance:

Template Inheritance
====================

The most powerful -- and thus the most complex -- part of Akshell
template engine is template inheritance. Template inheritance allows
you to build a base "skeleton" template that contains all the common
elements of your site and defines :dfn:`blocks` that child templates
can override.

It's easiest to understand template inheritance by starting with an
example::

   <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
             "http://www.w3.org/TR/html4/strict.dtd">
   <html>
     <head>
       <title>{% block title %}My amazing site{% endblock %}</title>
     </head>

     <body>
       <div id="sidebar">
         {% block sidebar %}
           <ul>
             <li><a href="/">Home</a></li>
             <li><a href="/blog/">Blog</a></li>
           </ul>
         {% endblock %}
       </div>

       <div id="content">
         {% block content %}{% endblock %}
       </div>
     </body>
   </html>

This template, which I'll call ``base.html``, defines a simple HTML
skeleton document that you might use for a simple two-column
page. It's the job of "child" templates to fill the empty blocks with
content.

In this example, the ``{% block %}`` tag defines three blocks that
child templates can fill in. All the ``block`` tag does is to tell the
template engine that a child template may override those portions of
the template.

A child template might look like this::

   {% extends 'base.html' %}

   {% block title %}My amazing blog{% endblock %}

   {% block content %}
     {% for entry in blogEntries %}
       <h2>{{ entry.title }}</h2>
       <p>{{ entry.body }}</p>
     {% endfor %}
   {% endblock %}

The ``{% extends %}`` tag is the key here. It tells the template
engine that this template "extends" another template. When the
template system evaluates this template, first it locates the parent
-- in this case, ``base.html``.

At that point, the template engine will notice the three ``{% block
%}`` tags in ``base.html`` and replace those blocks with the contents
of the child template. Depending on the value of ``blogEntries``, the
output might look like::

   <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
             "http://www.w3.org/TR/html4/strict.dtd">
   <html>
     <head>
       <title>My amazing blog</title>
     </head>

     <body>
       <div id="sidebar">
         <ul>
           <li><a href="/">Home</a></li>
           <li><a href="/blog/">Blog</a></li>
         </ul>
       </div>

       <div id="content">
         <h2>Entry one</h2>
         <p>This is my first entry.</p>

         <h2>Entry two</h2>
         <p>This is my second entry.</p>
       </div>
     </body>
   </html>

Note that since the child template didn't define the ``sidebar``
block, the value from the parent template is used instead. Content
within a ``{% block %}`` tag in a parent template is always used as a
fallback.

You can use as many levels of inheritance as needed. One common way of
using inheritance is the following three-level approach:

* Create a ``base.html`` template that holds the main look-and-feel of
  your site.
  
* Create a :samp:`base_{SECTION_NAME}.html` template for each "section"
  of your site. For example, ``base_news.html``,
  ``base_sports.html``. These templates all extend ``base.html`` and
  include section-specific styles/design.
  
* Create individual templates for each type of page, such as a news
  article or a blog entry. These templates extend the appropriate
  section template.

This approach maximizes code reuse and makes it easy to add items to
shared content areas, such as section-wide navigation.

Here are some tips for working with inheritance:

* If you use ``{% extends %}`` in a template, it must be the first
  template tag in that template. Template inheritance won't work
  otherwise.

* More ``{% block %}`` tags in your base templates are
  better. Remember, child templates don't have to define all parent
  blocks, so you can fill in reasonable defaults in a number of
  blocks, then only define the ones you need later. It's better to
  have more hooks than fewer hooks.

* If you find yourself duplicating content in a number of templates,
  it probably means you should move that content to a ``{% block %}``
  in a parent template.

* For extra readability, you can optionally give a *name* to your ``{%
  endblock %}`` tag. For example::

      {% block content %}
        ...
      {% endblock content %}

  In larger templates, this technique helps you see which ``{% block
  %}`` tags are being closed.

Finally, note that you can't define multiple ``{% block %}`` tags with
the same name in the same template. This limitation exists because a
block tag works in "both" directions. That is, a block tag doesn't
just provide a hole to fill -- it also defines the content that fills
the hole in the *parent*. If there were two similarly-named ``{% block
%}`` tags in a template, that template's parent wouldn't know which
one of the blocks' content to use.


.. _automatic_escaping:

Automatic HTML Escaping
=======================

When generating HTML from templates, there's always a risk that a
variable will include characters that affect the resulting
HTML. Clearly, user-submitted data shouldn't be trusted blindly and
inserted directly into your web pages, because a malicious user could
use this kind of hole to do potentially bad things. This type of
security exploit is called a `Cross Site Scripting`__ (XSS) attack.

.. __: http://en.wikipedia.org/wiki/Cross-site_scripting

To protect you from this problem Akshell *automatically* escapes the
output of every variable. Specifically, these five characters are
escaped:

* ``<`` is converted to ``&lt;``
* ``>`` is converted to ``&gt;``
* ``'`` (single quote) is converted to ``&#39;``
* ``"`` (double quote) is converted to ``&quot;``
* ``&`` is converted to ``&amp;``

Sometimes, template variables contain data that you *intend* to be
rendered as raw HTML, in which case you don't want their contents to
be escaped. For example, you might store a blob of HTML in your
database and want to embed that directly into your template.

Generally, template authors don't need to worry about auto-escaping
very much. Developers on the JavaScript side (people writing handlers
and custom filters) need to think about the cases in which data
shouldn't be escaped, and pass these data via the :func:`safe`
function, so things Just Work in a template.

You could also disable auto-escaping in a template via the
:filter:`safe` filter, but the :func:`safe` function should be
preferred, because escaping is a responsibility of the controller
side. Think of *safe* as shorthand for *safe from further escaping* or
*can be safely interpreted as HTML*.

Suppose you have this template::

    This will be escaped: {{ data }}
    This won't be escaped: {{ safeData }}
    This won't be escaped too: {{ data|safe }}

If ``data`` contains ``'<b>'``, ``safeData`` contains ``safe('<b>')``,
the output will be::

    This will be escaped: &lt;b&gt;
    This won't be escaped: <b>
    This won't be escaped too: <b>

As I mentioned earlier, filter arguments can be strings::

   {{ data|default:'This is a string literal.' }}
    
All string literals are inserted **without** any automatic escaping
into the template -- they act as if they were all passed through the
:filter:`safe` filter.  The reasoning behind this is that the template
author is in control of what goes into the string literal, so he can
make sure the text is correctly escaped when the template is written.

This means you would write::

   {{ data|default:'2 &lt; 3' }}

... rather than::

    {{ data|default:'2 < 3' }}  <-- Bad! Don't do this.

This doesn't affect what happens to data coming from the variable
itself.  The variable's contents are still automatically escaped, if
necessary, because they're beyond the control of a template author.
