# Testing in Django

Automated testing is an extremely useful bug-killing tool for the modern web developer. You can use a collection of tests -- a **test suite** -- to solve, or avoid, a number of problems:

* When you're writing new code, you can use tests to validate your code works as expected.
* When you're refactoring or modifying old code, you can use tests to ensure your changes haven't affected your application's behavior unexpectedly.

Testing a web application is a complex task, because a web application is made of several layers of logic -- from HTTP-level request handling, to form validation and processing, to template rendering. With Django's test-execution framework and assorted utilities, you can simulate requests, insert test data, inspect your application's output and generally verify your code is doing what it should be doing.

The preferred way to write tests in Django is using the [`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest) module built-in to the Python standard library. This is covered in detail in the [Writing and running tests](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Writing_Running_Tests#writing-and-running-tests) document.

You can also use any *other* Python test framework; Django provides an API and tools for that kind of integration. They are described in the [Using different testing frameworks](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Advanced_Testing#using-different-testing-frameworks) section of [Advanced testing topics](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Advanced_Testing#advanced-testing-topics).

* [Writing and running tests](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Writing_Running_Tests#writing-and-running-tests)
* [Testing tools](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#testing-tools)
* [Advanced testing topics](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Advanced_Testing#advanced-testing-topics)

<hr>

[[Previous module: Managing files]](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Managing_Files#managing-files) - [[Next module: User authentication in Django]](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication#user-authentication-in-django)