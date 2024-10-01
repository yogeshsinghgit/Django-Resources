## Reference Links

[python-dotenv](https://pypi.org/project/python-dotenv/)

[Rest Framework](https://www.django-rest-framework.org/)

[Cors](https://pypi.org/project/django-cors-headers/)

[Djoser](https://djoser.readthedocs.io/en/latest/index.html) 

[Celery docs](https://docs.celeryq.dev/en/stable/django/first-steps-with-django.html#using-celery-with-django)




## Create Virtual Environement

* #### To create Venv
``` cmd
python3 -m venv venvName
```

* #### To activate Venv
``` cmd
source venvName/bin/activate
```



## Required Modules/Libraries/Packages:

* ```pip install python-dotenv```
  
* ```pip install psycopg2``` # if using postgresql db

* ```pip install django```

* ```pip install djangorestframework```

* ```pip install django-cors-headers```

* ```pip install djoser```

* ```pip install celery[redis]```



## Databse connection in settings.py

``` python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'DbName',
        'USER': 'postgres',
        'PASSWORD': '********',
        'HOST': 'localhost',  # Or the address of your PostgreSQL server
        'PORT': '5432',       # Default PostgreSQL port
    }
}
```

## To load Env Variables.

[Reference](https://pypi.org/project/python-dotenv/)

``` python
from dotenv import load_dotenv

load_dotenv()  # take environment variables from .env.

# Code of your application, which uses environment variables (e.g. from `os.environ` or
# `os.getenv`) as if they came from the actual environment.

## Or

from dotenv import dotenv_values

config = dotenv_values(".env")  # config = {"USER": "foo", "EMAIL": "foo@example.org"}

```

## Implementing Djoser for Authentication

### Step 1. Update settings.py

``` python
# settings.py

INSTALLED_APPS = [
    # ...
    'rest_framework',
    'djoser',
    'rest_framework.authtoken',  # If you're using token authentication
    # ...
]

# Specify the user model if you're using a custom user model
AUTH_USER_MODEL = 'yourapp.CustomUser'  # Replace with your user model

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',  # or other auth classes
    ),
}

# DJOSER = {
#     'SERIALIZERS': {
#         'user_create': 'yourapp.serializers.CustomUserCreateSerializer',  # Custom user creation serializer
#         'user': 'yourapp.serializers.CustomUserSerializer',  # Custom user detail serializer
#     },
# }

# Djoser settings (optional)
# DJOSER = {
#     'LOGIN_FIELD': 'email',  # Specify the login field (default is 'username')
#     # You can add more settings here as needed
# }
```

### Step 2. Add URLs
you need to include Djoser URLs in your project. You can do this by adding the following to your urls.py:
```python
# main/urls.py
# urls.py

from django.urls import path, include

urlpatterns = [
    # Other paths...
    path('api/auth/', include('djoser.urls')),
    path('api/auth/', include('djoser.urls.authtoken')),  # For token authentication
]
```
### Step 3. Djoser Endpoints

5. Use Djoser Endpoints

You can now use the Djoser endpoints for user authentication and management. Here are some of the common endpoints provided by Djoser:

    * Registration: POST /api/auth/register/
    * Login: POST /api/auth/login/
    * Logout: POST /api/auth/logout/
    * Password reset: POST /api/auth/reset/
    * Change password: POST /api/auth/users/set_password/
    * User details: GET /api/auth/users/me/


**Note:** In this project I have used custom url endpoints to register user and login user.
* For Login http://127.0.0.1:8000/api/auth/jwt/create/
* For Refresh : http://127.0.0.1:8000/api/auth/jwt/refresh


## Implementing Celery

### Step 1. Create a Celery Configuration File. 
    Create a 'celery.py' file inside your main project directory

``` python
# MainProject/celery.py

from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# Set default Django settings module for 'celery'
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'my_project.settings')

# Create Celery app instance
app = Celery('my_project')

# Load task settings from Django settings file
app.config_from_object('django.conf:settings', namespace='CELERY')

# Auto-discover tasks from all registered Django app configs
app.autodiscover_tasks()

```

### Step 2. Update  `__init__.py` in the Project Directory.

``` python
from .celery import app as celery_app

__all__ = ('celery_app',)

```

### Step 3. Configure Celery in Django Settings.

``` python
# MainProject/settings.py

# Redis as Celery Broker
CELERY_BROKER_URL = 'redis://localhost:6379/0'

# Store task results
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'

# Task settings
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'

```

### Step 4. Create a Task File.
Create a `tasks.py` file inside one of your Django apps and define a sample task.

``` python
# myapp/tasks.py
from celery import shared_task
from time import sleep

@shared_task
def add(x, y):
    sleep(5)  # Simulating a long-running task
    return x + y

```

### Step 5. Use Celery Task in view.

``` python
# myapp/views.py
from rest_framework.response import Response
from rest_framework.views import APIView
from .tasks import add

class CalculateView(APIView):
    def get(self, request):
        # Call the Celery task
        task = add.delay(10, 20)
        return Response({"task_id": task.id, "status": "Task started!"})

```

### Step  6. Run Celery Worker.
Run the Celery worker in a separate terminal window.

``` cmd
celery -A my_project worker --loglevel=info
```
