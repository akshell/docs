
===============
Date Formatting
===============

In the `date_format.js`_ file the ``format()`` ``Date`` method is
defined. It significantly facilitates date formatting in
JavaScript. Its implementation is borrowed from `date.format.js`_ by
`Steven Levithan`_.

.. _date_format.js: http://www.akshell.com/apps/ak/code/date_format.js
.. _Steven Levithan: http://blog.stevenlevithan.com/archives/date-time-format
.. _date.format.js: http://stevenlevithan.com/assets/misc/date.format.js

.. class:: Date

   .. method:: format(mask='default')

      Format the date according to the given *mask* and return the
      resulting string. *mask* could be either a :ref:`named mask
      <named_mask>` or a string with special character sequences.


Format Strings
==============
      
+-----------+----------------------------------------------------------+
| Mask      | Description                                              |
+===========+==========================================================+
| ``d``     | Day of the month as digits; no leading zero for          |
|           | single-digit days.                                       |
+-----------+----------------------------------------------------------+
| ``dd``    | Day of the month as digits; leading zero for             |
|           | single-digit days.                                       |
+-----------+----------------------------------------------------------+
| ``ddd``   | Day of the week as a three-letter abbreviation.          |
+-----------+----------------------------------------------------------+
| ``dddd``  | Day of the week as its full name.                        |
+-----------+----------------------------------------------------------+
| ``m``     | Month as digits; no leading zero for single-digit        |
|           | months.                                                  |
+-----------+----------------------------------------------------------+
| ``mm``    | Month as digits; leading zero for single-digit months.   |
+-----------+----------------------------------------------------------+
| ``mmm``   | Month as a three-letter abbreviation.                    |
+-----------+----------------------------------------------------------+
| ``mmmm``  | Month as its full name.                                  |
+-----------+----------------------------------------------------------+
| ``yy``    | Year as last two digits; leading zero for years less     |
|           | than 10.                                                 |
+-----------+----------------------------------------------------------+
| ``yyyy``  |  Year represented by four digits.                        |
+-----------+----------------------------------------------------------+
| ``h``     | Hours; no leading zero for single-digit hours            |
|           | (12-hour clock).                                         |
+-----------+----------------------------------------------------------+
| ``hh``    | Hours; leading zero for single-digit hours               |
|           | (12-hour clock).                                         |
+-----------+----------------------------------------------------------+
| ``H``     | Hours; no leading zero for single-digit hours            |
|           | (24-hour clock).                                         |
+-----------+----------------------------------------------------------+
| ``HH``    | Hours; leading zero for single-digit hours               |
|           | (24-hour clock).                                         |
+-----------+----------------------------------------------------------+
| ``M``     | Minutes; no leading zero for single-digit minutes.       |
+-----------+----------------------------------------------------------+
| ``MM``    | Minutes; leading zero for single-digit minutes.          |
+-----------+----------------------------------------------------------+
| ``s``     | Seconds; no leading zero for single-digit seconds.       |
+-----------+----------------------------------------------------------+
| ``ss``    | Seconds; leading zero for single-digit seconds.          |
+-----------+----------------------------------------------------------+
| ``l``     | Milliseconds. l gives 3 digits. L gives 2 digits.        |
| or        |                                                          |
| ``L``     |                                                          |
+-----------+----------------------------------------------------------+
| ``t``     | Lowercase, single-character time marker string: a or p.  |
+-----------+----------------------------------------------------------+
| ``tt``    | Lowercase, two-character time marker string: am or pm.   |
+-----------+----------------------------------------------------------+
| ``T``     | Uppercase, single-character time marker string: A or P.  |
+-----------+----------------------------------------------------------+
| ``TT``    | Uppercase, two-character time marker string: AM or PM.   |
+-----------+----------------------------------------------------------+
| ``S``     | The date's ordinal suffix (st, nd, rd, or th).           |
|           | Works well with d.                                       |
+-----------+----------------------------------------------------------+
| ``'...'`` | Literal character sequence. Surrounding quotes are       |
| or        | removed.                                                 |
| ``"..."`` |                                                          |
+-----------+----------------------------------------------------------+


.. _named_mask:

Named Masks
===========

================== ================================ ============================
  Name               Mask                             Example
================== ================================ ============================
``default``        ``ddd mmm dd yyyy HH:MM:ss``     ``Sat Jun 09 2007 17:46:21``
``shortDate``      ``m/d/yy``                       ``6/9/07``
``mediumDate``     ``mmm d, yyyy``                  ``Jun 9, 2007``
``longDate``       ``mmmm d, yyyy``                 ``June 9, 2007``
``fullDate``       ``dddd, mmmm d, yyyy``           ``Saturday, June 9, 2007``
``shortTime``      ``h:MM TT``                      ``5:46 PM``
``longTime``       ``h:MM:ss TT``                   ``5:46:21 PM``
``isoDate``        ``yyyy-mm-dd``                   ``2007-06-09``
``isoTime``        ``HH:MM:ss``                     ``17:46:21``
``isoDateTime``    ``yyyy-mm-dd'T'HH:MM:ss``        ``2007-06-09T17:46:21``
================== ================================ ============================
