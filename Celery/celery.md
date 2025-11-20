# Celery

Celery es una librería muy usada en el ecosistema de Python para la ejecución de tareas de manera asíncrona o distribuida en el mismo equipo o en equipos diferentes. Aunque es de uso general, nos enfocaremos desde el punto de vista de la integración con **Django**.

El servidor de Celery debe tener implementadas las tareas (funciones) que se pueden ejecutar.
Además, como las Celery se ejecuta en un servidor diferente al de la app (aunque pudieran estar corriendo en el mismo equipo), pues debe haber un mecanismo de cominicación para que la app pueda decirle al servidor de Célery qué función ejecutar y con qué parámetros. El puente de comunicación más común es mediante un servidor de **Redis**, ya que este implementa mecanismos para compartir información entre servidores y además maneja un canal de suscripciones donde un servidor se puede suscribir para que sea avisado cada que haya algún mensaje nuevo, de esta forma y de manera opcional se le puede avisar a la app cuando la tarea haya terminado y esta pueda consultar el resultado.

Por las razones anteriores, el enfoque de esta documentación estará basada en **Celery** con **Django** y **Redis**.

La arquitectura funciona básicamente de la siguiente forma:
* Un servidor de Redis, para servir como puente de comunicación entre Celery y Django.
* Un servidor de Celery, que implementa y ejecuta las tareas asíncronas.
* Un servidor de Django, que ejecuta la aplicación web que solicita la ejecución de tareas asíncronas.

**⚠️ Importante**: como la comunicación con Celery nunca es directa sino a través de Redis, entonces no necesitamos exponer el servidor de Celery ni siquiera dentro de la red, es suficiente con que el servidor de Redis esté expuesto y que sea accesible por el servidor de Celery y de Django.

## Integración de Celery en Django

Lo principal es asegurarnos de levantar el servidor con Celery con las tareas que puede ejecutar. Por fortuna Celery provee mecanismos para facilitar este proceso incluyendo en apps con Django.

### Reutilizar la app de Django para ejecutar Celery

Como el servidor de Celery ejecutará tareas para una aplicación Django, pues lo más conveniente es que Celery se ejecute en un entorno igual al de la app Django, con los mismos ajustes, mismas librerías, mismas herramientas etc. Para lograr esto simplemente tenemos que realizar lo siguiente.

* Crear un script llamado `celery.py` que se coloca dentro de **la app base** del proyecto (aquella que contiene los ajustes, el `asgi.py`, el `wsgi.py`, etc), con una configuración muy similar al de `asgi.py` o `wsgi.py`:
    ```python
    import os
    from celery import Celery

    # Especificar cuales ajustes de Django se usarán
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djangoweb.settings.final')

    # Creamos una instancia de Celery (después se renombrará en el archivo `__ini__.py`)
    # El nombre que especifiquemos será el que tenemos que usar al ejecutar el comando que arranca el servidor de Celery
    # NOTE: confirmar si es así o si en realizad este nombre no es de importancia y en realidad se usa el nombre de la app base que contiene este script
    app = Celery('djangoweb')
    # Indicar que los ajustes correspondientes a Celery inician con 'CELEY'
    app.config_from_object('django.conf:settings', namespace='CELERY')
    # Looks for 'tasks.py' files in every app of the Django project
    # Buscar automáticamente los archivos 'tasks.py' que contienen las tareas que se pueden ejecutar en Celery
    app.autodiscover_tasks()
    ```

* Exponer la variable `celery.app` en el archivo `__init__.py` de la app base, bajo el nombre `celery_app`:
    ```python
    # Renombramos `app` de `celery.py`
    from .celery import app as celery_app

    __all__ = ('celery_app',)
    ```

* Agregar los siguientes ajustes a los ajustes de Django (los que especificamos en el archivo `celery.py`):
    ```python
    # Indicar que la lista de tareas y los resultados se manejarán mediante Redis
    # Recordemos que la recomendación de Django es usar al base de datos `0` para datos generales
    CELERY_BROKER_URL = f"redis://{redis_host}:6379/0"
    CELERY_RESULT_BACKEND = f"redis://{redis_host}:6379/0"
    # Tipo de estructura a usar en la comunicación
    CELERY_ACCEPT_CONTENT = ['json']
    CELERY_TASK_SERIALIZER = 'json'
    CELERY_RESULT_SERIALIZER = 'json'
    # Zona horaria
    CELERY_TIMEZONE = 'America/Mexico_City'
    ```

* Implementar las tareas de Celery, por lo general en archivos llamados `tasks.py` en cada app de la aplicación, que como vimos serán descubiertos automáticamente por Celery.

* Ya con las configuraciones anteriores, ejecutamos el siguiente comando para levantar el servidor Celery y esté listo para la ejecución de tareas. Obviamente debe estar corriendo el servidor de Redis en accesible mediante el host configurado en los ajustes.
    ```bash
    # Por defecto se usará un número de workers igual al número de CPUs en el equipo pero cada uno cargará en memoria una copia de la app completa lo que puede requerir mucha memoria
    # No olvidemos indicar el mismo nombre que definimos en 'celery.py'
    celery -A djangoweb worker --loglevel=INFO
    # Podemos indicar el número de workers deseados con '--concurrency'
    celery -A djangoweb worker --loglevel=INFO --concurrency=1
    ```
    **⚠️ Importante**: el parámetro `worker` en el comando indica que celery correrá en modo Worker, es decir, estará monitoreando la lista de tareas pendientes registradas en Redis para resolver la siguiente en la lista, si tenemos más de un worker corriendo entonces todos estarán monitoreando la lista y cada uno tomando una tarea diferente. Como veremos después podemos usar `beat` en lugar de worker para levantar un servidor de Celery encargado de ejecutar tareas periódicamente.

Con los ajustes anteriores, hemos reutilizado la app de Django completamente para dos propósitos, ejecutar la app de Django (`python manage.py runserver`, etc), o ejecutar Celery (`celery -A djangoweb worker --loglevel=INFO`). Así, si usamos contenedores Docker, podemos usar la misma imágen en ambos servicios.

## Implementar tareas para Celery

La forma directa de implementar tareas para Celery es usando el decorador `celery.shared_task`. Recordemos que las tareas son auto-descubiertas y cargadas por Celery al levantar el servidor (`celery -A djangoweb worker --loglevel=INFO`).
```python
# ejemplos/tasks.py
from celery import shared_task
import time

@shared_task
def sumar(a, b):
    time.sleep(3) # Simular una tarea pesada
    return a + b
```

Si la tarea quedó bien registrada por Celery entonces veremos algo como lo siguiente en el reporte de Celery de la terminal (log) al arrancar el servidor, en este caso nuestra app se llama `ejemplos`:
```bash
[tasks]
  . ejemplos.tasks.sumar
```

Para ejecutar dicha tarea solo tenemos que ejecutar lo siguiente en otra terminal:
```bash
python manage.py shell
>>> from ejemplos.tasks import sumar
>>> sumar.delay(2, 3)
>>> sumar.delay(2, 10)
# podemos almacenar el retorno para poder consultar el resultado
>>> s = sumar.delay(2, 10)
>>> s.result
```

Cada que ejecutemos la tarea (función), podremos ver en el log de Celery algo como lo siguiente. Me imagino que el UUID `517022ab-9da2-4073-a32b-a993162bd93b` corresponde a la entrada en Redis que contiene la invocación y los parámetros.
```bash
[2025-11-19 12:35:47,313: INFO/MainProcess] Task examples.tasks.sumar[517022ab-9da2-4073-a32b-a993162bd93b] received
[2025-11-19 12:35:47,319: INFO/ForkPoolWorker-15] Task examples.tasks.sumar[517022ab-9da2-4073-a32b-a993162bd93b] succeeded in 0.0030545909994543763s: 10
```

### Flujo de trabajo en la práctica

Para completar el flujo de trabajo Django-Celery y poder hacer uso de tareas asíncronas en nuestro proyecto solo basta realizar los siguientes pasos:

* Implementamos las taréas asíncronas (funciones) en los archivos `tasks.py` de cada app del proyecto, decoradas con `@shared_task`, para que sean registradas por Celery.
* Hacemos uso de dichas funciones en nuestras vistas, pero mediante el atributo `delay` (como `sumar.delay(2, 10)`), para por ejemplo procesar un video (bajar un poco la calidad), procesar una imagen (generar thumbnails), o enviar un email, etc.

**⚠️ Importante**: si solo invocamos la función como `sumar(2, 10)`, entonces se esperará a que termine la ejecución, para que el programa continúe sin esperar el resultado tiene que ser mediante `sumar.delay(2, 10)`.


