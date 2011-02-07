========
Glossary
========

.. glossary::
   :sorted:

   Attribute
      A pair of a name and a type.

   Class
      A function intended for creating objects via the ``new`` operator.

   CSRF
      `Cross-Site Request Forgery`_. This type of attack occurs when a
      malicious web site contains a link, a form button, or some
      client JavaScript that is intended to perform some action on
      your web site, using the credentials of a logged-in user who
      visits the malicious site in their browser.

   Decorator
      A function accepting a function and returning another function,
      usually used as a function transformation.

   DRY
      `Don't Repeat Yourself`_ principle of software development:
      every piece of knowledge must have a single, unambiguous,
      authoritative representation within a system.

   JSGI
      `JavaScript Gateway Interface`_.

   Metaclass
      A class whose instances are classes. Just as an ordinary class
      defines the behavior of certain objects, a metaclass defines the
      behavior of certain classes and their instances.

   MVC
      `Model–View–Controller`_. A software architecture isolating
      domain logic from input and presentation, permitting independent
      development, testing, and maintenance of each.

   Relation
      A data structure which consists of a header and a set of tuples
      sharing the same header.

   Relation variable
      A named variable whose value is a relation.

   Relational model
      A database model used in the majority of modern database
      systems. Proposed in 1969 by E.F. Codd. See the `Wikipedia
      page`_ for details.

   REST
      `Representational State Transfer`_. A style of software
      architecture for distributed systems. RESTful architectures
      consist of clients and servers. Clients initiate requests to
      servers; servers process requests and return appropriate
      responses. Requests and responses are built around the transfer
      of "representations" of "resources".

   Spot
      An independent version of an application owned by a developer.

   Subclass
      A class that inherits prototype properties from its
      superclass. If ``Derived`` is a subclass of ``Base``,
      ``Derived.prototype.__proto__ === Base.prototype``.

   Surrogate key
      A unique identifier of a tuple. The surrogate key is *not*
      derived from application data. Usually implemented using the
      ``'unique serial'`` type.

   Transaction
      A unit of work performed against a database. It bundles multiple
      steps into a single, all-or-nothing operation.

   Tuple
      A set of uniquely named attributes with their values.

.. _Cross-Site Request Forgery: http://en.wikipedia.org/wiki/Csrf
.. _Don't Repeat Yourself: http://en.wikipedia.org/wiki/Don%27t_repeat_yourself
.. _JavaScript Gateway Interface: http://wiki.commonjs.org/wiki/JSGI/Level0/A/Draft2
.. _Model–View–Controller: http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
.. _Wikipedia page: http://en.wikipedia.org/wiki/Relational_model
.. _Representational State Transfer: http://en.wikipedia.org/wiki/REST
