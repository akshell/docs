
=======
Testing
=======

Automated testing is an extremely useful bug-killing tool for a modern
web developer. You can use a collection of tests to solve, or avoid, a
number of problems:

* when you're writing new code, you can use tests to validate your
  code works as expected;

* when you're refactoring or modifying old code, you can use tests to
  ensure your changes haven't affected your application's behavior
  unexpectedly.

The Akshell unit testing framework makes writing automated tests
easy. The framework is a port of the Python unit testing framework,
which is, in turn, a port of JUnit, by Kent Beck and Erich Gamma. So
if you are coming from Java or Python, you'll feel at home.

This document describes the basics of writing unit tests; if you're
interested in details, see the :doc:`framework reference
</ref/ak/unittest>`.


Test Cases
==========

Let me immediately start with an example::

   var FirstTestCase = TestCase.subclass(
     {
       name: 'first',

       multiply: function (x, y) {
         return x * y;
       },

       setUp: function () {
         this._answer = 42;
       },
     
       testMultiply: function () {
         assertSame(this.multiply(21, 2), this._answer);
         assert(isNaN(this.multiply(42, 'text')));
       },

       testRange: function () {
         assertEqual(range(5), [0, 1, 2, 3, 4]);
         assertSame(sum(range(3, 10)), this._answer);
         assertThrow(ValueError, range, 1, 2, 0);
       }
     });

A test case class is created by subclassing
:class:`TestCase`. Individual tests are defined with methods whose
names start with the letters ``test``. This naming convention informs
Akshell which methods represent tests.

Each test calls the :ref:`assertion functions <debug_tools>` to verify
the behavior of the program. :func:`assert` checks a condition;
:func:`assertSame` checks values to be identical; :func:`assertEqual`
checks values to be :func:`equal <equal>`; :func:`assertThrow` checks
a function to throw an expected exception.

When a ``setUp()`` method is defined, Akshell will run that method
prior to each test. Likewise, if a ``tearDown()`` method is defined,
Akshell will invoke that method after each test.

You could define as many :class:`TestCase` subclasses as you need; in
a test output they are identified by the ``name`` prototype property.


Entry Point
===========

To run all your tests, just :ref:`evaluate <evaluate>` this
expression::

   test()

Yes, that easy. You don't have to declare anything -- Akshell will
guess tests to run via black magic (in fact, the :func:`test` function
simply scans the global namespace for :class:`TestCase` subclasses,
but don't tell anybody).

For the previous example it will produce the following output::

   testMultiply(first) ok
   testRange(first) ok
   -----
   Ran 2 tests
   OK

   
Test Client
===========

Real-world web applications often need to check for proper responses,
not for proper arithmetic. To simulate a real application client -- a
browser or another application -- use a :class:`TestClient`
object. It's a common practice to create such object in a ``setUp()``
method and use it in test methods.

For example::

   var SecondTestCase = TestCase.subclass(
     {
       setUp: function () {
         this._client = new TestClient();
         this._client.login('Bob');
       },

       testIndex: function () {
         var response = this._client.get()
         assertSame(response.handler, IndexHandler);
         assertSame(response.context.user, 'Bob');
       },

       testSomePage: function () {
         this._client.post(
           {path: '/some/path/', post: {parameter: 'value'}});
         assertSame(
           this._client.get({path: '/some/page/'}).context.parameter,
           'value');
         assertSame(
           this._client.get({path: '/some/page/', user: 'Alice'}).status,
           http.FORBIDDEN);
       }
     });

Use the :meth:`~TestClient.get`, :meth:`~TestClient.post`,
:meth:`~TestClient.head`, :meth:`~TestClient.put`, and
:meth:`~TestClient.del` client methods to perform test requests. The
:meth:`~TestClient.login` method sets the default user of test
requests.

The template framework hacks the :class:`Handler` class on the fly, so
each response has a ``handler`` property referencing the handler which
produced the response. If the response content was generated via the
template engine, the response object also has a ``context`` property
referencing the context object used for the template rendering. These
features are invaluable for testing.
