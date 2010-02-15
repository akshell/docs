
=================
Template Language
=================

.. _template_inheritance:

Template Inheritance
====================


.. _html_escaping:

HTML Escaping
=============

In order to prevent :term:`cross-site scripting` vulnerabilities the
template engine automatically escapes all strings coming from
variables. Specifically, these five characters are escaped:

* ``<`` is converted to ``&lt;``
* ``>`` is converted to ``&gt;``
* ``'`` (single quote) is converted to ``&#39;``
* ``"`` (double quote) is converted to ``&quot;``
* ``&`` is converted to ``&amp;``

For example::

   >>> (new Template('{{ value }}')).render({value: '<>\'"&'})
   &lt;&gt;&#39;&quot;&amp;

Constants are not escaped::

   >>> (new Template('{{ "<>" }}')).render()
   <>

The default escaping behavior could be altered by the
:filter:`escape`, :filter:`safe`, and :filter:`forceEscape`
filters. For example::

   >>> (new Template('{{ value|safe }}')).render({value: '<>\'"&'})
   <>'"&

