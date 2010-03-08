
=======
Testing
=======

To make your application reliable you should provide it with an
extensive set of tests. The Akshell unit testing framework (file
`unittest.js`_) targets at facilitation of this task. It supports test
automation, sharing of setup and shutdown code for tests, aggregation
of tests into collections, and independence of the tests from the
reporting machinery. It also has means of emulating field application
environment. The framework is a port of the Python unit testing
framework, which is, in turn, a port of JUnit, by Kent Beck and Erich
Gamma.

.. _unittest.js: http://www.akshell.com/apps/ak/code/unittest.js

Overview
========

The basic concepts of the unit testing framework are:

test fixture
   A :dfn:`test fixture` represents the preparation needed to perform
   one or more tests, and any associated cleanup actions. This may
   involve, for example, initializing test objects, or creating and
   populating relation variables.

test case
   A :dfn:`test case` is the smallest unit of testing. It checks for a
   specific response to a particular set of inputs. :class:`TestCase`
   should be used as a base class to create test cases.

test suite
   A :dfn:`test suite` is a collection of test cases, test suites, or
   both. It is used to aggregate tests that should be executed
   together.
   
test runner
   A :dfn:`test runner` is a function which orchestrates the execution
   of tests and provides the outcome to you. The runner may use
   various interfaces and output formats.

test loader
   A :dfn:`test loader` is a function which loads test cases from some
   test unit and incorporates them into a test suite.

The test case and test fixture concepts are supported through the
:class:`TestCase` class. Tests are created by subclassing it and
implementing :dfn:`test methods`, i.e., methods whose names start with
the letters ``test``. Each test method represents a single test case;
it should make assertions via the :ref:`debug tools<debug_tools>` to
check for an expected result. The :meth:`~TestCase.setUp()` and
:meth:`~TestCase.tearDown()` methods can be overridden to provide
initialization and cleanup for the fixture.

Test suites are implemented by the :class:`TestSuite` class. This
class allows individual tests and test suites to be aggregated; when
the suite is executed, all tests added directly to the suite and in
the "child" test suites are run.

A test runner is a function that accepts a ``TestCase`` or
``TestSuite`` object as a parameter and returns a result object. The
class :class:`TestResult` is provided for use as the result object. It
counts tests has been run and collects :dfn:`errors` (unexpected
exceptions) and :dfn:`failures` (:exc:`AssertionError` exceptions has
been thrown by ``assert*`` functions). ``unittest.js`` provides
:func:`runTestViaStream` as an example test runner which reports test
results on a :class:`stream<Stream>`.

The :func:`loadTestSuite` function is a main test loader. It could
load test suites from individual test cases, :class:`TestCase`
subclasses, modules, and arrays of the above.


TestCase
========

.. class:: TestCase(methodName)

   Each ``TestCase`` instance represents a single test, but each
   concrete subclass may be used to define multiple tests -- the
   concrete class represents a single test fixture. The fixture is
   created and cleaned up for each test case.

   .. attribute:: name

      The name of the test fixture; ``undefined`` by default.

   .. method:: setUp()

      Prepare the test fixture. Called immediately before calling the
      test method; any exception thrown by this method will be
      considered an error rather than a test failure. The default
      implementation does nothing.

   .. method:: tearDown()

      Method called immediately after the test method has been called
      and the result recorded. This is called even if the test method
      threw an exception; so the implementation in subclasses may need
      to be particularly careful about checking internal state. Any
      exception thrown by this method will be considered an error
      rather than a test failure. This method will only be called if
      :meth:`setUp()` succeeds, regardless of the outcome of the test
      method. The default implementation does nothing.
   
   .. method:: run(result)

      Run the test, collecting the result into the test result object
      passed as *result*.

   ::

      (function ()
      {
        var Test = TestCase.subclass(
          {
            setUp:       function () { this.answer = 42;            },
            testSuccess: function () { assertSame(this.answer, 42); },
            testFailure: function () { assertSame(2 + 2, 5);        },
            testError:   function () { throw 'error';               }
          });
        var result = new TestResult();
        loadTestSuite(Test).run(result);
        assertSame(result.testsRun, 3);
        assertSame(repr(result.errors),
                   '[[<TestCase testError>, "error"]]');
        assertSame(repr(result.failures),
                   '[[<TestCase testFailure>, ak.AssertionError("4 !== 5")]]');
      })()


TestSuite
=========
   
.. class:: TestSuite(tests=[])

   ``TestSuite`` objects behave much like :class:`TestCase` objects,
   except they do not actually implement a test. Instead, they are
   used to aggregate tests into groups of tests that should be run
   together.

   .. method:: addTest(test)

      Add a ``TestCase`` or ``TestSuite`` to the suite.

   .. method:: countTestCases()

      Return the number of tests represented by the suite, including
      the tests of the sub-suites.

   .. method:: run(result)

      Run the tests associated with this suite, collecting the result
      into the test result object passed as *result*.

   ::

      (function ()
      {
        var Test = TestCase.subclass(
          {
            name: 'fixture',
            test1: function () {},
            test2: function () {},
            test3: function () {}
          });
        var suite = new TestSuite();
        var subsuite = new TestSuite([new Test('test1'), new Test('test2')]);
        assertSame(repr(subsuite),
                   '<TestSuite test1(fixture), test2(fixture)>');
        suite.addTest(new Test('test3'));
        suite.addTest(subsuite);
        assertSame(suite + '',
                   'test3(fixture), test1(fixture), test2(fixture)');
      })()

   
TestResult
==========

.. class:: TestResult

   A ``TestResult`` object stores the results of a set of tests.  The
   :class:`TestCase` and :class:`TestSuite` classes ensure that
   results are properly recorded; test authors do not need to worry
   about recording the outcome of tests.

   ``TestResult`` instances have the following attributes that will be
   of interest when inspecting the results of running a set of tests.

   .. attribute:: errors

      An array containing 2-item arrays of :class:`TestCase` instances
      and exceptions representing a test which threw an unexpected
      exception.
   
   .. attribute:: failures
   
      An array containing 2-item arrays of :class:`TestCase` instances
      and exceptions representing a test where a failure was
      explicitly signaled using the ``assert*()`` functions (an
      :exc:`AssertionError` was thrown).
      
   .. attribute:: testsRun

      The total number of tests run so far.

   .. method:: wasSuccessful()

      Return ``true`` if all tests run so far have passed; otherwise
      return ``false``.

   The following methods of the ``TestResult`` class are used to
   maintain the internal data structures and may be extended in
   subclasses to support additional reporting requirements. This is
   particularly useful in building tools which support interactive
   reporting while tests are being run.

   .. method:: startTest(test)

      Called when the test case *test* is about to be run. The default
      implementation simply increments the instance's :attr:`testsRun`
      counter.

   .. method:: stopTest(test)

      Called after the test case *test* has been executed, regardless
      of the outcome. The default implementation does nothing.

   .. method:: addError(test, error)

      Called when the test case *test* throws an unexpected
      exception. The default implementation pushes a pair ``[test,
      error]`` to the instance's ``errors`` attribute.
   
   .. method:: addFailure(test, failure)

      Called when the test case *test* signals a failure (throws an
      :exc:`AssertionError`). The default implementation pushes a
      pair ``[test, failure]`` to the instance's ``failures``
      attribute.
   
   .. method:: addSuccess(test)

      Called when the test case *test* succeeds. The default
      implementation does nothing.

   
Functions
=========

.. function:: loadTestSuite(source)

   Return a suite of all tests contained in *source*. The following
   sources are supported:

   ``TestSuite`` object
      return *source* itself;

   ``TestCase`` object
      return a suite with this test case;

   ``TestCase`` subclass
      return a suite of all test cases contained in this subclass (the
      subclass is instantiated for each method whose name starts with
      the letters ``test``);

   ``Module`` object
      return a suite of suites loaded from the module properties
      matching one of three previous types; and

   array-like object
      return a suite of suites loaded from the items of the list.

   ::

      (function ()
      {
        var module = new Module();
        module.Test = TestCase.subclass(
          {
            test1: function () {},
            test2: function () {},
            func: function () {}
          });
        module.test = new module.Test('func');
        module.suite = new TestSuite();
        assertSame(loadTestSuite(module.suite), module.suite);
        assertSame(repr(loadTestSuite(module.test)),
                   '<TestSuite func>');
        assertSame(repr(loadTestSuite(module.Test)),
                   '<TestSuite test1, test2>');
        assertSame(repr(loadTestSuite(module)),
                   '<TestSuite test1, test2, func>');
        assertSame(repr(loadTestSuite([module.test, module.Test])),
                   '<TestSuite func, test1, test2>');
      })();

.. function:: runTestViaStream(test, stream=out)

   Create a :class:`TestResult` object, run *test* collecting the
   results in the result object, and return the result object. Test
   progress, errors, and failures are printed to the :class:`Stream`
   object *stream*, which defaults to :data:`out`. ::

      (function ()
      {
        var Test = TestCase.subclass(
          {
            test: function () {}
          });
        var stream = new Stream();
        var result = runTestViaStream(new Test('test'), stream);
        assertSame(result.testsRun, 1);
        assert(result.wasSuccessful());
        assertSame(stream.read(), 'test ok\n-----\nRan 1 tests\nOK');
      })()

.. function:: test(source=global, stream=out)

   Load a test from *source* by :func:`loadTestSuite`, run it by
   :func:`runTestViaStream`, and return ``stream.read()``. The
   ``test()`` function is a common launcher of your tests. All you
   need to run the tests is to evaluate the expression "``test()``" in
   a development spot. ::

      >>> (function ()
          {
            var module = new Module();
            module.Test = TestCase.subclass(
              {
                testSuccess: function () {},
                testFailure: function () { assertSame(2 + 2, 5); },
                testError:   function () { throw Error(); }
              });
            return test(module, new Stream());
          })()
      testError ERROR
      testFailure FAIL
      testSuccess ok
      =====
      ERROR: testError
      Error
          ...
      =====
      FAIL: testFailure
      ak.AssertionError: 4 !== 5 
          ...
      -----
      Ran 3 tests
      FAILED (failures=1, errors=1)

      
TestClient
==========

.. class:: TestClient(users=[], apps={})

   A ``TestClient`` object emulates a real application client, a
   browser or another application. Via ``TestClient`` methods one
   could make requests and check for expected responses. A client is
   usually created in the :meth:`~TestCase.setUp` method of a
   particular :class:`TestCase` subclass.

   The :meth:`request` ``TestClient`` method creates a sandbox
   environment throughout a handling of a test request. It temporarily
   substitutes the :func:`describeApp`, :func:`getAdminedApps`,
   :func:`getDevelopedApps` functions and instrument the
   :meth:`~Template.render` :class:`Template` method, the
   :meth:`~Controller.respond` :class:`Controller` method, and the
   :class:`Response` constructor.

   The sandbox environment is described by the ``TestClient``
   constructor arguments. *users* should be an array of user names to
   use in test requests. *apps* should be an object mapping
   application names to application descriptions. Each description
   should have an ``admin`` property (``string``) and could have a
   ``developers`` property (``Array``).

   .. method:: login(user)

      Log in the user with the name *user*. Subsequent test requests
      will be sent in the name of this user.

   .. method:: logout()

      Log out the logged in user. Subsequent test requests will be
      sent in the name of the anonymous user.

   .. method:: request(request)

      Send a test request to the application. The *request* object
      could have the following properties:

      method
         The request method; defaults to ``'get'``.

      path
         The path of the requested resource; defaults to ``'/'``.

      user
         The name of the user of the request; defaults to the name of
         the logged in user or an empty string if nobody is logged in.

      get
         An object mapping GET parameter names to their values;
         defaults to ``{}``.

      post
         An object mapping POST parameter names to their values;
         defaults to ``{}``.

      headers
         An object mapping the request header names to their values;
         defaults to ``{}``.

      files
         An object mapping the uploaded file names to
         :class:`TempFile` objects or file paths; defaults to ``{}``.
      
   .. method:: get(request)

      A shortcut for GET requests.
   
   .. method:: post(request)
   
      A shortcut for POST requests.
      
   .. method:: head(request)
   
      A shortcut for HEAD requests.
   
   .. method:: put(request)
   
      A shortcut for PUT requests.
   
   .. method:: del(request)

      A shortcut for DELETE requests.

   ::

      function __main__(request) {
        return (request.user
                ? new Response('Hi, ' + request.user + '!')
                : new Response('Please, log in', http.UNAUTHORIZED));
      }

      var Test = TestCase.subclass(
        {
          setUp: function () {
            this.client = new TestClient(['Bob', 'Alice']);
            this.client.login('Bob');
          },

          testGreeting: function () {
            var response = this.client.get({});
            assertSame(response.status, http.OK);
            assertSame(response.content, 'Hi, Bob!');
            assertSame(this.client.get({user: 'Alice'}).content,
                       'Hi, Alice!');
          },

          testError: function () {
            this.client.logout();
            var response = this.client.get({});
            assertSame(response.status, http.UNAUTHORIZED);
            assertSame(response.content, 'Please, log in');
          }
        });
