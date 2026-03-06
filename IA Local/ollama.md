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

# Comandos de chat

* `/bye` para salir de un chat en modo CLI.
* `/set system "<system-prompt>"`: definir un system prompt para el modelo.

# WebUI

Ollama es CLI por defecto, pero hay herramientas que integran un interfaz de usuario (UI). Auí veremos como usar WebUI que es la más popular.

Para esto usaremos docker


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