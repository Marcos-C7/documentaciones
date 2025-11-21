# Redis en Django

Documentación técnica sobre la integración, uso y funcionamiento interno de Redis en proyectos Django.

## Arquitectura y Conceptos Fundamentales

Antes de escribir código, es crucial entender cómo opera Redis para evitar problemas comunes en producción.

### 1. Single-Threaded (Un solo hilo)
Redis procesa los comandos **uno por uno** en un solo hilo principal.
*   **Implicación**: Si ejecutas un comando lento (como `KEYS *` en una base de datos grande), **bloqueas** a todos los demás clientes hasta que termine.
*   **Recomendación**: Evita comandos O(N) en claves grandes o colecciones gigantes. Usa `SCAN` en lugar de `KEYS`.

### 2. Bases de Datos Indexadas (0-15)
Por defecto, una instancia de Redis tiene 16 bases de datos lógicas, numeradas del 0 al 15.
*   **Aislamiento**: Las claves de la DB 0 no se ven en la DB 1.
*   **No es seguridad**: No hay control de acceso por DB. Si tienes la contraseña, tienes acceso a todas.
*   **Uso recomendado**: Separar entornos o aplicaciones lógicamente.
    *   Ejemplo: DB 0 para Caché, DB 1 para Celery (Colas), DB 2 para Session Storage.
    *   *¿Por qué?*: Si necesitas limpiar la caché (`FLUSHDB`), no borrarás accidentalmente las tareas pendientes de Celery.

### 3. Persistencia (RDB vs AOF)
Aunque Redis es "in-memory", puede guardar datos en disco.
*   **RDB (Snapshots)**: Guarda una "foto" de la memoria cada X tiempo. Rápido, pero puedes perder los últimos minutos de datos si el servidor falla.
*   **AOF (Append Only File)**: Guarda cada operación de escritura. Más seguro, pero el archivo crece más y es más lento de restaurar.
*   **Recomendación para Caché**: Si solo usas Redis como caché, puedes desactivar la persistencia para ganar rendimiento. Si usas Redis para Celery o datos persistentes, activa AOF o RDB.

### 4. Memoria y Evicción
Redis tiene un límite de memoria (`maxmemory`). Cuando se alcanza:
*   **Caché**: Debe estar configurado con `maxmemory-policy allkeys-lru` para borrar lo más viejo y dejar espacio a lo nuevo.
*   **Storage**: Si lo usas como DB persistente, usa `noeviction` (dará error al intentar escribir si está lleno, pero no borrará datos).

## Motivación

### ¿Por qué Redis?
Redis (Remote Dictionary Server) es un almacén de estructura de datos en memoria. A diferencia de una base de datos tradicional (PostgreSQL, MySQL) que escribe en disco, Redis mantiene todo en RAM, lo que ofrece tiempos de lectura/escritura de microsegundos.

**Ventajas clave:**
*   **Velocidad**: Al evitar el I/O de disco, es ideal para datos efímeros de alto acceso.
*   **Estructuras de Datos**: No solo guarda strings; soporta Listas, Sets, Hashes, Bitmaps, etc. nativamente.
*   **Persistencia Opcional**: Aunque es en memoria, puede configurar snapshots (RDB) o logs de transacciones (AOF) para no perder datos al reiniciar.

### Casos de Uso en Django
1.  **Caché**: Almacenar resultados de vistas pesadas o consultas SQL complejas.
2.  **Session Storage**: Guardar sesiones de usuario para evitar lecturas a la DB en cada request.
3.  **Throttling**: Controlar la tasa de peticiones (Rate Limiting) usando contadores atómicos.
4.  **Colas de Tareas**: Backend estándar para Celery.
5.  **Real-time**: Backend para WebSockets con Django Channels.

### Funcionamiento Interno (Conceptos Clave)
*   **TTL (Time To Live)**: La mayoría de las claves en caché tienen un tiempo de expiración. Redis elimina automáticamente las claves vencidas.
*   **Eviction Policy**: Cuando la memoria se llena, Redis debe decidir qué borrar. En caché, lo común es `allkeys-lru` (Least Recently Used) o `volatile-lru`.
    *   *Tip*: Si Redis se llena y no tiene política de desalojo, devolverá errores de escritura.

---

## Integración de Redis en Django

La librería estándar es `django-redis`, que es un wrapper robusto sobre el cliente oficial `redis-py`.

### Instalación
```bash
pip install django-redis
```

### Configuración (`settings.py`)

Django permite configurar diferentes mecanismos para manejar la caché y el mecanismo recomendado es Redis. A continuación vemos como configurar Django para que use el servidor de Redis como sistema de Caché y también para que maneje las sesiones de usuarios mediante la caché seleccionada (que será Redis).

```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        # Ubiación del servidor 'redis://<host>:6379/1' o servidores 'redis://<host1>:6379/1;redis://<host2>:6379/1'
        "LOCATION": "redis://127.0.0.1:6379/1", 
        "OPTIONS": {
            # Hay varias pero 'DefaultClient' funciona en el 99% de los casos
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            # Prefijo para evitar choques entre varias aplicaciones Django
            "KEY_PREFIX": "myapp"
            # En caso de que el servidor tenga contraseña y sea la misma en todos los servidores especificados
            "PASSWORD": "password"
            # Segundos para conectar (util en caso se caida del servidor Redis)
            "SOCKET_CONNECT_TIMEOUT": 3,
            # Timeout de operaciones (util en caso se caida del servidor Redis)
            "SOCKET_TIMEOUT": 3,
            # Algoritmo de compresión para datos grandes 
            "COMPRESSOR": "django_redis.compressors.zlib.ZlibCompressor",
            # Tamaño minimo para aplicar compresión
            "COMPRESS_MIN_LEN": 500,
            # Útil definor como 'True' en producción si Redis no es crítico
            "IGNORE_EXCEPTIONS": True,
            # Opciones del pool de conexiones
            "CONNECTION_POOL_KWARGS": {
                # Máximo de conexiones abiertas en el pool
                "max_connections": 100,
                # Reintentar la operación si llega al timeout definido
                "retry_on_timeout": True,
            }
        }
    }
}
```

Notas sobre algunas configuraciones:
* `LOCATION`: ubicación del servidor Redis, la sintaxis general es `redis://[:<password>@]<redis-host>:<port>/<db-number>`, lo recomendado por Django es usar la base de datos `1` para caché.
    * Si el servidor está corriendo en un contenedor o servicio podemos usar el nombre: `redis://redis:6379/1`.
    * Podemos indicar varios servidores Redis separados por punto y coma, por ejemplo: `redis://redis1:6379/1;redis://redis2:6379/1`.
    * En caso de usar varios servidores, si todos usan la misma contraseña podemos especificarla en el parámetro `OPTIONS.PASSWORD`, si son diferentes entonces ponemos la contraseña en cada servidor en `LOCATION`.

#### Parser de datos

⚠️ Django convierte los objetos de Python en strings antes de guardarlos en Redis y para esto usa Parsers. El mejor Parser por su velocidad es `hiredis`, que está implementado en `C` y para habiliarlo solo hay que instalarlo (`pip install hiredis`). El cliente de Django-Redis lo detectará y seleccionará automáticamente.

Para revisar cual parser se está usando podemos ejecutar lo siguiente:
```bash
python manage.py shell
```
```python
import redis
r = redis.Redis()
cls = r.connection_pool.connection_class()._parser
print(cls)
```

#### Sesiones de usuarios con Redis

También podemos configurar Django en los `settings.py` para que maneje las sesiones de usuarios mediante la caché seleccionada, que será Redis en este caso.

```python
# Usar Redis para guardar sesiones (Opcional pero recomendado)
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"
```

---

## Catálogo de ejemplos

### Ejemplos básicos

#### 1. API de Caché de Bajo Nivel
Uso directo de `django.core.cache`. Django serializa automáticamente los objetos de Python usando `pickle` antes de guardarlos en Redis.

```python
from django.core.cache import cache

# Identificadores
user_id = 42
cache_key = f"user_profile_{user_id}"

# SET: Guardar valor con expiración (timeout en segundos)
# Internamente: Redis SETEX user_profile_42 300 "serialized_data..."
cache.set(cache_key, {"name": "Juan", "role": "admin"}, timeout=300)

# GET: Recuperar valor
# Internamente: Redis GET user_profile_42 -> Deserializar pickle
profile = cache.get(cache_key)

if profile is None:
    # Cache miss: Recalcular y guardar
    profile = calculate_profile(user_id)
    cache.set(cache_key, profile, timeout=300)

# GET_OR_SET: Patrón atómico (o casi) para lo anterior
profile = cache.get_or_set(cache_key, lambda: calculate_profile(user_id), timeout=300)

# DELETE
cache.delete(cache_key)
```

#### 2. Caché de Vistas (`@cache_page`)
Cachea la respuesta HTTP completa. Ideal para páginas públicas que cambian poco.

```python
from django.views.decorators.cache import cache_page

# Cachea la vista por 15 minutos (900 segundos)
# La key interna se genera basada en la URL y headers (Vary)
@cache_page(60 * 15)
def home_view(request):
    # ... lógica pesada ...
    return render(request, 'home.html')
```

*   **Advertencia**: Cuidado con cachear vistas que contienen datos específicos del usuario (como "Hola, Juan"). `@cache_page` guardará la versión de Juan y se la mostrará a Pedro.

---

### Ejemplos avanzados

#### 1. Acceso Nativo a Redis (`get_client`)
A veces la API de Django (`set`/`get`) se queda corta porque solo maneja strings/pickles. Para usar estructuras nativas de Redis (Listas, Sets), necesitamos el cliente nativo.

```python
from django.core.cache import cache

def registrar_visita_reciente(producto_id):
    # Obtenemos el cliente nativo (redis-py)
    # bypass_proxy=True evita que django-redis intente envolver el cliente
    con = cache.get_client(default_client=False) 
    
    key = "recent_products"
    
    # Pipeline: Ejecutar múltiples comandos en un solo round-trip de red
    pipe = con.pipeline()
    pipe.lpush(key, producto_id) # Agregar al inicio de la lista
    pipe.ltrim(key, 0, 9)        # Mantener solo los últimos 10
    pipe.execute()

def obtener_visitas_recientes():
    con = cache.get_client(default_client=False)
    # LRANGE devuelve bytes, hay que decodificar si es necesario
    # Redis devuelve: [b'101', b'55', ...]
    ids_bytes = con.lrange("recent_products", 0, -1)
    return [int(id_b) for id_b in ids_bytes]
```

#### 2. Versionado de Claves (Invalidación por Grupo)
Django permite versionar claves. Esto es útil para invalidar un grupo entero de claves simplemente cambiando la versión, sin tener que borrarlas una por una.

```python
# Guardar datos con versión 1
cache.set("product_1", "data", version=1)
cache.set("product_2", "data", version=1)

# Recuperar
cache.get("product_1", version=1) # -> "data"

# "Invalidar" todo el grupo incrementando la versión en el código
# Al pedir versión 2, las claves viejas (v1) son ignoradas (y eventualmente desalojadas)
cache.get("product_1", version=2) # -> None
```

#### 3. Bloqueo Distribuido (Locks) para taréas asíncronas en Celery

Evitar condiciones de carrera cuando múltiples procesos (ej. workers de Celery o múltiples gunicorn workers) intentan modificar el mismo recurso.

```python
from django.core.cache import cache

lock_id = "procesar_pago_orden_123"

# cache.lock usa internamente SETNX (Set if Not Exists) de Redis
# blocking_timeout: cuánto esperar si está ocupado antes de rendirse
with cache.lock(lock_id, timeout=10, blocking_timeout=5):
    # Sección crítica: Solo un proceso entra aquí a la vez
    orden = Orden.objects.get(id=123)
    if orden.estado == 'PENDIENTE':
        orden.procesar_pago()
```

*   **Tip**: Siempre usa `timeout` en el lock para evitar deadlocks si el proceso muere antes de liberar el lock.

#### 4. Operaciones Atómicas con `watch`
Para contadores o transacciones donde necesitas leer, modificar y guardar sin que nadie se meta en medio.

```python
from django.core.cache import cache

def incrementar_contador_seguro(key):
    con = cache.get_client(default_client=False)
    
    with con.pipeline() as pipe:
        while True:
            try:
                # Vigilar la clave. Si cambia antes del execute(), lanza WatchError
                pipe.watch(key)
                
                valor_actual = pipe.get(key) or 0
                nuevo_valor = int(valor_actual) + 1
                
                # Iniciar transacción
                pipe.multi()
                pipe.set(key, nuevo_valor)
                pipe.execute()
                break # Éxito
            except redis.WatchError:
                # Alguien modificó la clave mientras mirábamos, reintentar
                continue
```

### Solución de Problemas (Troubleshooting)

1.  **`ConnectionError: Too many connections`**:
    *   **Causa**: El pool de conexiones se agotó o Redis alcanzó su límite `maxclients`.
    *   **Solución**: Aumentar `max_connections` en `CONNECTION_POOL_KWARGS` o revisar si hay fugas de conexiones (aunque `django-redis` las maneja bien).

2.  **`ResponseError: OOM command not allowed`**:
    *   **Causa**: Redis está lleno y no puede escribir más.
    *   **Solución**: Revisar la política de desalojo (`maxmemory-policy`) en `redis.conf`. Para caché debería ser `allkeys-lru`.

3.  **Errores de Pickle**:
    *   **Causa**: Cambiaste la estructura de una clase Python que estaba cacheada. Al intentar deserializar (`cache.get`), falla.
    *   **Solución**: Cambiar la versión de la caché o borrar la clave manualmente.
    *   *Advertencia*: Nunca uses pickle para datos que vienen de fuentes no confiables (seguridad).

4.  **Datos que no expiran**:
    *   **Causa**: Usaste `timeout=None` (infinito).
    *   **Solución**: Siempre define un timeout razonable a menos que sea estrictamente necesario.

---

## Redis CLI (comandos directos)

A veces es necesario interactuar directamente con Redis para verificar qué se está guardando, borrar claves manualmente o debuguear.

### Iniciar Redis CLI

*   **Local**: `redis-cli`
*   **Docker**: `docker exec -it <nombre-contenedor-redis> redis-cli`
*   **Remoto**: `redis-cli -h <host> -p <port> -a <password>`

### Gestión de Bases de Datos y Servidor

*   `SELECT <index>`: Cambiar de base de datos (ej. `SELECT 1`). El prompt cambiará a `[1]>`.
*   `DBSIZE`: Ver cuántas claves tiene la base de datos actual.
*   `FLUSHDB`: Borra TODAS las claves de la base de datos **actual**.
*   `FLUSHALL`: Borra TODAS las claves de **TODAS** las bases de datos.
*   `INFO keyspace`: Muestra estadísticas de claves por cada base de datos (ej. `db0:keys=10,expires=0...`).

### Catálogo de Comandos por Estructura

#### 1. Básicos y Strings (Cadenas)
Las claves de caché de Django suelen ser strings (serializados).

*   `SET clave valor`: Guardar un string.
*   `GET clave`: Obtener un string.
*   `DEL clave`: Borrar una clave.
*   `EXISTS clave`: Verificar si existe (1=sí, 0=no).
*   `EXPIRE clave segundos`: Asignar TTL (tiempo de vida).
*   `TTL clave`: Ver cuánto tiempo de vida le queda (-1=infinito, -2=no existe).
*   `INCR clave`: Incrementar un contador atómicamente.

#### 2. Listas (Lists)
Usadas para colas de tareas (Celery) o historiales.

*   `LPUSH clave valor`: Agregar al inicio.
*   `RPUSH clave valor`: Agregar al final.
*   `LPOP clave`: Sacar y borrar del inicio.
*   `RPOP clave`: Sacar y borrar del final.
*   `LRANGE clave start stop`: Ver rango (ej. `LRANGE milista 0 -1` para ver todo).
*   `LLEN clave`: Ver longitud de la lista.

#### 3. Sets (Conjuntos)
Colecciones desordenadas sin duplicados.

*   `SADD clave miembro`: Agregar miembro.
*   `SMEMBERS clave`: Ver todos los miembros.
*   `SISMEMBER clave miembro`: Verificar si existe.
*   `SREM clave miembro`: Borrar miembro.

#### 4. Hashes
Mapas de campo-valor dentro de una clave. Django los usa menos para caché simple, pero son útiles para objetos estructurados.

*   `HSET clave campo valor`: Guardar campo.
*   `HGET clave campo`: Obtener campo.
*   `HGETALL clave`: Ver todos los campos y valores.
*   `HDEL clave campo`: Borrar campo.

### Debugging y Mantenimiento

*   `KEYS *`: Listar TODAS las claves. **⚠️ PELIGRO**: No usar en producción si hay muchas claves, bloquea el servidor.
*   `SCAN 0`: Alternativa segura a `KEYS *` para iterar claves poco a poco.
*   `TYPE clave`: Te dice qué tipo de estructura es (string, list, set, etc.).
*   `MONITOR`: Muestra en tiempo real todos los comandos que recibe el servidor. Útil para ver qué está enviando Django.
*   `INFO`: Muestra estadísticas del servidor (memoria usada, conexiones, etc.).
