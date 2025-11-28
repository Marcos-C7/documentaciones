# Podman

Podman es un sistema de orquestación de contenedores similar a Docker pero sin necesidad de un demonio de Docker en el sistema. En lugar de ejecutar comandos como `docker run`, `docker build`, etc, se ejecutan comandos como `podman run`, `podman build`, etc. Los contenedores no son específicos de Docker, sino que siguen la especificación OCI para que puedan ser implementados por cualquier herramienta que lo desee, y tanto Docker como Podman son dos herramientas que siguen la especificación OCI de contenedores.

Como ambos son implementaciones diferentes, tienen sus diferencias en el manejo de los detalles. Por eso aquí enlistaré una lista de problemas y soluciones para que podamos usar el mismo entorno tanto en Docker como en Podman.

La arquitectura de Podman es la siguiente:
```
+-------------------------------+   <-- Vive en el host Windows (tu PC física), es la interfaz gráfica para administrar Podman
| Podman Desktop GUI (Electron) |       
| - Dashboard, Extensions       |       
| - kubeconfig (~/.kube/config) |       
+-------------------------------+
            | (SSH/Socket API)
            v
+-------------------------------+   <-- Vive en WSL2 como una distribución más, podemos verla ejecutanto `wsl -l -v`
| Podman Machine (WSL2 Distro)  |       Es la máquina que maneja los contenedores, todos los contenedores se crean dentro de esta
| - Podman Engine (rootless)    |       También ejecuta Kind que simula un cluster de Kubernetes de un solo nodo usando contenedores
| - Servicio Podman (socket)    |       
+-------------------------------+
            | (Podman Runtime)
            v
+-------------------------------+   <-- Contenedores dentro de Podman Machine
| Kind Cluster (nodos K8s)      |       Contenedores Podman (no VM separada)
| - Control-plane (etcd, API)   |       e.g., "kind-control-plane" container
| - Workers (kubelet, etc.)     |       Comunicación: kubeconfig desde host
+-------------------------------+
```

Recordemos que `kubectl` es una herramienta general para interactuar con clusters de Kubernetes (locales o remotos), no forma parte ni de Minikube, ni de Docker, ni de Podman, ni nada, solo es una herramienta que se encarga de interactuar con el API Server de clusters a través de la configuración `~/.kube/config`. Cuando ejecutamos un simulador de cluster, este se registra al arrancar en `kubectl` y se puede ver en `kubectl config get-contexts`, así podemos especificar hacia qué cluster enviar los comandos.

## Rootless/Rootful

Podman puede ejecutarse en modo rootless (sin privilegios de administrador) o rootful (con privilegios de administrador). En modo rootless, el usuario no necesita ser root para ejecutar comandos de Podman, pero en modo rootful, el usuario necesita ser root para ejecutar comandos de Podman.

La diferencia importante radica en que el modo rootless implica mayores restricciones en cuando a permisos, principalmente cuando montamos carpetas como volúmenes para que los contenedores generen datos.

## Solución de problemas

### Compartir permisos entre carpetas del host que son montadas como volúmenes

Por lo general necesitamos montar volúmenes en los contenedores para que puedan acceder a archivos del host. Algunas veces, aunque no es lo recomendado, queremos usar carpetas del host para almacenar los datos generados por un contenedor, como una base dedatos (postgres, mysql, etc). 

Lo recomendado es crear volúmenes nombrados con la herramienta compose (docker compose o podman compose) y usar este en el contenedor:
```yaml
services:
  postgres:
    image: postgres
    volumes:
      - db_data:/var/lib/postgresql/data
volumes:
  db_data:
```
* Funciona perfectamente en Docker y Podman (rootless o rootful)
* Los datos persisten
* No hay conflictos de permisos
* Pierdes la ventaja de “ver los archivos directamente en Windows”, pero ganas estabilidad total.
* Los datos no se comparten entre Docker y Podman.

**⚠️ Importante**: los volúmenes nombrados se crean dentro de la máquina virtual de Podman, si queremos ver la ruta donde fueron creados dentro de la VM, primero verifiquemos los nombres de los volúmenes con `podman volume ls` y luego ejecutamos el comando `podman volume inspect <volume-name>`.

Para compartir archivos entre Docker y Podman usualmente tendremos algo como:
```yaml
services:
  postgres:
    image: postgres
    volumes:
      - ./.docker_vols/postgresql:/var/lib/postgresql/data
```

El problema es que si inicializamos la carpeta con Docker Compose, luego Podman no tendrá permiso para hacer uso de dicho volúmen. Con esto, la primera vez que inicializas con Docker, crea los archivos con propietario `999:999`. Cuando Podman arranca después, el usuario mapeado no puede tocar esos archivos → Postgres se niega a iniciar.

#### Solución

Para este caso, la única solución que encontré fue activar el modo Rootful (settings -> Resources -> Podman -> Editar -> Activar "Machine with root privileges"). De esa forma Podman funciona exactamente igual que Docker y ya no habrá conflictos con los permisos.

Usar el modo Rootless es lo adecuado para producción, pero si vamos a usar Docker compose para desarrollo, entonces no hay problema.


