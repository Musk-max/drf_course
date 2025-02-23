## Steps/Commands
>Note: Please 'cd' into the root directory and fire up your virtual environment!

1) New app - We will apply token authentication on our ecommerce endpoints. However, we haven't created the app. Go ahead and create an ecommerce app with the following command in the docker desktop app container exec page it will then appear on your local machine because of volumes( mk R).

```
python manage.py startapp ecommerce
```
2) Settings - Open /drf_course/settings.py and replace the REST_FRAMEWORK with the following code. Notice the new DEFAULT_AUTHENTICATION_CLASSES.

```
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'rest_framework_json_api.exceptions.exception_handler',
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework_json_api.parsers.JSONParser',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework_json_api.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer'
    ),
    'DEFAULT_METADATA_CLASS': 'rest_framework_json_api.metadata.JSONAPIMetadata',
    'DEFAULT_FILTER_BACKENDS': (
        'rest_framework_json_api.filters.QueryParameterValidationFilter',
        'rest_framework_json_api.filters.OrderingFilter',
        'rest_framework_json_api.django_filters.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
    ),
    'SEARCH_PARAM': 'filter[search]',
    'TEST_REQUEST_RENDERER_CLASSES': (
        'rest_framework_json_api.renderers.JSONRenderer',
    ),
    'TEST_REQUEST_DEFAULT_FORMAT': 'vnd.api+json'
}
```

Now change INSTALLED_APPS to the following.
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django_extensions',
    'django_filters',
    'rest_framework',
    'rest_framework.authtoken', #Used to enable token authentication
    'core',
    'ecommerce', #New app
]
```
3) URL's - We now need to add a new endpoint to our urlconf file. Replace /drf_course/urls.py with the following code.

```
from django.urls import path
from django.contrib import admin
from core import views as core_views
from rest_framework import routers
from rest_framework.authtoken.views import obtain_auth_token

router = routers.DefaultRouter()

urlpatterns = router.urls

urlpatterns += [
    path('admin/', admin.site.urls),
    path('contact/', core_views.ContactAPIView.as_view()),
    path('api-token-auth/', obtain_auth_token), #gives us access to token auth
]
```

4) Migrate - You now need to migrate database changes. Use following code in the docker desktop app container exec page. 
```
python manage.py makemigrations
python manage.py migrate
```
4) Signals - We need a mechanism to create a token for every user that signs up to our app. This token is what will be returned when we call the new endpoint. Go ahead and create a new file in /ecommerce and call it signals.py.
Use the following code in the new file.

```
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

@receiver(post_save, sender=User, weak=False)
def report_uploaded(sender, instance, created, **kwargs):
    if created:
        Token.objects.create(user=instance)
```
Now open /ecommerce/app.py and add the following code it's just ready() that's been created.
```
from django.apps import AppConfig


class EcommerceConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'ecommerce'

    def ready(self):
        import ecommerce.signals
```
5) Create a user - Go ahead and create a new superuser. This will server 2 purposes. We we gain access to the built in Django admin page and we will also create a new token. Open a new terminal and use the following code in the docker desktop app container exec page.
```
python manage.py createsuperuser
```
Add a bobby, none and 1234.

6) Call the endpoint - You can call the new endpoint with the following commands in the docker desktop app container exec page. With any luck, you will receive a new token ID in the response.
http post http://api:8000/api-token-auth/ username=bobby password=1234
