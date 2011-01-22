======
Socket
======

.. function:: connect(host, port)

   Connect to *host* via *port* and return a :class:`Socket` object
   representing the connection.

.. class:: Socket

   A ``Socket`` object provides low-level means of Internet
   communication.

   .. attribute:: closed

      ``true`` if the socket is closed; ``false`` otherwise.

   .. attribute:: readable

      ``true`` if the socket is readable; ``false`` otherwise.

   .. attribute:: writable

      ``true`` if the socket is writable; ``false`` otherwise.

   .. method:: receive(size)

      Receive at most *size* bytes from the socket and return a
      :class:`Binary` object. May receive fewer bytes than requested
      even if the end of the stream hasn't been reached.

   .. method:: read([size])

      Read *size* bytes from the socket, or until the end of the
      stream has been reached, and return a :class:`Binary` object. If
      the argument is omitted, read until the end.

   .. method:: send(value)

      Convert *value* to :class:`Binary`, send it to the socket, and
      return the number of bytes sent. May send only a part of the
      data even if the peer hasn't closed connection.

   .. method:: write(value)

      Convert *value* to :class:`Binary`, send it to the socket until
      either all data has been sent or the peer has closed connection,
      and return the number of bytes sent.

   .. method:: close()

      Close the socket. All future operations on it will fail.

   .. method:: shutdown(how)

      Shut down one or both halves of the connection. If *how* is
      ``'receive'``, further receives are disallowed; if *how* is
      ``'send'``, further sends are disallowed; if how is ``'both'``,
      further sends and receives are disallowed.
