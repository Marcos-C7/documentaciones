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
        * **Controlador de deployment**: gestiona los deployments, actualiza los `ReplicaSet`.
            * **Depoloyment**: <span style="color:red;">FALTA DESCRIBIR ESTA PARTE</span>
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
    # Si creaste tus recursos con: kubectl apply -f <mi-app>.yaml:
    kubectl delete -f mi-app.yaml
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




