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
$ pipenv install django==2.1
```

## NOTE: pip 18.1 is buggy! Current fix (immediately after the previous step) is this:

```
$ pip3 install pipenv
$ pipenv run pip install pip==18.0
$ pipenv install
```

## Initial Project Setup (Continued)

```
$ pipenv install pylint
$ pipenv shell
(projdir) $ django-admin startproject my_project .
(projdir) $ python manage.py migrate
(projdir) $ python manage.py createsuperuser
(projdir) $ python manage.py runserver
```

Suggestion: Change the URL of the admin site in `my_project/urls.py`.

## Adding a Django app to the project

```
(projdir) $ python manage.py startapp products
```

2. Add the new app to `INSTALLED_APPS` list in `my_project/settings.py`.

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
    'products.apps.ProductsConfig',
]
```

## Creating a Model for the project

1. Go to `products/models.py` and make model:

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

3. Register the model in `products/admin.py`.

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

2. Add to `INSTALLED_APPS` and add `REST_FRAMEWORK` object in `my_project/settings.py`:

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

First, create paths for app (create `products/urls.py`):

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

## Allow Cross-Origin Resource Sharing (CORS)

1. Install package

```
(projdir) $ pipenv install django-cors-headers==2.4.0
```

2. Update maindir/settings.py
   2a. Add `corsheaders` in `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
  ...
  # 3rd party
  'rest_framework',
  'corsheaders',
  ...
]
```

2b. Add middlewares to the front of the `MIDDLEWARE` list:

```python
MIDDLEWARE = [
  'corsheaders.middleware.CorsMiddleware',
  'django.middleware.common.CommonMiddleware',
  ...
]
```

2c. Add `CORS_ORIGIN_WHITELIST`:

```python
CORS_ORIGIN_WHITELIST = (
  'mydomain.com' # Whatever domain
)
```

## Setting Permissions

#### View-Level Permissions

1. In the views.py file, import `permissions` from the `rest_framework` package.

```python
from rest_framework import generics, permissions
```

2. Add `permissions_classes` member variable to a view class.

```python
class ProductList(generics.ListCreateAPIView):
  permissions_classes = (permissions.IsAuthenticated,)
```

#### Project-Level Permissions

1. Add permissions settings in setting.py, under the `REST_FRAMEWORK` object and in the `'DEFAULT_PERMISSION_CLASSES'` list.

```python
# project_dir/settings.py
...
REST_FRAMEWORK = {
  'DEFAULT_PERMISSION_CLASSES': [
    'rest_framework.permissions.IsAuthenticated',
  ]
}
```

Built-in project-level permissions settings include:

- AllowAny
- IsAuthenticated
- IsAdminUser
- IsAuthenticatedOrReadOnly

#### Custom permissions

1. In file where custom permissions class is defined (Suggestion: make a permissions.py file), import `permissions` package:

```python
from rest_framework import permissions
```

2. Declare a class that extends `permissions.BasePermission`. Example:

```python
class IsOwnerOrReadOnly(permissions.BasePermission):
```

3. Override boolean methods `has_permission()` or `has_object_permission()`

```python
def has_object_permission(self, request, view_obj):
  # Read-only permissions allowed for SAFE_METHODS (GET, OPTIONS, HEAD)
  if request.method in permissions.SAFE_METHODS:
    return True

  # Write permissions for owner
  return obj.owner == request.user
```

## User Authentication

#### Basic Authentication

1. Client makes HTTP request
2. Server responds with 401 status (`Unauthorized`) and `www-Authenticate` HTTP header
3. Client sends credentials back with Authorization HTTP header
4. Server checks credentials; responds with 200 OK or 403 Forbidden code.
   `+` Simple
   `-` Must send credentials for every request
   `-` Insecure

#### Session Authentication

1. User enters credentials (logs in).
2. Server verifies credentials.
3. Server creates session object; stores in database.
4. Server sends client session ID; client stores as cookie.
5. When user logs out, session ID destroyed by client and server.
   `+` Credentials sent only once.
   `+` More efficient lookup for session ID.
   `-` Session ID only valid in browser where logged in.
   `-` Cookie sent out for every request, even no authorization required ones.

#### Token Authentication

1. User sends credentials to server.
2. Unique token generated and stored by client as cookie or local storage.
3. Token passed in header of each HTTP request.
4. Server verifies token validity to see if user is authenticated.
   `+` Tokens stored only in client.
   `+` Token can be shared by multiple front-ends.
   `-` Tokens can become large.
   `-` Token usually contains all user info.

### Setting Up Token Authentication

1. In settings.py, add `'DEFAULT_AUTHENTICATION_CLASSES'` list to the `REST_FRAMEWORK` object:

```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ]
}
```

- NOTE: `SessionAuthentication` needed for using Browsable API.

2. Add `'rest_framework.authtoken'` to `INSTALLED_APPS` in settings.py:

```python
INSTALLED_APPS = [
  ...
  # 3rd party
  'rest_framework',
  'rest_framework.authtoken',
  ...
]
```

3. Sync the database with `migrate` command.

```
(projdir) $ python manage.py migrate
(projdir) $ python manage.py runserver
```

## Setting Up Authentication APIs (using django-rest-auth)

1. Install packages via command line

```
(projdir) $ pipenv install django-rest-auth==0.9.3
```

2. Add to `INSTALLED_APPS` in settings.py

```python
INSTALLED_APPS = [
  ...
  # 3rd party
  'rest_framework',
  'rest_framework.authtoken',
  'rest_auth',
  ...
]
```

3. Include `'rest_auth.urls'` to project's urls.py.

```python
urlpatterns = [
  ...
  path('api/v1/rest-auth/', include('rest_auth.urls')),
]
```

## Setting Up User Registration APIs (using the django-allauth package)

1. Command line install:

```
(projdir) $ pipenv install django-allauth==0.37.1
```

2. Add multiple configs to the `INSTALLED_APPS` list in settings.py

```python
INSTALLED_APPS = [
  ...
  'django.contrib.sites',

  # 3rd party
  'rest_framework',
  'rest_framework.authtoken',
  'allauth', ##
  'allauth.account', ##
  'allauth.socialaccount', ##
  'rest_auth',
  'rest_auth.registration', ##
  ...
]
...
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

SITE_ID = 1
```

3. Add url route in `projdir/urls.py`.

```python
urlpatterns = [
  ...
  path('api/v1/rest-auth/registration/', include('rest_auth.registration.urls')),
]
```

## Implementing Viewsets (e.g., improving on views)

1. In an app's views.py, import `viewsets` from the `rest_framework` package.

```python
from rest_framework import viewsets
```

(We are additionally no longer requiring the importing of `generics` or `permissions`.)

2. Create view class that extends `viewset.ModelViewSet`.

```python
class ProductViewSet(viewset.ModelViewSet):
  permission_classes = (IsAuthorOrReadOnly,)
  queryset = Product.objects.all()
  serializer_class = ProductSerializer
```

## Implementing Routers (e.g., improving of url patterns)

1. Import `SimpleRouter` from the `rest_framework.routers` package.

```python
from rest_framework.routers import SimpleRouter
```

2. Import the Viewset classes (we also no longer import the old View classes).

```python
from .views import ProductViewSet
```

3. Create an instance of `SimpleRouter`, register routes for the `ViewSet`s, and then set the `urlpatterns` variable to the router's urls.

```python
router = SimpleRouter()
router.register('products', ProductViewSet, base_name='products')

urlpatterns = router.urls
```
