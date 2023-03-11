# Docker en WSL 2 + Ubuntu

La idea es documentar los pasos para instalar y usar docker en WSL 2 con Ubuntu.

Se asume que ya está instalado WSL 2 en el sistema y que también está instalada una distribuición de Ubuntu Linux en WSL 2. En nuestro caso estamos en `Ubuntu 22.04.2 LTS`.

## Habilitar Systemd en la instancia

Hay una herramienta nativa en Linux que se llama `Systemd`, que se describe como:
```
Systemd is a suite of basic building blocks for a Linux system. It provides a system and service manager that runs as PID 1 and starts the rest of the system.
```

Al parecer esa herramienta permite la ejecución de servicios como: 
* Docker: open platform for developing, shipping, and running applications.
* Snap: A handy binary that allows you to install and manage software inside Ubuntu. Try running: snap install spotify or snap install postman, etc.
* microk8s: Get Kubernetes running locally on your system quickly.
* systemctl: A tool that’s part of systemd, interact with services on your Linux machine Try systemctl list-units --type=service to see which services are available and their status.

Como es una característica recién implementada (Septiembre 2022), se tiene que activar manualmente simplemente creando/editando el archivo `/etc/wsl.conf` y agregando:
```bash
[boot]
systemd=true
```
Como es un archivo de sistema, esto se tiene que hacer con privilegios de `sudo`. 

Reiniciamos la instancia para que surta efecto. Fuera de la instancia en una terminal de Windows ejecutamos:
```powershell
wsl --terminate <NombreInstancia>
```
Y volvemos a iniciar la instancia.

## Instalar Docker

Opcionalmente podemos eliminar cualquier residuo de instalaciones previas de Docker:
```bash
sudo apt remove docker docker-engine docker.io containerd runc docker-ce-cli containerd.io
```

Actualizamos la lista de paquetes:
```bash
sudo apt update
```

Instalamos prerequisitos:
```bash
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release
```

Para verificar la integridad de las descargas desde los servidores de Docker, agregamos la llave GPG:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Agregamos el repositorio de Docker:
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Actualizamos la lista de paquetes de nuevo, porque acabamos de agregar un repositorio:
```bash
sudo apt update
```

Instalamos Docker;
```bash
sudo apt install docker-ce docker-ce-cli containerd.io
```

Agregamos el usuario al grupo Docker, para que no tener que usar `sudo` en los comandos de Docker:
```bash
sudo usermod -aG docker $USER
```

Revisamos que se haya instalado Docker:
```bash
docker --version
```

## Verificar la instalación de Docker

Salimos de WSL:
```bash
exit
```

Detenemos la instancia donde instalamos Docker:
```powershell
wsl --terminate <NombreInstancia>
```
Podemos ver la lista de instancias instaladas o de instancias corriendo:
```powershell
wsl -l
wsl -l --running
```

Volvemos a iniciar la instancia y dentro ejecutamos lo siguiente para que se ejecute un contenedor de prueba:
```bash
docker run hello-world
```

