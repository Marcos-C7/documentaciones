# Custom Resource Definitions (CRD)

Los Custom Resource Definitions (CRD) nacieron para permitir la implementación de recursos propios, similar a lo que es un deployment, Service, Ingress, etc. De esta forma podemos ampliar la API de Kubernetes para simplificar la implementación de nuestra arquitectura.

Un ejemplo son los CRDs que implementa el Traefik Ingress-Controller, que después de instalar sus CRDs, tenemos acceso a distintos recuros personalizados como el `IngressRoute`, que es similar al recurso estándar `Ingress`, pero que permite configurar el ruteo de Traefik de forma más clara, legible y mantenible en comparación con usar el `Ingress` estándar.

Mediante este mecanismo, cada herramienta puede implementar sus propios recursos para facilitar su integración en la arquitectura.

Una desventaja de esto, es que perdemos un poco de generalidad porque estamos saliendo de la API estándar de Kubernetes, así que los CRDs solamente funcionan con la herramienta que los implementó, pero de todas formas si una herramienta tiene configuraciones únicas, pues incluso usando recursos estándar no podremos reutilizarlos para otro componente.

### Ejemplos

Como ejemplo tomaremos al Traefik Ingress-Controller, cuyos CRDs se instalan con el siguiente comando:
```bash
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v3.6/docs/content/reference/dynamic-configuration/kubernetes-crd.yml
```

⚠️ Asegurarnos de especificar la misma versión en la URL de los CRDs que la que usamos para instalar el Ingress Controller de Traefik.

Los CRDs incluyen un recurso llamado `IngressRoute`, que está creado para usarse en lugar del recurso `Ingress` estándar. 

En el siguiente ejemplo que usa un Ingress estándar para configurar el ruteo de Traefik Ingress-Controller, bajo el dominio `dev.localhost`, redirige el tráfico hacia el Service `nginx` si la ruta comienza con `/static/` o `/media/` y a un Service ``django` todo lo demás:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: django-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
    - host: dev.localhost
      http:
        paths:
          # /static → nginx
          - path: /static/
            pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 80
          # /media → nginx
          - path: /media/
            pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 80
          # / → django (catch-all)
          - path: /
            pathType: Prefix
            backend:
              service:
                name: django
                port:
                  number: 8000
```

La misma configuración usando el recurso `IngressRoute` de los CRDs de Traefik quedaría como sigue, que es mucho más compacta y legible, además soporta configuraciones avanzadas que Traefik no soporta con el Ingress estándar:

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: django-ingressroute
  namespace: default
spec:
  entryPoints:
    - web
  routes:
    # static OR media → nginx
    - match: Host(`dev.localhost`) && (PathPrefix(`/static/`) || PathPrefix(`/media/`))
      kind: Rule
      services:
        - name: nginx
          port: 80
    # todo lo demás → django
    - match: Host(`dev.localhost`) && PathPrefix(`/`)
      kind: Rule
      services:
        - name: django
          port: 8000
```

