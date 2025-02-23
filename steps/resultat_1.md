# Introduction
    Date: 22/02/2025
# 1-Installer Docker desktop
    Docker desktop installé à partir du site officiel de docker a automatiquement installé wsl et les autres : il fonctionnement correctement sans rien ajouter;
# 2-Clone and setup our system
    1-J'ai forké le projet sur github from bobby-didcoding/drf_course
    2- j'ai cloné le fork: (Pour récupérer la structure d'un projet drf standarde et l'application déjà réalisée en front + la bd)
        git clone (https://github.com/Musk-max/drf_course.git)
    3-j'ai suivi les instructions de setup de module_1.md
        (Pour créer(venv,projet django, et .env) et installer tout ce qui est nécessaire (modules))

        1-créer et activer un environnement virtuel
        2-installer touts les thirds parties package ou module à partir du fich requirement.txt
            hérité de github ( après avoir fait cela, tu pourras utilisé les commandes des packets concernés 
            tel que django dans la suite)
        3-lancer la cmd django pour creer notre nouveau projet : drf_course dans: backend ( confirmercette interpretation avec la doc)
        4-créer un fichier .env à partir de: env.template pour stocker les informations de notre environnement de travail courant:
            Vous pouvez utiliser votre nouveau fichier .env pour stocker les clés API, secret_keys, app_passwords et vous y aurez accès dans l'application Django.
       
    4- J'ai suivi les instructions de module_2.md: (Pour configurer les urls de notre projet et lancer notre serveur local)
            Nous aurons 2 applications dans notre projet :
            La première sera core. Cette application contiendra la logique de contact us endpoint .
            La seconde sera ecommerce. Cette application contiendra la logique de nos endpoints pour les articles et les commandes.
            Allons-y et créons l'application principale.
          0- Se mettre dans le root directory (le grand drf_course) et activer le venv
          2-créer une l'application core dans un nouveau terminal (si on veut):
            python manage.py startapp core

           ## LES 4 CONFIGURATIONS DANS SETTING.PY
          3-aller dans settings.py et faire remplacer tous les imports par les suivants:
            from pathlib import Path
            from dotenv import load_dotenv
            import os
            load_dotenv()
          4-Dans le meme fichier : remplacer les variables présentes par le bloc suivant:
            SECRET_KEY = os.environ.get("SECRET_KEY")
            DEBUG = int(os.environ.get("DEBUG", default=0))
            ALLOWED_HOSTS = os.environ.get("DJANGO_ALLOWED_HOSTS").split(" ")
          5-Enregistrer notre nouvelle application: core dans installed_apps dans le fichier settings.py de petit drf_course
            Installed apps deviendra ainsi
                        ```
            INSTALLED_APPS = [
                'django.contrib.admin',
                'django.contrib.auth',
                'django.contrib.contenttypes',
                'django.contrib.sessions',
                'django.contrib.messages',
                'django.contrib.staticfiles',
                'django_extensions', #Great packaged to  access abstract models
                'django_filters', #Used with DRF
                'rest_framework', #DRF package
                'core', # New app
            ]
            ```
            6- Toujours dans settings.py, a la fin de ce fichier ajouter les variables qui vont permettre a django rest framework
                d'activer ses fonctionnalités dans nos applications, ces variables sont:

                REST_FRAMEWORK = {
                    'EXCEPTION_HANDLER': 'rest_framework_json_api.exceptions.exception_handler',
                    'DEFAULT_PARSER_CLASSES': (
                        'rest_framework_json_api.parsers.JSONParser',
                    ),
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
            7- Designons nos urls de l'application core en créant un fichier urls.py ou ouvrant s'il existe deja dans           drf_course et en ajoutant le snippet (extrait) suivant:
                ```
                    from django.urls import path
                    from django.contrib import admin
                    from rest_framework import routers

                    router = routers.DefaultRouter()

                    urlpatterns = router.urls

                    urlpatterns += [
                        path('admin/', admin.site.urls),
                    ]
                ```
            8-Créer et executer les migrations :
                > Note: We are using the default db.sqlite3 database.
                ```
                    python manage.py makemigrations
                    python manage.py migrate
                ```    
            9-lancer un local developement server:
                ```
                    python manage.py runserver
                ```
                * Our DRF course API interface is accessible at [http://localhost:8000](http://localhost:8000)
    5- J'ai suivi les instructions de module_3.md:Pour créer notre premier endPoint d'API :
        Joli et simple... Nous voulons un contact us endpoint pour qu'un utilisateur puisse envoyer son nom, son email et son message à notre backend.   

        Nos besoins:
        a) a model to capture store the incoming data
        b) a router/url called '/contact/'
        c) a serializer to process the data coming from the user and send a response back again.
        d) a view to encapsulates the common REST HTTP method calls

        0- Se mettre dans le root directory (le grand drf_course) et activer le venv
        1)  ouvrir /core/models.py et ajouter le code suivant.
            ```

            from django.db import models
            from utils.model_abstracts import Model
            from django_extensions.db.models import (
                TimeStampedModel, 
                ActivatorModel,
                TitleDescriptionModel
            )

            class Contact(
                TimeStampedModel, 
                ActivatorModel,
                TitleDescriptionModel,
                Model
                ):

                class Meta:
                    verbose_name_plural = "Contacts"

                email = models.EmailField(verbose_name="Email")

                def __str__(self):
                    return f'{self.title}'
            ```  
        2-Créeons notre propre abstract model qui utilise UUID au lieu de l'id field par défaut
           a) Go ahead and make a new directory in /backend called utils
           b) Create a new file called model_abstracts.py and another called __init__.py
           c) Paste the following code into /backend/utils/model_abstracts.py

           ```
            import uuid
            from django.db import models


            class Model(models.Model):
                id = models.UUIDField(primary_key=True, default=uuid.uuid4)

                class Meta:
                    abstract = True
          ```
        3-Serialization et déserialization:
        a) Create a new file called serializers.py in /backend/core
        b) Paste the following code into /backend/core/serializers.py

        ```
        from . import models
        from rest_framework import serializers
        from rest_framework.fields import CharField, EmailField



        class ContactSerializer(serializers.ModelSerializer):

            name = CharField(source="title", required=True)
            message = CharField(source="description", required=True)
            email = EmailField(required=True)
            
            class Meta:
                model = models.Contact
                fields = (
                    'name',
                    'email',
                    'message'
                )
        ```
        4- APIView- pour rediriger les req entrantes à une méthode comme get() ou post()
        Go ahead and paste the following code into /backend/core/views.py

            ```
            from json import JSONDecodeError
            from django.http import JsonResponse
            from .serializers import ContactSerializer
            from rest_framework.parsers import JSONParser
            from rest_framework import views, status
            from rest_framework.response import Response



            class ContactAPIView(views.APIView):
                """
                A simple APIView for creating contact entires.
                """
                serializer_class = ContactSerializer

                def get_serializer_context(self):
                    return {
                        'request': self.request,
                        'format': self.format_kwarg,
                        'view': self
                    }

                def get_serializer(self, *args, **kwargs):
                    kwargs['context'] = self.get_serializer_context()
                    return self.serializer_class(*args, **kwargs)

                def post(self, request):
                    try:
                        data = JSONParser().parse(request)
                        serializer = ContactSerializer(data=data)
                        if serializer.is_valid(raise_exception=True):
                            serializer.save()
                            return Response(serializer.data)
                        else:
                            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
                    except JSONDecodeError:
                        return JsonResponse({"result": "error","message": "Json decoding error"}, status= 400)

            ```
            5-les urls: Le framework REST ajoute la prise en charge du routage automatique des URL à Django et relie la logique de votre vue à un ensemble d'URL.
            go ahead and replace the code in /drf_course/urls.py with the following code.

                ```
                from django.urls import path
                from django.contrib import admin
                from core import views as core_views
                from rest_framework import routers


                router = routers.DefaultRouter()

                urlpatterns = router.urls

                urlpatterns += [
                    path('admin/', admin.site.urls),
                    path('contact/', core_views.ContactAPIView.as_view()),
                ]
                ```

            6) Register - Go ahead and open /core/admin.py and paste in the following code pour enregistrer les nouveaux modèles dans le built in admin page (page d'administration intégrée). 

                ```
                from django.contrib import admin
                from .models import Contact


                @admin.register(Contact)
                class ContactAdmin(admin.ModelAdmin):
                    list_display = ('id', 'title', 'description', 'email')
                ```

            7) Migrations 

                ```
                    python manage.py makemigrations
                    python manage.py migrate

                ```
            8) Appeler nos endpoints - voilà les req que nous pouvons faire sur notre nouveau endpoint.
                >Note: changer 'localhost' en 'api' car on a fait l'appel via Docker Decktop.
                taper: 
                docker-compose up -d --build
                coller cela dans la page exec de Docker Desktop:
                http http://api:8000/contact/ name="Bobby Stearman" message="test" email="bobby@didcoding.com"
            9) Erreurs rencontrées :
                1) the attribute 'version' is obsoleted:
                    On supprimer la ligne 'version' dans le docker-compose.yml
                2) package 'netcat' has no installation candidate:
                    Dans tous les fichiers de \docker\docker_files\ :
                        changer 'netcat' en 'netcat-tradional' 
                3) Could not find a version that satifies the requirement install (from version : none)
                4) No matching distribution found for install
                pip install --upgrade pip \, mettre un point virgule après pip dans le Dockerfile_app, ca donne:
                pip install --upgrade pip ;\
                cela résoud les deux problèmes précédents quand on refait à nouveau:
                docker-compose up -d --build
                5) Le conteneur api-1 ne démarrait pas alors que app-1 démarrait
                    * Dans Docker Desktop: on clique sur api-1, on regarde les logs: 
                        l'erreur est : entryPoint.sh no suchfile or directory 
                    Le problème:
                        la linge: "#!/bin/sh" dans le fichier entryPoint.sh (cette ligne est appelleé par le shebang en tache de fond)
                        Dans mon cas, le dépôt git avait unentry point script en Unix avec des fins de ligne (\n).
                         Mais lorsque le dépôt a été checkout  sur une machine Windows,
                         git a décidé d'essayer d'être intelligent et de remplacer les fins de ligne des fichiers unix par des fins de ligne Windows (\r\n).
                         Cela signifie que le shebang ne fonctionnait pas parce qu'au lieu de chercher /bin/bash, il cherchait /bin/bash\r.
                    Solution:
                        from: https://stackoverflow.com/questions/38905135/why-wont-my-docker-entrypoint-sh-execute
                        La solution pour moi a été de désactiver la conversion automatique de git :
                            git config --global core.autocrlf input   
                        Réinitialisez le repo en utilisant ceci (n'oubliez pas de sauvegarder vos modifications) :
                            git rm --cached -r .
                            git reset --hard
                        Et rebuildez (réexecuter vos commands: docker-compose up....)
                            D'autres informations utiles sont disponibles ici : How to change line-ending settings et ici http://willi.am/blog/2016/08/11/docker-for-windows-dealing-with-windows-line-endings/
                 10) Nouvelles cmd apprises :
                     1-chmod
                        chmod +x /code/docker/entrypoints/entrypoint.sh
                        execute le fichier entrypoint (accès en exécution, il y a aussi accès en lecture et en écriture)
                        D'autres informations utiles sont disponibles ici : 
                            https://dms.umontreal.ca/wiki/index.php/La_commande_chmod
                    2-Docker file COPY cmd
                         * COPY . /code/
                        Copie tous les fichiers ayant le meme chemin (dans le meme dossier) que le fichier dans le quel le code est écrit vers le dossier /code/ qui est la destination. Et les fichiers copiés sont appelés sources
                        Enfin j'ai appris que le dockerfile a ses propres commandes. 
                        VOIR ( https://docs.docker.com/reference/dockerfile/ )
                    3- pip freeze > requirements.txt
                        pip freeze: affiche toutes les bibliothèques Python installées avec leurs versions sous forme de liste.
                        > : redirige cette sortie vers le fichier requirements.txt.
                        VOIR la documentation pip (https://pip.pypa.io/en/stable/cli/pip_freeze/)
                    4- l'option -d pour docker-compose 
                        J'ai juste appris que l'option -d pour docker permet de le faire tourner en tache de fond
# Conclusion