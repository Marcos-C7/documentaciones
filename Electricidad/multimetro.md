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

#### 🔌 1. Potencia (W) = energía por segundo
$$W = J/s$$
La potencia te dice qué tan rápido se consume energía.
*   **Ejemplo:** Un foco de 60 W → consume 60 J cada segundo.

#### 🔋 2. Energía (J) = potencia × tiempo
$$J = W \cdot s$$
Esto ya lo viste:
*   Si un aparato usa 1 W durante 1 segundo → 1 J.
*   Si usa 1 W durante 3600 s (1 h) → 3600 J = 3.6 kJ.

Y en electricidad se usa mucho el **kW·h**, que es exactamente lo mismo pero a escala más grande:
$$1 \text{ kW}\cdot\text{h} = 1000 \text{ W} \times 3600 \text{ s} = 3,600,000 \text{ J}$$

#### ⚡ 3. Voltaje (V) = energía por carga
$$1 \text{ V} = 1 \text{ J/C (joule por coulomb)}$$
El voltaje dice cuánta energía recibe cada coulomb de carga.
*   **Ejemplo:** Una batería de 9 V da 9 joules por cada coulomb de carga que pasa.

#### 🔁 4. Corriente (A) = carga que pasa por segundo
$$1 \text{ A} = 1 \text{ C/s}$$
La corriente es “qué tan rápido” fluye la carga.
*   **Ejemplo:** 1 A significa 1 coulomb de electrones pasando cada segundo.

#### 🎯 5. La conexión clave entre voltaje, corriente y potencia
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

#### 🧱 6. Resistencia (Ω) = V/A
$$1 \text{ } \Omega = 1 \text{ voltio por ampere}$$
La resistencia dice cuánto “voltaje” necesitas para que pase 1 ampere.

Y de la ley de Ohm:
$$V = I \cdot R$$

Esto contiene otra conexión interesante:
$A \cdot \Omega = V$ (los amperes multiplicados por ohms dan voltaje).
Eso es como ver que $\text{metros/segundo} \times \text{segundos} = \text{metros}$. Las unidades “coinciden” porque la ley física las relaciona.

#### 🔥 7. Potencia usando resistencia
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

---

## Funcionalidades: Uso Detallado y Funcionamiento Interno

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
    1.  **¡CRÍTICO!** Debes abrir el circuito.
    2.  Conecta la punta negra en **COM**.
    3.  Conecta la punta roja en **mA** (corrientes bajas) o **10A/20A** (corrientes altas).
    4.  Coloca el multímetro en **SERIE** con el circuito.
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
**Símbolo:** )))

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
