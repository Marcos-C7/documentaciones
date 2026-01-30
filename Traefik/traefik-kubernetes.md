# Traefik en Kubernetes

Para usar Traefik en Kubernetes necesitamos usar una versión especial que integra un cerebro tipo Kubernetes que permite aplicar cambios sin interrumpir la disponibilidad de la aplicación y que además se comunica con la API de Kubernetes para autodescubrir los componentes y realizar las conexiones de manera automática apartir de los `Ingress` o `IngressRoute` (CRDs).

### Instalación

El Traefik Ingress-Controller no se instala en realidad, sino que se levanta como aplicativo en la red (Deployment, ConfigMap, Service, permisos RBAC, etc). Para esto se aplica un archivo YAML con todos los recursos necesarios, el cual podemos instalar directamente desde la URL o podemos descargar el archivo e incorporarlo a nuestra carpeta de archivos YAML para levantar todo en conjunto.

Primero creamos el namespace para traefik:
```bash
kubectl create namespace traefik
```

Cargamos los CRDs:
```bash
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v3.3/docs/content/reference/dynamic-configuration/kubernetes-crd-definition-v1.yml
```

Aplicamos los permisos RBAC para que Traefik pueda trabajar con los CRDs y con recursos estándar:
```bash
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v3.3/docs/content/reference/dynamic-configuration/kubernetes-crd-rbac.yml
```

La aplicación directa desde la URL es:
```bash
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v3.3/docs/content/reference/dynamic-configuration/kubernetes-crd-definition-v1.yml
```

