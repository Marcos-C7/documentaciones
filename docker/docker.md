# Docker


## Comandos

Mostrar la imágenes que han sido creadas:
```bash
docker images
```

Mostrar los contenedores que etán corriendo:
```bash
docker ps
```

Entrar a un contenedor que está corriendo, en modo consola:
```bash
docker exec -it <nombre_o_ID> <consola>
```
* `<nombre_o_ID>`: nombre o ID del contedor, que puede ser consultado con le comando `docker ps`.
* `<consola>`: nombre de la consola a usar, depende del sistema operativo del contenedor, unos usan `bash`, otros `sh`, etc.

Salir de un contedor en modo consola:
```bash
exit
```

Para detener un contenedor:
```bash
docker stop <nombre_o_ID>
```

Para iniciar un contenedor:
```bash
docker start <nombre_o_ID>
```

Para monitorear el consumo de recursos de los contenedores que están corriendo:
```bash
docker stats
```