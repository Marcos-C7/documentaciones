# Framework de PyTorch

PyTorch es un framework especializado en deep learning. Este framework se compone de varias librerías:

* `torch`: el núcleo de PyTorch.
* `torchvision`: datasets, modelos pre-entrenados y transformaciones de imágenes.
* `torchaudio`: herramientas para procesamiento de audio.

Las podemos instalar con el siguiente comando:

```bash
pip install torch torchvision torchaudio
```

## Tensores

Un tensor es una matriz generalizada en el número de dimensiones, es decir, que en lugar de tener solo 2 dimensiones `MxN` como las matrices regulares, un tensor puede tener cualquier número de dimensiones desde `1` en adelante, así que en este sentido un vector es un tensor de 1 dimensión, una matriz es un tensor de 2 dimensiones, etc.

Toda la IA se maneja con tensores, y hay hardware como las GPUs RTX de Nvidia que tienen implementación a nivel de hardware para operar sobre tensores (los Tensor Cores).

### Creación de tensores

Con Torch tenemos distintas formas para crear tensores:

* Mediante valores explícitos:

```py
a = torch.tensor(7)                # tensor escalar (0 dimensiones)
b = torch.tensor([1, 2, 3])        # vector (1 dimensión)
c = torch.tensor([[1, 2], [3, 4]]) # matriz (2 dimensiones)
```

* Mediante número de dimensiones:

```py
a = torch.zeros(2, 3)          # matriz 2×3 llena de ceros
b = torch.ones(3, 2)           # matriz 3×2 llena de unos
c = torch.eye(4)               # matriz identidad de 4×4
d = torch.rand(2, 4)           # matriz 2x4 de aleatorios uniformes [0, 1)
e = torch.randn(3, 3)          # matriz de 3x3 de distribución normal (mu= 0, sigma^2=1)
```

### Propiedades de tensores

Podemos consultar las propiedades de un tensor:

```py
print("Forma (shape): ", x.shape)       # o x.size()
print("Número de dimensiones: ", x.dim())
print("Cantidad total de elementos: ", x.numel())
print("Tipo de datos (dtype): ", x.dtype)
print("Está en dispositivo: ", x.device)     # dispositivo donde está el tensor: cpu, cuda, etc
print("Requiere gradiente?: ", x.requires_grad)
```

### Operaciones en tensores

Las operaciones se realizan elemento por elemento:

```py
x = torch.rand(2, 4)
2 * x + 5               # escalamiento/traslación
x ** 2                  # potencia
torch.sqrt(x.float())   # aplicar una función elemento por elemento
x_t = x.t()             # transpuesta
x * x_t                 # multiplicación elemento por elemento
x @ x_t                 # multiplicación matricial
```

### Tipo de dato de tensores

Podemos cambiar o consultar el tipo de dato de tensores:

```py
x = torch.rand(2, 4)
x.dtype         # consulta
x.float()       # float32
x.long()        # int64
```

### Indexación y slicing

```py
x = torch.rand(3, 4)
x[1, 0]     # elemento de fila 1 columna 0
x[0]        # primera fila
x[-1]       # última fila
x[:, 0]     # primera columna
x[:, -1]    # última columna
x[1:, 1:3]   # slice de fila 1 en adelante y columnas 1 a 2
```

### Gradientes

Este es un pilar muy importante, ya que incorpora en el tensor mismo los mecanismos para el cálculo automático de gradientes, lo que permite entrenar redes neuronales facilmente.

Para visualizar esto de forma más sencilla, nos enfocaremos en un ejemplo de un tensor de 0 dimensiones (escalar). 

Creamos un tensor `y` y indicamos que `requires_grad=True`:

```py
y = torch.tensor(4.0, requires_grad=True) # tensor escalar
```

Con `requires_grad=True` le estamos indicando a PyTorch que recuerde las operaciones realizadas al tensor en sucesivas asignaciones a otras variables. Por ejemplo:

```py
z = y ** 2 + 3 * y + 10     #z = y^2 + 3y + 10 = 38
```

El resultado debe ser `z=38`, pero como `y` tiene `requires_grad=True`, entonces `z` recordará cual fue la variable involucrada en la operación (`y`) y cual fue la expresión (`y^2 + 3y + 10`). La variable `z` no hereda `requires_grad=True`, pero sí queda marcada como parte de la relación para calcular gradientes.

Aquí viene la magia, ahora podemos calcular el gradiente de z respecto a sus varaibles, que en este caso como es univariable, el gradiente es $dz/dy = 2 * y + 3$, pero como `y=4`, entonces el gradiente es `2 * 4 + 3 = 11`.

Lo cual podemos confirmar con:

```py
z.backward()    # calcula el gradiente en propagación hacia atrás
y.grad          # muestra el gradiente calculado respecto a la variable `y`: tensor(11.)
```

Notemos que el gradiente de un tensor respecto a una de sus variables se almacena en la variable, no en el tensor, así cada que calculemos un gradiente en *backward propagation* con el método `backward()` se actualizarán lalos gradientes de las variables involucradas.

Ahora vemos un ejemplo de un tensor con 2 variables:

```py
a = torch.tensor(2.0, requires_grad=True)
b = torch.tensor(3.0, requires_grad=True)

c = a * b
d = c ** 2 + 5 * a - b # = a^2 * b^2 + 5a - b

print("a:", d) # 2
print("b:", d) # 3
print("c:", d) # 6
print("d:", d) # 43

# Propagamos el gradiente hacia atrás
d.backward()

# notemos que ∂d/∂a = 2ab^2 + 5, ∂d/∂b = 2ba^2-1
# como a=2 y b=3, entonces ∂d/∂a=41, ∂d/∂b=23
print("\nGradiente de d respecto a a (∂d/∂a):", a.grad)
print("Gradiente de d respecto a b (∂d/∂b):", b.grad)
```

Las variables terminales son `a` y `b`, así que no hay gradiente respecto a la variable intermedia `c`. Aunque sí se calcula como parte del backward propagation, no se almacena, PyTorch solo almacena los gradientes de tensores hoja, no de los intermedios en la gráfica de relaciones, y las hojas son solo los tensores que se crean con el constructor con `requires_grad=True`.

Podemos indicar que se almacene el gradiente de variables intermedias, en nuestro ejemplo con `c.retain_grad()` antes de ejecutar el cálculo. Así la variable `c` tendrá el gradiente `∂d/∂c`. Solo usar para debuguear ya que almacenar un gradiente consume memoria y es malo para modelos muy grandes.

El método `backward` por defecto solo se puede invocar una vez, si invocamos backward nuevamente en alguna de las variables involucradas (en la misma gráfica), obtendremos un error. Si se quiere volver a calcular gradientes en la misma gráfica tenemos que limpiar los gradientes anteriores.

## Optimizadores

Un optimizador es un objeto auxiliar que se encarga de facilitar las actualizaciones de los parámetros de un modelo durante el proceso de entrenamiento.

Solo tenemos que indicarle cuales son los parámetros y el *learning rate*. Durante el entrenamiento solo tenemos que calcular los gradientes de la función de pérdida (*loss function*) respecto a cada uno de los parámetros y el optimizador nos ayuda a actualizar todos los parámetros en función del gradiente para reducir el error.

Recordemos que un `lr` muy pequeño hace muy lento el entrenamiento y uno grande puede causar divergencia, hay que elegirlo con cuidado aunque depende del tipo de optimizador.

Veamos esto con un ejemplo muy sencillo, donde queremos entrenar un modelo que corresponde a una regresión lineal `y = a*x + b`. En este caso contamos con las parejas de entrada y salida $(x_1, y_1), (x_2, y_2),\ldots, (x_n, y_n)$, y necesitamos encontrar los valores de los números `a` y `b` que minimicen en error cuadrático medio: 

$$\frac{1}{n} \sum_i ((a \cdot x_i + b) - y_i)^2$$

recordemos que $(a \cdot x_i + b)$ es la predicción que el modelo tiene para $x_i$ mientras $y_i$ es el valor real para $x_i$.

```py
# 1. Creamos los parámetros 'a' y 'b' inicializados en '0'
# importante que `requires_grad=True` para los gradientes de la pérdida
a = torch.tensor(0.0, requires_grad=True)
b = torch.tensor(0.0, requires_grad=True)

# 2. Datos ficticios de entrada y salida (los x_i, y_i)
# Intencionalmente fueron generados para satisfacer "y_i = 7.3 * x_i + 0.5"
x = torch.tensor([1.0, 2.0, 3.0, 4.0])
y = torch.tensor([7.8, 15.1, 22.4, 29.7])

# 3. Definimos el optimizador de tipo SGD = Stochastic Gradient Descent (el más básico)
# sobre los parámetros 'a' y 'b' con un learning rate de 0.05
optimizer = torch.optim.SGD([a, b], lr=0.05)

# 4. Bucle de entrenamiento
n_epochs = 300   # cuántos pasos de gradient descent realizaremos

for epoch in range(n_epochs):
    # -----------------------
    # Forward pass (evaluación del desempeño actual de 'a' y 'b')
    predicciones = a * x + b
    
    # Calculamos el error cuadrático medio, como estamos usando tensores, `loss`
    # recordará la expresión con la que fue creada y las variables involucradas
    # para poder calcular gradienes
    loss = torch.mean((predicciones - y) ** 2)
    # también se puede escribir: torch.nn.functional.mse_loss(predicciones, y)
    
    # -----------------------
    # Backward pass (actualización de los parámetros 'a' y 'b')
    optimizer.zero_grad()     # ¡Muy importante Limpiar gradientes anteriores!
    loss.backward()           # calculamos los gradientes de loss respecto a 'a' y 'b'
    
    # -----------------------
    # Actualizar cada parámetro 'w' con gradient descent: w = w - lr * grad(loss),
    # Esta actualización la realiza el optimizador sobre todos los parámetros
    # usando los gradientes de `loss` calculados
    optimizer.step()

print("\nResultado final:")
print(f"a ≈ {a.item():.4f}")    # debería ser ~7.3
print(f"b ≈ {b.item():.4f}")    # debería ser ~0.5
```

Los métodos más comunes y que comparten todos los optimzadores son los siguientes:

| Método | Que hace | Cuando se usa |
|--------------|--------------|--------------|
| zero_grad() | Pone todos los gradientes .grad a cero (limpia memoria) |Antes de cada backward() |
| step() | Realiza la actualización real de los parámetros | Después de backward() |
| state_dict() | Guarda el estado interno (momentum, buffers de Adam, etc.) | Para guardar checkpoints |
| load_state_dict() | Carga el estado guardado | Para reanudar entrenamiento |
| add_param_group() | Añade nuevos parámetros (útil para fine-tuning) | Casos avanzados |

Hay muchos optimzadores (son 15 en Marzo-2026), pero los más importantes para iniciar son los siguientes:

|Optimizador|Diferencia clave|Cuándo usarlo|Learning rate típico|
|-----------|----------------|-------------|--------------------|
|SGD|Actualización simple + opcional momentum|Modelos pequeños, cuando quieres control total|0.01 – 0.1|
|Adam|Adaptive (cada parámetro tiene su propio lr) + momentum|Casi todo (visión, NLP, etc.)|0.001 – 0.0001|
|AdamW|Igual que Adam pero weight decay se aplica correctamente|Transformers, LLMs, todo lo moderno|0.001 – 0.0001|
|RMSprop|Similar a Adam pero más simple|Redes recurrentes (RNN/LSTM)|0.001|

### Función de los optimizadores

Ahora, si un optimizador solamente actualiza el valor de los parámetros, qué los diferencía entre sí?

La actualización del vector de parámetros `w` funciona mediante la siguiente fórmula general:

$$ w = w - lr \cdot f(\Delta loss) $$

donde $\Delta loss$ es el vector de gradientes de la función de error (pérdida) respecto a cada uno de los parámetros.

Pues cada optimizador implementa una función $f(\Delta loss)$ diferente para mejorar el entrenamiento, además de que algunos guardan "memoria" del pasado durante el entrenamiento como parte del estado interno. Algunos se funcionan mejor en cierto tipo de modelos que otros.

A continuación una explicación más detallada de las propiedades clave de algunos:

* SGD puro: 
    * No recuerda nada del pasado.  
    * Cada paso es independiente → puede oscilar mucho en valles estrechos o superficies ruidosas.
* SGD con momentum: 
    * Acumula "velocidad" (como una bola que rueda cuesta abajo y gana inercia).  
    * Ayuda a atravesar mesetas pequeñas y acelera en pendientes consistentes.  
    * El factor β (normalmente 0.9) decide cuánto "recuerda" del pasado.
* Adagrad:
    * Idea: parámetros que han cambiado mucho (gradientes grandes acumulados) deben moverse menos ahora.  
    * Problema: la suma acumulada crece indefinidamente → el learning rate efectivo tiende a 0 con el tiempo → entrenamiento se para.
* RMSprop (mejora de Adagrad): 
    * En vez de sumar todo el historial, usa una media móvil exponencial de grad².  
    * Olvida poco a poco el pasado lejano → el learning rate adaptativo no se apaga con el tiempo.
* Adam (el más popular):
    * Combina lo mejor de momentum + RMSprop.  
    * Tiene dos medias móviles:
        * Primera (m) → dirección promedio (como momentum)  
        * Segunda (v) → magnitud promedio (como RMSprop)
    * Corrección de bias (dividir por 1-βᵗ) porque al inicio las medias están sesgadas hacia 0.  
    * Resultado: convergencia rápida y estable en la mayoría de casos.

* AdamW (el recomendado hoy para casi todo en 2026):
    * Exactamente igual que Adam internamente… excepto en weight decay.  
    * En Adam clásico: weight decay se mezcla dentro del término adaptativo → se debilita cuando los gradientes son grandes.  
    * En AdamW: weight decay se aplica directamente al parámetro (como en SGD), independiente del adaptativo.  
    * Esto mejora mucho la generalización en modelos grandes (transformers, ViTs, LLMs, etc.).

Resumen visual de jerarquía evolutiva:

```
SGD
  ↓ (agrega inercia)
SGD + Momentum / Nesterov
  ↓ (agrega adaptación por parámetro)
Adagrad
  ↓ (arregla el problema de apagado)
RMSprop
  ↓ (combina momentum + RMSprop + bias correction)
Adam
  ↓ (arregla weight decay para mejor generalización)
AdamW  ← estado del arte para la mayoría de casos modernos
```

### Arquitectura de los optimizadores

Todos los optimizadores heredan de `torch.optim.Optimizer`. Internamente cada optimizador tiene dos cosas importantes, las cuales permiten generar entrenamientos multicapas en un solo optimizador:

* `param_groups`: Es una lista de diccionarios. Cada grupo puede tener:
    * `params`: lista de tensores (los pesos)
    * `lr`: learning rate (puede ser diferente por capa)
    * `weight_decay`, `momentum`, `betas` (para Adam), etc.
    * Ejemplo: puedes tener un grupo para la capa de embedding con lr=0.001 y otro para el resto con lr=0.01.
* `state`: Un diccionario que guarda información extra que el optimizador necesita entre pasos:
    * En SGD con momentum → guarda el "velocidad" anterior.
    * En Adam → guarda exp_avg (media móvil del gradiente) y exp_avg_sq (media móvil del gradiente al cuadrado). Esto es lo que permite que Adam “recuerde” el comportamiento pasado.

Esta arquitectura es muy flexible y es la razón por la que se puede cambiar de SGD a Adam con solo cambiar una línea.

## Modulos (`torch.nn.Module`)

La clase `torch.nn.Module` implementa de forma generalizada la administración automática de capas en un modelo de una o más capas (multi-capa). Provee una interfaz en la cual nosotros solo tenemos que implementar algunos métodos y el módulo se encargará de casi todo lo demás.

Para ejemplificar, reimplementaremos nuestro modelo de regresión lineal mediante un módulo.

```py
import torch
import torch.nn as nn

# Definimos nuestra red como una clase que hereda de nn.Module
class RegresionLineal(nn.Module):
    def __init__(self):
        super().__init__()
        # Definimos las capas / parámetros (no necesitamos requires_grad=True)
        self.a = nn.Parameter(torch.tensor(0.0))
        self.b = nn.Parameter(torch.tensor(0.0))

    def forward(self, x):
        # Aquí va SOLO la pasada hacia adelante (la predicción)
        # NO hacer loss, NO hacer backward, NO hacer .to(device) aquí
        return self.a * x + self.b

# ------------------------------
# Uso del modelo
model = RegresionLineal()

# Datos de entrenamiento
# Cada elemento debe ir en su propia lista, por la generalización a más de una dimensión
x = torch.tensor([[1.0], [2.0], [3.0], [4.0]])
y = torch.tensor([[7.8], [15.1], [22.4], [29.7]])

# Optimizador (ahora pasamos todos los parámetros del modelo de una vez)
# el método `.parameters()` del módulo devuelve una lista de todos los parámetros
optimizer = torch.optim.Adam(model.parameters(), lr=1)

# Bucle de entrenamiento
for epoch in range(300):
    # Forward (solo llamamos al modelo), aquí se ejecuta forward automáticamente
    y_pred = model(x)
    optimizer.zero_grad()
    # Gradiente del error
    loss = torch.mean((y_pred - y) ** 2)
    loss.backward()
    # Actualización de parámetros
    optimizer.step()

# Mostramos el valor final de los parámetros aprendidos
print("\nParámetros finales:")
for name, param in model.named_parameters():
    print(f"{name:8s} = {param.item():.4f}")
```

Hay módulos ya implementados que podemos reutilizar, por ejemplo hay un módulo que ya implementa regresiones lineales (`nn.Linear`):

```py
import torch
import torch.nn as nn

# Uso del modelo
model = nn.Linear(in_features=1, out_features=1, bias=True)

# Datos de entrenamiento
# Cada elemento de 'x' de tamaño `in_features` y de 'y' `out_features`
x = torch.tensor([[1.0], [2.0], [3.0], [4.0]])
y = torch.tensor([[7.8], [15.1], [22.4], [29.7]])

# Optimizador (ahora pasamos todos los parámetros del modelo de una vez)
# el método `.parameters()` del módulo devuelve una lista de todos los parámetros
optimizer = torch.optim.Adam(model.parameters(), lr=1)

# Bucle de entrenamiento
for epoch in range(300):
    # Forward (solo llamamos al modelo), aquí se ejecuta forward automáticamente
    y_pred = model(x)
    optimizer.zero_grad()
    # Gradiente del error
    loss = torch.mean((y_pred - y) ** 2)
    loss.backward()
    # Actualización de parámetros
    optimizer.step()

# Mostramos el valor final de los parámetros aprendidos
print("\nParámetros finales:")
for name, param in model.named_parameters():
    print(f"{name:8s} = {param.item():.4f}")
```

Para ver un poco más el potencial de los módulos, a continuación vemos uno con varias capas que implementa una Multi Layer Perceptrion (red neuronal) de 2 capas ocultas:

```py
class MLP(nn.Module):
    def __init__(self, input_size=1, hidden_size=32, output_size=1):
        super().__init__()
        # Recordemos que cada pareja de capas es simplemente una regresión
        # lineal con entrada y salida de varias dimensiones

        # Regresión capa-input a oculta-1
        self.layer1 = nn.Linear(input_size, hidden_size)
        # Regresión oculta-1 a oculta-2
        self.layer2 = nn.Linear(hidden_size, hidden_size)
        # Regresión oculta-2 a capa output
        self.layer3 = nn.Linear(hidden_size, output_size)
        
    def forward(self, x):
        x = self.layer1(x)
        x = self.relu(x)
        x = self.layer2(x)
        x = self.relu(x)
        x = self.layer3(x)
        return x


# Ejemplo de uso
model = MLP()
print(model)                    # Muestra la estructura bonita

# Ver todos los parámetros
print("\nNúmero de parámetros entrenables:", sum(p.numel() for p in model.parameters()))
```

