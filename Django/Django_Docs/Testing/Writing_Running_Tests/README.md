# Writing and running tests

<hr>

**See also**

The [testing tutorial](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Getting_Started/Tutorial_5#writing-your-first-django-app---part-5), the [testing tools reference](), and the [advanced testing topics]().

<hr>

This document is split into two primary sections. First, we explain how to write tests with Django. Then, we explain how to run them.

## Writing tests

Django's unit tests use a Python standard library module: [`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest). This module defines tests using a class-based approach.

Here is an example which subclasses from [`django.test.TestCase`](https://docs.djangoproject.com/en/4.0/topics/testing/tools/#django.test.TestCase), which is a subclass of [`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase) that runs each test inside a transaction to provide isolation:
```
from django.test import TestCase
from myapp.models import Animal

class AnimalTestCase(TestCase):
    def setUp(self):
        Animal.objects.create(name="lion", sound="roar")
        Animal.objects.create(name="cat", sound="meow")

    def test_animals_can_speak(self):
        """Animals that can speak are correctly identified"""
        lion = Animal.objects.get(name="lion")
        cat = Animal.objects.get(name="cat")
        self.assertEqual(lion.speak(), 'The lion says "roar"')
        self.assertEqual(cat.speak(), 'The cat says "meow"')
```
When you [run your tests](), <!-- below --> the default behavior of the test utility is to find all the test cases (that is, subclasses of [`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase)) in any file whose name begins with `test`, automatically build a test suite out of those test cases, and run that suite.

For more details about [`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest), see the Python documentation.

<hr>

**Where should the tests live?**

The default [`startapp`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-startapp) template creates a `tests.py` file in the new application. This might be fine if you only have a few tests but as your test suite grows, you'll likely want to restructure it into a tests package so you can split your tests into different submodules such as `test_models.py`, `test_views.py`, `test_forms.py`, etc. Feel free to pick whatever organizational scheme you like.

See also [Using the Django test runner to test reusable applications](). <!-- header in the upcoming folder, "Advanced testing" - "Using the Django test runner..." -->

<hr>

:warning: **Warning**: If your tests rely on database access such as creating or querying models, be sure to create your test classes as subclasses of [`django.test.TestCase`](https://docs.djangoproject.com/en/4.0/topics/testing/tools/#django.test.TestCase) rather than [`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase).

Using `unittest.TestCase` avoids the cost of running each test in a transaction and flushing the database but if your tests interact with the database, their behavior will vary based on the order that the test runner executes them. This can lead to unit tests that pass when run in isolation but fail when run in a suite.

<hr>

## Running tests

Once you've written tests, run them using the [`test`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-test) command of your project's `manage.py` utility:
```
$ ./manage.py test
```
Test discovery is based on the `unittest` module's [built-in test discovery](https://docs.python.org/3/library/unittest.html#unittest-test-discovery). By default, this will discover test in any file named "test*.py" under the current working directory.

You can specify particular tests to run by supplying any number of "test labels" to `./manage.py test`. Each test label can be a full Python dotted path to a package, module, `TestCase` subclass, or test method. For instance:
```
# Run all the tests in the animals.tests module
$ ./manage.py test animals.tests

# Run all the tests found within the 'animals' package
$ ./manage.py test animals

# Run just one test case
$ ./manage.py test animals.tests.AnimalTestCase

# Run just one test method
$ ./manage.py test animals.tests.AnimalTestCase.test_animals_can_speak
```
You can also provide a path to a directory to discover tests below that directory:
```
$ ./manage.py test animals/
```
You can specify a custom filename pattern match using the `-p` (or `--pattern`) option, if your test files are named differently from the `test*.py` pattern:
```
$ ./manage.py test --pattern="tests_*.py"
```
If you press `Ctrl-C` while the tests are running, the test runner will wait for the currently running test to complete and then exit gracefully. During a graceful exit, the test runner will output details of any test failures, report on how many tests were run and how many errors and failures were encountered, and destroy any test databases as usual. Thus pressing `Ctrl-C` can be very useful if you forget to pass the [`--failfast`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-test-failfast) option, notice that some tests are unexpectedly failing and want to get details on the failures without waiting for the full test run to complete.

If you do not want to wait for the currently running test to finish, you can press `Ctrl-C` a second time and the test run will halt immediately, but not gracefully. No details of the tests run before the interruption will be reported, and any test databases created by the run will not be destroyed.

<hr>

**Test with warnings enabled**

It's a good idea to run your tests with Python warnings enabled: `python -Wa manage.py test`. The `-Wa` flag tells Python to display deprecation warnings. Django, like many other Python libraries, uses these warnings to flag when features are going away. It also might flag areas in your code that aren't strictly wrong but could benefit from a better implementation.

<hr>