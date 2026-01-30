# Traefik

Traefik es un reverse-proxy, similar a Nginx pero con funcionalidades más avanzadas y además más fácil de configurar. Tiene soporte para reglas avanzadas de enrutamiento, middlewares, etc. integra un dashboard de monitoreo, entre otras cosas.

Su característica más resaltante es que tiene un mecanismo de autodescubrimiento que consulta la API del proveedor (docker, kubernetes, etc) para realizar las conexiones automáticamente. 

La parte que puede resultar más complicada es precisamente la conexión con el proveedor, además la configuración funciona diferente dependiendo del proveedor ya que en Docker se configura mediante etiquetas en los servicios del archivo compose y en Kubernetes se configura mediante anotaciones en el Ingress. 

En Docker se usa la versión normal y se le llama Traefik Reverse-Proxy, pero en Kubernetes se usa una versión adaptada para el paradigma de Kuberntes y se llama Traefik Ingress-Controller.


