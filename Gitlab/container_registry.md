# Gitlab Container Registry

Gitlab, como muchas otras plataformas de versionado (Github, etc), manejan un mecanismo propio de almacenamiento de contenedores tipo Docker.

Con este mecanismo podemos subir las imágenes de nuestras aplicaciones para usarlas en nuestros servidores para el despliegue. 

Hay versión gratuita y de pago, pero en Gitlab la gratuita es bastante generosa, de todas formas hay que estar cuidando el espacio para solo mantener las versiones más recionetes y evitar saturar nuestro espacio para imágenes.

## Acceder al Container Registry

Cada proyecto tiene su propio Container Registry, para abrirlo solo tenemos que ir al proyecto y en el panel izquiero ir a `Deploy` -> `Container registry`.

Ahí están las instrucciones básicas para el manejo, las cuales explicaremos a detalle más adelante.

## Manejo del Container Registry

Para poder hacer uso de este, lo primero es iniciar sesión desde Docker hacia el Container Registry del proyecto en Gitlab. 

Podemos iniciar sesión de dos formas, usando la contraseña de la cuenta de Gitlab o usando un token de acceso personal en lugar de la contraseña. El token de acceso personal se tiene que usar si tenemos habilitada la autenticación de dos pasos, si no podemos usar la contraseña regular.

Para iniciar sesión solo ejecutamos el siguiente comando en una terminal:

```bash
docker login registry.gitlab.com
```

Nos pedirá los datos de inicio de sesión donde podemos dar la contraseña regular o el token de acceso personal, lo que aplique en nuestro caso.

## Subir imágenes al registro

Para poder subir una imágen al registro de nuestro proyecto en Gitlab, es muy importante darle un prefijo específico al nombre de la imagen. Dicho pregijo se puede ver en la página del Container Registry del proyecto. La estructura de este prefijo es:

```bash
registry.gitlab.com/<namespace>/<proyecto>
# Ejemplo
registry.gitlab.com/dev-spingere/trueffort
```

Ya nosotros complementamos con el nombre de la imagen o lo que sea necesario:

```bash
registry.gitlab.com/<namespace>/<proyecto>/<nombre-imagen>:<tag>
# Ejemplo
registry.gitlab.com/dev-spingere/trueffort/frontend:3.1.0
```

Ya habiendo construido la imagen (`docker build`) usando el prefijo correcto en el nombre, con el siguiente comando la subimos:

```bash
docker push <nombre-imagen>
# Ejemplo
docker push registry.gitlab.com/dev-spingere/trueffort/frontend:3.1.0
```

El hecho de tener el prefijo correcto en el nombre de la imágen, guiará a docker al lugar correcto para subirla.





