# Certificate Signing Request (CSR)

Para poder comprar un Certificado de Firma de Código (Code Signing Certificate), tenemos hacelo mediante una autoridad (Certificate Authority) como [Digicert](https://www.digicert.com/) o [codesigningstore](https://codesigningstore.com/code-signing-certificates).

Hay dos tipos de firmas:
* Organization validation (OV): es un certificado emitido por una autiridad de certificación luego de una revisión para verificar la existencia de una organización y la propiedad del nombre de dominio.
* Extended validation (EV): es un certificado emitido luego de un proceso más riguroso de validación por parte de una autoridad certificadora.

Para poder comprar un certificado de firma de código necesitamos generar un Certificate Signing Request (CSR) que se puede generar con herramientas como OpenSSL. Un CSR simplemente es un formulario con los datos de la compañia y que está firmado con una llave criptográfica. La llave criptográfica se puede generar con OpenSSL o con SSH, aunque aquí haremos todo con OpenSSL.

### Generación del CSR

Los pasos para generar el CSR son los siguientes:
* Generamos una llave criptográfica asimétrica, requerimos la llave privada no la pública. Para generarla opdemos usar el siguiente comando, para esto las longitudes más usadas son 2048 bits y 4096 bits, que va especificado al final ``openssl genrsa -out <nombre_archivo_llave> <bits>``. Por ejemplo:
    * ``openssl genrsa -out mi_llave.key 2048``
* Generamos el CSR usando la llave generada con ``openssl req -new -key <nombre_archivo_llave> -out <nombre_archivo_csr>``. Por ejemplo:
    * ``openssl req -new -key mi_llave.key -out mi_csr.csr``
    * Al ejecutar el comando nos harán una preguntas para llenar el formulario antes de firmarlo, como los datos de la compañia, una contraseña, etc.
* Aunque también podemos realizar los dos pasos a la vez con un solo comando ``openssl req -new -nodes -keyout <nombre_archivo_llave> -out <nombre_archivo_csr> -newkey rsa:<bits>``. Por ejemplo:
    * ``openssl req -new -nodes -keyout mi_llave.key -out mi_csr.csr -newkey rsa:2048``

Los datos solicitados para el formulario son:
* **Country Name (2 letter code):** Código de país de dos letras.
* **State or Province Name (full name):** Estado o ciudad.
* **Locality Name (eg, city): Tipo de localidad City, Town, etc.
* **Organization Name (eg, company):** Nombre de la compañia.
* **Organizational Unit Name (eg, section):** Nombre de la Unidad de la compañía, lo recomendable es usar el sitio web.
* **Common Name (eg, YOUR name):** El nombre común al que se quiere aplicar el certificado (ej. www.domain.tld, secure.domain.tld, *.domain.tld). Esta parte es muy importante definirla bien.
* **Email Address:** La dirección de correo.

### Verificación del CSR

Podemos verificar que el CSR se generó de manera adecuada con el siguiente comando ``openssl req -text -in <nombre_archivo_csr> -noout -verify``. Ahí podremos ver los datos ingresados en la linea de `Subject`, aunque los nombres de los campos están abreviados, por ejemplo Country name es C, Common Name es CN, etc.

### Generar un Certificado Auto Firmado (Self Signed Certificate):

Se llama Self Signed Certificate porque fue un certificado emitido por uno mismo.
Podemos generar nuestro propio certificado `.crt`, que funcione como raiz autorizadora (Certificate Authority), mediante el comando ``openssl x509 -in <nombre_archivo_csr> -req -signkey <nombre_archivo_llave> -days <dias_expiracion> -out <nombre_archivo_crt>``, donde debemos indicar el archivo CSR, la llave y además un tiempo de expiración del certificado en días. Por ejemplo:
* ``openssl x509 -in mi_csr.csr -req -signkey mi_llave.key -days 360 -out mi_certificado.crt``

Los certificados tienen la cadena de confianza desde el mismo certificado hasta la raíz autorizadora, para que así las aplicaciones que hacen uso de certificados como los navegadores o el sistema operativo puedan validar si el certificado es confiable, es decir, si fue emitido por entidades autorizadas o no. En el caso de un SSC, pues el es el único en la cadena, y por lo tanto las aplicaciones no lo tomarán como válido, ya que dirán que la razín no es una entidad válida, tampoco tomarán como válidos a los certificados que firmemos con este.

Pero si nosotros queremos darnos de alta como Autoridad Certificadora en nuestra red local, pues podemos importar el SSC en la aplicación donde queramos, como puede ser el navegador Chrome, que tiene una opción para importar certificados para que así confíe en todos los certificaso emitidos o firmados por nuestro certificado. Obviamente solo nuestro navegador confiará en estos certificados. También puede instalarse en el sistema operativo.

