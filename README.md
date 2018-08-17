# django-rest-api-cheatsheet
A simple cheatsheet for a Django REST API backend.

## Development Environment (Mac OSX)
1. Install Python 3
```
$ brew install python3
```
2. Install pipenv
```
$ pip3 install pipenv
```

## Initial Project Setup
```
$ mkdir projdir && cd projdir
$ pipenv install django==2.0
$ pipenv install pylint
$ pipenv shell
(projdir) $ django-admin startproject my_project .
(projdir) $ python manage.py migrate
(projdir) $ python manage.py createsuperuser
(projdir) $ python manage.py runserver
```
Suggestion: Change the URL of the admin site in ```my_project/urls.py```.

## Adding a Django app to the project
```
(projdir) $ python manage.py startapp products
```
2. Add the new app to ```INSTALLED_APPS``` list in ```my_project/settings.py```.
```python
# my_project/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Local
    'products',
]
```

## Creating a Model for the project
1. Go to ```products/models.py``` and make model:
```python
# products/models.py
from django.db import models
from django.contrib.auth.models import User # if you need user here...

class Product(models.Model):
  created_by = models.ForeignKey(User, on_delete=models.CASCADE)
  name = models.CharField(max_length=250)
  description = model.TextField()
  image = models.ImagesField(uploaf_to='dir/of/image/')
  created_at = models.DateTimeField(auto_now_add=True)

  def __str__(self):
    return self.name
```
(See https://docs.djangoproject.com/en/2.1/ref/models/fields/ for model fields.] 

2. Create migration file (make sure you add the app name at the end).
```
(projdir) $ python manage.py makemigrations products
(projdir) $ python manage.py migrate
```

3. Register the model in ```products/admin.py```.
```python
# products/admin.py
from django.contrib import admin
from .models import Product # Added in this step

admin.site.register(Product) # Added in this step
```

## Setting up Django REST Framework
```unix
(projdir) $ pipenv install djangorestframework==3.8.2
```
2. Add to ```INSTALLED_APPS``` and add ```REST_FRAMEWORK``` object in ```my_project/settings.py```:
```python
# my_project/settings.py
...
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # 3rd-party
    'rest_framework', # Added in this step
    # Local
    'products', 
]

REST_FRAMEWORK = { # Added in this step
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny', # Just for now...
    ]
}
...
```

## Setting up APIs for an App
1. Create serializers for the models in the app.

```python
# products/serializers.py (a new file)
from rest_framework import serializers
from . models import Product

class ProductSerializer(serializers.ModelSerializer):

  class Meta:
    fields = {'id', 'created_by', 'name', 'description', 'image', 'created_at', }
    model = Product
```

2. Set up URLs for the app.

First, create paths for app (create ```products/urls.py```):

```python
# products/urls.py -- new file added

from . views import ProductList, ProductDetail

urlpatterns = [
  path('', ProductList.as_view()),
  path('<int:pk>/' ProductDetail.as_view()),
]
```

Then, create path in project-level URLS file:
```python
# my_project/urls.py
from django.contrib.import admin
from django.urls import path, include # 'include' added in this step

urlpatterns = [
  path('admin/', admin.site.urls),
  path('api/v1/products/', include('products.urls')) # Added in this step
]
```

3. Set up the views for the app:
```python
# products/views.py
from rest_framework import generics # Added

from . models import Product # Added
from . serializers import ProductSerializer # Added

class ProductList(generics.ListCreateAPIView):
  queryset = Product.objects.all()
  serializer_class = ProductSerializer

class ProductDetail(generics.RetrieveUpdateDestroyAPIView):
  queryset = Product.objects.all()
  serializer_class = ProductSerializer
```
