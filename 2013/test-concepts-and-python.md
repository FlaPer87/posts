<!---
$"metadata"$
{
  "md": true,
  "title": "Test strategies for your Python projects",
  "draft": false,
  "slug": "test-concepts-and-python",
  "tags": [
    "python",
    "openstack",
    "marconi"
  ]
}
$"metadata"$
-->

A couple of months ago, we started discussing Marconi's test suite and how it'd have been structured. The first thing to do was figuring out what tests made sense for the project, how we'd have integrated them with OpenStack's infrastructure and how those tests would have been organized in Marconi's source tree.

This is an important step for every project. Getting tests right at the very beginning of your project is necessary to guarantee as much stability, robustness and consistency as possible. Tests, for sure, will grow in terms of quantity, quality and they'll, eventually, evolve to support upcoming features.

Although this post is based on what we decided for Marconi, the concepts and ideas expressed here can definitely be used in other projects.

## What to test?

No matter how straight-forward the answer to this question is, it should never be skipped. Tests are not always the same and they vary depending on the language and the project. For example, you won't test types in statically typed languages but you'd want to do that, eventually, in dynamically typed ones.

As for Marconi, there are 3 types of test suites we'd like to provide.

1. **Unit test**: Used to verify that our code units work as expected. A code unit can be either a function, a class, a usage procedure, etc. These tests are the first thing a developer would run when developing a new feature or fixing a bug. The idea is to test a specific functionality and make sure it works, which means there will be one or more tests for each code unit emulating different calls that verify inputs, outputs, side effects and whatever it is suppose to do.

2. **Functional Tests**: These tests work as a black-box test. Ideally, there'll be a real environment - meaning that all required pieces to properly run live tests are installed - where Marconi's interoperability with its many components will be tested. Marconi runs an instance of marconi-server with the default configurations that will allow for testing the API, however, it is also possible to pass a different configuration to it and make it talk to a remote marconi-server or even just a remote backend instance.

3. **Load tests**: These tests verify Marconi's performance under heavy workloads. In Marconi's case, they won't run along with all other tests nor in OpenStack's CI gates. These tests are meant to be run manually by the interested parts - Marconi contributors, vendors, etc - and they should, eventually, be run on different environments in order to benchmark them.

## Tests Structure

Every language has its own standard with regard to where tests should live. In Go, for example, it's a standard to have tests in the [same package](http://golang.org/doc/code.html#Testing) where the code unit is but in a separate file suffixed with _test (mod.go and mod_test.go). In Rust, instead, it's good to either have tests defined in the crate they're testing within the module itself in a separate one or even in a separate crate. Going back to Python, it is a common standard to keep tests in a `tests` package within the package to test.

Organization is as important for tests as it is for your code. For example, keeping tests within their own package will make the inclusion or exclusion of tests easier. As for other places in an application, it can be useful to have a base class that other test suites can inherit from or even just an utility package with functions that are meant to be used throughout the test suite.

Depending on how big your application is or its structure, tests can be organized differently, for example, in a Django project, the best thing to do would be to have a test package per django app but, what if you want to share some code between your apps? A way to do that would be having a django app containing your common code for tests but, this post is not about Django, so back on topic.

Most OpenStack projects have their tests within the same test package and under the project 'src' package.

    |- nova/
    |   - tests/ # Everything is in here.
    |- README
    |- ...

As for Marconi, we decided to split that into 2 packages, pretty much like this:

    marconi
    |     |__tests # Test Classes
    |             |__base.py
    |             |__helpers.py
    |
    tests # Test Cases
    |    |__unit
    |           |__storage
    |           |__transport
    |           |__common
    |           |__test_decorators.py
    |           |__test_bootstrap.py
    |           |__test_config.py
    |
    |    |__functional
    |           | __wsgi
    |              | __v1
    |                    |__test_queue.py
    |                    |__test_messages.py
    |                    |__test_claims.py
    |    |__etc
    |          |__*.conf
    |__README.rst
    | .....


As shown above, we split actual tests from test base classes. Some of the reasons are:

* Test base classes and utilities are not suppose to be tested - meaning they are not actual tests - nor discovered.

* It makes packagers life easier. When creating a package, it won't be necessary to exclude the tests folder since it doesn't contains tests but code we do want to ship.

* Third-party modules can take advantage of Marconi's test suite. People writing their own modules will be able to import the test suite for the module they're working on - either it's a frontend or backend module - and make sure the module is compliant with Marconi's API.

## Setup and tear down.

It's commonly ignored how important it is to have a fast setup and tear down process. If your test suite stays small, you might not notice it but while your test suite keeps growing this becomes more and more critical.

A good test suite should share as much resources as possible between the sub units. This resources should be initialized during the setup process - there are a setUp method and a setUpClass classmethod that can be used - and be cleaned up in the tear down process - just like for the setup process, there are a setUp method and a setUpClass classmethod. The different between setUp and setUpClass is that the former will be called before each test is executed while the later will be called once per test case.

Some things you may want to avoid running multiple times in your test suite are:

### Database synchronization:

This can be a very expensive task depending on how big your schema is but it is definitely worth to avoid nevertheless. A good way to avoid synchronizing the database in every test is creating a "base database" and then copying it for every test needing it. if the schema is not big, it could be worth truncating all tables instead.

### Embedded services:

In order to test Marconi's API functionally, it is necessary, unless there's a remote instance running, to execute and embedded instance of marconi-server as a child process. Launching a server is an expensive task for tests, even more if it requires a database synchronization.

### Don't use globals

This is not exactly about running things multiple times but, it's definitely a good practice.

Don't use common data between tests, instead, generate it during the setUp process or in the test itself. For example, in Marconi, every queue test generates a random queue name to use. Sharing the queue name would break tests isolation - will talk about this later - and might cause conflicts between them.

## Tests Isolation

Don't, ever, make your tests depend on each other. Every test should be an isolated unit and each one of them should be capable of running without any dependency on other tests. People may argue saying that creating dependencies can be good, and that in some cases it is worthless testing B if A fails, which is not a good enough excuse to break tests isolation. In a case like this, a test for a resource lifecycle should be written.

The main advantage of keeping tests isolated is the possibility to parallelize their execution. There are plenty of tools to do this, one of them is [testrepository](https://testrepository.readthedocs.org), which is currently being used in OpenStack. Although we're currently not using it right now, we're totally looking forward to that.

## Tools and libs

There are many libraries for testing. Some of them extend test classes, others tests execution and others help with replicating tests on different environments. The ones we're currently using in Marconi are:

* [Testools](https://pypi.python.org/pypi/testtools): Extensions to the Python standard unittest framework.
* [nose](http://nose.readthedocs.org/en/latest/): Extends unittest to ease tests' discovery, loading and execution.
* [tox](http://testrun.org/tox/latest/): Testing virtualenv manager. Makes sure tests pass on different Python environments.

## Closing

There's still a huge road to go through, however, the above states the current, on-going, test refactoring we're working on Marconi. If you've any comment or feedback, please, leave a comment or contribute back.

Some things I'd like to highlight and leave as take aways from this post are:

* Tests are not just meaningful for the app itself, they are also important for future contributions and implementations.

* Tests performance is as important as app's performance.

* Share resources throughout your test suite as much as possible.

* Don't, by any means, break test isolation.