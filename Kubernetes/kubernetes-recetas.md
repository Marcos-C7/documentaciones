# Kubernetes: Guía de Referencia y Recetas

Esta documentación está diseñada para ser una referencia rápida y práctica. Cada sección es independiente y contiene todo lo necesario para trabajar con ese recurso específico.

---

## ⚡ Quick Reference (Cheat Sheet)

### Ciclo de Desarrollo Básico
1. **Codificar**: Crear la app y el `Dockerfile`.
2. **Construir**: `docker build -t mi-imagen:v1 .` (o usar el docker env de minikube).
3. **Desplegar**: `kubectl apply -f mi-recurso.yaml`.
4. **Verificar**: `kubectl get all` / `minikube service <nombre>`.

### Comandos Esenciales de Kubectl

| Acción | Comando | Descripción |
| :--- | :--- | :--- |
| **Estado** | `kubectl get all` | Ver todo en el namespace actual. |
| **Pods** | `kubectl get pods -o wide` | Ver pods con IP y nodo. |
| **Logs** | `kubectl logs <pod-name>` | Ver logs de un pod. |
| **Detalle** | `kubectl describe <tipo> <nombre>` | Ver detalle y eventos (útil para debug). |
| **Ejecutar** | `kubectl exec -it <pod> -- /bin/sh` | Entrar a la terminal del pod. |
| **Borrar** | `kubectl delete -f archivo.yaml` | Borrar recursos definidos en el archivo. |
| **Contexto** | `kubectl config get-contexts` | Ver clusters disponibles. |

---

## 🛠️ Configuración del Entorno

### Minikube
Simula un cluster de un solo nodo. Ideal para desarrollo local.

*   **Iniciar**: `minikube start` (o `minikube start --cpus=4 --memory=4096` para más recursos).
*   **Detener**: `minikube stop`.
*   **Borrar**: `minikube delete` (⚠️ Borra todo, incluidos volúmenes).
*   **Dashboard**: `minikube dashboard` (abre interfaz web).
*   **Docker Env**: `& minikube -p minikube docker-env --shell powershell | Invoke-Expression` (Para usar el daemon de docker de minikube y no tener que subir imágenes a un registry externo).

### Docker Desktop
Alternativa integrada en Docker Desktop.
*   **Activar**: Settings -> Kubernetes -> Enable Kubernetes.
*   **Contexto**: `kubectl config use-context docker-desktop`.

---

## 📦 Receta: Namespace
**Concepto**: Un cluster virtual dentro del cluster físico. Sirve para aislar proyectos o entornos (dev, prod).

### YAML Template
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mi-proyecto-dev # Nombre del namespace
```

### Comandos Útiles
*   **Crear**: `kubectl apply -f namespace.yaml`
*   **Listar**: `kubectl get namespaces`
*   **Usar por defecto**: `kubectl config set-context --current --namespace=mi-proyecto-dev` (Evita poner `-n <ns>` en cada comando).

---

## 🐳 Receta: Pod
**Concepto**: La unidad atómica de Kubernetes. Envuelve uno o más contenedores. Generalmente no se crean directamente, sino a través de Deployments.

### YAML Template
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-pod
  labels: # Etiquetas para identificarlo (usadas por Services y Deployments)
    app: mi-app
    tier: backend
spec:
  containers:
  - name: nginx-container
    image: nginx:alpine
    ports:
    - containerPort: 80
    resources: # Solicitud y límites de hardware
      requests:
        memory: "64Mi"
        cpu: "250m" # 250 milicores (1/4 de CPU)
      limits:
        memory: "128Mi"
        cpu: "500m"
    env: # Variables de entorno directas
    - name: ENVIRONMENT
      value: "development"
```

### Tips y Advertencias
*   **Efímeros**: Si un Pod muere, muere. No se "reinicia" solo (para eso usa Deployments).
*   **IP Dinámica**: Su IP cambia cada vez que se recrea. No confíes en la IP del Pod, usa un **Service**.
*   **Debug**: Si falla, usa `kubectl describe pod <nombre>` para ver la sección "Events" al final.

---

## 🚀 Receta: Deployment
**Concepto**: Administra copias (réplicas) de Pods. Asegura que siempre haya X número de pods corriendo. Permite actualizaciones sin caída (Rolling Updates).

### YAML Template
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-web-app
spec:
  replicas: 3 # Cantidad de pods deseados
  selector:
    matchLabels: # DEBE coincidir con los labels del template
      app: mi-web
  strategy:
    type: RollingUpdate # Actualización gradual
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template: # La plantilla del Pod (ver receta de Pod)
    metadata:
      labels:
        app: mi-web # Etiqueta fundamental
    spec:
      containers:
      - name: web
        image: nginx:1.19
        ports:
        - containerPort: 80
```

### Comandos Comunes
*   **Escalar**: `kubectl scale deployment mi-web-app --replicas=5`
*   **Actualizar Imagen**: `kubectl set image deployment/mi-web-app web=nginx:1.20`
*   **Deshacer Cambio**: `kubectl rollout undo deployment/mi-web-app`
*   **Historial**: `kubectl rollout history deployment/mi-web-app`

### Tips
*   **MatchLabels**: Es inmutable. Si te equivocas aquí, tienes que borrar el Deployment y volver a crearlo.
*   **Reiniciar**: `kubectl rollout restart deployment/mi-web-app` (útil si cambiaste un ConfigMap/Secret y quieres que los pods lo recarguen).

---

## 🌐 Receta: Service
**Concepto**: Una IP estable y DNS para un grupo de Pods. Balancea la carga entre ellos.

### YAML Template
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-servicio # Este nombre será el DNS interno (http://mi-servicio)
spec:
  type: NodePort # Opciones: ClusterIP (default, solo interno), NodePort (expone puerto en nodo), LoadBalancer (nube)
  selector: # Conecta con los Pods que tengan estos labels
    app: mi-web
  ports:
  - protocol: TCP
    port: 80      # Puerto del Servicio (el que usas para llamar)
    targetPort: 80 # Puerto del Contenedor (donde escucha tu app)
    nodePort: 30080 # (Opcional, solo NodePort) Puerto externo 30000-32767
```

### Tipos de Servicio
1.  **ClusterIP**: Solo accesible desde DENTRO del cluster.
2.  **NodePort**: Abre un puerto en la IP del nodo (ej. `minikube ip`:30080).
3.  **LoadBalancer**: Pide una IP pública al proveedor de nube (AWS, GCP). En Minikube requiere `minikube tunnel`.

### Acceso en Minikube
*   **Comando Mágico**: `minikube service mi-servicio` (Abre el navegador directamente).
*   **URL**: `minikube service mi-servicio --url`.

---

## ⚙️ Receta: ConfigMap
**Concepto**: Guarda configuración no sensible (archivos .conf, variables, properties) para desacoplar la configuración del código.

### YAML Template
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Como variable simple
  COLOR_TEMA: "azul"
  # Como archivo completo
  nginx.conf: |
    server {
      listen 80;
      server_name localhost;
    }
```

### Cómo usarlo en un Pod
**Como Variables de Entorno:**
```yaml
envFrom:
- configMapRef:
    name: app-config
```

**Como Volumen (Archivo):**
```yaml
volumes:
- name: config-vol
  configMap:
    name: app-config
volumeMounts:
- name: config-vol
  mountPath: /etc/nginx/conf.d # Donde aparecerán los archivos
```

---

## 🔐 Receta: Secret
**Concepto**: Igual que ConfigMap pero para datos sensibles (passwords, tokens). Se guardan en base64 (pero no están encriptados por defecto en etcd).

### YAML Template
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
type: Opaque
data:
  # Valores DEBEN estar en base64 (echo -n "admin" | base64)
  username: YWRtaW4= 
  password: czNjcjN0
```

### Tips
*   **Creación rápida**: `kubectl create secret generic db-creds --from-literal=username=admin --from-literal=password=secret123` (te ahorra el base64 manual).
*   **Seguridad**: No subas el YAML de secrets a Git. Usa herramientas como SealedSecrets o inyéctalos en el CI/CD.

---

## 💾 Receta: PersistentVolume (PV) y PVC
**Concepto**: Almacenamiento que sobrevive al Pod.
*   **PV**: El disco físico (o virtual).
*   **PVC**: El "ticket" o solicitud que hace el Pod para pedir disco.

### YAML Template (PVC - Lo que usas normalmente)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce # RWO: Montado por un solo nodo (común en DBs)
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard # En minikube 'standard' crea el PV automáticamente
```

### Cómo usarlo en un Pod
```yaml
volumes:
- name: mysql-store
  persistentVolumeClaim:
    claimName: mysql-pvc
volumeMounts:
- name: mysql-store
  mountPath: /var/lib/mysql
```

### Tips Minikube
*   Minikube tiene un provisionador automático (`standard`). Solo crea el PVC y el PV aparecerá solo.
*   Los datos se guardan dentro de la VM de Minikube. Si haces `minikube delete`, adiós datos.

---

## 📈 Receta: Horizontal Pod Autoscaler (HPA)
**Concepto**: Escala automáticamente el número de Pods según CPU o Memoria.

### Requisitos
*   Habilitar métricas: `minikube addons enable metrics-server`.
*   El Deployment **DEBE** tener `resources.requests` definidos (CPU/Memoria).

### YAML Template
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: mi-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mi-web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70 # Escala si el promedio de CPU > 70%
```

### Comandos
*   **Ver estado**: `kubectl get hpa` (Muestra TARGETS ej: 15%/70%).
*   **Monitoreo**: `kubectl top pods`.

---

## 🕵️ Troubleshooting & Advanced

### Debugging Paso a Paso
1.  **¿Está corriendo el Pod?**
    *   `kubectl get pods` -> Si dice `ImagePullBackOff` o `CrashLoopBackOff`.
2.  **¿Por qué falló?**
    *   `kubectl describe pod <nombre>` -> Lee los eventos al final.
    *   `kubectl logs <nombre>` -> Lee la salida de la aplicación.
3.  **¿Problema de Red?**
    *   Entra al pod: `kubectl exec -it <pod> -- sh`.
    *   Prueba conexión: `curl http://otro-servicio`.

### Port Forwarding (Túnel Manual)
Si quieres probar un servicio o pod específico desde tu máquina sin exponerlo:
```bash
# Redirige puerto 8080 de tu PC al 80 del servicio
kubectl port-forward service/mi-servicio 8080:80
```
Ahora entra a `localhost:8080`.

### Imágenes Locales en Minikube
Si tu pod da error `ErrImagePull` o `ImagePullBackOff` con una imagen local:
1.  Asegúrate de haber construido la imagen DENTRO de minikube (ver sección Quick Reference).
2.  En el YAML del Deployment, pon: `imagePullPolicy: Never` o `IfNotPresent`.
