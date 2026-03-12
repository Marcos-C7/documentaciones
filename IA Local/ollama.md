# Ollama

Ollama es una herramienta de IA local especializada en modelos de texto y visión, es decir que los modelos soportan mayormente solo texto como entrada, algunos pocos soportan texto e imágenes como input, pero solo devuelven texto como salida.

Ollama está inspirado completamente en la estrategia de Docker, así que muchos comandos son y se sienten muy similares. En Ollama en lugar de administrar contenedores, administras modelos de IA. Puedes tener corriendo más de un modelo a la vez.

Permite correr modelos directamente:
```bash
ollama rm phi3:mini
```

También crear modelos personalizados con `Modelfile`, aquí se empieza a ver la similtud con Docker. En este archivo podemos personalizar completamente el modelo mediante `PARAMETER`s:
```dockerfile
FROM llama3.1:8b
SYSTEM Eres un asistente experto en programación Python, responde en español y con código limpio.
PARAMETER temperature 0.7
PARAMETER num_ctx 8192
```
el cual ejecutamos con:
```
ollama create mi-python-experto -f Modelfile
ollama run mi-python-experto
```

Ollama también expone una API en `http://localhost:11434` para poder usar la librería de OpenAI conectada a Ollama en Python:
```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")  # api_key dummy

response = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[{"role": "user", "content": "Hola, ¿qué es Ollama?"}]
)
print(response.choices[0].message.content)
```


# Comandos CLI

* `ollama pull <modelo>`: descarga un modelo sin correrlo.
* `ollama run <modelo>`: ejecuta un modelo, descargandolo si no está descargado.
* `ollama rm <modelo>`: elimina el modelo del equipo.
* `ollama list`: lista los modelos descargados.
* `ollama ps`: modelos corriendo ahora.
* `ollama stop <modelo>`: detener un modelo.
* `ollama show <modelo> --modelfile`: mostrar el modelfile original del modelo.
* `ollama show <modelo> --info`: muestra información del modelo.
* `ollama show <modelo> --parameters`: muestra los parámetros del modelo.

# Comandos de chat

* `/bye` para salir de un chat en modo CLI.
* `/set system "<system-prompt>"`: definir un system prompt para el modelo.
* `/show info`: muestra información del modelo.
* `/show parameters`: muestra los parámetros del modelo.
* `/set nothink`: deshabilitar el razonamiento previo a la respuesta.

# Razonamiento

Muchos modelos incorporan una característica de razonamiento (thinking) para dirigir al modelo a realizar razonamiento paso a paso (Chain of Thought) antes de dar la respuesta definitiva. El razonamiento es el primer bloque que se genera en la respuesta y viene envuelto en las etiquetas `<think>...</think>` o similar. 

Esto solo busca enriquecer el contexto para generar respuestas más lógicas y estructuradas, aunque es mayormente útil en tareas complejas como programación, matemáticas, lógica, problemas multi-paso.

El problema es que esto alenta mucho al modelo (más tokens generados) y consume gran parte del contexto (VRAM).

Por fortuna podemos desactivarlo si no vamos a resolver problemas complejos:
* Para la sesión actual de chat:
    * Con el comando de chat `/set nothink`.
* Para el prompt actual:
    * Agregar `/no_think` al final del prompt.
    * Usar `/think` en lugar para activarlo por turnos.
* Para el system prompt:
    * Agregar `/no_think` al final del system prompt.

# WebUI

Ollama es CLI por defecto, pero hay herramientas que integran un interfaz de usuario (UI). Auí veremos como usar WebUI que es la más popular.

Para esto usaremos docker, levantando la imágen de WebUI en un contenedor:

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

# Parámetros de los modeos de texto

Para entender los parámetros, tenemos que tener en cuenta que el objetivo de los modelos de texto es calcular una distribución de probabilidad sobre el vocabulario completo de tokens (~128k-200k) para así elegir cual será el siguiente token en la respuesta. 

Para esto, al final del modelo se obtienen unos pesos llamados *logits* ($z_i \in \mathbb{R}$) para todos los tokens posibles y estos se convierten en una distribución de probabilidad mediante la siguiente expresión:

$$p_i = \frac{e^{z_i / T}}{\sum_j (e^{z_j / T})}$$

Es decir, el logit de cada token se mapea mediante $z_i \rightarrow e^{z_i / T} > 0$ y se calcula su proporción respecto a todos los demás tokens.

El valor $T$ es un parámetro llamado *temperatura*, el cual veremos más adelante, pero podemos observar que converme $T$ se hace más grande, todos los mapeos convergen a $1$ y por lo tanto la distribución subyacente converge a una uniforme. Es decir, si usamos una temperatura muy grande, entonces todos los tokens resultan con la misma probabilidad de ser elegidos como el siguiente token, lo cual deja completamente sin sentido al modelo.

### Parámetros

Con esto en mente ya podemos revisar los parámetros más importantes que se pueden configurar desde afuera para cada modelo en Ollama, ya sea mediante comandos de chat o mediante un Modelfile (recomendado). La lista completa está en [https://docs.ollama.com/modelfile#parameter]().

* `num_predict`: número máximo de tokens en cada respuesta.
    * Valor por defecto en ollama: `-1`, sin límite.
    * Recomendado: `-1`.
* `num_ctx` (tamaño del contexto): al momento de generar cada token se toma en cuenta un contexto limitado de tokens anteriores, que incluye al system prompt, los prompts anteriores del usuario, las respuestas anteriores del modelo el prompt actual del modelo y la respuesta actual generada hasta el momento. Como el contexto implica almacenar en cache VRAM muchas matrices, el objetivo es usar un tamaño que permita convivir al modelo y al cache en la VRAM.
    * Valor por defecto en ollama: `2048`.
    * Recomendado: haer pruebas mientras se monitorea el consumo de VRAM con el comando `nvidia-smi` para ver cual cantidad queda mejor.
* `temperature`: el valor de `T` en la expresión anterior que calcula las probabilidades.
    * Valor por defecto en ollama: `0.8`.
    * Recomendado: `T=0` → simpre elegir el token más probable (determinista, coherente, aburrido), `T=0.5–0.7` → equilibrado (ideal para coding, explicaciones técnicas), `T=0.8–1.0` → creativo (bueno para escritura creativa), `T>1.2` → caótico, alucinaciones.
* `top_k`: se queda con los `k` tokens tokens más probables y renormaliza. 
    * Valor por defecto en ollama: `40`.
    * Recomendado: `top_k=40` → buena mezcla, `top_k=10–20` → respuestas más conservadoras y coherentes, `top_k=100+` → más creativo pero puede decir tonterías.
* `top_p`: se queda con los tokens más probables que justo acumulen probabilidad `>p` en total. Mejor que `top_k` ya que adapta dinámicamente el número de tokens que pueden ser seleccionados (si hay mucha certeza usa pocos, si no usa más).
    * Valor por defecto en ollama: `0.9`.
    * Recomendado: `0.9-0.95` para uso general, usar este en lugar de `top_k`.
* `min_p`: descartar cualquier token que tenga probabilidad < `(probabilidad del token más probable × min_p)`, ejemplo, si la probabilidad máxima es `0.8` y `min_p=0.1`, entonces descartar todos los tokens con probabilidad < `0.8 * 0.1 = 0.08`.
    * Valor por defecto en ollama: `0.0` (desactivado).
    * Recomendado: `0.05–0.1`, muy recomendable usarlo en modelos modernos.
* `repeat_last_n`: define cuantos tokens hacia atrás en la respuesta actual se buscan repeticiones, va en conjunto con `repeat_penalty`.
    * Valor por defecto en ollama: `64`.
    * Recomendado: `64-128`, pero tener en cuenta que `0` (deshabilitado), `-1` (todo el contexto).
* `repeat_penalty` (evitar loops infinitos): penaliza tokens que ya aparecieron en la respuesta. El penalty se aplica en el logit dividiéndolo por el valor especificado en este parámetro. Notemos que un valor `<1` implica incentivar las repeticiones.
    * Valor por defecto en ollama: `1.1`.
    * Recomendado: `1.15` para conversaciones más variadas, `1` para poesía o código.
* `seed`: define la semilla inicial para las decisiones aleatorias, si especificamos una semilla implica que el modelo generará lo mismo para prompts idénticos en el mismo orden.
    * Valor por defecto en ollama: `0`, no se cambia.
    * Recomendado: dejarlo sin cambiar a menos que queramos replicar una conversación.
* `stop`: secuencias de texto generado que indican al modelo que deje de generar más y retorne. Se pueden indicar varias secuencias mediante un Modelfile especificando varias veces el parámetro `stop`.

Hay otros parámetros avanzados y bastante buenos que me mencionó la IA de Grok, pero que en la documentación oficial no aparecen, como:

* `mirostat`, `mirostat_tau` (sampling adaptativo modo "pro"): ajusta dinámicamente la temperatura para mantener una entropía objetivo configurable en el parámetro `mirostat_tau` (preferible sobre `temperature` + `top_p`). Respuestas mucho más coherentes y naturales en textos largos. Ideal para escritura creativa o razonamiento largo. 
    * Valores: `0` desactivado, `1` versión 1 de mirostat, `2` versión 2 de mirostat.
    * Valor por defecto en ollama: `0`, desactivado.
    * Recomendado: mirostat `2`, mirostat_tau `5.0-6.0`.

# Procedimientos

## Ajustar el system prompt

Se puede hacer desde un comando de chat:
```bash
/set system "<system-prompt>"
```

O en el Modelfile, por ejemplo:

```docker
FROM llama3.1:8b
SYSTEM Eres un profesor de programación Python estricto pero paciente. Corriges errores con explicación clara y das ejemplos reales. Responde solo en español.
PARAMETER temperature 0.7
PARAMETER num_ctx 8192
```

Tambien se puede hacer en interfaces UI como WebUI mediante los ajustes del chat.
