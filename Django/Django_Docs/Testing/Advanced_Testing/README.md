# Advanced testing topics

## The request factory

##### `class RequestFactory`

The `RequestFactory` shares the same API as the test client. However, instead of behaving like a browser, the `RequestFactory` provides a way to generate a request instance that can be used as the first argument to any view. This means you can test a view function the same way as you would test any other function -- as a black box, with exactly known inputs, testing for specific outputs.