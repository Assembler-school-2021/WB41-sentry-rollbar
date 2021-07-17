# WB41-sentry-rollbar

> Pregunta 1 : Añade a tu django el middleware de rollbar como explican en la documentación. Durante el proceso te harán generar un error para comprobar que todo funciona correctamente.

Necesitamos PIP:
```
apt install python-pip python3-pip
pip install setuptools
pip install wheel
pip install rollbar
pip3 install rollbar
```

In your *settings.py*, add '*rollbar.contrib.django.middleware.RollbarNotifierMiddleware*' as the last item in MIDDLEWARE 
```
MIDDLEWARE = (
# ... other middleware classes ...
'rollbar.contrib.django.middleware.RollbarNotifierMiddleware',
)
```
Add the following in *settings.py*: (al final)
```
ROLLBAR = {
'access_token': '***********',
'environment': 'development' if DEBUG else 'production',
'root': BASE_DIR,
}
import rollbar
rollbar.init(**ROLLBAR)
```
Y lanzamos el error:
```
python3 manage.py shell
>>> import rollbar
>>> rollbar.report_message("Hello world")
'e5cdbda2-172d-45fe-9a46-68de8bfc665d'
```
 ![image](https://user-images.githubusercontent.com/65896169/126047088-3cbaad83-0f99-4040-936a-f09e0c294924.png)


> Pregunta 2 : Repite el proceso anterior pero con sentry, borrando primero django del proyecto para evitar problemas.

Instalamos sentry-sdk:
```
	pip install --upgrade sentry-sdk
	pip3 install --upgrade sentry-sdk
```
To configure the SDK, initialize it with the Django integration in your *settings.py* file:
```
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn="https://*****************3@o922130.ingest.sentry.io/5869124",
    integrations=[DjangoIntegration()],

    # Set traces_sample_rate to 1.0 to capture 100%
    # of transactions for performance monitoring.
    # We recommend adjusting this value in production.
    traces_sample_rate=1.0,

    # If you wish to associate users to errors (assuming you are using
    # django.contrib.auth) you may enable sending PII data.
    send_default_pii=True
)
```
Incluimos en el fichero *urls.py* la ruta para forzar un error:
```
from django.contrib import admin
from django.urls import path

def trigger_error(request):
    division_by_zero = 1 / 0

urlpatterns = [
    path('sentry-debug/', trigger_error),
    path('opensesame/', admin.site.urls),
]
```
Reiniciamos el servicio:

`service uwsgi restart`

Al acceder a https://django.devops-alumno08.com/sentry-debug/ nos genera un error.

 ![image](https://user-images.githubusercontent.com/65896169/126047093-f9d419de-042c-4743-8c8c-b2faacffb891.png)
 
> Pregunta 3 : Pon la instalación de wordpress que estamos usando estos días en sentry.

> Pregunta 4 : Haz lo mismo con rollbar.
