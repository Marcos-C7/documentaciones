Prompt para generar las bases de una documentación

En este archivo vacío quiero generar una documentación para mí mismo de <TEMA>. Que el contenido tenga amplia explicación, principalmente en la funcionalidad interna, que sección tenga códigos de ejemplo, descripción precisa de variables o identificadores usados en el código, tips, advertencias, solución de problemas. A continuación te doy el índice en forma de objeto JSON que describe la estructura del documento: 
```json
{
    "title": "<titulo del documento>",
    "content": "<describir contenido deseado>",
    "subsections": [
        {
            "title": "<titulo de subsección>",
            "content": "<describir contenido deseado>",
            "subsections": [
                {
                    "title": "<titulo de subsección>",
                    "content": "<describir contenido deseado>"
                },
                {
                    "title": "<titulo de subsección>",
                    "content": "<describir contenido deseado>"
                }
            ]
        }
    ]
}
```