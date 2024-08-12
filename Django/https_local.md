Django sobre HTTPS en red local
-----------------------------------

Para levantar nuestra apliacción de Django pero sobre el protocolo `HTTPS` en lugar de `HTTP`, tenemos que tener un certificado para validar nuestra identidad y una llave para cifrar la comunicación.

El problema es que los certificados tiene que estar validados por una autoridad certificadora y para esto hay que validar ante la autoridad nuestra identidad para que vea que somos confiables. Con el certificado, el navegador verá que está firmado por una autoridad y por lo tanto confiará en nosotros. La mayoría de certificados son de pago, pero podemos conseguir uno gratuito con la autoridad `Let's Encrypt`.

Aún así, nosotros podemos generar un certificado auto-firmado (Self-Signed Certificate), aunque al no estar firmado por una autoridad, el navegador de preguntará a los usuarios si quieren confiar en el certificado y una vez aceptado ya no volverá a preguntar.

### Usar un certificado auto-firmado (self-signed certificate)

Para esto podemos usar la herramienta openssl con los siguientes comandos:

```bash
# Generamos una llave privada en el archivo 'certificate.key'
openssl genrsa -out certificate.key 2048

# Generamos la solicitud de firma de certificado (Certificate Signing Request o CSR)
# Yo no definí ninguna contraseña durante el llenado del formulario y funcionó bien
openssl req -new -key certificate.key -out certificate.csr

# Nosotros mismos firmamos el certificado el cual generamos en el archivo 'certificate.crt'
openssl x509 -req -days 365 -in certificate.csr -signkey certificate.key -out certificate.crt
```

En el ejemplo generamos un certificado que expira en `365` días y luego tendremos que generar otro.

Una vez tenemos el certificado, tenemos que agregar las siguientes configuraciones en Django. Tenemos que asegurarnos de no subir esos archivos a ningún repositorio.

```python
# Habilitamos HTTPS
SECURE_SSL_REDIRECT = True  # Redirect HTTP to HTTPS
SECURE_HSTS_SECONDS = 3600  # HTTP Strict Transport Security
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
# Indicamos la ubucación del certificado y la llave
SECURE_SSL_CERTIFICATE = 'ruta/a/certificate.crt'
SECURE_SSL_PRIVATE_KEY = 'ruta/a/certificate.key'
```

Luego hay que instalar los siguientes paquetes para habilitar un comando que nos permite servir sobre HTTPS.

```bash
pip install django-extensions
pip install Werkzeug
pip install pyOpenSSL
```

Tenemos que registrar `django-extensions` en `INSTALLED_APPS`:

```bash
INSTALLED_APPS = [
    ...
    'django_extensions',
    ...
]
```

Luego ya podremos correr sobre HTTPS. Aunque en los ajustes configuramos la ruta del certificado y de la llave, en el siguiente comando también tenemos que indicarlos ya que estos son para el protocolo https, y los otros son para el funcionamiento interno de la app.

```bash
python manage.py runserver_plus --cert-file server.crt --key-file server.key <ip>:<port>
```
