# Git Submodules

Un submódulo es un repositorio completo dentro de otro. Git no copia el código al repositorio principal, solo un puntero para poder manejarlo. 

El submódulo sigue siendo un repositorio normal en el cual podemos trabajar directamente, haciendo cambios, commits, creando ramas, etc. El repositorio que lo incluye simplemente tiene una copia de su contenido y almacena el estado en el que está el submódulo (en qué rama necesita que esté el submódulo, etc) para poder replicar el funcionamiento de la aplicación conjunta.

Es decir, al hacer un cambio de rama o hacer checkout en un commit/tag específico, el repositorio principal almacenará ese estado y podemos hacer commit de este cambio en el repositorio principal para que cuando alguien más lo clone, los submódulos sean posicionados en las ramas/commits/tags/etc adecuados.

Al agregar el primer submódulo, se creará el archivo `.gitmodules` que contiene la información necesaria para clonar/administrar los repositorios de los submódulos.

La carpeta git de cada submódulo se pone en la ruta `.git/modules/<submodule-name>` del repositorio principal. En el directorio del submódulo en lugar de la carpeta `.git` habitual en su lugar se genera un archivo con el mismo nombre `.git` que es un enlace simbólico que contiene algo como `gitdir: ../.git/modules/py-services`, es decir, indica la ruta donde está la verdadera carpeta `.git` del submódulo.

Esta es la manera recomendada y la que se maneja por defecto a partir de Git 2.12+ (2017) para administrar los repositorios de submódulos. Esto facilita el manejo de submódulos anidados y evita problemas con herramientas que escanean recursivamente en busca de carpetas `.git`, además de mantener más limpio el espacio de trabajo.

Al agregar un submódulo, este apunta a un commit (detached HEAD), no a una rama.

### Añadir un submódulo al repositorio actual

Clonará el repositorio indicado en la ruta destino indicada.

```bash
git submodule add <repo-url> <ruta-destino>
# Ejemplo:
git submodule add git@gitlab.com:dev-spingere/trueffort-py-services.git backend-python
```

Una versión más general es la siguiente:

```bash
# -b <branch-name> pone el submódulo en esa rama
git submodule add --name <submodule-name> -b <branch-name> <repo-url> <ruta-destino>
```

Donde:
* `submodule-name`: es el nombre con el que se hará referencia al submódulo en los comandos. El nombre por defecto es la ruta que se indique al submódulo.
* `branch-name`: es la rama del submódulo en la que quedará posicionado el submódulo después de actualizarlo con comandos como `git submodule update --remote` y similares. Esta configuración se guarda en el archivo `.gitmodules`. Por defecto no se configura ninguna rama. 

### Clonar un repositorio completo, con todo y sus submódulos

```bash
git clone --recurse-submodules <repo-url>
# Ejmplo:
git clone --recurse-submodules git@gitlab.com:dev-spingere/trueffort.git
```

Si ya se realizó la clonación sin los submódulos, estos se pueden integrar con el siguiente comando:

```bash
git submodule update --init --recursive
```

### Ver el estado de los submódulos

```bash
# - = no inicializado, + = modificado, U = conflicto, nada = inicializado y sin cambios
git submodule status
```

### Trabajar sobre un submódulo

Solo hay que entrar en la carpeta del sumbódulo y trabajar sobre el como si estuvieramos en su repositorio original (ya que lo estamos):

```bash
cd <submodule-path>
git checkout -b feature/nueva-cosa
# ... haces cambios, realizas commits, etc
# Subes los cambios (si se requiere)
git push origin feature/nueva-cosa
```

Después de implementar los cambios volvemos al repositorio pricipal y actualizamos su puntero:

```bash
cd ..
# esto registra el nuevo SHA
git add <submodule-name>
git commit -m "Actualizar componente a nueva feature"
git push
```

### Cambiar de rama/commit/tag

Hay que entrar al submódulo para realizar el cambio y luego actualizar el puntero del principal:

```bash
cd <submodule-path>
git checkout develop    # checkout a rama
git checkout v2.3.0     # checkout a tag
git checkout abc1234    # o checkout a commit específico

# Volvemos al directorio principal y actualizamos
cd ../..
git add <submodule-name>
git commit -m "Rollback componente"
```

### Actualizar uno o todos los componentes

Cada submódulo tiene configurada una rama, podemos actualizar la rama (pull) configurada de uno o todos los módulos.

```bash
# Un solo submódulo
git submodule update --remote <submodule-name>
# Todos los submódulos
git submodule update --remote --recursive

# y luego actualizar los apuntadores
git commit -am "Update all submodules to latest"
```

### Cambiar la rama por defecto de un submódulo

```bash
git submodule set-branch -b <branch-name> <submodule-name>
```

### Cambiar URL de un submódulo

```bash
git submodule set-url <submodule-name> <url>
```
