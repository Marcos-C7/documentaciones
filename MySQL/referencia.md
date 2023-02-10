## Comandos

Crear una base de datos:
* CREATE DATABASE `<nombre>`;

Eliminar una base de datos y todo su contenido:
* DROM DATABASE `<nombre>`;

Seleccionar una base de datos sobre la que se ejecutarán los comandos:
* USE `<base_datos>`;

Crear una tabla una vez seleccioanda una base de datos:
* CREATE TABLE `<nombre>` (\
    `<columna1>` `<tipo_de_dato>` `[restricciones]`,\
    `<columna2>` `<tipo_de_dato>` `[restricciones]`,\
    ...\
    PRIMARY KEY (`<columna_a_usar_como_primary_key>`)\
    FOREIGN KEY (`<columna>`) REFERENCES `<otra_tabla>`(`<columna>`)\
  );

Un prymary key es un índice en la tabla. Para crear un primary key que se genere automáticamente a partir de más de una columna:
* CONSTRAINT `<nombre>` PRIMARY KEY (`<columna1>`, `<columna2>`, ...);

También se pueden agregar primary keys usando ALTER TABLE.

Tipos de datos:
* INT
* VARCHAR(`<long_maxima>`)

Restricciones de campos:
* NOT NULL
* AUTO_INCREMENT
* UNIQUE

Agregar columnas a una tabla:
* ALTER TABLE `<tabla>`
  ADD `<columna_nueva>` `<tipo_de_dato>`;

ELiminar una tabla:
* DROP TABLE `<tabla>`;

Insertar registros a una tabla en un subconjunto de columnas:
* INSERT INTO `<tabla>` (`<columna1>`, `<columna2>`, ...)\
  VALUES (`<val_col1>`, `<val_col2>`, ...),\
         (`<val_col1>`, `<val_col2>`, ...)\
         ...;

Obtener datos de una tabla:
* SELECT columna1, columna2, ... FROM `<tabla>`;
* SELECT * FROM `<tabla>`

Obtener solo los valores distintos de una columna:
* SELECT DISTINCT `<columna>` FROM `<tabla>`;
Contar los valores distintos de una columna:
* SELECT COUNT(DISTINCT `<columna>`) FROM `<tabla>`;

Aplicar flitro a una consulta:
* SELECT columna1, ... FROM `<tabla>` WHERE `<condicion>`;
Se pueden combinar varias condiciones con AND, OR y NOT, usar paréntesis para
agrupar las condiciones:
* SELECT columna1, ... FROM `<tabla>` WHERE `<condicion>` AND `<condicion>`;
* SELECT columna1, ... FROM `<tabla>` WHERE `<condicion>` OR `<condicion>`;
* SELECT columna1, ... FROM `<tabla>` WHERE NOT `<condicion>`;

Tipos de condiciones:
* columna = valor
* columna < valor
* columna > valor
* columna <= valor
* columna >= valor
* columna <> valor
* columna BETWEEN valor1 AND valor2
* columna LIKE patron
* columna IN (valor1, valor2, ...)
* Para valores de tipo texto, se requiere poner comillas **'simples'**.
* Para valores numéricos no se requieren comillas.
* El patrón para LIKE aplica solo para textos y puede ser por ejemplo, 
  los textos que empiecen con M 'M%'.

Aplicar un ordenamiento a una consulta, si no se especifica una dirección, por
defecto será ascendente. Se puede indicar una o más columnas para ordenar 
jerarquías:
* `<consulta>` ORDER BY columna1, columna2, ... `[ASC|DESC]`;
Se puede combinar las direcciones de ordenamiento en las jerarquías:
* `<consulta>` ORDER BY columna1 ASC, columna1 DESC;
Por ejemplo:
* SELECT columna1, ... FROM `<tabla>` WHERE NOT `<condicion>` ORDER BY columna1;

