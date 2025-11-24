# Multímetros: Guía Completa de Uso y Funcionamiento

## Descripción General
Un multímetro (o polímetro) es un instrumento de medición eléctrica portátil que combina varias funciones de medición en una sola unidad. Es la herramienta fundamental para electricistas, técnicos en electrónica e ingenieros.

Los multímetros modernos son digitales (DMM - Digital Multimeter), mostrando los valores en una pantalla numérica, aunque todavía existen análogos con aguja. Internamente, un multímetro digital es esencialmente un voltímetro digital con circuitos adicionales para convertir otras magnitudes (corriente, resistencia) en voltaje que el convertidor analógico-digital (ADC) pueda leer.

### Partes Principales
1.  **Pantalla (Display):** Muestra la lectura, unidades y símbolos de advertencia.
2.  **Selector (Dial):** Rueda giratoria para seleccionar la función y el rango.
3.  **Puertos (Jacks):**
    *   **COM (Común):** Siempre se conecta la punta negra (negativo/tierra).
    *   **VΩmA:** Para medir voltaje, resistencia y corrientes pequeñas (punta roja).
    *   **10A / 20A:** Puerto exclusivo para medir corrientes grandes (punta roja). Sin fusible o con fusible de alta capacidad.

---

## Simbología Común

| Símbolo | Significado | Descripción |
| :--- | :--- | :--- |
| **V⎓** o **DCV** | Voltaje Corriente Directa | Línea recta sobre puntos. Para baterías, electrónica, fuentes DC. |
| **V~** o **ACV** | Voltaje Corriente Alterna | Línea ondulada. Para enchufes domésticos, transformadores. |
| **mV** | Milivoltios | Para voltajes muy bajos (sensores, circuitos lógicos). |
| **A**, **mA**, **µA** | Amperios (Corriente) | Puede tener el símbolo de DC (⎓) o AC (~). |
| **Ω** (Omega) | Resistencia | Mide la oposición al flujo de corriente. |
| **)))** o **🔊** | Continuidad | Emite un pitido si hay conexión eléctrica (baja resistencia). |
| **➔+** (Diodo) | Prueba de Diodo | Mide la caída de voltaje en un diodo. |
| **⫞** (Capacitor) | Capacitancia | Mide la capacidad de almacenamiento de carga (Faradios). |
| **Hz** | Frecuencia | Mide ciclos por segundo. |
| **HOLD** | Retención de datos | Congela el valor en pantalla. |
| **REL** | Medición Relativa | Establece el valor actual como cero y mide la diferencia. |

---

## Conceptos de Física y Matemáticas para el Multímetro

Para entender qué estamos midiendo, es crucial recordar la **Ley de Ohm**:
$$V = I \times R$$
Donde $V$ es Voltaje (Volts), $I$ es Corriente (Amperios), y $R$ es Resistencia (Ohmios).

### 1. Impedancia de Entrada ($Z_{in}$)
*   **Concepto:** Es la resistencia interna del multímetro cuando mide voltaje.
*   **Importancia:** Un buen multímetro digital tiene una impedancia de entrada muy alta (típicamente **10 MΩ**).
*   **Por qué importa:** Si mides un circuito de alta impedancia con un medidor de baja impedancia, el medidor "carga" el circuito (roba corriente), alterando el voltaje que intentas medir y dando una lectura falsa (Efecto de Carga).

### 2. True RMS vs. Promedio (Average Responding)
*   **Concepto:** En Corriente Alterna (AC), el voltaje cambia constantemente.
*   **Promedio:** Mide el valor promedio y asume que es una onda senoidal perfecta, aplicando un factor de corrección ($1.11$). Falla si la onda está distorsionada (ej. variadores de frecuencia, fuentes conmutadas).
*   **True RMS (Valor Eficaz Verdadero):** Calcula matemáticamente la raíz cuadrada media ($V_{rms} = \sqrt{\frac{1}{T} \int_{0}^{T} v(t)^2 dt}$).
*   **Aplicación:** Necesario para medir con precisión en electrónica moderna y sistemas no lineales.

### 3. Resolución y Dígitos (Cuentas)
*   **Cuentas (Counts):** Un multímetro de 2000 cuentas muestra de 0 a 1999.
*   **Dígitos:** 3 ½ dígitos significa 3 dígitos completos (0-9) y uno que solo puede ser 0 o 1 (o signo).
*   **Precisión:** Se expresa como $\pm(\% \text{ de lectura} + \text{número de dígitos})$. Ejemplo: $\pm(1\% + 2)$. El error es el porcentaje del valor medido más un error fijo en el último dígito.

## Conexión entre Potencia, Energía, Corriente, Voltaje y Resistencia

A continuación se explica cómo se conectan estas magnitudes de forma clara y visual para que todas las unidades tengan sentido.

Cuando hablamos de la relación entre potencia, energía, corriente, voltaje y resistencia, nos referimos a la forma en se comportan en un circuito eléctrico en concreto. Por ejemplo, si un equipo eléctrico consume 100 W de potencia en un circuito de 110 V, con las siguientes fórmulas podemos obtener por ejemplo la corriente que el circuito está permitiendo pasar para poder consumir esos 100 W de potencia (que equivale a 100 J/s).

Es decir, si queremos calcular algún valor específico de la corriente eléctrica en un circuito o dispositivo eléctrico, podemos recopilar lo que sabemos y buscar una fórmula que relacione lo que quieremos con algo de lo que sabemos. 

Por ejemplo, si queremos calcular la corriente eléctrica que entra en un aparato eléctrico, sabiendo que su consumo es de 100W (que es 100 J/s) y que el voltaje de entrada es de 110V (que es 110 J/C), donde C es coulomb, podemos usar la relación $P=V\cdot I$, que en nuestro caso equivale a $100 W = 110 V \cdot I$, que al despejar nos deja ver que la corriente que entra en el aparato es de $I = \frac{100 W}{110 V} = \frac{100 J/s}{110 J/C} = \frac{100}{110} C/s \approx 0.909 A$.

#### 📊 Coulomb (C) (carga eléctrica)

El coulomb es la unidad de carga eléctrica, que mide la cantidad de electricidad que fluye por un conductor. 
*  Equivale a 6.241509345972621 x 10^18 electrones.

#### 🔌 Watt (W) (potencia)

👉 Un watt (W) es la unidad de potencia que te dice qué tan rápido se consume energía (joules) por unidad de tiempo (segundos).

* En fórmula: $$1 W = 1 J/s$$

#### 🔋 Energía (Joule) = potencia × tiempo

👉 Un joule (J) es la unidad de energía que representa la energía necesaria para aplicar una fuerza de 1 newton y mover un objeto 1 metro en la dirección de esa fuerza.

*   Si un aparato usa 1 W durante 1 segundo → 1 J.
*   Si usa 1 W durante 3600 s (1 h) → 3600 J = 3.6 kJ.

* En fórmula: $$1 J = 1 N \cdot 1 m$$

No hay que complicarnos mucho con la fórmula, solo tener en cuenta que representa energía real para realizar un trabajo como hacer funcionar un aparato eléctrico y por esta razón es la unidad en la que se mide el consumo de elctricidad en un circuito como el hogar, ya que el consumo del hogar se mide en $$kW\cdot h = (1000 W) \cdot (3600 s) = (1000 J/s) \cdot (3600 s) = 3,600,000 J$$

#### ⚡ Voltaje (V) = energía por carga

$$1 \text{ V} = 1 \text{ J/C (joule por coulomb)}$$
El voltaje dice cuánta energía recibe cada coulomb de carga, es decir, qué tan potentes (cargados de energía) vienen los electrones o cuanta energía trae cada coulomb.

*   **Ejemplo:** Una batería de 9 V da 9 joules por cada coulomb de carga que sale de esta.

#### 🔁 Ampere (A) = Corriente = carga por segundo

👉 Un ampere (A) es la unidad de corriente que te dice cuántos coulombs de carga eléctrica fluyen por unidad de tiempo (segundos).

$$1 \text{ A} = 1 \text{ C/s}$$

La corriente es “qué tan rápido” fluye la carga.
*   **Ejemplo:** 1 A significa 1 coulomb de electrones pasando cada segundo.

#### 🎯 La conexión clave entre voltaje, corriente y potencia
La potencia eléctrica es:
$$P = V \cdot I$$
$$(W = V \cdot A)$$

**Interpretación:**
*   El voltaje te dice cuánta energía gana cada coulomb.
*   La corriente te dice cuántos coulombs pasan cada segundo.

Entonces:
$$\text{energía por coulomb} \times \text{coulombs por segundo} = \text{energía por segundo}$$
$$\rightarrow \text{que son watts.}$$

Eso hace que $W = V \cdot A$ tenga todo el sentido.

Volviendo al ejemplo de la batería de 9 V, si supiéramos también la corriente que sale de esta, podríamos calcular la potencia que da la batería (Watts).

#### 🧱 Resistencia (Ω) = V/A

La resistencia limita el paso de la corriente, con el objetivo de asegurarse que solo entre la corriente requerida por el aparato (A) para lograr el consumo requerido (W).

$$ I = \frac{V}{R} $$

De esa forma, si conocemos el voltaje (V=J/C) y la corriente (C/s) objetivo para lograr el consumo requerido (W), podemos despejar para calcular la resistencia (Ω) necesaria.

#### 🔥 Potencia usando resistencia
Podemos combinar todo:
*   $P = I^2 \cdot R$ (Corriente al cuadrado multiplicada por la resistencia)
*   $P = V^2 / R$ (Voltaje al cuadrado dividido entre la resistencia)

Estas fórmulas derivan de $P = V \cdot I$ y $V = I \cdot R$.

#### 📦 Resumen mental (muy útil)

| Concepto | Fórmula | Interpretación |
| :--- | :--- | :--- |
| **Voltaje** | $V = J/C$ | Energía por carga |
| **Corriente** | $I = C/s$ | Cantidad de carga por segundo |
| **Potencia** | $P = V \cdot I$ | Energía por segundo |
| **Resistencia** | $R = V/A$ | Dificultad al paso de corriente |
| **Energía** | $E = P \cdot t$ | Potencia acumulada en el tiempo |

#### 📦 Notas simplificadas

* Por estas razones la corriente no es lo mismo que la energía, ya que la corriente son electrones que llevan la energía, y los electrones no se consumen, solo su energía. Por eso necesitamos un cable de linea y otro de neutro, porque el cable de linea mete electrones con energía, y el neutro regresa los electrones sin energía.
* En una red eléctrica doméstica entran en juego dos factores:
    * La corriente (V=C/s): que mide los electrones que pasan por segundo.
    * El voltaje (A=J/C): que mide la energía que lleva cada electrón, este suele ser constante en una red eléctrica doméstica (127 V o 220 V).
    * Por eso al multiplicar las cantidades anteriores podemos conocer la energía por segundo (W=J/s).
* Los electrodomésticos meten en juego los otros dos factores:
    * La potencia (W/s): la cantidad de energía que consume el electrodoméstico por segundo.
    * La resistencia (V/A=Ω): que limita la corriente que entra en el equipo y de esa forma asegurarse de que solo llegue la potencia requerida por el aparato (W)

### Consumo en el hogar ($kW\cdot h$)

Aunque la unidad de medida del consumo de energía es $kW\cdot h$ o más comunmente denotado $kWh$, en realidad equivale a enería consumida (Joules), como derivamos a continuación usando las fórmulas anteriores:

$$1 kW\cdot h = (1000 W) \cdot (3600 s) = (1000 J/s) \cdot (3600 s) = 3,600,000 J$$

Por lo tanto, si un electrodoméstico consume 1 kW•h, está consumiendo 3,600,000 J de energía. Solo que es más fácil calcular el consumo en $kW\cdot h$ que en Joules, incluso los electrodomésticos expresan su potencia (W) en su lista de características técnicas, para facilitar este cálculo. Por ejemplo, si un electrodoméstico tiene una potencia de 60W, el cálculo del consumo en kWh por cada hora que dure encendido sería de:

$$(60W)\cdot (1h) = (0.06kW)\cdot (1h) = 0.06kWh$$

ni siquiera tuvimos que meter al cálculo el voltaje de la red eléctrica.

**⚠️ Importante**: 
* En la práctica hay factores que alteran el consumo real de energía, el voltaje que llega al aparato no es exactamente 127V, puede variar entre 110V y 127V.
* Además hay aparatos que varían lo que consumen dependiendo de su estado de funcionamiento, por ejemplo: 
    * Una pantalla, que va a consumir menos o más enería dependiendo de su brillo.
    * Además algunos aparatos no dejan de trabajar incluso apagados aunque a un consumo mucho menor que estando encendidos, como las pantallas que tienen que estar al pendiente del control remoto.
    * Otro ejemplo sería un aire acondicionado inverter, que su consumo varía dependiendo de la potencia que esté usando el compresor, el cual suele ser al máximo los primeros 20 minutos y luego se reduce a un 25-35% de su potencia máxima.

Por estas razones es importante tener herramientas de medición como el multímetro para medir el voltaje y la corriente reales, entre otras cosas. Hay varios tipos de multímetros que se pueden usar para medir diferentes magnitudes y además unos simplifican más la medición que otros. El objetivo de esta guía, además de explicar simplificadamente los conceptos eléctricos, es documentar como tomar medidas con algunos tipos de multímetros.

---

## Ejemplos numéricos simplificados (sin tomar mediciones reales)

Para entender cómo se relacionan la potencia (W), corriente (A), voltaje (V), resistencia (Ω) y energía (Wh / kWh / J), veamos algunos ejemplos numéricos paso a paso. Asumiremos una red eléctrica doméstica con voltaje de 127 V, una tensión típica en muchas zonas de México, además son ejemplos solo con el fin de ilustrar el cálculo de las magnitudes eléctricas ya que no estamos tomando mediciones reales.

En la práctica, para obtener consumos más reales, realizamos lo siguiente:
* Medimos el voltaje de la red eléctrica doméstica con un multímetro, puede ser desde un toma-corrientes.
* Medimos la corriente que consume el electrodoméstico con un multímetro, es más facil con un multímetro de gancho y posiblemente un line-splitter.
* Calculamos la potencia que consume el electrodoméstico multiplicando el voltaje por la corriente obtenidos ($P = V \cdot I$).
* Calculamos el consumo de energía ($E = P \cdot t$) (en kWh), o cualquier otro valor usando los valores reales obtenidos anteriormente.

### Ejemplo 1: Una lámpara de 60 W conectada a 127 V

*   **Corriente ($I$)** usando $P = V \cdot I \Rightarrow I = P/V$:
    $$I = \frac{60\ \text{W}}{127\ \text{V}} \approx 0.472\ \text{A} \text{ (o 472 mA)}$$

*   **Resistencia ($R$)** usando $V = I \cdot R \Rightarrow R = V/I$:
    $$R = \frac{127\ \text{V}}{0.47244\ \text{A}} \approx 269\ \Omega$$

*   **Validar potencia** usando $P = I^2 \cdot R$:
    $$P = (0.472\ \text{A})^2 \cdot 269\ \Omega \approx 60\ \text{W} \text{ (Coincide, como debe ser)}$$

*   **Energía consumida** si la enciendes 5 horas ($E = P \cdot t$):
    *   En Wh: $E = 60\ \text{W} \times 5\ \text{h} = 300\ \text{Wh}$.
    *   En kWh: $300\ \text{Wh} = 0.300\ \text{kWh}$.
    *   En joules: Sabiendo que $1\ \text{kWh} = 3,600,000\ \text{J}$:
        $$E = 0.300\ \text{kWh} \times 3,600,000\ \frac{\text{J}}{\text{kWh}} = 1,080,000\ \text{J}$$

    Si conoces la tarifa por kWh, puedes calcular el costo:
    $$\text{Costo} = \text{Energía (kWh)} \times \text{Tarifa (\$kWh)}$$

**Ejemplo de cálculo de costo (con tarifa de ejemplo):**
Si la tarifa fuera 0.20 (moneda) / kWh, y usas la lámpara del Ejemplo 1 durante 5 h:
Energía = 0.300 kWh
Costo = $0.300 \times 0.20 = 0.06$ (moneda).
(Las tarifas varían; sustituye tu tarifa real).

### Resumen práctico para recordar

*   **Watts**: Qué tan rápido consumes energía (J/s).
*   **W $\times$ tiempo**: Energía total (Wh o kWh).
*   **V $\times$ A = W** (voltaje $\times$ corriente = potencia).
*   **A $\times$ $\Omega$ = V** (corriente $\times$ resistencia = voltaje).

Si algo te cuesta entender: elige un valor concreto (ej.: 60 W) y repite los pasos ($I = P/V$, $E = P \cdot t$) — los números hacen que la interpretación sea clara.

---

## Multímetros, mediciones

### 1. Medición de Voltaje (Voltímetro)
**Símbolo:** V⎓ (DC) o V~ (AC)

*   **Funcionamiento Interno:**
    *   El multímetro utiliza un **Divisor de Tensión** (resistencias de precisión en serie) para reducir el voltaje de entrada a un rango que el ADC pueda leer (usualmente 0-2V o 0-200mV).
    *   El ADC convierte este voltaje analógico en un número digital.
*   **Cómo usar:**
    1.  Conecta la punta negra en **COM** y la roja en **V**.
    2.  Selecciona el rango mayor al esperado (si no es auto-rango).
    3.  Conecta las puntas en **PARALELO** al componente o fuente.
*   **Precauciones:**
    *   Nunca exceder el voltaje máximo (ej. 600V o 1000V).
    *   Verificar la categoría de seguridad (CAT III/IV) para altos voltajes.

### 2. Medición de Corriente (Amperímetro)
**Símbolo:** A, mA, µA

*   **Funcionamiento Interno:**
    *   La corriente debe fluir *a través* del multímetro.
    *   Internamente, la corriente pasa por una resistencia de valor muy bajo y preciso llamada **Resistencia Shunt**.
    *   El multímetro mide la caída de voltaje en esta resistencia ($V = I \times R_{shunt}$) y calcula la corriente.
*   **Cómo usar:**
    1.  **¡CRÍTICO!**: con un multímetro convencional debes abrir el circuito para tener acceso al cable de linea que llega al dispositivo. 
    2. Con un multímetro de gancho puedes medir la corriente sin abrir el circuito, en especial si se cuenta con un line-splitter que facilita la separación de la fase y el neutro y tiene un hueco enmedio para el multímetro de gancho.
    3.  Conecta la punta negra en **COM**.
    4.  Conecta la punta roja en **mA** (corrientes bajas) o **10A/20A** (corrientes altas).
    5.  Coloca el multímetro en **SERIE** con el circuito.
*   **Advertencias:**
    *   **NUNCA** conectes en paralelo a una fuente de voltaje mientras estás en modo Amperios. Crearás un cortocircuito directo a través del Shunt, fundiendo el fusible interno o dañando el equipo (y riesgo de arco eléctrico).
    *   Empieza siempre por el puerto de 10A si no estás seguro de la magnitud.

### 3. Medición de Resistencia (Ohmetro)
**Símbolo:** Ω

*   **Funcionamiento Interno:**
    *   El multímetro utiliza una **Fuente de Corriente Constante** interna.
    *   Inyecta una pequeña corriente conocida ($I_{ref}$) a través del componente.
    *   Mide el voltaje resultante ($V_{medido}$).
    *   Calcula $R = V_{medido} / I_{ref}$.
*   **Cómo usar:**
    1.  **¡IMPORTANTE!** El circuito debe estar **DESENERGIZADO**.
    2.  Conecta las puntas en paralelo al componente.
    3.  Si mides una resistencia en un circuito, desconecta una pata para evitar medir componentes paralelos.

### 4. Prueba de Continuidad
**Descripción**: La continuidad eléctrica es la presencia de un camino ininterrumpido para que la corriente fluya entre dos puntos. La prueba de continuidad verifica si existe este camino, es decir, si un circuito está cerrado o si un cable no está roto. Es fundamental para diagnosticar fallas como cables cortados, fusibles quemados o conexiones defectuosas.

**Símbolo:** 🔊 

*   **Funcionamiento:** Similar al Ohmetro. Si la resistencia medida es menor a un umbral (típicamente < 30Ω o 50Ω), un comparador activa un buzzer (zumbador).
*   **Uso:** Verificar cables rotos, pistas de PCB, fusibles o interruptores.
*   **Tip:** Es la función más rápida para diagnósticos de "pasa/no pasa".

### 5. Prueba de Diodos
**Símbolo:** ➔+

*   **Funcionamiento:** Inyecta una corriente pequeña (aprox 1mA) y mide el voltaje.
*   **Resultados:**
    *   **Polarización Directa (Roja en Ánodo, Negra en Cátodo):** Muestra la caída de voltaje (Silicio: ~0.5V - 0.7V, Germanio: ~0.2V - 0.3V, LED: 1.5V - 3V).
    *   **Polarización Inversa:** Muestra "OL" (Over Limit) o 1.
    *   **Corto:** Muestra ~0V.
    *   **Abierto:** Muestra "OL" en ambos sentidos.

### 6. Capacitancia
**Símbolo:** ⫞

*   **Funcionamiento:** El multímetro carga el capacitor con una corriente conocida y mide el tiempo que tarda en llegar a cierto voltaje ($dV/dt = I/C$), o mide la constante de tiempo RC.
*   **Precaución:** **DESCARGAR** siempre los capacitores antes de medir. Un capacitor cargado puede destruir el multímetro.

---

## Tips y Solución de Problemas

### Voltaje Fantasma (Ghost Voltage)
*   **Síntoma:** El multímetro lee un voltaje bajo (ej. 20V-50V) en cables desconectados.
*   **Causa:** La alta impedancia de entrada capta el campo electromagnético de cables activos cercanos (inducción capacitiva).
*   **Solución:** Usar un multímetro con modo **LoZ** (Baja Impedancia) para drenar ese voltaje fantasma y confirmar si es real.

### Lecturas Inestables
*   Asegura un buen contacto de las puntas (limpia óxido o suciedad).
*   En rangos de mV, el ruido ambiental puede afectar; usa cables más cortos o trenzados.
*   Batería baja del multímetro: Produce lecturas erráticas y peligrosamente incorrectas.

### Fusibles Quemados
*   Si el multímetro marca 0.00 en modo Amperios incluso con corriente, el fusible interno está fundido.
*   Para probar el fusible sin abrirlo:
    1.  Pon el multímetro en Continuidad/Ohmios.
    2.  Pon la punta roja en el jack de VΩ.
    3.  Toca con la punta roja dentro del jack de A o mA.
    4.  Deberías leer una resistencia baja (el valor del shunt + fusible). Si dice "OL", el fusible está abierto.

## Seguridad (Categorías CAT)
No todos los multímetros sirven para todo.
*   **CAT I:** Electrónica protegida, bajo voltaje.
*   **CAT II:** Enchufes domésticos, electrodomésticos.
*   **CAT III:** Tableros de distribución, interruptores principales, sistemas trifásicos en edificios.
*   **CAT IV:** Origen de la instalación, líneas exteriores, medidores de compañía eléctrica. **Alto riesgo de arco.**
