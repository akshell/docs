==========
Formatting
==========

In the `format.js`_ file the ``format()`` ``String`` method and the
``toString()`` ``Number`` and ``Date`` methods are defined. They
significantly facilitate string formatting in JavaScript. The
interface is inspired by the .NET Framework; the implementation is
based on the `String.format for JavaScript`__ by Daniel Mester
Pirttijärvi.

.. _format.js: http://www.akshell.com/apps/ak/code/0.2/format.js
__ http://www.masterdata.dyndns.org/r/string_format_for_javascript/


format()
========

.. class:: String
   :noindex:

   .. method:: format(args...)

      Substitute substrings of the form
      ``{index[,length][:formatString]}`` by the corresponding
      arguments and return the resulting string. ::

         >>> 'Name: {0}\nAge: {1}'.format('John', 42)
         Name: John
         Age: 42

      If ``length`` is defined, a parameter is padded by spaces up to
      the given length. If ``length`` is positive, the parameter is
      right-aligned; if negative, it's left-aligned. ::

         >>> 'Name: {0,-10} Age: {1,3}'.format('Ann', 7)
         Name: Ann        Age:   7

      If ``formatString`` is defined, it's passed to the parameter's
      ``toString()`` method. Its syntax is specific to the object being
      formatted. ::

         >>> '{0:dd MMMM}: {1,6:0.00%}'.format(new Date('03/26/2010'), .012)
         26 March:  1.20%


Numbers
=======

.. class:: Number

   .. method:: toString(format='g')

      Format the number according to the given *format*. ::

         >>> (12.34).format('000.000')
         012.340
         >>> (42).toString('X')
         2A
         >>> (1234567).toString('c')
         $1,234,567.00
         >>> (-1).toString('positive;negative;zero')
         negative
         >>> (8005551212).toString('(###) ###-####')
         (800) 555-1212


Standard Specifiers
-------------------

Standard specifiers denote particular, commonly used formats.

=========  ===========================  =======  =============
Specifier  Type                         Number   Output
=========  ===========================  =======  =============
c          Currency                     1234567  $1,234,567.00
g          General                      123.456  123.456
f          Fixed point                  .1234    0.12
n          Number with commas           1234567  1,234,567
           for thousands
x          Lowercase hexadecimal        42       2a
X          Uppercase hexadecimal        42       2A
=========  ===========================  =======  =============


Custom Specifiers
-----------------

Custom specifiers are intended for defining application-specific
formats.

=========  =================  =====================  ======  ======  ======
Specifier  Type               Note                   Format  Number  Output
=========  =================  =====================  ======  ======  ======
0          Zero placeholder   Pads with zeroes       00.000  1.2     01.200
#          Digit placeholder                         (#).##  1.2345  (1).23
.          Decimal point                             0.0     12.34   12.3
,          Thousands          Must be between        0,0     12345   12,345
           Separator          two zeroes
,.         Number scaling     Scales by 1000         0,.     12345   12
%          Percent            Multiplies by 100,     0%      .1234   12.34%
                              adds % sign
;          Group separator    Positive, negative,    p;n;z   0       z
                              and zero formats
=========  =================  =====================  ======  ======  ======


Dates
=====

.. class:: Date

   .. method:: toString(format='ddd MMM dd yyyy HH:mm:ss')

      Format the date according to the given *format*.


Standard Specifiers
-------------------

Standard specifiers denote particular, commonly used formats.
Examples are given for the time of writing, ``new Date('Fri Mar 26
2010 10:38:44')``.

=========  ==============================  ==========================
Specifier  Description                     Example
=========  ==============================  ==========================
``d``      Short date                      03/26/2010
``D``      Long date                       March 26, 2010
``t``      Short time                      10:38 AM
``T``      Long time                       10:38:44 AM
``M``      Month/day                       26 March
``Y``      Year/month                      March, 2010
``s``      Sortable date/time              2010-03-26T10:38:44
``f``      Full date/time (short time)     March 26, 2010 10:38 AM
``F``      Full date/time (long time)      March 26, 2010 10:38:44 AM
``g``      General date/time (short time)  03/26/2010 10:38 AM
``G``      General date/time (long time)   03/26/2010 10:38:44 AM
=========  ==============================  ==========================


Custom Specifiers
-----------------

Custom specifiers are intended for defining application-specific
formats.

+-----------+----------------------------------------------------------+
| Specifier | Description                                              |
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
| ``M``     | Month as digits; no leading zero for single-digit        |
|           | months.                                                  |
+-----------+----------------------------------------------------------+
| ``MM``    | Month as digits; leading zero for single-digit months.   |
+-----------+----------------------------------------------------------+
| ``MMM``   | Month as a three-letter abbreviation.                    |
+-----------+----------------------------------------------------------+
| ``MMMM``  | Month as its full name.                                  |
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
| ``m``     | Minutes; no leading zero for single-digit minutes.       |
+-----------+----------------------------------------------------------+
| ``mm``    | Minutes; leading zero for single-digit minutes.          |
+-----------+----------------------------------------------------------+
| ``s``     | Seconds; no leading zero for single-digit seconds.       |
+-----------+----------------------------------------------------------+
| ``ss``    | Seconds; leading zero for single-digit seconds.          |
+-----------+----------------------------------------------------------+
| ``tt``    | Time marker string: AM or PM.                            |
+-----------+----------------------------------------------------------+
