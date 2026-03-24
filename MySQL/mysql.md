Web: https://www.mysql.com/
----------------------------
Podemos instalar solo el servidor de MySql, para administrar las bases de datos y el servidor a través de comandos.

- Para esto necesitamos ir a las "Descargas" -> "MySQL Community (GPL) Downloads" -> "MySQL Community Server" y bajar el "Windows (x86, 64-bit), ZIP Archive".
- Esto nos descargará un ZIP con la versión portable que contiene los ejecutables (carpeta "bin") necesarios para manejar el servidor y todo lo relacionado a las bases de datos.
- Para poder utilizar estas herramientas necesitamos instalar "Microsoft Visual C++ Redistributable" desde: 
  "https://support.microsoft.com/es-mx/help/2977003/the-latest-supported-visual-c-downloads"
  Yo descargué la versión de 64-bits para mi sistema operativo de 64-bits y con eso fue suficiente.
- La carpeta extraida la podemos poner donde queramos, pero yo la puse en "C:\Users\Marcos C7\AppData\Local\Programs\MySQL\[carpeta extraida]". A este directorio se le llamara el "Directorio Raiz".
- Lo más conveniente es agregar la carpeta "bin" al PATH del sistema, para facilitar el uso de las distintas herramientas desde la consola.
- De ahora en adelante asumiremos que la carpeta "bin" está en el PATH del sistema.
- Si la carpeta "data" no existe en le Directorio Raiz, se puede generar con el comando:
  mysqld --initialize --console
  Esto nos generará una contraseña temporal que tendremos que resetear apenas iniciemos sesión con 'root' y dicha contraseña temporal.
- Para ejecutar el servidor solo necesitamos ejecutar:
  mysqld --console
- Todos los comandos relacionados con la administración de las bases de datos tienen que ejecutarse mientras el servidor está corriendo, de otra forma habra errores.
- Para iniciar sesión con un usuario en MySQL, abrimos una terminal nuevo y usamos el siguiente comando:
  mysql -u root -p
  Nos pedira la contraseña, ingresamos la temporal y una vez dentro la cambiaremos. 
- Con el siguiente comando cambiamos la contraseña:
  ALTER USER 'root'@'localhost' IDENTIFIED BY 'nueva_contraseña';
-------------------------------------------------------------------------------
Siempre que necesitemos poner a correr el servidor de la base de datos tenemos que ejecutar el siguiente comando:

Si queremos conectarnos a la base de datos tenemos que abrir otra terminal y el comando para entrar es el siguiente:
> mysql -u [usuario] -p
Aunque podemos especificar un Host diferente al local con el siguiente comando:
  mysql -h [hostname] -u [usuario] -p

Una vez iniciada sesión podemos ejecutar comandos, a continuación están algunos de ellos:
(los comando pueden ser en cualquiera de minusculas o mayusculas)

- Podemos ver todas las bases de datos a las que tiene acceso el usuario logeado, con el comando:
  SHOW DATABASES;
- Podemos crear una base de datos con el comando:
  CREATE DATABASE nombre;
- Podemos crear un usuario nuevo con el comando:
  CREATE USER '[usuario]'@'%' IDENTIFIED WITH mysql_native_password BY '[contraseña]';
- Podemos darle todos los privilegios a un usuario sobre una base de datos con el comando:
  GRANT ALL ON [base_datos].* TO '[usuario]'@'%';
- Para que los privilegios se apliquen tenemos que ejecutar el comando:
  FLUSH PRIVILEGES;
- Podemos seleccionar una base de datos con el comando:
  USE [base_datos];
- Podemos ver todas las bases de datos de la base de datos seleccionada con el comando:
  SHOW TABLES;
- Podemos obtener una descripcion de los campos de una tabla con el comando:
  DESCRIBE [tabla];
- Podemos mostrar toda una tabla con el comando:
  SELECT * FROM [tabla];
- Podemos borrar una base de datos con el comando:
  DROP DATABASE [base_datos];
- Podemos borrar una tabla de la base de datos seleccionada, con el comando:
  DROP TABLE [tabla];
- Podemos borrar un usuario con el comando:
  DROP USER <usuario>;
- Para borrar un usuario asociado a un host específico:
  DROP USER '<usuario>'@'<HOST>';

-------------------------------------------------------------------------------
https://www.digitalocean.com/community/tutorials/how-to-create-a-django-app-and-connect-it-to-a-database
Para conectar a Django una base de datos 'nombre_base_datos' sobre la que un usuario 'nombre_usuario', con contraseña 'contraseña', tiene privilegios: 

- Instalamos el conector de MySQL para Python a traves de pip, con el comando:
  pip install mysqlclient
- Definimos el ajuste "DATABASES" con los siguientes valores:
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'nombre_base_datos',
          'USER': 'nombre_usuario',
          'PASSWORD': 'contraseña',
          'HOST': 'localhost',
          'PORT': '3306',
      }
  }





