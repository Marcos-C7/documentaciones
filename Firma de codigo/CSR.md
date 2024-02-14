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
    * ``openssl req -new -nodes -keyout private.key -out server.csr -newkey rsa:2048``

Los datos solicitados para el formulario son:
* **Country Name (2 letter code):** Código de país de dos letras.
* **State or Province Name (full name):** Estado o ciudad.
* **Locality Name (eg, city): Tipo de localidad City, Town, etc.
* **Organization Name (eg, company):** Nombre de la compañia.
* **Organizational Unit Name (eg, section):** Nombre de la Unidad de la compañía, lo recomendable es usar el sitio web.
* **Common Name (eg, YOUR name):** El nombre común al que se quiere aplicar el certificado  (ej. www.domain.tld, secure.domain.tld, shop.domain.tld).
* **Email Address:** La dirección de correo.
