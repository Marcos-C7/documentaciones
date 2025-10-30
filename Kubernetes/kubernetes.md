# Kubernetes

Kubernetes es un sistema open-source de oquestación de contenedores Docker para automatizar el despliegue, escalado y mantenimiento de software en la nube. Su arquitectura, a grandes rasgos se compone de las siguientes partes:

### Componentes de la arquitectura de Kubernetes

* **Cluster**: es un conjunto de nodos (máquinas) enlazadas mediante Kubernetes.
* **Nodo**: una máquina física o virtual, cada nodo ejecuta las siguientes componentes:
    * **kubelete**: responsable del estado de ejecución del nodo asegurándose de que todos contenedores del nodo estén saludables, así como de iniciarlos, detenerlos y organizarlos en Pods. Consulta las especificaciones de los Pods mediante el API Server y se asegura de que los contenedores de cada Pod estén corriendo.
    * **Container runtime**: encargado del ciclo de vida de los contenedores, los crea, los detiene y elimina. Tiene su propia interfaz de comandos llamada **Container Runtime Interface (CRI)**. Los más usuales son Containerd (el motor de Docker), CRI-O.
    * **Kube proxy**: es una red proxy y laod balancer, se encarga de dirigir el tráfico al contenedor apropiado en base a la IP y el Puerto así como de balancear la carga entre las replicas de un deployment.
* **Pod**: es la unidad básica que puede ser agendada para desplegarse, este contiene los contenedores y se le asigna una IP única en el cluster, dentro del cluster sus contenedores tienen comunicación entre sí. Estos se suelen crear al crear un Deployment y su nombre suele tener la estructura `<deploy-name>-<replicaset>-<hash>`, esto ya que un deployment tiene un replicaset que se encarga de crear un Pod para cada replica.
* **Contenedor**: un contenedor de Docker, que vive dentro de un Pod y además ejecuta una aplicación como parte de un microservicio.
* **Control plane**: es un nodo cerebro en el que se ejecutan las principales componentes que administran el cluster, como son:

    * **API Server** (`kube-apiserver`): es la interfaz de comuniación tipo REST por HTTP para poder darle instrucciones de actualización del cluster a Kubernetes. Es el único punto de comunicación tanto para el humano administrador como entre las mismas componentes, como los controladores.
    * **Etcd** (`etcd`): tienda de datos persistente llave-valor que sirve como punto de encuentro para matener todo el cluster sincronizado, aquí se almacena y consulta todo el estado del cluster (pods, services, deployments, secrets, etc), aquí se guarda absolutamente todo lo relacionado con el estado actual y futuro.
    * **Scheduer** (`kube-scheduler`): se encarga de determinar en qué nodo va a ejecutarse cada Pod, tomando en cuenta los recursos disponibles (RAM, CPU, etc), solo agenda la ejecución no lo ejecuta. Si detecta un pod no asignado, evalúa los nodos existentes y asigna el pod al nodo mejor puntuación. Se puede correr más de un scheduler y podemos implementar los nuestros propios tipo plug-ins para extender o modificar el funcionamiento por defecto.
    * **Controller Manager** (`kube-controller-manager`): se encarga de la ejecución y administración de los controladores, los cuales se encargan de mantener el cluster en el estado deseado. Algunos de los controladores son:
        * **Controlador de replicación**: cada deployment debe tener un `ReplicaSet` que almacena cuantas replicas se desean y cuantas existen, este controlador se encarga de que ambos números coincidan, si falta o muere alguno levanta nuevos y si hay de más los elimina. Esto convierte a Kubernetes en auto-reparable.
        * **Controlador de nodos**: vigila la salud de los nodos y si alguno deja de responder reasigna sus Pods a otros nodos.
        * **Controlador de deployment**: gestiona los deployments, creándoles o actualizándoles su `ReplicaSet` el cual se encargará de crear las réplicas de los Pods.
            * **Depoloyment**: es la descripción de una aplicación, qué imágen va a usar, cuantas réplicas quieres corriendo, etc. Tú la describes y kubernetes se encarga de mantenerla, acualizarla y repararla recreando pods muertos, entre otras cosas.
        * **Controlador de servicios**: gestiona los servicios y configura balanceadores de carga.
            * **Servicio**: para que un Pod sea accesible desde el exterior se tiene que exponer mediante un servicio, el cual agrupa bajo una IP a uno o más Pods de un deployment. De esta forma, podemos tener 3 pods de un deployment levantados con IP diferentes y agruparlos bajo una misma IP de un servicio que se encargue de reenviar las solicitudes hacia las IPs de los pods.

En resumen, el cliclo de vida del cluster se resume en lo siguiente:
```
Loop infinito:
1. Observar el estado actual (desde etcd vía el API Server).
2. Comparar con el estado deseado.
3. Si hay diferencia, tomar acciones.
```

### Componentes de la red del cluster

* **Pod IP**: cada Pod tiene IP única en el cluster y cambia si el Pod se recrea.
* **Service Cluster-IP**: IP virtual estable, no cambia y solo es accesible dentro del cluster.
* **Service Node-Port**: para exponer el servicio en un puerto de cada nodo donde tiene pods el servicio, con este puerto ya podemos acceder al servicio externamente, no solo desde dentro del cluster, para acceder al servicio en alguno de los nodos es con `<NodeIP>:<NodePort>`.
* **Service Load-Balancer**: solo en clouds públicos, crea un balanceador de carga externo.
* **CNI (Container Network Interface)**: modelo de red, Minikube usa `bridge` o `calico` por defecto, pero existe también `flannel`, `cilium` o `weave`.

### Volúmenes

* **Volume**: directorio asociado a un Pod que es accesible por sus contenedores, vive mientras viva el Pod, puede ser de tipo **emptyDir**, **hostPath**, **configMap** o **secret**.
* **Persistent Volume (PV)**: recurso de almacenamiento asociado al cluster, existe independientemente de los Pods.
* **Persistent Volume Claim (PVC)**: solicitud de almacenamiento, ej. un Pod hace un PVC `"Quiero 5GB de disco"` y Kubernetes le enlaza el espacio con un PV.


### Minikube

Al ser Kubernetes open-source, pueden existir implementaciones diferentes adaptadas para la infraestructura objetivo. En nuestro caso, usaré `Minikube` que es una implementación de Kubernetes para una computadora local que simula un cluster de Kubernetes de un solo nodo.

#### Iniciar el cluster

Antes de iniciar el cluster tenemos que asegurarnos de que Docker está corriendo, si tenemos **Docker Desktop** en Windows instalado hay que iniciarlo. Una vez iniciado ejecutamos el siguiente comando:

```bash
minikube start
```

otros comandos de minikube:
```bash
# SSH al nodo de Minikube
minikube ssh

# Una vez dentro podremos explorar sus procesos:
# Ver procesos del kubelet
ps aux | grep kubelet
# Ver reglas de iptables (Services)
sudo iptables-save | grep <service-name>
```

### Kubectl

Kubernetes tiene una API tipo REST via HTTP llamada `API Server` (`kube-apiserver`) para poder interactuar y darle instrucciones cuando queramos realizar alguna actualización en el cluster. Hay varias herramientas oficiales que sirven de intermediarias con las cuales podemos interactuar con la API de manera más sencilla sin tener que realizar las peticiones HTTP nosotros mismos, algunas son:

* `kubectl`: una interfaz de linea de comandos integrada en Kubernetes.
* Librerías para distintos lenguajes de programación: **Python**, **C**, **Javascript** entre otros.

En nuestro caso usaremos `kubectl` pero no debería ser difícil implementar automatizaciones mediante las librerías de los lenguajes mencionados anteriormente.


## Flujos de trabajo

A continiuación se organizan flujos básicos de trabajo para desarrollar alguna actividad de principio a fin:

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

### Flujo de ejecución de los procesos de eliminación

A continuación se describe lo que ocurre internamente al eliminar algún recurso.

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

Una funcionalidad de Kubernetes muy importante es que permite realizar actualizaciones a los deployments en vivo sin tener que apagarlas o reiniciarlas manualmente, solo le indicas el cambio que quieres y Kubernetes lo realiza de la manera más adecuada y automáticamente. Algunos ejemplos son los siguientes:


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

### Modelo de comunicación imperativo y declarativo

Hay dos formas e indicarle a Kubernetes qué hacer:

* **Modelo imperativo**: es mediante comandos con `kubectl` como `kubectl describe pod <nombre>`, le dices que hacer paso por paso.
* **Modelo declarativo**: es mediante un archivo **YAML** como el siguiente, en este le describes el estado final al que quieres que lleguen tus deployments, tomando en cuenta pods, deployments, etc, y Kubernetes calcula los pasos para llegar ahí desde el estado actual.
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


El modo preferido es el declarativo que es mediante archivos YAML, ya que es más predecible y además versionable con herramientas como Git.

## Estrucrura del archivo YAML

A continuación revisaremos ejemplos de algunas configuraciones que se pueden especificar en un archivo YAML para crear un recurso, en este caso un Pod. Se revisarán las configuraciones gradualmente solo manteniendo en cada ejemplo las nuevas configuraciones junto con sus ancestros para no perder la estructura.

Reglas del formato YAML:
* YAML usa espacios, no tabulaciones.
* Cada nivel de identación es de 2 espacios, no de 4.

#### Ejemplo básico para crear un Pod:

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

#### Más configuraciones para los metadatos

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

#### Más configuraciones para puertos y ejecución de los contenedores:

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

#### Configuraciones de variable sde entorno para el contenedor

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

#### Más configuraciones para recursos del sistema para el contenedor

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

#### Configuraciones para prueba de vida de los contenedores

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

#### Configuraciones para volúmenes básicos

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

#### Ejemplo de YAML para un Deployment

Ahora en lugar de definir un Pod, definiremos la estructura de un Deployment, comenzaremos con algo básico. Obviamente la mayoría de las configuraciones son opcionales.

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

#### Ejemplo de YAML para un Service

Ahora veremos un ejemplo para definir un Service en un archivo YAML, también las mayoría de las configuraciones son opcionales.

```yaml
apiVersion: v1 # versión para Services
kind: Service
metadata:
  name: mi-service
spec:
  type: NodePort # Tipo de Service, también hay `ClusterIP`, `LoadBalancer`, `ExternalName`.
  selector: # Qué Pods incluir en el servicio
    app: mi-app
  ports:
  - port: 80 # Puerto del Service
    targetPort: 80 # Puerto del contenedor
    nodePort: 30080 # Puerto en el nodo (30000-32767)
    protocol: TCP
    name: http
```

#### Ejemplo YAML para crear entradas para el configMap

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

#### Ejemplo YAML para crear entradas de Secret

Los secrets deben ser almacenados en codifiación `base64`. Los Secrets se almacenan en el `etcd` encriptados (si está configurado así). Solo se envían a los nodos que tienen al menos un Pod que los necesita. Kubelet guarda los Secrets en RAM, y son montados en los contenedores como archivos en memoria y al morir el Pod el Secret se elimina de la memoria.

**⚠️ Importante:** Los Secrets NO están encriptados en etcd por defecto. Debes configurar encryption-at-rest.

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

#### Ejemplo YAML para crear Persistent Volume (PV) y Persistent Volume Claim (PVC)


**Persistent Volume (PV)**: recurso de almacenamiento asociado al cluster, existe independientemente de los Pods, es como una unidad de almacenamiento virtual y podemos reservar espacio de esta unidad para un Pod mediante un Persistent Volume Claim (PVC).

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

**Persistent Volume Claim (PVC)**: solicitud de almacenamiento de un Pod para reservar espacio en un PV, algo como `"Quiero 5GB de disco"` y Kubernetes le enlaza el espacio con un PV. Aquí solo lo estamos creando, pero nocesitamos enlazarlo a un contenedor creando un volumen para el PVC en `volumes` y montando el volumen con `volumeMounts` (ver ejemlos arriba).

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

#### Ejmplo de puesta en práctica para un Deployment con almacenamiento persistente para MySQL

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

#### Instrucciones para validar un archivo YAML

```bash
# Validar sintaxis sin crear
kubectl apply -f mi-archivo.yaml --dry-run=client

# Ver qué se creará exactamente
kubectl apply -f mi-archivo.yaml --dry-run=server -o yaml
```

#### Comandos para aplicar/desaplicar un archivo YAML al cluster

```bash
# Aplicar
kubectl apply -f <file>.yaml:
# Desaplicar
kubectl delete -f <file>.yaml
```


# Glosario de comandos

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


