# IA Local - Teoría

La IA generativa local permite hacer uso de modelos de de IA Generativa (texto, imágenes, video) sin enviar información a empresas externas, todo se queda en la máquina local, su uso es completamente gratuito si se tiene el Hardware. Estas iniciativas buscan democratizar el uso de IA, aquí no hay censura, ni restricciones, ni límites éticos/legales impuestos por terceros.

## Hardware

No todos los modelos de IA pueden correr localmente, ya que requieren una alta cantidad de recursos, pero hay versiones optimizadas/recortadas (eg. compresión) para uso en computadoras personales. Para que estos modelos se ejecuten a buena velocidad se requiere de GPUs Nvidia RTX o AMD (aquí no sé que modelos). Los modelos se ejecutan en la GPU, así que mientras más memoria gráfica tenga la GPU, modelos más grandes (o menos recortadas) se podrán ejecutar en local. 

Con 4GB de VRAM de VRAM se puden ejecutar modelos muy principiantes, con 8 GB se pueden ejecutar modelos intermedios y con 12GB-16GB se pueden ejecutar modelos superiores.

## Proyectos Open-Source de IA

En 2022 inició el projecto Stable Diffusion, que popularizo la IA local permitiendo generar arte en PCs caseras. Después surgió también el proyecto Hugging Face que impulsó aún más esta democratización.

En segmentos como generación de imágenes, en 2026, iniciativas locales como Stable Diffusion compiten con DALL-E para imágenes, y Llama compite con GPT en texto.

En la IA Open Source los modelos se comparten libremente con todo y acceso al código fuente, además que fomenta la innovación comunitaria ya que los modelos base son mejorados en modelos derivados por la comunidad.

## Nivel conceptual interno

Componentes básicos internos:

* **Modelo**: El núcleo. Es un archivo con los pesos del modelo entrenado. Ej: Un modelo de texto como GPT tiene billones de parámetros (conexiones neuronales).
* **Framework**: Librerías de software para cargar y correr el modelo. Ej: PyTorch (de Meta) o TensorFlow (de Google).
* **Backend/Interfaz**: Herramientas que simplifican el uso. Ej: Ollama (para texto) o Automatic1111 (para imágenes). Actúan como "puentes" entre el modelo y el usuario.
* **Hardware**: CPU para control, GPU para cálculos paralelos. RAM almacena datos temporales; VRAM (en GPU) carga el modelo.
* **Datos de entrada/salida**: Convierte un prompt e.g. "genera una imagen de un gato" en una salida.

Flujo conceptual de funcionamiento:

* **Carga**: Descargas el modelo (e.g., 5-50GB) y se carga en memoria (VRAM prioritaria para velocidad).
* **Inferencia**: El modelo usa algoritmos como **transformers** (para texto) o **diffusion** (para imágenes). 
* **Optimizaciones**: Cuantización (reducir precisión de números para ahorrar memoria, e.g., de 32-bit a 8-bit) o LoRA (adaptadores que modifican modelos sin reentrenar todo).
* **Ejecución**: En el PC, el backend envía datos a GPU, computa y devuelve resultado. Todo local: no sale de la máquina.

## Advertencias

* No sobrecargar la GPU sin monitorear temperaturas; usa herramientas como MSI Afterburner para vigilar.

## IA para texto

### Modelos base

* Llama 3 (modelo base de Meta)

### Backends

* Ollama (backend GUI/simple)

**Importante**: aprende `prompt engeenering` para mejorar los resultados ya que prompts vagos generan resultados malos.

## IA para imágenes

### Modelos base

### Backends

* ComfyUI

## Portales/repositorios de modelos IA

* **Hugging Face**: El "GitHub de IA". Miles de modelos open-source, búsqueda por tipo (texto, imagen). Seguro, comunitario.
* **Civitai**: Especializado en imágenes (Stable Diffusion derivados). Comunidad creativa, ratings.
* **Ollama Library**: Para modelos de texto listos para Ollama.
* **LM Studio**: modelos tipo GPT, Qwen, Gemma, DeepSeek, etc.
* Modelos cuantizados: TheBloke en Hugging Face (especialista en optimizaciones).


Tener en Cuenta al Descargar (Técnico)

* **Tamaño y Memoria**: Modelos de 7B parámetros ~4-8GB; chequea tu VRAM (RTX 4060 Ti tiene 8/16GB; apunta a <70% uso para estabilidad).
* **Formato**: .gguf/.safetensors (seguros, sin virus); evita .exe dudosos.
* **Compatibilidad**: CUDA para Nvidia; verifica versión (tu CPU/GPU soporta bien).
* **Licencia**: Open-source (MIT/Apache) vs restrictivas; chequea uso comercial.
* **Seguridad**: Descarga de fuentes oficiales; escanea con antivirus. Error común: Descargar de torrents riesgosos → malware.
* **Versión Hardware**: Para Nvidia Instala CUDA 11.8+; usa Python 3.10+.
* **Pruebas**: Descarga pequeño primero (e.g., 1B params) para testear.

# Notas