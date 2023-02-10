# [SQLAlchemy](https://www.sqlalchemy.org/)

SQLAlchemy es un manejador de bases de datos SQL para Python con un ORM (Obect Relational Mapper) integrado, que le da a los desarrolladores control y flexibilidad total de SQL. 

Contiene una colección completa de patrones de persistencia deseñados para la eficiencia y el alto rendimiento, adaptado a un lenguaje simple y Pythonico. Considera las basaes de datos no solo como una colección de tablas, sino como un motor de algebra relacional.

Es más conocido por su ORM, un componente que provee un patrón de mapeo de datos (Data Mapper Pattern), donde clases de Python pueden ser mapeadas a la base de datos permitiendo que el modelo de objetos y el esquema de la base de datos se desarrollen de manera limpia y sepasada desde el principio.

No esconde los detalles de la relación entre SQL y los objetos, sino que todos los procesos están completamente expuestos en una serie de herramientas transparentes y combinables.

Para [instalar SQLAlchemy](https://pypi.org/project/SQLAlchemy/) en Python usamos pip:
```
> pip install SQLAlchemy
```

## Módulos relacionados con SQLAlchemy

Hay varios módulos relacionados con SQLAlchemy, pero no los exploré a profundidad, solo revisé un poco la documentación y parecían útiles:
* [Aldjemy](https://github.com/aldjemy/aldjemy): Aldjemy integrates SQLAlchemy into an existing Django project, to help you build complex queries that are difficult for the Django ORM.

## Ligas de interés

Algunas ligas de iterés que no pude probar pero parecían útiles, son las siguientes:
* [Merging Django ORM with SQLAlchemy for Easier Data Analysis](https://djangostars.com/blog/merging-django-orm-with-sqlalchemy-for-easier-data-analysis/)
* [Best way to integrate SqlAlchemy into a Django project](https://stackoverflow.com/questions/6606725/best-way-to-integrate-sqlalchemy-into-a-django-project)

## [Configurar una conexión SQLAlchemy en Python](https://docs.sqlalchemy.org/en/14/core/engines.html#database-urls)

Para crear una conexión necesitamos hacer uso de la función `sqlalchemy.create_engine`, la cual recibe una URL con los datos necesarios para llevar a cabo la conexión. La URL se compone de los siguiente forma:

```python
url = f'{dialect}+{driver}://{username}:{password}@{host}:{port}/{database_name}'
```

Donde [`dialect`](https://docs.sqlalchemy.org/en/14/core/engines.html#backend-specific-urls) representa un lenguaje SQL como pueden ser:
* MySQL (`'mysql'`)
* MariaDB (`'mariadb'`)
* PostgreSQL (`'postgresql'`)
* Oracle (`'oracle'`)
* Microsoft SQL Server (`'mssql'`)
* SQLite (`'sqlite'`)
* entre [otros](https://docs.sqlalchemy.org/en/14/dialects/index.html#external-dialects).

El driver se refiere a un paquete que se encarga de manejar la conexión con la base de datos, para el caso de MySQL, lo principales son los siguientes, en orden de rendimiento, siendo `mysqlclient` hasta 10 veces más rápido que los demás según [stackoverflow](https://stackoverflow.com/a/46396881/10902201):
* [mysqlclient](https://github.com/PyMySQL/mysqlclient) (`'mysqldb'`)
    * Se pueden ajustar [parámetros SSL](https://docs.sqlalchemy.org/en/14/dialects/mysql.html#ssl-connections) (también otros pueden)
    * Se puede usar para [conectar con Google Cloud SQL](https://docs.sqlalchemy.org/en/14/dialects/mysql.html#using-mysqldb-with-google-cloud-sql) (también otros pueden)
* [PyMySQL](https://github.com/PyMySQL/PyMySQL) (`'pymysql'`)
* [mysql-connector-python](https://github.com/mysql/mysql-connector-python) (`'mysqlconnector'`), en la documentación se recomiendo no usar este.
* [PyODBC](https://pypi.org/project/pyodbc/) (`'pyodbc'`)
* Entre [otros](https://docs.sqlalchemy.org/en/14/dialects/mysql.html)

## Ejemplos de conexiones

A continuación algunos ejemplos de conexiones con dialecto MySQL y distintos dirvers.

### MySQL con mysqlclient
Tenemos que tener instalado el paquete `mysqlclient` en Python.
```
> pip install mysqlclient
```

```python
import mysqlclient
from sqlalchemy import create_engine

db_url = 'mysql+mysqldb://db_user:password@localhost:3306/my_db'
con = create_engine(db_url)
```

La conexión obtenida se puede usar por ejemplo en `pandas` con el método `pandas.read_sql_query`.

La URL también se puede hacer más legible con argumentos a `str.format`:
```python
db_url = 'mysql+mysqldb://{user}:{password}@{host}:{port}/{db_name}'.format(
    user='db_user',
    password='password',
    host='localhost',
    port='3306',
    db_name='my_db'
)
```

También directamente con variables en un `f-string`:
```python
db_url = f'mysql+mysqldb://{user}:{password}@{host}:{port}/{db_name}'
```

