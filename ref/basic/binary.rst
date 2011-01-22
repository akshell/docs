======
Binary
======

.. class:: Binary()

   A ``Binary`` object represents raw binary data. ``Binary`` is a
   mutable fixed-length numeric byte storage type. It can be
   instantiated in a few ways:

   ``new Binary(length, byte=0)``
      Create a new ``Binary`` with the given *length* and fill it with
      the given *byte*.

   ``new Binary(string, charset='utf-8')``
      Convert the *string* to ``Binary`` using the given *charset*.

   ``new Binary(array)``
      Initialize ``Binary`` bytes from the *array* values.

   ``new Binary(binary, toCharset, fromCharset='utf-8')``
      Transcode *binary* from *fromCharset* to *toCharset*.

   ``new Binary(binary[, binary1...])``
      Create new ``Binary`` concatenating the given binaries.

   The index operator ``[]`` can be used to get and set byte values.

   .. attribute:: length

      The length of the byte sequence. Cannot be changed.

   .. method:: toString(charset='utf-8')

      Convert to ``string`` using the given *charset*.

   .. method:: range(start=0, stop=length)

      Return a new ``Binary`` that views the given range of this
      ``Binary``.

   .. method:: fill(byte=0)

      Fill the ``Bynary`` by the given *byte*.

   .. method:: indexOf(value, start=0)

      Return the index of the first occurence of *value*, starting
      search at *start*; return ``-1`` if *value* is not
      found. *value* can be ``Binary`` or ``string``.

   .. method:: lastIndexOf(value, start=length)

      Return the index of the last occurence of *value*, starting
      search at *start*; return ``-1`` if *value* is not
      found. *value* can be ``Binary`` or ``string``.

   .. method:: md5()

      Calculate the MD5 hash and return it as a ``string`` hex dump.

   .. method:: sha1()

      Calculate the SHA1 hash and return it as a ``string`` hex dump.

.. exception:: ConversionError

   Failed to encode, decode, or transcode data.
