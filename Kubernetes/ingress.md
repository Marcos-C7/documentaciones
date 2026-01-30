# Ingress

### El problema

Lo común en una aplicación web de varios componentes desplegada en Docker o Kubernetes, es usar un **reverse-proxy** (nginx, caddy, traefik) como única puerta de entrada a la aplicación y desde este redirigir el tráfico hacia el componente que le corresponda atender la petición, además se encarga de manejar de forma centralizada aspectos de seguridad como HTTPS, límites de tráfico, etc. De esta forma centralizamos los aspectos más importantes de seguridad, y aligeramos las responsabilidades o preocupaciones de todos los demás componentes, ya que si una petición llegó a un componente, quiere decir que ya pasó los filtros de seguridad más importantes impuestos por el reverse-proxy.

El detalle con esto es que hay muchos reverse-proxy y cada uno tiene configuraciones y mecanismos de configuración diferente, lo que hace difícil elegir entre uno u otro ya que si se decide cambiar después será algo muy complicado, además de que rompería con la filosofía base de Kubernetes que es mantener la app siempre disponible incluso durante un cambio. 

### La solución

Para resolver este problema, Kubernetes diseñó el componente **Ingress**, que no es un componente activo de Kubernetes sino solo una estandarización de la configuración de un reverse-proxy. Para que todo funcione correctamente en el entorno de Kubernetes, necesitamos versiones especiales de los reverse-proxy, no podemos usar las versiones regulares de nginx, caddy, etc. Para esto se implementarion versiones especiales que interactúan y se comunican con Kubernetes y no se llaman reverse-proxy, sino **Ingress-Controllers**. La mayoría de los reverse-proxy tienen su versión Ingress-Controller para Kubernetes.

Un Ingress se relaciona con el Ingress-Controller en el sentido de que el Ingress es un componente YAML para Kubernetes que describe de forma estándar configuraciones de ruteo (qué reglas de URL van a qué servicio, etc), HTTPS, etc. y luego Kubernetes le pasa dicho Ingress al Ingress-Controller que estemos usando y este lo convierte en configuraciones propias de forma automática, sin ninguna interacción de nuestra parte.

En resumen, el Ingress es la configuración estandárizada y el Ingress-Controller es el reverse-proxy que interpreta dicha configuración a su propia versión interna. De esta forma, podemos definir nuestras reglas de ruteo hacia los serivios, HTTPS, etc en un Ingress y cambiar de un ingress-controller a otro de manera transparente sin tocar ninguna configuración interna del ingress controller. Además, todo ocurre al estilo Kubernetes, es decir, si aplicamos alguna mofificación en nuetro Ingress o cambiamos el Ingress-Controller, el cambio se realizará sigilosamente sin interrumpir el funcionamiento de la aplicación.

Un Ingress-Controller es un reverse-proxy con un cerebro Kubernetes que incluye:

* Proxy (Nginx / Caddy)
* Watcher de API
* Parser de Ingress
* Generator de config
* Hot reload

#### Diagrama
```
           Internet
            ↓
Ingress →  Ingress Controller
           (Nginx / Traefik / etc)
            ↓
           Services
            ↓
           Pods
```
#### Ejemplo

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: django-ingress
spec:
  rules:
    # Si dominio "dev.localhost" y URL "/" mándala al Service "django" puerto 8000
    - host: dev.localhost
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: django
                port:
                  number: 8000
```

### Instalación

Como habíamos mencionado, no podemos usar las versiones regulares de los reverse-proxy como Ingress-Controllers, los tenemos que instalar directamente en Kubernetes con `kubectl apply`, solo que el archivo YAML correspondiente está dado mediante una URL, solo hay que buscar cual es la que corresponde al que queremos instalar. También se puede instalar mediante Helm.

La instalación a mano (sin Helm) del Nginx-Ingress-Controller se realiza con:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Como podemos ver si abrimos la URL del archivo en el navegador, la instalación incluye todos los componentes necesarios: namespace `ingress-nginx`, permisos RBACs, deployment, service, ConfigMaps, el IngressClass `nginx`, entre otras cosas.

Si nos damos cuenta, no es que lo instalemos, sino que levantamos un deployment/service con la versión modificada de Nginx configurado perfectamente con los permisos RBACs, y demás elementos necesarios para funcionar, todo levantado bajo el namespace `ingress-nginx`. También analizando los Services del archivo, el Ingress-Controller escucha en el puerto 80 y 443.

Otra opción sería descargar el archivo directamente e incluirlo a nuestro directorio de archivos YAML de Kubernetes, para levantarlo junto con todo lo demás.

### Manejo de varios Ingress-Controllers

Podemos tener más de un Ingress-Controller corriendo en nuestro Cluster, la parte clave para administrarlos es el componente de tipo `IngressClass` definido en el archivo, este le asigna un nombre al controller para poder especificarlo en nuestros Ingress, en el caso de Nginx, el IngressClass es `nginx`.

La forma más adecuada de consultar el nombre de un IngressClass es con el siguiente comando, que nos muestra los nombres y cual es el default:
```bash
kubectl get ingressclass
```
Resultado:
```
NAME    CONTROLLER             DEFAULT
nginx   k8s.io/ingress-nginx   true
```

Así podemos especificar en el Ingress cual es el controller que queremos utilizar:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: django-ingress
spec:
  ingressClassName: nginx   # 👈 aquí
  rules:
    - host: dev.localhost
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: django
                port:
                  number: 8000
```

Si queremos usar otro Ingress-Controller, simplemente cambiamos el nombre en `ingressClassName` y listo.

Si no especificamos ningún Ingress-Class name entonces se usará el que esté marcado como default. Lo recomendable es siempre especificar el nombre para reducir errores.

El siguiente comando nos ayuda a saber qué Controller está usando un Ingress:

```bash
kubectl describe ingress <ingress-name>
```

## Anotaciones para configuraciones únicas de cada Ingress-Controller

Aunque los Ingress son el intento de estandarizar el ruteo interno de la aplicación, cada Ingress-Controller tiene características únicas y la forma en la que Kubernetes permite configurarlas es mediante `annotations` (anotaciones). Cada Ingress-Controller tiene implementado internamente el manejo de sus propias anotaciones, así que solo tenemos que usar las que implementa el Ingress que estemos usando.

Un Ingress estándar solo sabe:

* host
* path
* service
* port
* tls

Pero en la práctica necesitamos:

* timeouts
* body size
* websockets
* auth
* headers
* redirects
* rate limit
* cache
* buffering

En el siguiente ejemplo configuramos anotaciones específicas del Ingress-Controller Nginx:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: django-ingress
  annotations:
    # Tiempo máximo esperando respuesta
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    # Tiempo máximo enviando respuesta
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
    # Tamaño máximo de un request, si tiene archivos mayores falla
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    # Habilitar websockets
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    # Forzar redirección a HTTPS
    nginx.ingress.kubernetes.io/enable-websocket: "true"
    # Rate limit
    nginx.ingress.kubernetes.io/limit-rps: "10"
    nginx.ingress.kubernetes.io/limit-burst: "20"
spec:
  ingressClassName: nginx
  rules:
    - host: dev.localhost
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: django
                port:
                  number: 8000
```

Con las anotaciones permites que cada Controller conserve fucionalidades únicas y fáciles de configurar. Podemos usar anotaciones para varios Controllers, ya que las que no correspondan al utilizado actualmente serán ignoradas, pero lo recomendado es manejar un Ingress para cada Controller.

## TLS (HTTPS)

Podemos integrar HTTPS en el Ingress de distintas formas, una es con renovación automática con letsencrypt, y otra es con certificados autofirmados (mkcert, etc).

### Certificados autofirmados con mkcert

Asumiendo que tenemos instalada la CA de mkcert, y tenemos el comando `mkcert` disponible (revisar la documentación de **mkcert**):

* Generamos los certificados para `dev.localhost`:
    ```bash
    mkcert dev.localhost *.localhost localhost 127.0.0.1
    ```
* Con lo anterior tendremos los archivos `dev.localhost+3-key.pem` and `dev.localhost+3.pem`, que podemos guardar donde queramos.
* Empaquetamos los certificados en un Secret de Kubernetes:
    ```bash
    # El secret-type puede ser 'tls' (recomended) o 'generic'
    kubectl create secret <secret-type> <secret-name> --cert=<crt-file> --key=<key-file>
    # Ejemplo
    kubectl create secret tls tls-secret --cert=tls/certs/dev.localhost+3.pem --key=tls/certs/dev.localhost+3-key.pem
    ```
* Esto generará un Secret llamado `tls-secret` con las entradas `tls.crt` y ``tls.key` listo para ser usado, podemos dejarlo así o si queremos tener el Secret en un archivo explícitamente, realizamos los pasos a continuación.
    * Exportamos el Secret recién creado a un archivo YAML:
        ```bash
        kubectl get secret <secret-name> -o yaml > <file-name>.yaml
        # Ejemplo
        kubectl get secret tls-secret -o yaml > tls.yaml
        ```
    * Ponemos el archivo donde mejor se nos acomode siempre que no sea versionado con Git.
    * Eliminamos el Secret inicial de Kubernetes: 
        ```bash
        kubectl delete secret tls-secret
        ```

Cualquier forma que hayamos elegido para administrar el Secret, al final tiene que aplicarse de alguna forma, y por lo tanto podemos hacer uso de este en cualquier lugar. Solo hay que especificar en el Ingress que use ese Secret para TLS:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: django-ingress
  # annotations
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - dev.localhost
      secretName: tls-secret # 👈 aquí va el secret que contiene los certificados
  rules:
    - host: dev.localhost
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: django
                port:
                  number: 8000
```

## CRDs

Algunos Ingress-Controllers, como Traefik, implementan sus propios Custom Resource Definitions (CRD) (ver la documentación de CRDs para más información), con los cuales podemos tener una mejor configuración, más compacta, más legible y más avanzada.

Esto ya depende del propio Ingress-Controller y al ser una parte exclusiva y opcional de cada uno, pues no es algo generalizable, nos toca investigar si el Controller implementa CRDs, como se agregan a nuestro cluster, cuales son y como se usan.

⚠️ Solo hay que tener bien presentes que el Ingress-Controller es una cosa y sus CRDs son otra, y que podemos usar sus CRDs si queremos mejor funcionalidad o podemos usar lo estándar.

