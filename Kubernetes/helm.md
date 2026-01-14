# Helm

Helm es un manejador de paquetes de Kubernetes, sería el quivalente de pip, apt, npm, etc, pero para Kubernetes. Un paquete de Helm se llama **Chart** y contiene:

* Plantillas YAML de kubernetes.
* Valores configurables.
* Metadatos de la app.

[Helm](https://helm.sh/) no es parte de kubernetes, es un proyecto aparte que potencia y simplifica el despliegue en Kubernetes, y funciona mediante una herramienta de linea de comandos (CLI).

## Motivación

Helm resuelve el siguiente problema. Al desplegar una app en Kubernetes, necesitamos de varios archivos YAML para deployment, service, configmap, secret, etc. Si quiero el mismo deployment pero con otro nombre, en otro namespace y otro dominio, sin Helm tenemos que crear copias de todos los archivos y adaptarlos. Con Helm podemos tener plantillas reutilizables, variables de entorno, versionado para rollback, instalación/desinstalación limpia de aplicaciones. **En términos simplificados, Helm convierte configuraciones estáticas en aplicaciones configurables**.

Esta herramienta se ha vuelto muy importante y la opción recomendada para desplegar, por esta razón casi todas las aplicaciones tienen su versión Helm en forma de Chart, como `nginx`, `traefik`, entre otras. Durante la instalación de una app (chart) podemos especificar parámetros.

El comando básico de instalación de un chart es:
```bash
helm install <release-name> <chart-name>
```

A la plantilla reutilizable se le llama Chart y a la aplicación instalada se le llama `release`.

Por ejemplo, podemos instalar dos instancias de nginx con distintos nombres:
```bash
helm install web nginx-chart
helm install api nginx-chart
```

## Instalación

Podemos instalar la herramienta Helm (CLI) de distintas formas, pero aquí lo haremos desde el instalador oficial.

* Vamos a los releases en el repositorio oficial: https://github.com/helm/helm/releases
* Descargamos la versión para Windows-AMD64.
* Descomprimimos el archivo ZIP que se descargó y renombramos la carpeta a `helm` y la ponemos en `C:\Users\<user>\AppData\Local\Programs`.
* Agregamos el directorio `C:\Users\<user>\AppData\Local\Programs\helm` a la variable de entorno PATH del sistema.
* Reiniciamos la aplicaion donde queremos hacer uso de helm para que se recarguen las variables de entorno.
* Y ya tendremos disponible el comando `helm`.

Revisamos que todo funciona correctamente:
```bash
# kubectl está corriendo
kubectl cluster-info
# Helm está conectado a kubernetes (si responde sin errores entonces todo bien)
helm list
```

## Flujo

Cuando tenemos nuestra aplicación en un chart y la instalamos mediante:
```bash
helm install mi-app .
```

Ocurre lo siguiente:

* Lee Chart.yaml
* Carga values.yaml
* Renderiza las plantillas
* Genera YAML final
* Envía ese YAML a Kubernetes
* Guarda el estado del release en el cluster

Y podemos ver el YAML final con:
```bash
helm template .
```

## Uso

Similar a como cuando inicializamos un proyecto Git, inicializamos un proyecto Helm (chart) con:
```bash
helm create <chart-name>
```

Esto generará una serie dearchivos estructurados con contenido inicial para el Helm del proyecto. Genera mucho más de lo necesario, así que por lo general podemos eliminar varias de las cosas generadas si no las vamos a necesitar.

Helm en realidad usa los archivos de configuración dinámica para generar todas las configuraciones en un solo archivo YAML estático el cual es aplicado al cluster. Pero podemos solo renderizarlo sin aplicarlo para revisar el contenido:
```bash
helm template <chart-name>
```

Para generar la configuración final y aplicarla (instalar el chart) usamos el siguiente comando:
```bash
helm install <release-name> <chart-name>
```

Podemos instalar el release en el namespace que queramos:
```bash
helm install <release-name> <chart-name> -n <namespace-name>
```

Podemos revisar las aplicaciones instaladas con helm:
```bash
helm list
```

Y podemos revisar que los recursos fueron creados:
```bash
# Pods
kubectl get pods
# Servicios
kubectl get svc
```

Podemos desinstalar un chart con:
```bash
helm uninstall <release-name>
```

## Actualización y rollback

Helm nos permite actualizar un chart y si algo sale mal hacer rollback instantaneamente y muy fácil:

* Primero hay que tener instalado el chart: `helm install <release-name> <chart-name>`
* Realizamos algún cambio por ejemplo en el archivo `values.yaml`.
* Aplicamos el cambio: `helm upgrade <release-name> <chart-name>`
* Revisamo el historial de cambios del release: `helm history <release-name>`
* Aplicamos rollback hacia la revisión deseada, digamos la 1: `helm rollback <release-name> 1`

## Configuraciones de entorno de desarrollo

Lo recomendado para trabajar con distintos entornos de desarrollo es manejar un archivo de valores diferente para cada uno. Por ejemplo `values-prod.yaml` (prod), `values-dev.yaml` (dev), etc. Y aplicar el que queramos con:
```
helm install <release-name> <chart-name> -f values-dev.yaml
```

## Inspección

* Podemos revisar los valores actuales de un release: `helm get values <release-name>` o `helm get values <release-name> --all`

## Plantillas, variables y valores

Helm funciona mediante plantillas de configuraciones de Kubernetes, solo que están adaptadas para usar variables dinámicas. Dichas variables salen de algunos archivos como los siguientes:

* `values.yaml`: valores de configuración, se usan en las plantillas como `{{ .Values.<value-chain> }}`
* `Chart.yaml`: información del paquete `{{ .Chart.<value-chain> }}`
* `_helpers.tpl`: nombres, etiquetas o funciones reutilizables.
* Además de los anteriores, hay otro conjunto de datos que no se definen en ningún archivo pero se injectan a la plantilla en el momento de instalación/actualización que vienen empaquedados en el objeto `Release` y pueden ser usados como `{{ .Release.<value-chain> }}`.
    * Este objeto no representa el Chart, sino la instalación (release) y contiene información como el nombre, el namespace, la revisión, si es instalación o es actualización. 
    * Se suelen usar para definir las etiquetas de los objetos creados en Kubernetes, para derivar otros nombres (recomendado), etc.

Una analogía con Django es que las plantillas son como las plantillas HTML y lo que definamos en alguno de los archivos anteriores corresponde al contexto que es pasado a la plantilla.

Un ejemplo sencillo de una plantilla que hace uso de variables es la siguiente:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

Además podemos indicarle a la plantilla que elimine el salto de linea anterior iniciando con `{{-` o que elimine el salto de linea siguiente terminando con `-}}`.

También soporta etiquetas condicionales para poder crear recursos en base a las variables `{{- if <variable> }}...{{- end}}`.

También podemos generar configuraciones en loops con `range`: `{{- range <lista> }}...{{- end }}`.

Podemos especificar valores por defecto en caso de que una variable no esté definida: `{{ .Values.replicaCount | default 1 }}`


### Estructura del archivo values.yaml

Este archivo no impone ninguna estructura en términos de datos, solamente una estructura de identación. Podemos definir cualquer estructura de datos que queramos y usarla en las plantillas. Un ejemplo sería el siguiente:
```yaml
banana: 123
foo:
  bar:
    baz: "hola"
lista:
  - a
  - b
  - c
```

Y podemos usar los valores en una plantilla como sigue:
```yaml
{{ .Values.banana }}
{{ .Values.foo.bar.baz }}
{{ .Values.lista }}
# Loop de la lista
{{- range lista }}
...
{{- end }}
```

Si bien no hay una estructura impuesta, lo recomendable es definir los valores para cada objeto de forma separada, algo como el siguiente ejemplo, para que sea entendible y fácil de editar:
```bash
replicaCount: 2

image:
  repository: nginx
  tag: latest

service:
  enabled: true
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: "500m"
    memory: "256Mi"

config: # Para los ConfigMap
  APP_ENV: dev
  LOG_LEVEL: debug
```

## Configmaps y secrets

Helm facilita el manejo de estos, principalmente los secrets. Ya que permite definir los secrets en texto plano e indicar en la plantilla que los codifique en Base64:

```yaml
# values.yaml
secrets:
  DB_PASSWORD: supersecret
```
```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
type: Opaque
data:
{{- range $key, $value := .Values.secrets }}
  {{ $key }}: {{ $value | b64enc }}
{{- end }}
```

Para manejar los secrets con Git, una opción es mantener un archivo `values-template.yaml` con todas las configuraciones excepto los secrets y otro `values-prod.yaml` o `values-dev.yaml` con los secrets pero registrados en `.gitignore`. Pero no es lo recomendado, lo recomendado es mandar los secretos como parte del comando de instalación:
```bash
helm upgrade app . --set secrets.DB_PASSWORD=<password>
```

## Etiquetas

Se recomienda la siguiente convención para las etiquetas:
```yaml
metadata:
  labels:
    app.kubernetes.io/name: {{ .Chart.Name }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    app.kubernetes.io/managed-by: Helm
```

Aunque se puede definir una macro en `_helpers.tpl` para no estar manteniéndolas en todos lados:
```yaml
{{- define "hello-helm.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
app.kubernetes.io/managed-by: Helm
{{- end }}
```
```yaml
metadata:
  labels:
{{ include "hello-helm.labels" . | indent 4 }}
```



