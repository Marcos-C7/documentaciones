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

## Optimizaciones

### Cuantización

Los modelos originales se entrenan en alta precisión (FP16~16 bits por parámetro e incluso FP32), cada parámetro es un número flotante. Un modelo de `8B` (8 billones) de parámetros necesita ~16 GB solo para los pesos que corresponden a cada parámetro.

La cuantización reduce la precisión de los parámetros a menor tamaño, dependiendo del propósito del modelo, pueden ser reducidos 8, 5, 4, 3 e incluso 2 bits por parámetro, lo cual reduce el tamaño del modelo sutancialmente para poder ejecutarse con menos memoria.

Con esto tenemos modelos más ligeros, más rápidos, pero con menos calidad como respuestas menos coherentes, más errores en razonamiento complejo, repeticiones, etc. La cuantización es weight-only (solo pesos del modelo; el KV cache suele quedar en FP16 o Q8 para no perder contexto)

En 2026, las cuantizaciones modernas (especialmente en formato GGUF de llama.cpp) son tan buenas que un modelo cuantizado a 4-5 bits puede ser casi indistinguible del original en la mayoría de tareas cotidianas (chat, código simple, escritura). 

El formato GGUF es muy usado en modelos para Llama.cpp, ollama, LM Studio, KoboldCPP y otros. Otros formatos como GPTQ/AWQ/EXL2 existen (más para GPU Nvidia), pero GGUF es el rey para compatibilidad amplia (CPU, AMD, Apple, Nvidia).

Idea superficial de la cuantización: en la cuantización tradicional no se cuantiza peso por peso, sino que se agrupan los pesos en bloques (eg, de 32, 64, 256 pesos), a cada bloque se le calcula una escala y posiblemente un zero-point para mapear los valores flotantes originales a enteros de pocos bits. A veces se aplica otra cuantización sobre la cuantización para un mayor ahorro. Hay cuantizaciones modernas (K-quants) que usan estructuras jerárquicas para una cuantización no uniforme en tamaño de bits y mezclas de pesos inteligente por tensor.

Las notaciones generales de cuantización son  `Q<N>_K_<S/M/L>` o `IQ<N>_<XXS/XS/S/M>` o legacy como `Q4_0`:

* `Q`: modelo cunatizado.
* `N`: número de bits base por peso, aunque no es exacto en métodos no uniformes como `K-quants`.
* `_K`: indica que está usando la cuantización moderna `K-quants`, lo recomendado es usar esta cuantizaicón.
* `S/M/L` (solo para `K-quants`): indica el nivel de mezcla o precisión selectiva:
    * `S` (Small) para un menor tamaño pero más pérdida de calidad.
    * `M` (Medium) balance ideal con mucho mejor calidad por un poco más de tamaño.
    * `L` (Large) menos común con mejor calidad pero mucho mayor peso.
* `IQ` (I-Quants, más nuevos): Mejores en bits muy bajos (IQ3_XXS, IQ4_XS). A veces superan a K-quants equivalentes en calidad por bit, pero más sensibles a cómo se cuantizó.
* `_0` o `_1` (legacy): Métodos antiguos sin `K` (peor calidad al mismo tamaño → evitar si es posible).

Ejemplos:

* `Q8_0`: Bits por peso ~8.5, tamaño total ~7-8 GB, VRAM necesaria ~7-8 GB (legacy, evitar los que no tengan K).
* `Q5_K_S`: Bits por peso ~5.3-5.5, tamaño total ~4.3-4.8 GB, VRAM necesaria ~5 GB
* `Q5_K_M`: Bits por peso ~5.5-5.7, tamaño total ~4.5-5 GB, VRAM necesaria ~5-6 GB.

**Punto importante**: Muchos creadores de cuantizaciones (TheBloke, city96, etc.) usan imatrix (importance matrix) durante la cuantización → mejora drásticamente la calidad en Q3/Q4/Q5 (reduce pérdida en ~20-50%). Si es posible usa versiones con "imatrix" o "imat". Busca "TheBloke" o "bartowski" para buenas cuantizaciones.








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