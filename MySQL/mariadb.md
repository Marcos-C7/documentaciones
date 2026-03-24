# MariaDB

MariaDB es un sistema de gestión de bases de datos relacionales (RDBMS) de código abierto, desarrollado como un fork de MySQL por sus creadores originales después de que Oracle Corporation adquiriera MySQL.

Es mucho mejor que MySQL, ya que es más rápido, seguro y tiene más características. Es la opción recomendada para bases de datos relacionales tipo SQL.

## Características principales

* **Código abierto**: Es completamente gratuito y de código abierto, licenciado bajo GPL.
* **Compatibilidad con MySQL**: Es altamente compatible con MySQL, lo que facilita la migración de aplicaciones existentes.
* **Rendimiento**: Ofrece un rendimiento mejorado en comparación con MySQL en varias áreas.
* **Seguridad**: Incluye características de seguridad avanzadas, como cifrado de datos en reposo y en tránsito.
* **Escalabilidad**: Soporta escalabilidad horizontal y vertical para manejar grandes volúmenes de datos.
* **Almacenamiento**: Ofrece múltiples motores de almacenamiento, incluyendo InnoDB, Aria, MyISAM, y más.
* **Comunidad**: Cuenta con una comunidad activa de desarrolladores y usuarios.

## Instalación

La instalación de MariaDB varía según el sistema operativo. A continuación se muestran algunos ejemplos:

### Windows

Se puede descargar el instalador desde [https://mariadb.org/download/](), pero yo prefiero descargar la versión en `zip` desde la misma liga e integrarla manualmente al sistema:

* En la página de la descarga en el campo `Package Type` elegir `ZIP file`.
* Descomprimir la carpeta y ponerla en `C:\Users\<usuario>\AppData\Local\Programs\MariaDB`, para tener la posibilidad de tener varias versiones.
* Incluir la carpeta `bin` de la versión deseada en la variable de entorno `Path` del sistema, la ruta se verá algo como `C:\Users\<usuario>\AppData\Local\Programs\MariaDB\mariadb-<version>-winx64\bin`.
* La primera vez hay que inicializar la base de datos con el siguiente comando, el cual generará los directorios de datos (`data/`), y el usuario root quedará sin contraseña:
    ```bash
    mariadb-install-db
    ```
* En una terminal levantamos el servidor con el comando:
    ```bash
    mariadbd --console
    ```
* En otra terminal entramos a la base de datos con el comando:
    ```bash
    mariadb -u root
    ```
* Cambiamos la contraseña del usuario root (aunque no tenga una) con el comando:
    ```bash
    ALTER USER 'root'@'localhost' IDENTIFIED BY '<nueva-contraseña>';
    ```
* La proxima vez que entremos al servidor con el usuario root tendrá que ser con el siguiente comando e ingresar la contraseña cuando la pida:
    ```bash
    mariadb -u root -p
    ```
* Una vez dentro de la base de datos, podemos ejecutar cualquier comando SQL, por ejemplo:
    ```sql
    SHOW DATABASES;
    SHOW TABLES;
    ```

