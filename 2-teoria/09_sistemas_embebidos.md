# Capítulo: Sistemas Embebidos

> Objetivo del capítulo: entender **el hardware sobre el que corre el software que pruebas**: qué hay dentro de una ECU, cómo se organizan la memoria, las interrupciones y los periféricos, y cómo se conectan sensores y actuadores. En una entrevista de Embedded SW Engineer te pedirán razonar con cierto detalle de hardware; en una de Test Engineer o Function Lead, lo que se valora es que puedas **conectar un síntoma observado en HIL con su causa probable en el hardware** (un timeout, un reset inesperado, una lectura analógica fuera de rango).
>
> **Límites con otros capítulos (para no repetir):**
> - Hardware, memoria, interrupciones, periféricos, sensores/actuadores, energía y reloj → **este capítulo**.
> - RTOS vs Linux, QNX, planificación de tareas, IPC, proceso de arranque (boot) y debugging a nivel de sistema → *Sistemas Operativos Embebidos*.
> - Concurrencia desde el diseño de software (race conditions, deadlock, inversión de prioridades) → ya en *Ingeniería de Software*.
> - Protocolos CAN/LIN/Ethernet a nivel de trama → ya en *Protocolos de Comunicación*. Aquí solo el **periférico** que los implementa.
> - Fault injection y bancos HIL → ya en *HIL Testing*. Aquí se enlazan con la causa de hardware.

---

## 1. Qué es un sistema embebido

Un **sistema embebido** es un sistema de cómputo dedicado a una función específica, integrado dentro de un producto mayor, que interactúa con el mundo físico mediante sensores y actuadores. Una **ECU (Electronic Control Unit)** es un sistema embebido automotriz.

**Características típicas:**
- **Función dedicada**: hace una tarea (o un conjunto acotado), no es de propósito general.
- **Recursos limitados**: memoria, CPU, consumo de energía y costo ajustados.
- **Tiempo real**: debe responder dentro de plazos definidos (determinismo, ya visto en HIL).
- **Confiabilidad y seguridad**: opera años en condiciones adversas y, en ciertas funciones, un fallo puede ser peligroso.
- **Entorno exigente**: rango amplio de temperatura, vibración, ruido eléctrico, tensión de batería variable.
- **Actualización costosa**: reprogramar implica flasheo (UDS) y control formal.

**Embebido vs propósito general:**

| Aspecto | Sistema embebido (ECU) | PC / servidor |
|---|---|---|
| Función | Dedicada | General |
| Recursos | Limitados y fijos | Abundantes y ampliables |
| Tiempo real | Frecuentemente exigido | Normalmente no |
| Vida útil | 10–15 años | 3–5 años |
| Interfaz de usuario | Limitada o ninguna | Rica |
| Actualización | Controlada y escasa | Frecuente |

## 2. Anatomía de una ECU

Una ECU típica integra los siguientes bloques (de forma esquemática):

```
 Batería 12 V ──► [Protección + Regulación de alimentación] ──► MCU / SoC
                                                                 │
 Sensores ──► [Acondicionamiento de señal] ──► ADC / GPIO / Timers
 Actuadores ◄── [Drivers de potencia] ◄──── PWM / GPIO
 Buses vehiculares ◄──► [Transceptores CAN / LIN / Ethernet PHY] ◄──► Controladores de bus
 Memorias externas (opcional) · Watchdog externo · Reloj (cristal) · Conectores
```

- **MCU / SoC**: el "cerebro" (ver sección 3).
- **Alimentación**: protección contra polaridad inversa y picos, reguladores que generan las tensiones internas (por ejemplo, 5 V, 3.3 V, 1.2 V) y **supervisores** que mantienen la MCU en reset si la tensión no es válida.
- **Acondicionamiento de señal**: filtros, divisores de tensión, protección ESD y adaptación de niveles antes del ADC o de las entradas digitales.
- **Drivers de potencia**: interruptores *high-side/low-side*, puentes H y etapas que manejan cargas que la MCU no puede alimentar directamente. Muchos incluyen **diagnóstico** (detección de circuito abierto, corto a tierra o a batería, sobrecorriente).
- **Transceptores**: convierten las señales lógicas del controlador de bus al medio físico (el transceptor CAN que vimos en Protocolos de Comunicación).
- **Watchdog externo, supervisores y reloj**: soporte de confiabilidad (ver secciones 9 y 10).

*Conexión con HIL:* cada bloque es un punto donde el banco puede **estimular** (señales analógicas y digitales) o **inyectar fallas** (abrir o cortocircuitar una línea, variar la tensión de alimentación). Los módulos del VT System que viste corresponden a estos bloques.

## 3. Microcontrolador, microprocesador, SoC, FPGA y ASIC

| Elemento | Qué es | Uso típico automotriz |
|---|---|---|
| **Microcontrolador (MCU)** | CPU + memoria (Flash/RAM) + periféricos en un solo chip, con tiempo real y bajo consumo | ECUs de control: motor, frenos, carrocería, gateways |
| **Microprocesador (MPU/CPU de aplicaciones)** | CPU de mayor desempeño que usa memoria externa y suele correr un SO como Linux o QNX | Infotainment, cluster, ADAS de alto desempeño |
| **SoC (System on Chip)** | Integra CPU(s), GPU, aceleradores (NPU/DSP), memoria y periféricos | Cockpit digital, cómputo central, percepción ADAS |
| **FPGA** | Lógica reconfigurable por hardware | Procesamiento de señal de alta velocidad, prototipos |
| **ASIC** | Chip diseñado a medida | Funciones de alto volumen y desempeño específico |

Ejemplos de familias que conviene reconocer (no se espera memorizar detalles): Infineon AURIX, NXP S32K/S32G, Renesas RH850 y R-Car, STMicroelectronics SPC5, y SoCs de Qualcomm y NVIDIA para cockpit y ADAS. La relación con tu experiencia: infotainment sobre QNX/Qualcomm es el mundo **SoC/microprocesador con SO**, mientras que muchas ECUs de control con AUTOSAR Classic son el mundo **MCU con RTOS**.

## 4. Arquitectura del procesador

- **Familias ARM Cortex**: **Cortex-M** (microcontroladores, bajo consumo), **Cortex-R** (tiempo real y seguridad funcional, comunes en ECUs críticas) y **Cortex-A** (procesadores de aplicaciones, corren Linux/QNX). Existen también otras arquitecturas (TriCore, RH850, Power Architecture).
- **Von Neumann vs Harvard**: en Harvard, instrucciones y datos tienen buses separados (más paralelismo); muchas MCUs usan variantes de Harvard modificada.
- **RISC vs CISC**: conjunto reducido de instrucciones simples (ARM) frente a instrucciones complejas (x86). En embebido domina RISC por eficiencia.
- **Pipeline, caché y predicción de saltos**: mejoran el desempeño promedio pero **complican el determinismo**, por lo que en sistemas de tiempo real crítico se usan con cuidado (por ejemplo, memorias de acceso predecible como TCM).
- **FPU y DSP**: aceleran operaciones de punto flotante y de procesamiento de señal.
- **MPU/MMU**: la **MPU (Memory Protection Unit)** define regiones de memoria con permisos (lectura/escritura/ejecución) para aislar software; la **MMU** añade memoria virtual y la usan los SO de propósito general.
- **Lockstep (núcleos en paso sincronizado)**: dos núcleos ejecutan las mismas instrucciones y un comparador detecta divergencias. Es un mecanismo de seguridad de hardware común en MCUs para ASIL alto.
- **Tamaño de palabra**: 8/16/32/64 bits; determina rango de datos y direcciones.

### 4.1 Endianness (orden de bytes)
Define cómo se almacenan en memoria los valores de varios bytes. El valor `0x1234`:
- **Little-endian**: se guarda `34 12` (el byte menos significativo primero). Es lo común en x86 y en la mayoría de configuraciones ARM.
- **Big-endian**: se guarda `12 34`.

Es relevante en tu trabajo: en las bases de datos DBC, el formato **Intel** corresponde a little-endian y **Motorola** a big-endian. Un error de orden de bytes entre emisor y receptor produce valores aparentemente "aleatorios" aunque el bus funcione perfectamente (un defecto de interfaz clásico).

## 5. Memoria

### 5.1 Tipos y funciones
| Memoria | Volátil | Uso | Notas |
|---|---|---|---|
| **Flash** | No | Código del programa, constantes, calibraciones | Se borra por **sectores** y se escribe por páginas; tiene límite de ciclos de borrado/escritura (*endurance*). Es lo que reprograma el flashing UDS |
| **SRAM** | Sí | Variables, pila (*stack*), montón (*heap*) | Rápida; se pierde al quitar la alimentación |
| **EEPROM / Data Flash** | No | Datos persistentes (contadores, configuración, DTCs) | Gestionada por una capa (en AUTOSAR, el NVRAM Manager) para repartir el desgaste |
| **ROM** | No | Código inmutable (bootloader de fábrica) | No se puede modificar |
| **Caché / TCM** | Sí | Aceleran el acceso a código o datos | La caché reduce el determinismo; la TCM (memoria fuertemente acoplada) lo conserva |
| **DRAM externa** | Sí | Grandes volúmenes en SoCs (infotainment, ADAS) | Controlador de memoria más complejo, mayor consumo |

### 5.2 Mapa de memoria y secciones del programa
Cada MCU tiene un **mapa de memoria**: regiones de direcciones asignadas a Flash, RAM, periféricos y registros. El *linker* organiza el programa en **secciones**:
- `.text`: código ejecutable (en Flash).
- `.rodata`: constantes (en Flash).
- `.data`: variables globales inicializadas (valor inicial en Flash, copiadas a RAM en el arranque).
- `.bss`: variables globales sin inicializar (puestas a cero en el arranque).
- **Stack**: variables locales y direcciones de retorno. Un **desbordamiento de pila** (*stack overflow*) corrompe memoria y puede producir fallos difíciles de reproducir.
- **Heap**: memoria dinámica. En sistemas críticos suele evitarse (MISRA) por fragmentación y no determinismo.

El **archivo de mapa (.map)** generado por el linker indica el tamaño y la dirección de cada símbolo. Es la fuente de las direcciones que vimos en XCP/A2L: el A2L se genera a partir de él y por eso debe corresponder al build exacto que corre en la ECU.

### 5.3 Registros y E/S mapeada en memoria
Los periféricos se controlan leyendo y escribiendo **registros** ubicados en direcciones específicas del mapa de memoria (*memory-mapped I/O*). Por eso las variables que apuntan a registros se declaran `volatile`: el valor puede cambiar por el hardware y el compilador no debe optimizar los accesos.

### 5.4 ECC y protección
- **ECC (Error Correcting Code)**: detecta y corrige errores de un bit (y detecta de dos) en Flash y RAM. Es habitual en MCUs de seguridad.
- **Protección de memoria (MPU)**: impide que una tarea de menor criticidad corrompa la memoria de otra (*freedom from interference*).
- **Particiones duales (dual-bank Flash)**: permiten programar un banco mientras se ejecuta desde el otro, base de las actualizaciones seguras y de OTA.

## 6. Interrupciones y excepciones

### 6.1 Concepto
Una **interrupción** es una señal que suspende la ejecución normal para atender un evento (llegada de una trama CAN, desbordamiento de un timer, fin de conversión del ADC). La CPU guarda el contexto, ejecuta la **rutina de servicio (ISR)** y regresa al punto interrumpido.

### 6.2 Elementos clave
- **Tabla de vectores**: asigna a cada fuente de interrupción la dirección de su ISR.
- **Controlador de interrupciones** (por ejemplo, NVIC en Cortex-M): gestiona prioridades, habilitación y anidamiento.
- **Prioridades y anidamiento**: una interrupción de mayor prioridad puede interrumpir a una ISR de menor prioridad.
- **Latencia de interrupción**: tiempo desde el evento hasta que se ejecuta la primera instrucción de la ISR. Depende de la prioridad, de si las interrupciones estaban enmascaradas, del guardado de contexto y de la arquitectura.
- **Enmascaramiento**: inhabilitar temporalmente interrupciones para proteger secciones críticas (a costa de aumentar la latencia).

### 6.3 Sondeo (polling) vs interrupciones vs DMA
| Método | Cómo funciona | Ventaja | Desventaja |
|---|---|---|---|
| **Polling** | El software consulta periódicamente el estado | Simple, determinista | Consume CPU, puede perder eventos |
| **Interrupción** | El hardware avisa cuando ocurre el evento | Respuesta rápida, eficiente | Añade latencia y complejidad (concurrencia) |
| **DMA** | Un controlador mueve datos entre periférico y memoria sin la CPU | Libera la CPU en transferencias grandes | Configuración más compleja, contención de bus |

### 6.4 Buenas prácticas para ISRs
Mantenerlas **cortas** (marcar un evento y diferir el procesamiento a una tarea), evitar bloqueos y llamadas largas, proteger datos compartidos con el contexto principal y no asumir orden estricto entre fuentes. Un defecto típico: una ISR demasiado larga hace que se pierdan tramas CAN o se incumplan plazos.

### 6.5 Excepciones y fallos
Además de las interrupciones externas existen **excepciones** generadas por la propia CPU: acceso a memoria inválido (*bus fault*, *memory fault*), instrucción no válida, división por cero o desbordamiento de pila. Un **hard fault** suele terminar en un **reset** o en un estado seguro, y su análisis (registro de la dirección que falló, contenido de la pila) es una tarea habitual de debugging a nivel de código (conecta con tu experiencia en Bosch).

## 7. Periféricos

### 7.1 Entradas y salidas básicas
- **GPIO**: pines digitales de entrada o salida. Pueden tener *pull-up/pull-down*, filtros y capacidad de interrupción. En una entrada ruidosa se aplica **antirrebote (debounce)**.
- **ADC (Convertidor analógico-digital)**: convierte una tensión en un número.
  - **Resolución**: n bits → 2ⁿ niveles. **LSB = Vref / 2ⁿ**.
  - Ejemplo: ADC de 12 bits con Vref = 5 V → LSB = 5 / 4096 ≈ 1.22 mV. Una lectura de 2048 corresponde a 2.5 V.
  - Factores que afectan la lectura: **frecuencia de muestreo** (criterio de Nyquist: más del doble de la frecuencia máxima de la señal), ruido, impedancia de la fuente, precisión de Vref, error de cuantización y calibración.
- **DAC**: convierte un número en una tensión.
- **PWM (modulación por ancho de pulso)**: señal digital con frecuencia fija y **ciclo de trabajo (duty)** variable, usada para controlar potencia en motores, iluminación y válvulas.
  - Ejemplo: PWM a 20 kHz → periodo 50 µs. Con duty de 30% la señal está en alto 15 µs.
- **Timers/contadores**: base de la temporización. Funciones comunes: **generación de periodos**, **captura de entrada** (medir el ancho de un pulso o la frecuencia, por ejemplo, la señal de un sensor de velocidad de rueda) y **comparación de salida**.

### 7.2 Comunicación serie dentro de la ECU y hacia dispositivos
| Interfaz | Hilos | Característica | Uso típico |
|---|---|---|---|
| **UART** | 2 (TX/RX) | Asíncrona, punto a punto, requiere acordar baud rate | Consola de depuración, módulos simples |
| **SPI** | 4 (SCLK, MOSI, MISO, CS) | Síncrona, full-duplex, rápida, un CS por esclavo | Memorias, ADCs externos, drivers de potencia con diagnóstico |
| **I²C** | 2 (SDA, SCL) | Síncrona, multi-dispositivo por direcciones, más lenta | Sensores, EEPROM pequeñas |

### 7.3 Controladores de bus vehicular
El **controlador CAN/CAN FD** implementa el protocolo en hardware (arbitraje, CRC, ACK, bit stuffing); el software interactúa con él mediante **buzones (mailboxes)** y **filtros de aceptación** (los que mencionamos al hablar de *broadcast*). Se configura el **bit timing**.
- **Ejemplo de bit timing**: a 500 kbit/s el tiempo de bit es 2 µs. Si se divide en 16 *time quanta* (TQ), cada TQ dura 125 ns. Con un **punto de muestreo** del 87.5%, la muestra se toma tras 14 TQ y quedan 2 TQ de segmento final. Si la configuración de bit timing no coincide entre nodos (o la tolerancia del reloj es insuficiente), aparecen errores de bit y de CRC en el bus aunque el cableado esté bien.
- El **controlador LIN** y el **controlador/MAC Ethernet** (con su PHY externo) cumplen un rol análogo para LIN y Ethernet automotriz.

### 7.4 Otros periféricos relevantes
- **DMA**: transferencias sin CPU (ver 6.3).
- **Watchdog (WDT)**: temporizador que reinicia el sistema si el software no lo "alimenta" a tiempo (ver sección 10).
- **RTC**: reloj de tiempo real.
- **HSM (Hardware Security Module)**: bloque de seguridad con aceleración criptográfica y almacenamiento de claves, base del *secure boot* y de la autenticación (ligado a ciberseguridad).
- **Unidades de depuración y trazado** (JTAG/SWD, ETM; ver sección 11).

## 8. Sensores y actuadores

### 8.1 Sensores
Convierten una magnitud física en una señal eléctrica. Lo importante es **cómo llega esa señal a la ECU**:
| Tipo de salida | Ejemplos automotrices | Cómo lo lee la ECU |
|---|---|---|
| **Analógica** (tensión/resistencia) | Temperatura NTC, posición de pedal (potenciómetro o Hall), presión | ADC (a menudo con divisor resistivo y filtro) |
| **Digital / frecuencia / PWM** | Velocidad de rueda (Hall/inductivo), posición del cigüeñal | GPIO, captura de timer |
| **Bus digital** | Sensores inteligentes (radar, IMU) | SPI, I²C, CAN, Ethernet |
| **Imagen / nube de puntos** | Cámara, lidar | Interfaces de alta velocidad hacia un SoC |

**Redundancia y plausibilidad**: los sensores críticos (por ejemplo, el pedal del acelerador) suelen tener **dos canales** con características distintas y la ECU compara ambos; una discrepancia genera un DTC y un modo degradado.

### 8.2 Actuadores y etapas de potencia
La MCU maneja señales de baja potencia, por lo que necesita **drivers** para cargas como relés, solenoides, motores, inyectores o LEDs:
- **Interruptor high-side / low-side**: conecta la carga a la batería o a tierra.
- **Puente H**: controla el sentido de giro de un motor DC.
- **Smart drivers**: integran protección (sobrecorriente, sobretemperatura) y **diagnóstico** reportado a la MCU (por SPI o por una salida de estado).

### 8.3 Fallas eléctricas típicas
**Circuito abierto**, **corto a tierra**, **corto a batería** y **corto entre pines**. Se detectan por mediciones de corriente o tensión en el driver o en la entrada, y la ECU responde con un DTC y una reacción segura. Son exactamente las fallas que inyectas con el VT System (módulos de relés y cargas) en HIL, y la validación de la detección y la reacción es una prueba central de *fault management*.

## 9. Reloj y alimentación

### 9.1 Reloj
- Fuente: **oscilador de cristal** (y a veces un oscilador interno RC, menos preciso).
- **PLL**: multiplica la frecuencia base para generar la frecuencia del núcleo y de cada periférico (**árbol de reloj**).
- **Monitor de reloj**: detecta pérdida o desviación y fuerza una reacción segura.
- La precisión del reloj afecta directamente al **bit timing** de los buses y a la exactitud de los timers.

### 9.2 Alimentación en un vehículo
La batería nominal es de 12 V, pero la ECU debe tolerar mucho más que eso:
- **Arranque en frío (cold crank)**: la tensión puede caer drásticamente durante el arranque del motor. Muchas ECUs deben seguir operativas o reiniciarse de forma controlada.
- **Load dump**: pico de sobretensión al desconectarse la batería con el alternador cargando.
- **Polaridad inversa** y **transitorios** de conmutación de cargas inductivas.
- Estas condiciones se definen en normas como **ISO 16750** (condiciones eléctricas y ambientales) e **ISO 7637** (transitorios). Los valores exactos dependen de la especificación del OEM y del tipo de ECU.
- **Reguladores y supervisores**: mantienen tensiones estables y generan **reset por baja tensión (brown-out)** cuando la alimentación cae por debajo del umbral.

### 9.3 Modos de energía, wake-up y fuentes de reset
- **Modos**: operación normal, bajo consumo, *sleep*, apagado. En vehículos, las ECUs entran en reposo para no descargar la batería y se despiertan por **ignición**, una **trama de bus** (*CAN wake-up*, gestión de red) o una señal local.
- **Fuentes de reset**: encendido (*power-on reset*), baja tensión, watchdog, reset por software, fallo de reloj, excepción grave. Conocer la **causa del último reset** (registro de estado) es una pista fundamental al depurar reinicios inesperados.

*Pruebas relacionadas en HIL:* variación de la tensión de alimentación (bajo voltaje, sobrevoltaje, microcortes), ciclos de encendido y apagado, comportamiento al despertar por bus y recuperación tras un brown-out. Los módulos de fuente programable del VT System habilitan estas pruebas.

## 10. Confiabilidad y seguridad a nivel de hardware embebido

- **Watchdog**: se debe "alimentar" periódicamente. Si el software se cuelga, el watchdog expira y reinicia el sistema o lo lleva a estado seguro. Un **watchdog de ventana** exige que la alimentación ocurra dentro de una ventana de tiempo (ni muy pronto ni muy tarde), lo que detecta también bucles anómalos. Los diseños críticos añaden un **watchdog externo** independiente de la MCU.
- **Lockstep y ECC** (ver secciones 4 y 5): detección de errores de cómputo y de memoria.
- **BIST (Built-In Self Test)**: autotest de memorias y lógica al arranque o periódicamente.
- **Estado seguro y modos degradados**: la ECU debe saber a qué estado ir ante una falla (concepto de FTTI y estado seguro del capítulo de Ingeniería de Sistemas).
- **Redundancia**: duplicación de sensores, canales o procesadores.
- **Grados de calificación AEC-Q100** para circuitos integrados automotrices, por temperatura ambiente de operación: Grado 0 (−40 a +150 °C), Grado 1 (−40 a +125 °C), Grado 2 (−40 a +105 °C), Grado 3 (−40 a +85 °C).
- **EMC/EMI**: la ECU no debe emitir interferencias excesivas ni ser vulnerable a ellas. Recuerda que una causa de fallo puede ser **ambiental** (campo electromagnético) y no un defecto de software.

## 11. Herramientas de desarrollo y depuración de hardware

- **Interfaces de depuración (JTAG / SWD)**: permiten parar el procesador, poner *breakpoints*, leer y escribir memoria y registros, y programar la Flash. Es lo que usa la **debug board** del banco HIL que mencionamos.
- **Depuradores**: por ejemplo, Lauterbach TRACE32, SEGGER J-Link, iSystem winIDEA y los integrados en los IDEs.
- **Trazado de instrucciones (ETM/trace)**: registra la ejecución sin detenerla; clave para problemas de timing donde un *breakpoint* alteraría el comportamiento.
- **Osciloscopio**: observar formas de onda reales (PWM, señales de bus, rizado de alimentación, tiempos de reset).
- **Analizador lógico**: ver varias señales digitales y decodificar SPI, I²C o UART.
- **Multímetro, fuente de laboratorio y cargas**: mediciones básicas y control de la alimentación.
- **Herramientas de bus**: CANoe y sus interfaces VN (ya vistas).
- **Archivo de mapa y A2L**: correlación de símbolos y direcciones para depurar y medir con XCP.

*Conexión con tu CV:* "source-level debugging" y "patch development" en Bosch implican usar estas herramientas para correlacionar una falla observada en HIL con una línea de código, una región de memoria o un evento de hardware.

## 12. Cómo conectar un síntoma de HIL con una causa de hardware

Una tabla práctica para razonar en entrevista (hipótesis típicas, no reglas absolutas):

| Síntoma observado | Posibles causas de hardware o de bajo nivel a revisar |
|---|---|
| Reinicios inesperados de la ECU | Brown-out por caída de alimentación, expiración del watchdog, excepción grave (hard fault), stack overflow |
| Lectura analógica incorrecta o ruidosa | Referencia o calibración del ADC, impedancia de fuente, filtro, ruido de alimentación, conexión abierta o corto |
| Mensajes CAN perdidos o errores de bus intermitentes | Bit timing o tolerancia del reloj, terminación del bus, carga de bus, ISR demasiado larga o buffer saturado |
| Valores "raros" en una señal multibyte | Endianness o formato Intel/Motorola mal configurado, escalado |
| Fallo solo con alta carga o a ciertas temperaturas | Margen de timing, calentamiento, límites de la alimentación |
| ECU no despierta o se descarga la batería | Fuente de wake-up, gestión de red, consumo en reposo |
| Variable observada con XCP no coincide con lo esperado | A2L de otra versión, dirección de memoria equivocada, acceso no atómico |
| Falla de flasheo intermitente | Alimentación inestable durante la programación, timeouts de CAN-TP, sectores de Flash |
| Actuador no responde o DTC de carga | Circuito abierto, corto, driver en protección térmica, falta de alimentación de potencia |

## Ejercicios numéricos de práctica
1. **ADC**: 10 bits, Vref = 3.3 V. ¿Cuál es el LSB? ¿Qué tensión representa una lectura de 512? *(LSB = 3.3/1024 ≈ 3.22 mV; 512 ≈ 1.65 V)*
2. **PWM**: frecuencia de 10 kHz y duty de 25%. ¿Cuánto dura el estado alto? *(periodo 100 µs → 25 µs en alto)*
3. **CAN bit timing**: a 1 Mbit/s con 10 TQ por bit, ¿cuánto dura un TQ? *(bit = 1 µs → TQ = 100 ns)*
4. **Endianness**: ¿cómo se almacena `0x0A0B0C0D` en memoria little-endian? *(0D 0C 0B 0A)*
5. **Latencia**: CPU a 100 MHz y una ISR de 500 ciclos. ¿Cuánto tarda la ISR? *(500 × 10 ns = 5 µs)*
6. **Presupuesto de ISR**: si llegan tramas CAN cada 100 µs en el peor caso y la ISR tarda 5 µs, ¿qué porcentaje de CPU ocupa solo esa ISR? *(5%)*

## Preguntas de teoría
1. ¿Qué distingue a un sistema embebido de uno de propósito general? Da cuatro diferencias.
2. Describe los bloques principales de una ECU y qué función cumple cada uno.
3. ¿Qué diferencia hay entre MCU, microprocesador y SoC? ¿Dónde ubicarías tu experiencia en infotainment con QNX?
4. ¿Para qué sirven los núcleos Cortex-M, Cortex-R y Cortex-A, y qué es el lockstep?
5. ¿Por qué la caché complica el determinismo en tiempo real?
6. Explica qué es endianness y cómo se relaciona con los formatos Intel y Motorola de un DBC.
7. Describe las secciones de memoria de un programa (.text, .data, .bss, stack, heap) y dónde vive cada una.
8. ¿Por qué un archivo A2L debe corresponder al build exacto de la ECU?
9. ¿Qué es una ISR, qué es la latencia de interrupción y qué hace que una ISR sea mala práctica?
10. Compara polling, interrupciones y DMA.
11. ¿Qué es un hard fault y qué información usarías para analizarlo?
12. Calcula el LSB de un ADC de 12 bits con Vref de 5 V y explica qué factores afectan la exactitud real.
13. Explica el ciclo de trabajo de una PWM y para qué se usa en una ECU.
14. Compara UART, SPI e I²C.
15. ¿Qué es el bit timing de CAN y qué síntomas aparecen si está mal configurado?
16. ¿Qué es un watchdog de ventana y por qué es mejor que uno simple? ¿Para qué sirve uno externo?
17. ¿Qué condiciones de alimentación debe tolerar una ECU en un vehículo (cold crank, load dump) y cómo las probarías en HIL?
18. ¿Cuáles son las fuentes de reset de una MCU y por qué importa conocer la del último reinicio?
19. ¿Qué fallas eléctricas se inyectan en un driver de potencia y cómo se detectan?
20. Una ECU se reinicia de forma intermitente durante una prueba HIL. Plantea tu proceso de diagnóstico, desde hardware hasta software.

## Errores comunes
- Describir una ECU solo como "un microcontrolador con software" e ignorar alimentación, acondicionamiento de señal, transceptores y drivers, que son fuente frecuente de problemas.
- Confundir MCU con SoC o asumir que todo corre un SO: muchas ECUs de control ejecutan firmware sobre un RTOS o incluso sin SO.
- Pensar que `volatile` hace una variable segura entre ISR y tarea: solo evita optimizaciones del compilador, no garantiza atomicidad.
- Ignorar la endianness y los formatos de byte order al interpretar señales de bus.
- Olvidar que la caché y el pipeline mejoran el desempeño promedio pero dificultan el peor caso en tiempo real.
- Atribuir cualquier reinicio a "un bug de software" sin revisar alimentación, watchdog y causa del reset.
- Asumir que si el bus CAN "se ve bien" en el trace, el problema no puede ser de hardware (bit timing, reloj, terminación).
- Usar interrupciones largas con la intención de "hacer todo ahí", provocando pérdida de tramas y violaciones de plazos.
- Tratar el ADC como si midiera valores exactos: tiene resolución finita, ruido, error de cuantización y dependencia de Vref.
- Olvidar que una causa de fallo puede ser **ambiental** (EMI, temperatura, tensión) y no un defecto en código.
- No distinguir entre el periférico (hardware que implementa CAN) y el protocolo (reglas del bus) al explicar una falla.