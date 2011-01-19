========================
Default Template Filters
========================

.. highlightlang:: html+django

The default template filters are represented by the
``template.env.filters`` properties.


.. filter:: add

add
===

Add the argument to the value.

For example::

   {{ value|add:2 }}

If ``value`` is ``4``, the output will be ``6``.


.. filter:: addSlashes

addSlashes
==========

Add slashes before single and double quotes. Useful for escaping
strings in CSV.

For example::

   {{ value|addSlashes }}

If ``value`` is ``"I'm using Akshell"``, the output will be ``"I\'m
using Akshell"``.


.. filter:: breakLines

breakLines
==========

Convert all newlines in a piece of plain text to the HTML line breaks
(``<br>``).

For example::

   {{ value|breakLines }}

If ``value`` is ``'Joel\nis a slug'``, the output will be
``'Joel<br>is a slug'``.


.. filter:: capFirst

capFirst
========

Capitalize the first character of the value.

For example::

   {{ value|capFirst }}

If ``value`` is ``'akshell'``, the output will be ``'Akshell'``.


.. filter:: countWords

countWords
==========

Return the number of words.

For example::

   {{ value|countWords }}

If ``value`` is ``'Joel is a slug'``, the output will be ``4``.


.. filter:: cut

cut
===

Remove all values of the argument from the given string.

For example::

   {{ value|cut:' '}}

If ``value`` is ``'String with spaces'``, the output will be
``'Stringwithspaces'``.


.. filter:: default

default
=======

If the value evaluates to ``false``, use the given default. Otherwise,
use the value.

For example::

   {{ value|default:'nothing' }}

If ``value`` is ``''`` (an empty string), the output will be
``'nothing'``.


.. filter:: defaultIfNull

defaultIfNull
=============

If (and only if) the value is ``null``, use the given
default. Otherwise use the value.

Note that if an empty string is given, the default value will **not**
be used.  Use the ``default`` filter if you want to fallback for empty
strings.

For example::

   {{ value|defaultIfNull:'nothing' }}

If ``value`` is ``null``, the output will be the string ``'nothing'``.


.. filter:: defaultIfUndefined

defaultIfUndefined
==================

If (and only if) the value is ``undefined``, use the given
default. Otherwise use the value.

Note that if an empty string is given, the default value will **not**
be used.  Use the ``default`` filter if you want to fallback for empty
strings.

For example::

   {{ value|defaultIfUndefined:'nothing' }}

If ``value`` is ``undefined``, the output will be ``'nothing'``.


.. filter:: divisibleBy

divisibleBy
===========

Return ``true`` if the value is divisible by the argument.

For example::

   {{ value|divisibleBy:3 }}

If ``value`` is ``21``, the output will be ``true``.


.. filter:: encodeURI

encodeURI
=========

Apply the ``encodeURI()`` JavaScript function to the value.

For example::

   {{ value|encodeURI }}

If ``value`` is ``'%^ []'``, the output will be ``'%25%5E%20%5B%5D'``.


.. filter:: encodeURIComponent

encodeURIComponent
==================

Apply the ``encodeURIComponent()`` JavaScript function to the value.

For example::

   {{ value|encodeURIComponent }}

If ``value`` is ``'/some/path/?some&params'``, the output will be
``'%2Fsome%2Fpath%2F%3Fsome%26params'``.


.. filter:: escape

escape
======

Mark the value to be HTML escaped before the output. The escaping
is performed by the :func:`escapeHTML` function.

Applying ``escape`` to a value that is already marked for escaping
will do nothing; so it is safe to use this function in auto-escaping
context. If you want multiple escaping passes to be applied, use the
:filter:`forceEscape` filter.


.. filter:: escapeJS

escapeJS
========

Escape characters for use in JavaScript strings. This does **not**
make the string safe for use in HTML, but does protect you from syntax
errors when using templates to generate JavaScript/JSON.

For example::

   {{ value|escapeJS }}

If ``value`` is ``'testing\r\njavascript \'string" <b>escaping</b>'``,
the output will be ``'testing\\x0D\\x0Ajavascript \\x27string\\x22
\\x3Cb\\x3Eescaping\\x3C/b\\x3E'``.


.. filter:: first

first
=====

Return the first item in the list.

For example::

   {{ value|first }}

If ``value`` is the array ``['a', 'b', 'c']``, the output will be
``'a'``.


.. filter:: forceEscape

forceEscape
===========

Apply HTML escaping to the string via the :func:`escapeHTML`
function. This filter is applied *immediately*, and returns a new,
escaped string. This is useful in the rare cases when you need
multiple escaping or want to apply other filters to the escaped
results. Normally, you want to use the :filter:`escape` filter.


.. filter:: formatFileSize

formatFileSize
==============

Format the value like a human-readable file size (i.e., ``'13 KB'``,
``'4.1 MB'``, ``'102 bytes'``, etc).

For example::

   {{ value|formatFileSize }}

If ``value`` is 123456789, the output will be ``'117.7 MB'``.


.. filter:: getDigit

getDigit
========

Given a whole number, return the requested digit, where 1 is the
right-most digit, 2 is the second-right-most digit, etc. Return the
original value for invalid input (if input or argument is not an
integer, or if argument is less than 1). Otherwise output is always an
integer.

For example::

   {{ value|getDigit:2 }}

If ``value`` is ``123456789``, the output will be ``8``.


.. filter:: hyphen

hyphen
======

Remove non-word characters (alphanumerics and underscores), strip
leading and trailing white space, and replace spaces by hyphens.

For example::

   {{ value|hyphen }}

If ``value`` is ``'Joel is a slug'``, the output will be
``'Joel-is-a-slug'``.


.. filter:: items

items
=====

Return an ``Array`` of ``[key, value]`` pairs of the object ordered by
keys.

For example::

   {{ value|items }}

If ``value`` is the object ``{a: 1, b: 2, c: 3}``, the output will
be ``[['a', 1], ['b', 2], ['c', 3]]``.


.. filter:: join

join
====

Join an array with a string by JavaScript's ``array.join(string)``.

For example::

   {{ value|join:' // ' }}

If ``value`` is the array ``['a', 'b', 'c']``, the output will be the
string ``'a // b // c'``.


.. filter:: last

last
====

Return the last item in a list.

For example::

   {{ value|last }}

If ``value`` is the array ``['a', 'b', 'c', 'd']``, the output will be
the string ``'d'``.


.. filter:: numberLines

numberLines
===========

Display text with line numbers.

For example::

   {{ value|numberLines }}

If ``value`` is::

   one
   two
   three

the output will be::

   1. one
   2. two
   3. three


.. filter:: paragraph

paragraph
=========

Replace line breaks in plain text with the appropriate HTML; a single
newline becomes an HTML line break (``<br>``) and a new line followed
by a blank line becomes a paragraph break (``</p>``).

For example::

   {{ value|paragraph }}

If ``value`` is ``'Joel\nis a slug'``, the output will be
``'<p>Joel<br>is a slug</p>'``.


.. filter:: pluralize

pluralize
=========

Return a plural suffix if the value is not 1. By default, this suffix
is ``'s'``.

Example::

   You have {{ count }} message{{ count|pluralize }}.

If ``count`` is ``1``, the output will be ``You have 1 message.`` If
``count`` is ``2`` the output will be ``You have 2 messages.``

For words that require a suffix other than ``'s'``, you can provide an
alternate suffix as a parameter to the filter.

Example::

   You have {{ count }} walrus{{ count|pluralize:'es' }}.

For words that don't pluralize by simple suffix, you can specify both
a singular and plural suffix, separated by a comma.

Example::

   You have {{ count }} cherr{{ count|pluralize:"y,ies" }}.


.. filter:: removeTags

removeTags
==========

Remove a space-separated list of HTML tags from the text.

For example::

   {{ value|removeTags:'b span'|safe }}

If ``value`` is ``'<b>Joel</b> <button>is</button> a
<span>slug</span>'``, the output will be ``'Joel <button>is</button> a
slug'``.


.. filter:: safe

safe
====

Mark the value as not requiring further HTML escaping prior to
output.


.. filter:: slice

slice
=====

Return a slice of the string or the list. The argument can have either
``begin`` or ``begin,end`` form.

Example::

   {{ someList|slice:'2,4' }}

If ``someList`` is ``['a', 'b', 'c', 'd', 'e']``, the output will be
``['c', 'd']``.


.. filter:: sortObjects

sortObjects
===========

Take a list of objects and return that list sorted by the key given in
the argument.

For example::

   {{ value|sortObjects:'name' }}

If ``value`` is:

.. code-block:: javascript

   [
     {name: 'zed', age: 19},
     {name: 'amy', age: 22},
     {name: 'joe', age: 31}
   ]

... then the output will be:

.. code-block:: javascript

   [
     {name: 'amy', age: 22},
     {name: 'joe', age: 31},
     {name: 'zed', age: 19}
   ]


.. filter:: sortObjectsReversed

sortObjectsReversed
===================

Take a list of objects and return that list sorted in reverse order
by the key given in the argument. This works exactly the same as the
above filter, but the returned value will be in reverse order.


.. filter:: stripTags

stripTags
=========

Strips all HTML tags.

For example::

   {{ value|stripTags }}

If ``value`` is ``'<b>Joel</b> <button>is</button> a
<span>slug</span>'``, the output will be ``'Joel is a slug'``.


.. filter:: timeSince

timeSince
=========

Apply the :func:`timeSince` function to the date. Takes an optional
argument that is a variable containing the date to use as the
comparison point.


.. filter:: timeUntil

timeUntil
=========

Apply the :func:`timeUntil` function to the date. Takes an optional
argument that is a variable containing the date to use as the
comparison point.


.. filter:: toLowerCase

toLowerCase
===========

Convert the value to a lowercase string.

For example::

   {{ value|toLowerCase }}

If ``value`` is ``'Still MAD At Yoko'``, the output will be ``'still
mad at yoko'``.


.. filter:: toString

toString
========

Convert the value to a string via its ``toString()`` method.

For example::

   {{ value|toString:'MMM yyyy' }}

If ``value`` is ``new Date('Wed, 24 Mar 2010 17:36:07')``, the output
will be the string ``'Mar 2010'``.


.. filter:: toTitleCase

toTitleCase
===========

Convert the value to a title-case string.

For example::

   {{ value|toTitleCase }}

If ``value`` is ``'my first post'``, the output will be ``'My First
Post'``.


.. filter:: toUpperCase

toUpperCase
===========

Convert the value to an uppercase string.

For example::

   {{ value|toUpperCase }}

If ``value`` is ``'Joel is a slug'``, the output will be ``'JOEL IS A
SLUG'``.


.. filter:: truncateWords

truncateWords
=============

Truncate a string after a certain number of words.

For example::

   {{ value|truncateWords:2 }}

If ``value`` is ``'Joel is a slug'``, the output will be ``'Joel is ...'``.


.. filter:: yesno

yesno
=====

Return ``'yes'`` if the value evaluates to ``true``; return ``'no'``
otherwise. An optional argument can specify alternative strings.

For example::

   {{ true|yesno }}
   {{ false|yesno:'sure,no way' }}

... will output::

   yes
   no way

