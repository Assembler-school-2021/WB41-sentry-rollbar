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

Instalamos composer:
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```
Mediante composer podemos instalar sentry:
```
composer require sentry/sdk
```
Insertamos en *wp-config.php* del proyecto (al final) las siguientes líneas:
```
if ( is_readable( __DIR__ . '/vendor/autoload.php' ) ) {
    require __DIR__ . '/vendor/autoload.php';
}
Sentry\init(['dsn' => 'https://***********************.ingest.sentry.io/5869751' ]);
```
Y lanzamos el error:
```
throw new Exception("Primera prueba de sentry en wordpress!");
```
![image](https://user-images.githubusercontent.com/65896169/126063308-09bcbce8-1979-4780-886f-ee7884c2a0b0.png)

> Pregunta 4 : Haz lo mismo con rollbar.

Instalamos con composer en el proyecto:
```
composer require rollbar/rollbar:~1.5
```
Y sustituimos lo que hemos puesto en wp-config para sentry por lo de rollbar:
```
use \Rollbar\Rollbar;
use \Rollbar\Payload\Level;
Rollbar::init(
    array(
        'access_token' => '************************',
        'environment' => 'production'
    )
);
```
Y lanzamos un error de prueba:
```
Rollbar::log(Level::info(), 'Primera prueba de rollbar en wordpress');
throw new Exception('Fallo lanzado a proposito para testing');
```
Y aquí tenemos el resultado de ambas pruebas:
![image](https://user-images.githubusercontent.com/65896169/126063584-04915218-2bdc-4c05-b9eb-6e6e975ede60.png)
![image](https://user-images.githubusercontent.com/65896169/126063590-d8b2024f-e182-4758-977d-e3373b8e36bc.png)





