# Form handling with class-based views

Form processing generally has 3 paths:

* Initial `GET` (blank or prepopulated form)
* `POST` with invalid data (typically redisplay form with errors)
* `POST` with valid data (process the data and typically redirect)

Implementing this yourself often results in a lot of repeated boilerplate code (see [Using a form in a view](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Working_with_Forms#the-view)). To help avoid this, Django provides a collection of generic class-based views for form processing.