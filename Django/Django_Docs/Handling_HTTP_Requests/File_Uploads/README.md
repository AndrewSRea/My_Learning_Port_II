# File uploads

When Django handles a file upload, the file data ends up placed in [`request.FILES`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.FILES) (for more on the `request` object, see the documentation for [request and response objects](https://docs.djangoproject.com/en/4.0/ref/request-response/)). This document explains how files are stored on disk and in memory, and how to customize the default behavior. 

<hr>

:warning: **Warning**: There are security risks if you are accepting uploaded content from untrusted users! See the security guide's topic on [User-uploaded content](https://docs.djangoproject.com/en/4.0/topics/security/#user-uploaded-content-security) for mitigation details. <!-- possible future file? -->

<hr>

## Basic file uploads

Consider a form containing a [`FileField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.FileField):

`forms.py`
```
from django import forms

class UploadFileForm(forms.Form):
    title = forms.CharField(max_length=50)
    file = forms.FileField()
```

A view handling this form will receive the file data in [`request.FILES`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.FILES), which is a dictionary containing a kew for each `FileField` (or [`ImageField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.ImageField), or other `FileField` subclass) in the form. So the data from the above form would be accessible as `request.FILES['file']`.

Note that `request.FILES` will only contain data if the request method was `POST`, at least one file field was actually posted, and the `<form>` that posted the request has the attribute `enctype="multipart/form-data"`. Otherwise, `request.FILES` will be empty.

Most of the time, you'll pass the file data from `request` into the form as described in [Binding uploaded files to a form](https://docs.djangoproject.com/en/4.0/ref/forms/api/#binding-uploaded-files). This would look something like:

`views.py`
```
from django.http import HttpResponseRedirect
from django.shortcuts import render
from .forms import UploadFileForm

# Imaginary function to handle an uploaded file.
from somewhere import handle_uploaded_file

def upload_file(request):
    if request.method == 'POST':
        form = UploadFileForm(request.POST, request.FILES)
        if form.is_valid():
            handle_uploaded_file(request.FILES['file'])
            return HttpResponseRedirect('/success/url/')
        else:
            form = UploadFileForm()
        return render(request, 'upload.html', {'form': form})
```

Notice that we have to pass `request.FILES` into the form's constructor; this is how file data gets bound into a form.

Here's a common way you might handle an uploaded file:
```
def handle_uploaded_file(f):
    with open('some/file/name.txt', 'wb+') as destination:
        for chunk in f.chunks():
            destination.write(chunk)
```
Looping over `UploadedFile.chunks()` instead of using `read()` ensures that large files don't overwhelm your system's memory.

There are a few other methods and attributes available on `UploadedFile` objects; see [`UploadedFile`](https://docs.djangoproject.com/en/4.0/ref/files/uploads/#django.core.files.uploadedfile.UploadedFile) for a complete reference.

### Handling uploaded files with a model

If you're saving a file on a [`Model`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model) with a [`FileField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.FileField), using a [`ModelForm`](https://docs.djangoproject.com/en/4.0/topics/forms/modelforms/#django.forms.ModelForm) makes this process much easier. The file object will be saved to the location specified by the [`upload_to`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.FileField.upload_to) argument of the corresponding `FileField` when calling `form.save()`:
```
from django.http import HttpResponseRedirect
from django.shortcuts import render
from .forms import ModelFormWithFileField

def upload_file(request):
    if request.method == 'POST':
        form = ModelFormWithFielField(request.POST, request.FILES)
        if form.is_valid():
            # file is saved
            form.save()
            return HttpResponseRedirect('/success/url/')
    else:
        form = ModelFormWithFileField()
    return render(request, 'upload.html', {'form': form})
```
If you are constructing an object manually, you can assign the file object from [`request.FILES`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.FILES) to the file field in the model:
```
from django.http import HttpResponseRedirect
from django.shortcuts import render
from .forms import UploadFileForm
from .models import ModelWithFileField

def upload_file(request):
    if request.method == 'POST'
        form = UploadFileForm(request.POST, request.FILES)
        if form.is_valid():
            instance = ModelWithFileField(file_field=request.FILES['file'])
            instance.save()
            return HttpResponseRedirect('/success/url/')
    else:
        form = UploadFileForm()
    return render(request, 'upload.html', {'form': form})
```
If you are constructing an object manually outside of a request, you can assign a [`File`](https://docs.djangoproject.com/en/4.0/ref/files/file/#django.core.files.File) like object to the [`FileField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.FileField):
```
from django.core.management.base import BaseCommand
from django.core.files.base import ContentFile

class MyCommand(BaseCommand):
    def handle(self, *args, **options):
        content_file = ContentFile(b'Hello world!', name='hello-world.txt')
        instance = ModelWithFileField(file_field=content_file)
        instance.save()
```

### Uploading multiple files

If you want to upload multiple files using one form field, set the `multiple` HTML attribute of field's widget:

`forms.py`
```
from django import forms

class FileFieldForm(forms.Form):
    file_field = forms.FileField(widget=forms.ClearableFileInput(attrs={'multiple': True}))
```
Then override the `post` method of your [`FormView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-editing/#django.views.generic.edit.FormView) subclass to handle multiple file uploads:

`views.py`
```
from django.views.generic.edit import FormView
from .forms import FileFieldForm

class FileFieldFormView(FormView):
    form_class = FileFieldForm
    template_name = 'upload.html'   # Replace with your template
    success_url = '...'             # Replace with your URL or reverse()

    def post(self, request, *args, **kwargs):
        form_class = self.get_form_class()
        form = self.get_form(form_class)
        files = request.FILES.getlist('file_field')
        if form.is_valid():
            for f in files:
                ...   # Do something with each file
            return self.form_valid(form)
        else:
            return self.form_invalid(form)
```
