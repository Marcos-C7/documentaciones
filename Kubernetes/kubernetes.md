# Kubernetes

Kubernetes es un sistema open-source de orquestación de contenedores Docker para automatizar el despliegue, escalado y mantenimiento de software en la nube. Su arquitectura, a grandes rasgos se compone de las siguientes partes:

* **Cluster**: es un conjunto de nodos (máquinas) enlazadas mediante Kubernetes.
* **Nodo**: una máquina física o virtual, cada nodo ejecuta las siguientes componentes:
    * **kubelete**: responsable del estado de ejecución del nodo asegurándose de que todos contenedores del nodo estén saludables, así como de iniciarlos, detenerlos y organizarlos en Pods. Consulta las especificaciones de los Pods mediante el API Server y se asegura de que los contenedores de cada Pod estén corriendo.
    * **Container runtime**: encargado del ciclo de vida de los contenedores, los crea, los detiene y elimina. Tiene su propia interfaz de comandos llamada **Container Runtime Interface (CRI)**. Los más usuales son Containerd (el motor de Docker), CRI-O.
    * **Kube proxy**: es una red proxy y laod balancer, se encarga de mantener las iptables y dirigir el tráfico al contenedor apropiado en base a la IP y el Puerto así como de balancear la carga entre las replicas de un deployment.
* **Pod**: es la unidad básica que puede ser agendada para desplegarse, este contiene los contenedores y se le asigna una IP única en el cluster, dentro del cluster sus contenedores tienen comunicación entre sí. Estos se suelen crear al crear un Deployment y su nombre suele tener la estructura `<deploy-name>-<replicaset>-<hash>`, esto ya que un deployment tiene un replicaset que se encarga de crear un Pod para cada replica.
* **Contenedor**: un contenedor de Docker, que vive dentro de un Pod y además ejecuta una aplicación como parte de un microservicio.

Además, además de los nodos creados para desplegar aplicaciones (**Worker Nodes**), hay un nodo llamado Plano de Control (**Control Plane**) que es el cerebro que ejectuta las principales componentes que administran el cluster, como son:

* **API Server** (`kube-apiserver`): es la interfaz de comuniación tipo REST por HTTP para poder darle instrucciones de actualización del cluster a Kubernetes. Es el único punto de comunicación tanto para el humano administrador como entre las mismas componentes, como los controladores.
* **Etcd** (`etcd`): tienda de datos persistente llave-valor que sirve como punto de encuentro para matener todo el cluster sincronizado, aquí se almacena y consulta todo el estado del cluster (pods, services, deployments, secrets, etc), aquí se guarda absolutamente todo lo relacionado con el estado actual y futuro.
* **Scheduer** (`kube-scheduler`): se encarga de determinar en qué nodo va a ejecutarse cada Pod, tomando en cuenta los recursos disponibles (RAM, CPU, etc), solo agenda la ejecución no lo ejecuta. Si detecta un pod no asignado, evalúa los nodos existentes y asigna el pod al nodo de mejor puntuación. Se puede correr más de un scheduler y podemos implementar los nuestros propios tipo plug-ins para extender o modificar el funcionamiento por defecto.
* **Controller Manager** (`kube-controller-manager`): se encarga de la ejecución y administración de los controladores, los cuales se encargan de mantener el cluster en el estado deseado. Algunos de los controladores son:
    * **Controlador de replicación**: cada deployment debe tener un `ReplicaSet` que almacena cuantas replicas del Pod se desean y cuantas existen, este controlador se encarga de que ambos números coincidan, si falta o muere alguno levanta nuevos y si hay de más los elimina. Esto convierte a Kubernetes en auto-reparable.
    * **Controlador de nodos**: vigila la salud de los nodos y si alguno deja de responder reasigna sus Pods a otros nodos.
    * **Controlador de deployment**: gestiona los deployments, creándoles o actualizándoles su `ReplicaSet` el cual se encargará de crear las réplicas de los Pods.
        * **Depoloyment**: es la descripción de una aplicación (Pod), qué contenedores va a tener, que imágenes van a usar los contenedores, cuantas réplicas del Pod quieres corriendo, etc. Tú la describes y kubernetes se encarga de mantenerla, acualizarla y repararla recreando pods muertos, entre otras cosas mediante los demás controladores.
    * **Controlador de servicios**: gestiona los Services y configura iptables y balanceadores de carga en el `kube-proxy`.
        * **Servicio**: para que un Pod sea accesible desde el exterior se tiene que exponer mediante un servicio, el cual agrupa bajo una IP a uno o más Pods de un deployment. De esta forma, podemos tener 3 pods de un deployment levantados con IP diferentes y agruparlos bajo una misma IP de un servicio que se encargue de reenviar las solicitudes hacia las IPs de los pods.

En resumen, el cliclo de vida del cluster se resume en lo siguiente:
```
Loop infinito:
1. Observar el estado actual (desde etcd vía el API Server).
2. Comparar con el estado deseado (también desde etcd vía el API Server).
3. Si hay diferencia, tomar acciones (mediante los controladores).
```

### Volúmenes

Los volúmenes son unidades de almacenamiento persistente, similar a los que se usan en Docker pero en Kubernetes son mucho más potentes. Hay varios tipos de volúmenes y cada uno de ellos tiene un ciclo de vida y alcance diferente (ver abajo los ejemplos YAML para crear volúmenes).

* **Volume**: directorio asociado a un Pod que es accesible por sus contenedores, vive mientras viva el Pod, puede ser de tipo **emptyDir**, **hostPath**, **configMap** o **secret**.
* **Persistent Volume (PV)**: recurso de almacenamiento asociado al cluster, existe independientemente de los Pods.
* **Persistent Volume Claim (PVC)**: solicitud de almacenamiento, ej. un Pod hace un PVC `"Quiero 5GB de disco"` y Kubernetes le enlaza el espacio con un PV.

## Herramientas para simular un cluster de Kubernetes

Kubernetes es open-source y por lo tanto hay varias herramientas para simular un cluster de Kubernetes en una máquina local, cada uno con sus ventajas sobre otros, a continuación describo los dos principales:

* **Minikube (Windows, Mac, Linux)**: es una implementación de Kubernetes para una máquina local que simula un cluster de un solo nodo, dicho nodo se simula en una máquina virtual o un contenedor, depende del driver que se esté usando el cual se puede consultar con `minikube profile list`. Además, dicho nodo cumple dos funciones Worker y Control Plane, es decir que ejecuta los servicios que administran el cluster y también despliega aplicaciones. El cluster se inicia y apaga con comandos `minikube start` y `minikube stop`, además necesita de Docker para funcionar, por lo tanto hay que tener corriendo Docker (Windows, Mac, Linux) o Docker Desktop (Windows, Mac) para que funcione. Su ventaja es que está más apegado a una implementación de producción de Kubernetes y por lo tanto soporta muchas funcionalidades de Kubernetes.
* **Docker Desktop (Windows, Mac)**: en Docker Desktop hay una sección llamada `Kubernetes` que contiene un botón para iniciar y apagar el cluster, además permite virtualizar uno o más nodos, la ventaja es que funciona completamente integrado con el docker daemon de Docker Desktop por lo que no crea máquina virtual separada y así consume menos recursos, la desventaja que leí decía que soporta menos funcionalidades que `Minikube` y además no puedes cambiar la versión de Kubernetes que se usará, no lo he probado en la práctica y no sé si ya cambiarían las cosas para este momento. Otra ventaja es que al venir integrado en el mismo Docker Desktop pues no se necesita tener 2 herramientas instaladas así que para desplegar en una máquina local funcionaría bien.

**IMPORTANTE**: podemos tener corriendo ambos clusters, el de Minikube y el de Docker Desktop, al mismo tiempo y decidir con cual interactuar, esto lo veremos en la sección de `Kubectl`.

### Iniciar y apagar el cluster con Minikube

Antes de iniciar el cluster tenemos que asegurarnos de que Docker está corriendo, si tenemos **Docker Desktop** en Windows instalado hay que iniciarlo. Una vez iniciado ejecutamos el siguiente comando:

```bash
minikube start
```

Apagar: ejecutamos el comando
```bash
minikube stop
```

También lo podemos eliminar con:
```bash
minikube delete
```

### Iniciar y apagar el cluster con Docker Desktop

Iniciar:

* Solo hay que ir a la sección `Kubernetes`.
* Presionar el botón `Create Cluster`.
* Configurar el cluster (aunque podemos configurarlo también una vez esté iniciado).
* Presionamos el botón `Create` una vez lista la configuración.
* Presionamos el botón `Install` en la nueva ventana, será tardado solo la primera vez.

Apagar:
* Solo presionamos el botón `Stop` en la sección de `Kubernetes`.

## Kubectl (modelo de comunicación imperativo)

Recordemos que Kubernetes tiene un centro de comunicación llamado `API Server` (`kube-apiserver`) el cual funciona mediante una API HTTP tipo REST. Con esta podemos interactuar y darle instrucciones cuando queramos realizar alguna actualización en el cluster o consultar información del cluster. A este tipo de comunicación se le llama **Modelo imperativo** ya que se le dice que hacer paso por paso `kubectl describe pod <nombre>`.

Lo bueno es que no es necesario interactuar forzosamente mediante dicha API HTTP, sino que Kubernetes viene integrado con una herramienta llamada `kubectl` que funciona mediante una interfaz de comandos (Command Line Interface - CLI) y se encarga de recibir comandos y convertirlos a peticiones HTTP propias para el API Server y además realizar las peticiones y mostrarnos las respuestas en forma más legible.

**⚠️ Importante**: para facilitar la automatización de interacción con el API Server, también hay librerías oficiales para distintos lenguajes de programación como **Python**, **C**, **Javascript** entre otros.

### Apuntar kubectl a Minikube o a Docker Desktop

Como habíamos mencionado anteriormente, podemos tener corriendo el cluster de Minikube y el de DOcker Desktop al mismo tiempo, la herramienta `kubectl` puede informarnos los clusters que están corriendo (les llama contextos) y nos permite redirigirla al cluster que queramos.

Para saber cuales son los clusters (contextos) disponibles:
```bash
kubectl config get-contexts
```
Nos mostrará algo como (marcanado con arterisco el cluster seleccionado):
```
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          docker-desktop   docker-desktop   docker-desktop
*         minikube         minikube         minikube         default
```

Podemos seleccionar otro cluster:
```bash
# Sintaxis
kubectl config use-context <context-name>
# Ejemplo
kubectl config use-context docker-desktop
```

### Ejemplos de comandos de kubectl

Manejar el cluster mediante comandos no es lo ideal, ya que solo permite aplicar una configuración a la vez y además no es versionable, hay una mejor forma que es mediante archivos YAML que tiene una estructura mucho más legible y sí es versionable. De todas formas, lo que sí es útil con comandos es consultar el estado del cluster, por eso veremos algunos ejemplos a continuación.

* Crear un deployment en base a una imagen, exponerlo, levantarlo, revisarlo:
    ```bash
    # Crear el deployment (esto creará y pondrá a correr los Pods, aunque no estarán expuestos al exterior)
    kubectl create deployment <deploy-name> --image=<image-name> --replicas=<num-replicas>
    # Crear un servicio para exponer el deployment, tendrá el mismo nombre que el deploy (esto solo expondrá los Pods del deployment bajo una IP ya que los Pods ya están corriendo, además el servicio solo se creará pero no será levantado)
    kubectl expose deployment <deploy-name> --type=NodePort --port=<port>
    # Levantar el servicio (en la terminal se mostrará la URL), de preferencia correrlo en otra consola ya que queda bloqueada (con esto ya podemos acceder a la app del deployment desede el exterior)
    minikube service <service-name> --url

    # Revisar el estado de todos los recursos existentes (pods, servicios, deployments, replicasets, etc)
    kubectl get all
    # Revisar el estado de los deployments existentes
    kubectl get deployments
    # Revisar el estado de los ReplicaSets del deployment
    kubectl get replicaset -l app=<deploy-name>
    # Revisar el estado de los pods del deployment
    kubectl get pods -l app=<deploy-name>
    # Revisar el estado de los servicios del deployment
    kubectl get services -l app=<deploy-name>
    # Ver las IPs de los Pods del servicio
    kubectl get endpoints <service-name>
    ```
* Realizar limpieza completa de un deployment:
    ```bash
    # Ver qué existe
    kubectl get all

    # Eliminar Deployment y Service
    kubectl delete deployment mi-app
    kubectl delete service mi-app

    # Verificar que se eliminó todo
    kubectl get all

    # Si quedó algo huérfano
    kubectl delete pod <nombre-pod> --grace-period=0 --force
    ```
* Eliminar un deployment por etiqueta (más eficiente):
    ```bash
    # Si todos los recursos tienen el mismo label
    kubectl delete all -l app=<deploy-name>
    ```
* Eliminar deployments por archivo YAML:
    ```bash
    # Si creaste tus recursos con: kubectl apply -f <file>.yaml:
    kubectl delete -f <file>.yaml
    ```
* Eliminar todo lo de un namespace:
    ```bash
    # ⚠️ CUIDADO: Borra TODO en el namespace actual
    kubectl delete all --all
    # O para borrar todo en un namespace específico
    kubectl delete namespace <namespace-name>
    ```

### ¿Qué ocurre internamente cuando eliminamos algún recurso del cluster?

Eliminar un Deployment:
* **Comando**: `kubectl delete deployment <deploy-name>`
* **Consecuencias**: se elimina el deployment, su ReplicaSet, todos sus Pods, pero no el servicio ni los configMaps/Secrets.

Eliminar un Service:
* **Comando**: `kubectl delete service <deploy-name>`
* **Consecuencias**: elimina el service, los endpoints asociados las iptables del kube-proxy, pero no se eliminan los Pods (siguen corriendo pero sin exposición hacia el exterior).

Eliminar un Pod:
* **Comando**: `kubectl delete pod <pod-name>`
* **Consecuencias**: se elimina el Pod, pero el ReplicaSet detecta que falta un Pod y lo crea nuevamente con otra IP actualizando el enrutamiento en las iptables del kube-proxy.

### Actualización en vivo

Una funcionalidad de Kubernetes muy importante es que permite realizar actualizaciones a los deployments en vivo sin tener que apagar o reiniciar el cluster, solo se le indica el cambio y Kubernetes lo realiza de la manera más adecuada y automáticamente. Algunos ejemplos son los siguientes:

* Cambiar el número de replicas de la aplicación:
    * Comando: `kubectl scale deployment <deploy-name> --replicas=<nuevas-replicas>`
    * Ejmplo: `kubectl scale deployment mi-app --replicas=5`
    * Descripción: le estamos indicando a Kubernetes que la app correspondiente al deployment `mi-app` se ajuste para que corra `5` réplicas.
    * Acciones: Kubernetes actualizará la cantidad en el ReplicaSet correspondiente para que este elimine o genere nuevas réplicas y las conecte a la iptable del Kube-Proxy para que este balancee la carga considerando las nuevas réplicas.
* Actualizar la versión de la imagen de la aplicación sin apagarla:
    * Comando: `kubectl set image <resource-type>/<resource-name> <container-name>=<nueva imagen>`
    * Ejemplo: `kubectl set image deployment/my-app nginx=nginx:1.23.4`
    * Descripción: le estamos indicando a kubernetes que cambie la iamgen de un recurso tipo `deployment`, cuyo nombre es `mi-app`, y que la imagen usada por el contenedor llamado `nginx` sea cambiada por la imagen llamada `nginx:1.23.4` manteniendo el mismo nombre del contenedor que es `nginx`. Podemos especificar múltiples cambios `<container-name>=<nueva imagen>`.
    * Acciones: Kubernetes crea los un nuevo ReplicaSet con la nueva imágen el cual crea Pods con la nueva versión. El anterior ReplicaSet queda en `0` para poder deshacer el cambio.
* Regresar a una versión anterior si los últimos cambios aplicados están fallando:
    * **Comando**: `kubectl rollout undo <resource-type>/<resource-name>`
    * **Ejemplo**: `kubectl rollout undo deployment/mi-app`
    * **Descripción**: estamos indicando que queremos deshacer los últimos cambios del recurso tipo `deployment` con nombre `mi-app`.
    * **Acciones**: Kubernetes realiza los cambios gradualmente sin interrumplir el funcionamiento. 

## Archivos YAML (modelo de comunicación declarativo)

Además de poder actualizar el cluster con `kubectl`, también podemos realizarlo más eficientemente a escala mediante archivos YAML. A este tipo de comunicación se le llama **Modelo declarativo**. En un mismo archivo YAML podemos describir uno o más recuros (Pod, Deployment, Service, Secret, Congig Map, Persistent Volume, Persistent Volume Clain, etc) y Kubernetes se encargará de llevar el estado de dichos recursos al descrito en el archivo YAML, es decir, si los recursos no existen los crea y si ya existen los actualiza, además registra un historial de actualizaciones para poder regresar en caso de errores.

Este es el modelo preferido ya que es más predecible y además versionable con herramientas como Git.

Reglas del formato YAML:
* YAML usa espacios, no tabulaciones.
* Cada nivel de identación es de 2 espacios, no de 4.

A continuación un ejemplo sencillo de un deployment que contiene un Pod con un solo contenedor `nginx` escuchando en el puerto `80`, pero levanta 2 réplicas de dicho Pod.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### Pod

A continuación revisaremos ejemplos de algunas configuraciones que se pueden especificar en un archivo YAML para crear un recurso, en este caso un Pod. Se revisarán las configuraciones gradualmente solo manteniendo en cada ejemplo las nuevas configuraciones junto con sus ancestros para no perder la estructura.

```yaml
# Archivo YAML
apiVersion: v1 # versión de la API de Kubernetes -> v1 (usado para Pods) o apps/v1 (para Deployments, StatefulSets, etc), se supone que cada tipo de recurso puede usar ciertas versiones
kind: Pod # tipo de recurso -> Pod, Deployment, Service, ConfigMap, Secret, etc.
metadata: # información del recurso
  name: mi-primer-pod # nombre del recurso
spec: # especificación del recurso
  containers: # lista de contenedores que correrán dentro del Pod (en este caso solo 1)
  - name: nginx # nombre que tendrá el contenedor
    image: nginx # imágen de Docker que usará el contenedor
```

#### Pod - Metadatos

```yaml
metadata:
  namespace: default # namespace para organizar los recursos creados y poder manejarlos en grupo, el valor por defecto es `default`
  labels: # etiquetas para organizar y manejar en sub-grupos dentro del namespace, podemos inventar las que queramos
    app: mi-aplicacion
    tier: frontend
    version: v1
  annotations: # metadatos para documentar el recurso, podemos inventar las que queramos
    description: "Pod de prueba"
    createdBy: "juan"
```

#### Pod - Contenedor - puertos y ejecución

```yaml
spec:
  containers:
  - name: nginx
    ports: # lista de puertos que el contenedor expone, no abre puertos, es para que los Services sepan como a qué puerto mapear
    - containerPort: 8080 # puerto en el que estará escuchando el contenedor
      name: http # nombre descriptivo del puerto (opcional)
      protocol: TCP # protocolo, puede ser TCP o UDP
    command: ["nginx"] # comando a ejecutar (sobreescribe al ENTRYPOINT de la imagen)
    args: ["-g", "daemon off;"] # argumentos para el comando (sobreescribe CMD de la imagen)
```

#### Pod - Contenedor - configuraciones de variables de entorno

```yaml
spec:
  containers:
  - name: nginx
    env: # lista de variables de entorno para el contenedor
    - name: ENVIRONMENT # nombre de la variable
      value: "production" # valor crudo de la variable
    - name: DATABASE_URL
      valueFrom: 
        configMapKeyRef: # Podemos sacar datos de los configMaps (ver abajo como crearlos con YAML)
          name: app-config # nombre del config map a usar
          key: database.conf # nombre de la entrada a usar (recordemos que un configmap puede tener varias entradas) cada entrada puede tener varias parejas llave=valor, me imagino que todas ellas se inyectan como variables de entorno.
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef: # Podemos sacar datos de un Secret (ver abajo como crearlos con YAML)
          name: db-secret # nombre del Secret a usar
          key: password # nombre de la entrada a montar, en este caso la entrada es una única llave.
```

#### Pod - Contenedor - recursos del sistema

```yaml
spec:
  containers:
  - name: nginx
    resources: # requerimientos de recursos para el contenedor
      requests: # Recursos MINIMOS (el scheduler usa esto para decidir en qué nodo colocar el Pod)
        memory: "128Mi" # unidades de memoria (M=Megabytes, Mi=Mebibytes, G=Gigabyte, Gi=Gibibyte)
        cpu: "250m" # unidades de CPU (1=un CPU completo, 500m=500 milicores=0.5CPU)
      limits:
        memory: "256Mi"
        cpu: "500m"
```

#### Pod - Contenedor - prueba de vida

```yaml
spec:
  containers:
  - name: nginx
    livenessProbe: # configuraciones para revisar si el contenedor sigue vivo, si falla se recrea el contenedor, para estas validaciones, el contenedor debe implementar el endpoint que retornará Código HTTP 200 si todo sigue bien. Hay otros mecanismos para realizar la prueba.
      httpGet: # Tipo de petición que usa el endpoint del contenedor
        path: / # url relativa a donde se hará la petición de la prueba dentro del contenedor
        port: 8081 # Puerto donde se hará la petición
        httpHeaders: # podemos agregar las cabeceras que queramos a la petición http
        - name: Custom-Header
          value: "value"
      initialDelaySeconds: 30 # Espera antes de la primera verificación
      periodSeconds: 10 # Cada cuántos segundos verifica
      timeoutSeconds: 5 # Timeout para cada verificación
      successThreshold: 1 # Éxitos consecutivos = sano
      failureThreshold: 3 # Fallos consecutivos = no sano
    readinessProbe: # configuraciones para revisar si el contenedor está listo par recibir tráfico, si falla se deja de mandar tráfico pero no se recrea el contenedor. Similar a `livenessProbe`.
      #...configuraciones similares a `livenessProbe`
    startupProbe: # configuraciones para revisar si el contenedor inició correctament, deshabilita livenessProbe y readinessProbe hasta que esta prueba pase, si falla se recrea el contenedor. Similar a `livenessProbe`.
      #...configuraciones similares a `livenessProbe`
```

#### Pod - Contenedor - volúmenes

```yaml
spec:
  containers:
  - name: nginx
    volumeMounts: # Dónde montar volúmenes en el contenedor
    - name: cache # nombre del volumen a montar (definido más abajo en volumes)
      mountPath: /usr/share/nginx/cache # ruta del volumen dentro del conenedor
    - name: config # nombre del volumen a montar (definido más abajo en volumes)
      mountPath: /etc/nginx/conf.d
      readOnly: true # siempre true cuando montmos un Secret
    - name: data-volume # nombre del volumen a montar (definido más abajo en volumes)
      mountPath: /data
  volumes: # Definición de los volúmenes, no los monta para esto hay que especificarlo en `volumeMounts` del contenedor
  - name: cache # nombre para el volumen
    emptyDir: {} # directorio vacío temporal, creado cuando el Pod inicia y es compartido entre los contenedores del Pod, se elimina cuando el Pod muere.
  - name: docker-socket # nombre para el volumen
    hostPath: # Monta un directorio para el nodo (⚠️ Peligroso en producción)
      path: /var/run/docker.sock
  - name: config # nombre para el volumen
    configMap: # Monta un las entradas de un ConfigMap como archivos
      name: nginx-config
      items: # Lista de entradas a montar de el config map, si no se especifica se montarán todas un archivo por entrada con el mismo nombre de la entrada
      - key: database.conf # nombre de la entrada a montar
        path: db.conf # Renombrar la entrada (el archivo) al montar
  - name: credentials # nombre para el volumen
    secret: # Monta un Secret como archivos
      secretName: db-credentials
  - name: data-volume  # nombre para el volumen
    persistentVolumeClaim:
      claimName: pvc-local # nombre del Persistent Volume Claim (ver abajo como crearlo con YAML)
```

`NOTA`: podemos combinar varias fuentes (secrets, configMaps, etc) en un mismo volumen con la configuración `projected` del volumen, no lo documentaré aquí pero lo dejo anotado como futura investigación.

Tipos de volúmenes:

* `emptyDir`: define un directorio virtual vacío que se crea cuando el Pod inicia y es accesible por todos los contenedores del Pod, se elimina cuando el Pod muere. Sus configuraciones son:
    * `medium`: para indicar si se usará memoria (valor `Memory`) en lugar de disco (tmpfs).
    * `sizeLimit`: límite de tamaño, ej. `1Gi`, `1M`, etc.
* `hostPath`: define un directorio o archivo del nodo, es peligroso en producción porque depende del nodo específico. Sus configuraciones son:
    * `path`: directorio del volumen en el nodo, ej. `/data/nginx`
    * `type`: Tipo de verificación (`DirectoryOrCreate` Crea si no existe, `File` Debe existir como archivo, `FileOrCreate` Crea archivo si no existe, entre otros)
* `configMap`: define un volumen con datos de configuración desde un ConfigMap como archivos en el Pod. Recordemos que los configMaps se almacenan en el `etcd`. Sus configuraciones son:
    * `name`: nombre del configMap a usar, 
    * `items`: lista de entradas a montar, si no se especifica se montarán todas un archivo por entrada con el mismo nombre de la entrada.
    * `NOTA`: además de montar configMaps como volúmenes, podemos integrarlos en los contenedores mediante las variables de entorno.
* `secret`: Similar a ConfigMap pero para datos sensibles (contraseñas, tokens, certificados). Codificados en base64 (ver abajo como crear Secrets con YAML). Además de inyectar secrets como volúmenes, podemos inyectarlos como variables de entorno (ver arriba). Sus configuraciones son:
    * `secretName`: nombre del Secret a montar, un archivo por cada entrada del Secret.
* `persistentVolumeClaim`: define un PVC como volumen. Sus configuraciones son:
    * `claimName`: nombre del PVC a usar.

### Deployment

Un deployment es el siguiente nivel después de un Pod, ya que además de describir las características de la aplicación, también permite describir un Pod dentro de la sección `spec.template`, dentro de esta sección es donde podemos usar todo lo que vimos anteriormente de Pod.

Kubernetes es muy flexible, y por esa razón tenemos que asignarele una o más etiquetas a los Pods, de esta forma podemos indicar cuales Pods estarán corriendo en el deployment especificando en la sección `spec.selector.matchLabels` las etiquetas que queramos que concuerden. Así podemos crear los Pods fuera del Deployment y usar las etiquetas para integrarlos en el Deployment.

```yaml
apiVersion: apps/v1 # versión usada para deployments
kind: Deployment # tipo de recurso
metadata:
  name: mi-deployment
  labels:
    app: mi-app
spec:
  replicas: 3 # número de réplicas de la aplicación
  selector: # para indicar cuales Pods gestionará este Deployment
    matchLabels: # los pods que conincidan en todas estas etiquetas pertenecerán a este Deployment
      app: mi-app
  strategy: # Estrategia de actualización
    type: RollingUpdate # RollingUpdate es la estrategia por defecto para aplicar actualizaciones gradualmente, pero también está `Recreate` para aplicar actualizaciones a todo de golpe
    rollingUpdate:
      maxUnavailable: 1 # Máximo de Pods caídos antes de recrearlos, puede ser en cantidad o en porcentaje (25%)
      maxSurge: 1 # Máximo de Pods extras antes de eliminar los excedentes
  template: # Plantilla del Pod, todo lo que vimos para la creación de un Pod va en esta sección
    metadata:
      labels:
        app: mi-app # IMPORTANTE: DEBE coincidir con la etiqueta definida en selector, no se integra automáticamente porque los Pods pueden crearse por separado, el deploy tomará todos los Pods que tengan la misma etiqueta y los tomará como parte de sí
    spec: # Spec del Pod (todo lo que aprendimos de la creación de Pods)
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

### Service

Ahora veremos un ejemplo para definir un Service en un archivo YAML, también las mayoría de las configuraciones son opcionales.

```yaml
apiVersion: v1 # versión para Services
kind: Service
metadata:
  name: mi-service
spec:
  type: NodePort # Tipo de Service, `NodePort` (exposición al exterior, desarrollo local), `ClusterIP` (solo acceso interno para servicios que no serán expuestos al exterior), `LoadBalancer` (exposición al exterior, al subir a la nube), `ExternalName`.
  selector: # Qué Pods incluir en el servicio
    app: mi-app
  ports:
  - name: http # nombre desciptivo (opcional)
    protocol: TCP # TCP o UDP (default: TCP)
    port: 80 # Puerto del Service (podemos hacerle peticiones mediante http://mi-service:<port>)
    targetPort: 8000 # Puerto donde el contenedor está recibiendo peticiones (el service redirigirá el tráfico a este puerto en el contenedor, no olvidemos configurar el `ports.containerPort` en el contenedor)
    nodePort: 30080 # solo para NodePort/LoadBalancer, puerto en el nodo (máquina) (rango 30000-32767), tal que al recibir una petición en el nodo en este puerto la redirigirá a http://mi-service:<port>.
```

En resumen, especifica el camino a seguir de una petición desde el nodo hasta el contenedor:
`http://<IP-del-Nodo>:<nodePort> -> http://<service-name>:<port> -> http://<container-name>:<targetPort>`.

Componentes de la red del cluster:

* **Pod IP**: cada Pod tiene IP única en el cluster y cambia si el Pod se recrea.
* **Service Cluster-IP**: IP virtual estable, no cambia y solo es accesible dentro del cluster.
* **Service Node-Port**: para exponer el servicio en un puerto de cada nodo donde tiene pods el servicio, con este puerto ya podemos acceder al servicio externamente, no solo desde dentro del cluster, para acceder al servicio en alguno de los nodos es con `<NodeIP>:<NodePort>`.
* **Service Load-Balancer**: solo en clouds públicos, crea un balanceador de carga externo.
* **CNI (Container Network Interface)**: modelo de red, Minikube usa `bridge` o `calico` por defecto, pero existe también `flannel`, `cilium` o `weave`.


Podemos revisar las IP-Tables:
```bash
# Entrar nodo de Minikube
minikube ssh
# Ver reglas de iptables (Services)
sudo iptables-save | grep <service-name>
```

### configMap

Recordemos que los `configMap` se almacenan en el componente `etcd` de Kubernetes. Aquí veremos como crear un `configMap`, los ConfigMaps se usan para almacenar configuraciones de las aplicaciones y cada uno puede contener varias entradas con lo cual podemos almacenar todas las configuraciones de una app o de un servicio en un mismo `configMap`.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config # nombre del config map, todas las entradas definidas serán asociadas en este
data:
  database.conf: | # nombre de la entrada seguida por los datos en formato llave=valor
    host=postgres.default.svc.cluster.local
    port=5432
    database=myapp
  app.properties: |
    log.level=INFO
    max.connections=100
```

### Secret

Similar al los Config Maps solo que, en este caso, los Secrets se usan para almacenar credenciales, certificados criptográficos, y todos tipo de información altamente sensible. Los secrets deben ser almacenados en codifiación `base64`. También se almacenan en el `etcd` y pueden ser encriptados si se configura el `encryption-at-rest`. Estos solo se envían a los nodos que tienen al menos un Pod que los necesita. Kubelet guarda los Secrets en RAM, y son montados en los contenedores como archivos en memoria y al morir el Pod el Secret se elimina de la memoria.

**⚠️ Importante:** Los Secrets NO están encriptados en `etcd` por defecto, debes configurar `encryption-at-rest`.


```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret # nombre del secret
type: Opaque # Tipo genérico
data: # datos, podemos agregar tantos como queramos
  username: YWRtaW4= # "admin" en base64
  password: czNjcjN0 # "s3cr3t" en base64
```

Tipos de Secret:
* Almacenar datos genéricos, podemos poner las entradas que queramos:
    ```yaml
    type: opaque
    ```
* Almacenar credenciales de Docker:
    ```yaml
    type: kubernetes.io/dockerconfigjson
    data:
      .dockerconfigjson: <base64-encoded-docker-config>
    ```
* Autenticación básica:
    ```yaml
    type: kubernetes.io/basic-auth
    data:
      username: <base64>
      password: <base64>
    ```
* Autenticación SSH:
    ```yaml
    type: kubernetes.io/ssh-auth
    data:
      ssh-privatekey: <base64>
    ```
* certificados TLS:
    ```yaml
    type: kubernetes.io/tls
    data:
      tls.crt: <base64>
      tls.key: <base64>
    ```
* Service Account Token:
    ```yaml
    type: kubernetes.io/service-account-token
    ```

### Persistent Volume (PV)

**Persistent Volume (PV)**: recurso de almacenamiento asociado al cluster, existe independientemente de los Pods, es como una unidad de almacenamiento virtual y podemos reservar espacio de esta unidad para un Pod mediante un Persistent Volume Claim (PVC).

**⚠️ Importante**: estos solo se pueden crear por un **administrador**.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-local # nombre del volumen
spec:
  capacity:
    storage: 10Gi # Tamaño
  volumeMode: Filesystem # Filesystem o Block
  accessModes:
  - ReadWriteOnce # más modos abajo
  persistentVolumeReclaimPolicy: Retain # Retain, Delete, Recycle
  storageClassName: manual # Clase de almacenamiento
  hostPath: # Tipo de volumen (hostPath solo para testing)
    path: /mnt/data
```

Tipos de Access Modes:
* **ReadWriteOnce (RWO)**: Montado R/W por UN solo nodo.
* **ReadOnlyMany (ROX)**: Montado R/O por MÚLTIPLES nodos.
* **ReadWriteMany (RWX)**: Montado R/W por MÚLTIPLES nodos.

Tipos de Reclaim POlicy:
* **Retain**: Cuando se borra el PVC, el PV queda pero requiere limpieza manual:
* **Delete**: Borra el volumen automáticamente (cuidado con datos).

Tipos de Storage Class:
* `manual`: crearlo manualmente ya sea con comandos o con YAML (ver ejemplos abajo).
* Además de creación manual, podemos usar StorageClass para creación automática en provedores en la nube como (AWS, Azure, GCE, etc), no voy a entrar en detalle pero solo lo documento para futura investigación. Para usar un Storage Class, primero tenemos que crearlo con YAML donde indicamos las configuraciones de creación en el provedor deseado.

### Persistent Volume Claim (PVC)

Es una solicitud de almacenamiento de un Pod para reservar espacio en un PV, algo como `"Quiero 5GB de disco"` y Kubernetes le enlaza el espacio con un PV. Aquí solo lo estamos creando, pero nocesitamos enlazarlo a un contenedor creando un volumen para el PVC en `volumes` y montando el volumen con `volumeMounts` (ver ejemlos arriba).

Ciclo de vida: PVC creado → Bound a PV → Pod usa PVC → Pod muere → PVC sigue existiendo → Datos persisten.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-local # nombre del claim
spec:
  accessModes:
  - ReadWriteOnce # Debe coincidir con la definición en el PV
  resources:
    requests:
      storage: 5Gi # Cantidad que necesito (≤ PV)
  storageClassName: manual # Debe coincidir con PV
```

### Ejmplo práctico y completo para un Deployment con almacenamiento persistente para MySQL

```yaml
# Secret con credenciales para MySQL
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret # nombre del secret
type: Opaque # Tipo genérico
data: # datos, podemos agregar tantos como queramos
  username: YWRtaW4= # "admin" en base64
  password: czNjcjN0 # "s3cr3t" en base64
---
# PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-local # nombre del volumen
spec:
  ... demás configuraciones
---
# MySQL PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
 ... demás configuraciones
---
# MySQL Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: username
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql    # Datos de MySQL
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```

### Instrucciones para validar un archivo YAML

```bash
# Validar sintaxis sin crear
kubectl apply -f mi-archivo.yaml --dry-run=client

# Ver qué se creará exactamente
kubectl apply -f mi-archivo.yaml --dry-run=server -o yaml
```

### Comandos para aplicar/desaplicar un archivo YAML al cluster

```bash
# Aplicar
kubectl apply -f <file>.yaml
# Desaplicar
kubectl delete -f <file>.yaml
```

**⚠️ Importante**: si especificamos un directorio en lugar de un archivo YAML, entonces se aplicarán todos los archivos YAML que se encuentren en el directorio.

## Como hacer uso de una app levantada con un deployment

Una vez que el Deployment está corriendo en Kubernetes junto con un Service, ya podemos acceder a la app mediante la URL del nodo y el puerto que hayamos configurado en `nodePort` del Service:
```
http://<node-ip>:<nodePort>
```
Solo que hay un inconveniente muy importante, Minikube emula un cluster de un solo nodo (máquina) en la máquina local, pero resulta que dicho nodo no es la máquina local en sí, sino en una máquina virtual o contenedor, así que para acceder a la app tenemos que hacer uso de la IP de dicha máquina virtual (nodo). Podemos ver cual driver de virtualización estamos usando con:
```bash
minikube profile list
```

El problema es que el Nodo está en una red virtual aislada del host por seguridad, así que no será posible acceder a la app desde el host de manera directa, pero a continuación vemos algunas formas de acceder de manera indirecta.

### Conectarse al nodo virtual via SSH y hacer peticiónes desde ahí:

* Primero obtenemos la IP del nodo:
    ```bash
    # Regresa la IP que tiene el nodo emulado
    minikube ip
    ```
* Nos conectamos al nodo y ejecutamos la petición:
    ```bash
    # Nos conectamos al nodo
    minikube ssh
    # Realizamos la petición a la IP del nodo obtenida en el paso anterior y el nodePort configurado en el Service
    curl http://192.168.49.2:30801
    ```

### Crear un tunel temporal desde el host hacia el nodo virtual

Con esta forma podremos acceder a la app como si estuviera corriendo en el host (127.0.0.1), solo tenemos que crear un tunel (port-forward) entre el host y el nodo virtual, como si el host fuera el nodo:
```bash
# Además de crear el tunel, muestra la tabla de direcciones internas del nodo virtual y abre automáticamente el navegador con la app usando el tunel
minikube service <service-name>

# O el siguiente que crea el tunel y solo muestra la URL que generó el tunel para usarla
minikube service <service-name> --url

# Ayuda acerca del comando
minikube service -h
```

⚠️ El problema es que cada que se abre un tunel el puerto es diferente, así que complica un poco las cosas ya que hay que estar actualizando el puerto en las peticiones.

⚠️ En cualquier caso la terminal queda bloqueada porque el tunel queda corriendo, podemos cerrar el tunel con `CTRL+C` en cuyo caso la app dejará de respoder en la URL local.

### Crear un tunel temporal a un Deployment, Pod o Service directamente

En el método anterior, se creaba un tunel desde el host hacia el nodo virtual de minikube. En este caso veremos como crear un tunel del host directamente a un Depolyment, Pod o Service, es útil para realizar pruebas hacia algún enpoint o conjunto de endpoints. La sintaxis general es:
```bash
kubectl port-forward <resource>/<resource-name> <hostPort>:<resourcePort>
```
Ejemlplos:
```bash
# Tunel hacia un servicio que redirige de 127.0.0.1:8000 -> mi-service:7000
kubectl port-forward service/mi-service 8000:7000

# Tunel a un Pod que redirige de 127.0.0.1:8888 -> mi-pod:5000
kubectl port-forward pod/mi-pod 8888:5000
```

⚠️ En este caso como nosotros definimos el puerto, este puede quedar igual, a diferencia del tunel hacia el nodo.

⚠️ En cualquier caso, la terminal quedará bloqueada con el puente abierto, el cual puede cerrarse con `CTRL+C`.

### Simular un cluster real en la nube

Cuando usamos Kubernetes en un cluster real en la nube no necesitamos ningún tunel ni nada, desafortunadamente en clusters simulados localmente no hay forma de evitarlo ya que el cluster se simula en una máquina virtual la cual no está expuesta al exterior.

Otra forma de hacer uso de los servicios desde el host es cambiar los servicios de tipo `NodePort` a `LoadBalancer`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-service
spec:
  type: LoadBalancer # <-- aquí se realiza el cambio
  # ... demás configuraciones
```

**⚠️ Importante**: de hecho esta es la mejor estrategia uando `type: LoadBalancer` ya que así es como se tiene que definir al subir a la nube, ya que los servicios expuestos deben poder ser accedidos en el puerto fijo definido en el archivo YAML.

Una vez así, y habiendo aplicado los cambios al cluster, ejecutamos el siguiente comando en una terminal, la cual quedará bloqueada, para abrir un tunel hacia los servicios de tipo LoadBalancer:
```bash
minikube tunnel
```

Con eso ya podremos conectarnos a los servicios de tipo `LoadBalancer` directamente desde el host via `http://127.0.0.1:<service-port>`, donde `<service-port>` es el puerto (`port`) que le hayamos configurado al servicio en el archivo YAML.

## Crear imágenes desde Dockerfile que puedan integrarse en un Deployment

No podemos usar el Dockerfile directamente en un deployment, primero tenemos que construir la imágen desde el Dockerfile, subirla aun registro que puede ser `Docker Hub` o para pruebas puede ser el registro de `Minikube` y finalmente referenciarla en el Deployment. Para estos ejemplos usaremos el registro de Minikube.

Recordemos que Minikube usa un Docker daemon, para facilitar la integración con el registro de Minikube, tenemos que configurar la terminal para que use el Docker daemon de Minikube en lugar de el del sistema. Para esto ejecutamos el siguiente comando:

```bash
# Windows (Powershell)
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
# Linux
eval $(minikube docker-env)

# Verifica que estás usando el Docker de Minikube: deberías ver los contenedores de Kubernetes
docker ps
```

`Nota`: De hecho el comando para realizarlo debería verse en el mensaje que se muestra al ejecutar `minikube docker-env`.

Con esto las imágenes que construyamos ya estarán disponibles directamente en Minikube sin necesidad de hacer `docker push`.

El flujo completo para integrar una imágene en un deployment sería el siguiente:

* Primero implementamos el Dockerfile.
* Construimos la imágen:
    ```bash
    # Construir la imágen
    docker build -t <image-name> .

    # Opcionalmente podemos verificar que existe
    docker images | grep <image-name>
    ```
* La usamos directamente en el Deployment:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mi-deployment
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: mi-app
      template:
        metadata:
          labels:
            app: mi-app
        spec:
          containers:
          - name: mi-app
            image: <image-name> # nombre de la imagen recién creada
            imagePullPolicy: Never # ← CRÍTICO: No intentes descargar
            ports:
            - containerPort: 8000
    ```


# Glosario de comandos para kubectl

Algunos de los comandos básicos y principales son los siguientes:

Información general:
* `kubectl cluster-info`: revisar el estado del cluster.
* `kubectl get all`: revisar el estado de todos los recursos.
* `kubectl get nodes`: revisar el estado de los nodos.
* `kubectl get deployments`: revisar el estado de los depoyments.
* `kubectl get pods`: ver los Pods.
* `kubectl get pods -o wide`: ver los Pods incluyendo IP y Nodo.
* `kubectl get pods -l app=<deploy-name>`: ver pods de un deployment.
* `kubectl get pods -n kube-system`: Ver componentes del Control Plane.
* `kubectl get pod <pod-name> -o yaml`: ver pod en formato YAML.
* `kubectl get services`: revisar el estado de los servicios.
* `kubectl get replicasets`: ver los ReplicaSet.
* `kubectl get rs -l app=<deploy-name>`: ver cuál ReplicaSet pertenece a un Deployment.
* `kubectl get endpoints`: ver los endpoints existentes de los distintos servicios (IPs de los Pods).
* `kubectl get endpoints <deploy-name>`: ver endpoints de un deploy (IPs de los Pods).
* `kubectl get configmaps`: ver los config maps.
* `kubectl get secrets`: ver los secrets.
* `kubectl get pvc`: ver los persistent volume claims.
* `kubectl get events`: ver todos los eventos (Pod Scheduled, Pulling image, Container started, etc).
* `kubectl get events --sort-by='.lastTimestamp'`: eventos ordenados por tiempo.
* `kubectl get events --field-selector involvedObject.name=<deploy-name>`: eventos de un deployment.

Información detallada:
* `kubectl describe pod <pod-name>`: detalle de un Pod (incluye eventos al final).
* `kubectl describe rs <replicaset-name>`: detalle de un ReplicaSet.
* `kubectl describe service <deploy-name>`: detalle de un servicio.
* `kubectl describe endpoints <service-name>`: ver los endpoints de un servicio en particular.
* `kubectl describe endpoints <service-name>`: ver los endpoints de un servicio en particular.
* `kubectl logs <pod-name>`: logs de un pod.
* `kubectl logs -n kube-system kube-apiserver-minikube`: logs del API Server.
* `kubectl logs -n kube-system kube-scheduler-minikube`: logs del Scheduler.
* `kubectl api-resources`: Lista de tipos de recursos disponibles.

Métricas:
* `kubectl top pods`: recursos consumidos por los Pods.
* `kubectl top nodes`: recursos consumidos por los nodos.

Crear recursos:
* `kubectl create deployment <deploy-name> --image=<imagen>`: crear un deployment desde una imagen.
* `kubectl expose deployment <deploy-name> --type=NodePort --port=<puerto>`: exponer un deployment mediante un puerto, si no ponemos el argumento `--type=NodePort` el servicio será de tipo `ClusterIP` que no debe exponerse al público por seguridad.
* `minikube service <deploy-name> --url`: obtener la url del deployment una vez expuesta a un puerto.
* `kubectl scale deployment <deploy-name> --replicas=3`: cambiar el número de replicas de un deployment.

Eliminar recursos:
* `kubectl delete deployment <deploy-name>`: eliminar un deployment.
* `kubectl delete service <deploy-name>`: eliminar un servicio, los pods siguen vivos solo que ya no están expuestos al exterior.
* `kubectl delete pod <pod-name>`: eliminar un Pod (Se recreará automáticamente por el ReplicaSet).

Actualización:
* `kubectl exec -it <pod-name> -- /bin/bash`: entrar a un contenedor en modo terminarl.
* `kubectl scale deployment <deploy-name> --replicas=0`: detener los Pods del deployment sin eliminarlo.
* `kubectl rollout pause deployment <deploy-name>`: pausa todas las actualizaciones pero sigue corriendo.
* `kubectl rollout resume deployment <deploy-name>`: reanuda las actualizaciones.
* `kubectl rollout history deployment <deploy-name>`: ver historial de cambios.




