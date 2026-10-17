# Capítulo: Diseño Basado en Modelos (Model-Based Design, MBD)

> Objetivo del capítulo: al terminar podrás (1) calcular la resolución/rango de un tipo de punto fijo y relacionarlo con el factor/offset de una señal DBC; (2) explicar por qué HIL exige un solver de paso fijo y detectar cuándo un modelo va a producir un overrun; (3) aplicar los tres criterios de cobertura de transición de estado a un chart de Stateflow real (como el de sesiones UDS); (4) explicar qué causa una discrepancia *back-to-back* entre MIL y SIL, y cuándo es aceptable; y (5) construir un modelo de ECU/planta para HIL a partir de una especificación y una matriz de comunicación. Conecta tu experiencia en Ford (modelos Simulink/Stateflow desde especificaciones y datos de CAN) con lo que ya viste de MIL/SIL/HIL.
>
> **Límites con otros capítulos (para no repetir):**
> - Definiciones de MIL, SIL, PIL, HIL y el concepto de "planta" → ya en *HIL Testing*. Aquí: cómo se construyen y verifican esos modelos.
> - MBSE y SysML (modelado de arquitectura de sistema) → ya en *Ingeniería de Sistemas*. Aquí: modelos **ejecutables** de comportamiento.
> - Scripting de MATLAB y su comparación con Python → ya en *Lenguajes de Programación*.
> - Técnica de prueba de transición de estado → ya en *Fundamentos de Software Testing*. Aquí: cómo se aplica a un diagrama de Stateflow.
> - MISRA C, análisis estático y cobertura de código → ya en *Ingeniería de Software*. Aquí: su equivalente a nivel de modelo.

---

## 1. Qué es el Diseño Basado en Modelos

**Model-Based Design (MBD)** es un enfoque de desarrollo en el que el **modelo ejecutable** es el artefacto central: se diseña el comportamiento en un entorno gráfico, se **simula** para validarlo, se **verifica** contra requisitos y, finalmente, se **genera código** automáticamente para la ECU. El flujo clásico:

```
Requisitos → Modelo (Simulink/Stateflow) → Simulación MIL → Generación de código
          → SIL → PIL → Integración en la ECU → HIL → Vehículo
```

**Por qué se usa en automotriz:**
- Permite **validar la lógica antes de que exista hardware** (shift-left llevado al diseño).
- El modelo sirve como **especificación ejecutable**: es menos ambiguo que un texto.
- La generación automática de código reduce errores de codificación manual y acelera iteraciones.
- Facilita la **trazabilidad** (requisitos vinculados a bloques) y las pruebas **back-to-back** entre niveles.
- Fomenta la **reutilización** de componentes entre proyectos y variantes.

**Retos:**
- Curva de aprendizaje y disciplina de modelado (un modelo mal estructurado es tan difícil de mantener como un código mal escrito).
- El código generado puede ser menos legible y debe cumplir restricciones del proyecto.
- Costo de licencias y de **calificación de herramientas** en proyectos de seguridad.
- Riesgo de "validar el modelo y olvidar el software real": las diferencias de precisión numérica y de tiempo entre simulación y ECU existen.

## 2. Simulink: fundamentos

### 2.1 Conceptos básicos
- **Modelo**: diagrama de **bloques** conectados por **señales**. Cada bloque realiza una operación; las señales llevan valores a lo largo del tiempo.
- **Simulación**: un **solver** calcula el estado del modelo paso a paso.
- **Bloques comunes**: `Constant`, `Gain`, `Sum`, `Product`, `Switch`, `Saturation`, `Rate Limiter`, `Relational Operator`, `Logical Operator`, `Unit Delay`, `Memory`, `Integrator`, `Transfer Fcn`, `PID Controller`, **tablas de consulta (Lookup Tables 1-D/2-D)** (muy usadas para mapas de calibración) y `Stateflow Chart`.
- **Subsistemas**: agrupan bloques. Los **virtuales** son solo organización visual; los **no virtuales/atómicos** se ejecutan como una unidad y se reflejan como funciones en el código generado.
- **Subsistemas condicionales**: *Enabled* (se ejecutan mientras una señal de habilitación es verdadera), *Triggered* (se ejecutan en un flanco) y *Function-Call* (se ejecutan cuando otro elemento los invoca; útiles para modelar tareas del SO).
- **Referencia a modelos (Model Reference)**: un modelo incluido dentro de otro como componente reutilizable, con compilación independiente. Es la base de la modularidad en proyectos grandes.
- **Bibliotecas**: bloques personalizados reutilizables.
- **Variantes (Variant Subsystems)**: seleccionan en tiempo de compilación o simulación entre alternativas del modelo (enlaza con la gestión de variantes de Ingeniería de Sistemas).

### 2.2 Tiempo y solvers
- **Solver de paso fijo (fixed-step)** vs **de paso variable (variable-step)**: el de paso variable ajusta el tamaño de paso para balancear precisión y velocidad (adecuado para simulación offline con dinámicas continuas). El de **paso fijo** es el que se usa para **generación de código y para HIL**, porque la ECU y el simulador de tiempo real ejecutan a un ritmo constante.
- **Tiempo de muestreo (sample time)**: cada bloque se ejecuta con una periodicidad. En ECUs hay **multirate** (tareas de 1 ms, 10 ms, 100 ms). Los bloques `Rate Transition` gestionan la transferencia de datos entre tasas distintas de forma determinista y segura.
- **Sistemas continuos y discretos**: la planta física suele modelarse con dinámica continua (o discretizada a un paso fijo), mientras que el controlador de la ECU es **discreto**.
- **Elección del paso para HIL**: debe ser lo bastante pequeño para representar la dinámica más rápida del modelo (guía práctica: varias veces por debajo del criterio de Nyquist) y lo bastante grande para que el modelo **complete su cómputo dentro del paso** en el simulador de tiempo real. Si no lo logra, ocurre un **overrun** (ver capítulo de HIL).

Configuración típica por script (ejemplo):
```matlab
mdl = 'mi_modelo';
set_param(mdl, 'SolverType', 'Fixed-step', ...
               'Solver',     'FixedStepDiscrete', ...
               'FixedStep',  '0.001');   % paso fijo de 1 ms
```

### 2.3 Tipos de datos y punto fijo
- Tipos: `double`, `single`, enteros con y sin signo (`int8`…`uint32`), `boolean`, enumeraciones y **punto fijo**.
- En ECUs sin unidad de punto flotante, o por determinismo y consumo, se usa **punto fijo**: un entero que representa un valor real mediante una **escala** (y un posible *offset*): `valor_real = entero × escala + offset`. Es exactamente el mismo concepto que **factor y offset de una señal en un DBC**.
  - Ejemplo: una señal de 16 bits con signo y 8 bits fraccionarios (`sfix16_En8`) tiene resolución 1/256 ≈ 0.0039 y rango de −128 a ≈ 127.996.
  - Ejemplo de DBC: velocidad en 16 bits sin signo con factor 0.01 → el valor crudo 12345 equivale a 123.45 km/h; el rango máximo es 655.35.
- **Saturación y desbordamiento**: las operaciones pueden exceder el rango. Hay que decidir si se **satura** (el valor se queda en el límite) o se **envuelve** (*wrap-around*). En software de seguridad suele preferirse la saturación explícita.
- **Conversiones implícitas** entre tipos y **pérdida de precisión** son fuentes clásicas de diferencias entre MIL y SIL.

### 2.4 Señales, buses y datos
- **Buses (bus objects)** agrupan señales relacionadas bajo una interfaz con nombre (equivalente a una `struct`). Preferibles a `Mux/Demux` para interfaces entre componentes.
- **Diccionario de datos (Data Dictionary)** o archivos de parámetros: centralizan constantes, calibraciones, tipos y definiciones de señales, con **unidades, rangos y valores por defecto**. Evita los "números mágicos" dentro de los bloques.
- **Parámetros de calibración**: se exponen como variables calibrables y se pueden ajustar en la ECU (por ejemplo, vía XCP/A2L, que ya conoces).
- **`Goto/From`**: conectan sin cables visibles, pero abusar de ellos vuelve el modelo difícil de leer.

## 3. Stateflow

### 3.1 Qué es
Stateflow añade a Simulink **máquinas de estados y diagramas de flujo** para modelar **lógica de modos, secuenciación y control de eventos**. Se coloca como un bloque `Chart` dentro del modelo de Simulink.

### 3.2 Elementos
- **Estados**: representan modos del sistema. Tienen **acciones**:
  - `entry (en:)`: al entrar al estado.
  - `during (du:)`: en cada paso mientras el estado está activo.
  - `exit (ex:)`: al salir.
- **Transiciones**: se etiquetan con la sintaxis `evento [condición] {acción de condición} / acción de transición`.
- **Transición por defecto**: indica el estado inicial.
- **Uniones (junctions)**: conectivas para decisiones y bifurcaciones (equivalente a diagramas de flujo), y **unión de historia** (`H`), que recuerda el último subestado activo.
- **Eventos y datos**: entradas y salidas del chart, datos locales y eventos de entrada/locales.
- **Lógica temporal**: operadores como `after(5, sec)`, `before`, `at` y `every` permiten expresar temporizaciones directamente en la transición (por ejemplo, un timeout).
- **Funciones**: gráficas, de MATLAB, de Simulink y **tablas de verdad** (Truth Table) para lógica combinatoria.

### 3.3 Jerarquía y descomposición
- **Descomposición exclusiva (OR)**: dentro de un estado padre, solo **un** subestado está activo a la vez.
- **Descomposición paralela (AND)**: **varios** subestados están activos al mismo tiempo (útil para funciones independientes, como "gestión de errores" en paralelo con "operación normal").
- La jerarquía permite estructurar la complejidad y reutilizar comportamiento común.

### 3.4 Semántica de ejecución (lo que más se pregunta)
- Un chart se ejecuta cuando recibe un **evento** (periódico, por flanco, por llamada de función).
- **Orden de evaluación de transiciones**: cuando salen varias transiciones de un estado, el orden afecta el resultado. Se debe fijar de forma **explícita** y no depender de la geometría del dibujo.
- **Nondeterminismo accidental**: transiciones con condiciones que se solapan, o dependencias en el orden de ejecución de estados paralelos, generan comportamientos difíciles de reproducir.
- **Estados sin salida**, transiciones inalcanzables y condiciones que nunca se cumplen son defectos típicos que las herramientas de análisis pueden detectar.
- **Tipos de máquina**: *Classic*, *Mealy* y *Moore* (diferencian si las salidas dependen del estado, de las entradas o de ambos).

### 3.5 Ejemplo conceptual: gestión de sesión de diagnóstico (UDS)
Estados: `Default`, `Extended`, `Programming`. Eventos: **E1** = solicitud `0x10 01`, **E2** = `0x10 02`, **E3** = `0x10 03`, **E4** = vencimiento del temporizador de sesión (S3, por ejemplo `after(5, sec)` sin actividad).
Transiciones válidas del modelo (simplificado):

| Desde | Evento | Hacia |
|---|---|---|
| Default | E3 | Extended |
| Extended | E1 | Default |
| Extended | E4 | Default |
| Extended | E2 | Programming |
| Programming | E1 | Default |
| Programming | E4 | Default |

La tabla de estados completa tiene 3 × 4 = 12 celdas: **6 transiciones válidas** y **6 inválidas** (por ejemplo, `Default` + E2 debe responder con el NRC correspondiente y no cambiar de estado). Esto se conecta de forma directa con la **prueba de transición de estado** y con lo que ya viste de las sesiones UDS.

## 4. Modelado de ECUs y plantas para HIL

Esta es la parte de MBD más cercana a tu experiencia en Ford ("desarrollo de modelos de ECU en Simulink a partir de especificaciones técnicas y datos de CAN, integración y validación con equipos de software y hardware").

### 4.1 Qué se modela y por qué
| Elemento | Qué representa | Para qué |
|---|---|---|
| **Modelo de planta (plant)** | Dinámica del vehículo, sensores, actuadores, entorno | Alimentar a la ECU real con el comportamiento físico del entorno |
| **Modelo de ECU "virtual"** | Comportamiento de una ECU que no está físicamente | Completar la red en un banco HIL cuando faltan ECUs reales |
| **Simulación de bus restante (rest-bus simulation)** | Mensajes y señales que las demás ECUs enviarían, definidos por la matriz de comunicación (DBC/ARXML) | Que la ECU bajo prueba "vea" una red completa y coherente |
| **Modelo de falla** | Inyección de anomalías (señal fuera de rango, retardo, congelamiento) | Pruebas de robustez y de *fault management*, además de la inyección eléctrica del hardware |

### 4.2 De especificación a modelo
Un flujo típico para un modelo de ECU a partir de documentación y datos de bus:
1. Analizar la **especificación técnica** y la matriz de comunicación (señales, periodos, timeouts, valores por defecto).
2. Identificar **entradas, salidas y estados** de la ECU simulada.
3. Modelar la **lógica** (Stateflow para modos y secuencias, Simulink para cálculos), con los **tiempos de ciclo** definidos.
4. Implementar el **comportamiento de comunicación**: periodicidad de mensajes, contadores y *checksums* (como en los ejemplos de CAPL), reacciones a ausencia de mensajes.
5. **Validar contra mediciones reales**: comparar las salidas del modelo con trazas capturadas de la ECU o del vehículo (correlación).
6. **Parametrizar** y documentar supuestos y limitaciones.
7. **Integrar** en el banco HIL y comprobar que no hay overruns ni desviaciones de tiempo.

### 4.2.1 Fidelidad del modelo
Un modelo no necesita ser perfecto, necesita ser **suficiente para el objetivo de prueba**. Un modelo muy detallado cuesta tiempo de cómputo y mantenimiento; uno demasiado simple puede ocultar defectos o producir falsos positivos. Incluye, según el caso, **ruido, retardos, cuantización y saturaciones** de los sensores reales. Una buena práctica: **validar el modelo** contra datos reales y registrar el rango de validez.

### 4.3 Ejecución en tiempo real
- El modelo se compila para un **objetivo de tiempo real** (hardware de simulación de proveedores como dSPACE, Speedgoat, ETAS u otros; el nombre depende del proyecto y del banco).
- Se vincula a las **interfaces de E/S** (señales eléctricas) y a las **interfaces de bus**; esa asignación (*I/O mapping*) es parte de la configuración del banco.
- Se vigila el **tiempo de ejecución por paso** frente al tamaño del paso para evitar overruns.
- Con herramientas del ecosistema Vector, los modelos pueden coexistir con CANoe y el VT System (visto en Herramientas).

## 5. Generación de código

### 5.1 Herramientas
- **Simulink Coder / Embedded Coder** (MathWorks) generan código C/C++ desde el modelo; Embedded Coder está orientado a código de producción embebido y puede producir código compatible con **AUTOSAR** (SWCs y archivos ARXML).
- **TargetLink** (dSPACE) es otra solución usada ampliamente en automotriz.
- Los nombres de herramientas varían entre organizaciones; lo importante es el principio.

### 5.2 Qué esperar del código generado
- Es **determinista y estructurado**, pero su legibilidad depende de la disciplina de modelado (nombres de señales, subsistemas atómicos, tipos de datos).
- Debe integrarse con el resto del software (BSW, drivers, SO) mediante **interfaces definidas**: puertos del modelo → variables y funciones del código.
- **Restricciones de proyecto**: reglas como MISRA C aplican al código generado (con las configuraciones adecuadas) y al código escrito a mano que lo rodea.
- **Reproducibilidad**: la versión de la herramienta, la configuración del generador y del compilador forman parte de la **línea base**.

### 5.4 Equivalencia entre modelo y código (back-to-back)
Se compara la salida del **modelo (MIL)** con la del **código (SIL, luego PIL)** para los mismos estímulos. Las diferencias aceptables se definen con **tolerancias**. Causas típicas de discrepancia: precisión numérica (`double` vs `single` vs punto fijo), orden de evaluación, conversiones de tipo, optimizaciones del compilador y comportamiento ante desbordamiento. Una discrepancia no explicada es un hallazgo a investigar: puede ser un defecto del modelo, de la generación, de la configuración o del entorno.

## 6. Verificación y validación a nivel de modelo

### 6.1 Revisión estática del modelo
- **Guías de modelado** (por ejemplo, las del *MathWorks Advisory Board*, y variantes del sector): reglas sobre nombres, estructura, tipos de datos, uso de bloques y legibilidad. Equivalen a las reglas de codificación en C.
- **Model Advisor** y herramientas similares: comprueban automáticamente reglas y detectan problemas (lazos algebraicos, conversiones inseguras, bloques no recomendados).
- **Métricas del modelo**: complejidad, profundidad de jerarquía, número de bloques y subsistemas.
- **Revisión por pares** de modelos, con los mismos tipos de revisión que viste (informal, guiada, técnica, inspección).

### 6.2 Pruebas del modelo
- **Pruebas basadas en requisitos**: casos de prueba derivados de los requisitos y vinculados a ellos (con herramientas como Simulink Test y Simulink Requirements, o equivalentes). Las técnicas de caja negra (partición de equivalencia, valores frontera, tabla de decisión, transición de estado) aplican directamente a las entradas y estados del modelo.
- **Arneses de prueba (test harness)**: estructura que aísla un componente y le aplica estímulos y verificaciones.
- **Secuencias de prueba y evaluaciones**: definen entradas en el tiempo y criterios de aceptación (aserciones sobre las salidas).
- **Cobertura del modelo**: análoga a la cobertura de código. Incluye **cobertura de decisión, de condición y MC/DC** sobre bloques de lógica y transiciones de Stateflow. Sirve para detectar partes del modelo no ejercitadas por las pruebas.
- **Verificación formal**: herramientas como Simulink Design Verifier buscan propiedades (por ejemplo, demostrar que una condición nunca ocurre, o generar casos de prueba que alcancen un objetivo de cobertura).
- **Pruebas de regresión del modelo**: se re-ejecutan automáticamente ante cualquier cambio (candidatas ideales a CI).

### 6.3 Aplicación de las técnicas de prueba a Stateflow
- **Cobertura de estados**: cada estado se visita al menos una vez (la más débil).
- **Cobertura de transiciones válidas**: cada transición definida se ejerce.
- **Cobertura de todas las transiciones** (incluye eventos que deben ser ignorados o rechazados): la más fuerte.
- Se añaden pruebas de **temporización** (¿la transición ocurre exactamente al cumplirse `after(5, sec)`?), de **prioridad entre transiciones** y de **entradas simultáneas**.

### 6.4 Modelos, ASPICE e ISO 26262
- El modelo es un **producto de trabajo** de diseño (SWE.2/SWE.3) y, si se verifica a nivel de modelo, aporta evidencia a SWE.4.
- La **trazabilidad** requisito ↔ modelo ↔ prueba debe mantenerse en las herramientas del proyecto (DOORS/RQM).
- En seguridad funcional, las **herramientas de modelado y generación de código** requieren **evaluación y, según su impacto, calificación** (concepto de calificación de herramientas visto en Ingeniería de Software). Existen **kits de certificación** de los proveedores para apoyar este proceso.
- Las **restricciones de modelado** se endurecen con el ASIL (por ejemplo, subconjuntos de bloques permitidos y reglas de estructura).

## 7. Buenas prácticas de modelado

- **Arquitectura modular y por capas**, con interfaces claras (buses con nombre) y separación entre lógica de control, acceso a E/S y calibraciones.
- **Una responsabilidad por subsistema**, jerarquía razonable y nombres descriptivos con unidades.
- **Parámetros fuera de los bloques**: usar diccionario de datos o archivos de parámetros, nunca números mágicos dentro del diagrama.
- **Tipos de datos explícitos** y control de rangos, saturaciones y desbordamientos.
- **Inicialización definida** de todos los estados y salidas (valores iniciales conocidos).
- **Evitar lazos algebraicos** (si es necesario, romperlos con retardos de forma consciente), **Goto/From excesivos** y líneas cruzadas ilegibles.
- **Control explícito del orden de ejecución** en Stateflow y de las tasas de muestreo en Simulink.
- **Reutilización**: bibliotecas y referencia a modelos.
- **Control de versiones** de modelos y de su configuración (los archivos de modelo son binarios o XML; conviene usar herramientas de comparación y *merge* específicas).
- **Automatizar** comprobaciones (guías de modelado, pruebas de regresión, cobertura) en el pipeline.
- **Documentar supuestos**: rango de validez del modelo, fuentes de datos de calibración y limitaciones.

## 8. Del síntoma a la causa probable (nivel modelo/HIL)

| Síntoma | Hipótesis a considerar |
|---|---|
| Diferencias entre MIL y SIL | Tipos de datos (`double` vs punto fijo), conversión implícita, orden de ejecución, desbordamiento, solver o paso distinto |
| Overrun en el simulador de HIL | Modelo demasiado costoso para el paso elegido, solver o paso inadecuado, falta de particionado entre núcleos |
| Comportamiento distinto cada vez que se simula | Estados sin inicializar, dependencia en el orden de transiciones, semillas aleatorias no fijadas |
| La ECU real reacciona distinto que con el modelo de planta | Fidelidad insuficiente, parámetros mal calibrados, retardos o ruido de sensores no modelados |
| Un mensaje del bus simulado no llega con el periodo esperado | Tasa de muestreo del modelo, `Rate Transition` mal configurado, asignación de tareas |
| Valores de señal distintos a los esperados al decodificar | Escala/offset incorrectos, orden de bytes, tipo de dato del modelo vs DBC |
| Estado atascado en un chart de Stateflow | Condición que nunca se cumple, transición con prioridad incorrecta, evento no recibido |
| Cobertura de modelo incompleta | Entradas de prueba insuficientes, código muerto del modelo, requisitos sin pruebas |

## Ejercicios de práctica
1. **Punto fijo/DBC:** una señal de temperatura de 8 bits sin signo tiene factor 0.5 y offset −40. ¿Qué valor físico representa el crudo 130? ¿Cuál es el rango físico?
   *Respuesta:* 130 × 0.5 − 40 = **25 °C**. Rango: 0 × 0.5 − 40 = −40 °C hasta 255 × 0.5 − 40 = **87.5 °C**.
2. **Resolución:** un tipo `sfix16_En8`, ¿cuál es su resolución y rango? *(1/256 ≈ 0.0039; de −128 a ≈ 127.996)*.
3. **Paso fijo y overrun:** un modelo se ejecuta con paso de 1 ms y su cómputo tarda 0.8 ms por paso. ¿Cuál es la utilización y qué pasa si tras una modificación sube a 1.2 ms?
   *Respuesta:* 80%; con 1.2 ms excede el paso (**overrun**) y la simulación en tiempo real deja de ser válida.
4. **Elección del paso:** la dinámica más rápida relevante del modelo es de 100 Hz. ¿Cuál es el paso máximo que permite el criterio de Nyquist y cuál sería una elección práctica?
   *Respuesta:* muestreo mínimo > 200 Hz (paso < 5 ms); en la práctica se elige bastante menor (por ejemplo, 0.5–1 ms).
5. **Stateflow:** con el modelo de sesión de la sección 3.5, ¿cuántas transiciones válidas hay y cuántas celdas inválidas en la tabla de estados? ¿Cuántos casos de prueba mínimos necesitas para cobertura de transiciones válidas?
   *Respuesta:* 6 válidas y 6 inválidas. Un único caso largo que encadene las 6 transiciones cubre el 100%, pero para facilitar el diagnóstico conviene casos cortos (por ejemplo, 3 o 4 secuencias). La cobertura de **todas** las transiciones requiere ejercer además las 6 inválidas.
6. **Diseño:** modela en Stateflow un monitor de "timeout de mensaje" que pasa a `Fallo` si no llega el mensaje durante 300 ms y vuelve a `OK` cuando llega. Describe estados, eventos, `after(...)` y qué casos de prueba incluirías.
7. **Back-to-back:** una salida difiere en 0.002 entre MIL y SIL. ¿Cuándo es aceptable y qué investigarías si la tolerancia definida es 0.001?

## Preguntas de teoría
1. ¿Qué es el Diseño Basado en Modelos y qué ventajas y riesgos tiene frente al desarrollo manual de código?
2. Describe el flujo MBD desde el requisito hasta HIL y qué se valida en cada etapa.
3. ¿Qué diferencia hay entre un solver de paso fijo y uno de paso variable, y por qué HIL exige paso fijo?
4. ¿Qué es el tiempo de muestreo y cómo se gestionan sistemas multirate en Simulink?
5. ¿Para qué sirve un bloque `Rate Transition`?
6. Explica el punto fijo y su relación con factor y offset en un DBC.
7. ¿Qué diferencia hay entre saturación y *wrap-around* y cuál preferirías en software de seguridad?
8. ¿Qué ventajas tiene un bus (*bus object*) frente a `Mux/Demux`?
9. ¿Para qué sirve el diccionario de datos y por qué evita números mágicos?
10. Describe las acciones `entry`, `during` y `exit` de un estado en Stateflow.
11. ¿Cómo se escribe la etiqueta de una transición en Stateflow y qué papel juega `after(n, sec)`?
12. Compara la descomposición exclusiva (OR) y la paralela (AND).
13. ¿Qué riesgos trae depender del orden geométrico de evaluación de transiciones?
14. ¿Qué es un lazo algebraico y cómo se evita?
15. ¿Qué es la simulación del bus restante (rest-bus) y cómo se construye desde un DBC?
16. ¿Qué factores determinan la fidelidad necesaria de un modelo de planta para HIL?
17. ¿Cómo validarías un modelo de ECU contra datos reales capturados?
18. Explica qué es la prueba *back-to-back* y qué causas explican diferencias entre MIL y SIL.
19. ¿Qué mide la cobertura de modelo y cómo se relaciona con MC/DC del código?
20. ¿Qué herramientas de verificación existen a nivel de modelo (guías, análisis estático, verificación formal)?
21. ¿Por qué las herramientas de modelado y generación de código pueden requerir calificación en proyectos de seguridad?
22. ¿Cómo aplicarías la prueba de transición de estado a un chart de Stateflow? Menciona los tres criterios de cobertura.
23. Describe cómo trazarías requisitos hacia bloques del modelo y hacia casos de prueba.
24. ¿Qué cambios en un modelo disparan una regresión y cómo la automatizarías en CI?

## Errores comunes
- Pensar que "si funciona en MIL, funcionará igual en la ECU": las diferencias de tipos, precisión, tiempo y entorno existen y deben verificarse con SIL, PIL y HIL.
- Usar `double` en todo el modelo y sorprenderse cuando el código en punto fijo se comporta distinto.
- Elegir un paso de simulación sin analizar la dinámica más rápida ni el tiempo de cómputo, y terminar con overruns en HIL.
- Esconder constantes dentro de bloques en vez de usar un diccionario de datos o parámetros con nombre.
- No inicializar estados y salidas, lo que genera comportamiento distinto entre simulaciones.
- Depender del orden geométrico de transiciones o de ejecución implícita de estados paralelos en Stateflow.
- Abusar de `Goto/From`, de subsistemas gigantes y de jerarquías profundas que vuelven el modelo ilegible.
- Confundir la planta (entorno físico simulado) con el controlador (lógica de la ECU).
- Aceptar una discrepancia MIL/SIL sin explicarla "porque la diferencia es pequeña".
- Medir solo cobertura de código y olvidar la cobertura del modelo (o al revés), y creer que alta cobertura implica pruebas buenas.
- Dar por hecho que el modelo de planta es perfecto: tiene un rango de validez que debe documentarse.
- Tratar los modelos como archivos sin control: no versionarlos, no revisarlos y no comparar cambios.
- Ignorar la calificación de herramientas y las restricciones de modelado en proyectos de seguridad funcional.

## Puntos clave

Repaso rápido antes de una entrevista — un bullet por idea, sin reexplicar. (Para cálculos y mapeo síntoma→causa, usa también los bloques "Ejercicios de práctica" y "Del síntoma a la causa probable" de arriba — no se repiten aquí.)

### 1. Qué es el Diseño Basado en Modelos
- Flujo: Requisitos → Modelo (Simulink/Stateflow) → MIL → generación de código → SIL → PIL → ECU → HIL → Vehículo.
- Ventajas: validar lógica sin hardware, modelo como especificación ejecutable, trazabilidad requisito↔bloque, pruebas back-to-back entre niveles.
- Riesgos: curva de aprendizaje del modelado, código generado menos legible, calificación de herramientas en seguridad, "validar el modelo y olvidar el software real".

### 2. Simulink: fundamentos
- Subsistemas virtuales (solo organización visual) vs atómicos (se ejecutan como unidad, se reflejan como función en el código generado).
- Paso fijo = generación de código y HIL (ritmo constante); paso variable = simulación offline.
- Punto fijo = entero × escala + offset — el mismo concepto que factor/offset de una señal en un DBC.
- Saturación (se queda en el límite) vs *wrap-around* (se desborda) — en software de seguridad se prefiere la saturación explícita.
- Los *bus objects* agrupan señales con nombre (como un `struct`) — preferibles a `Mux`/`Demux`.

### 3. Stateflow
- Acciones de un estado: `entry`/`during`/`exit`; una transición se etiqueta `evento [condición] {acción} / acción`.
- Descomposición exclusiva (OR, un solo subestado activo) vs paralela (AND, varios activos a la vez).
- El orden de evaluación de transiciones debe fijarse **explícitamente**, no depender de la geometría del dibujo — fuente de nondeterminismo accidental.
- Ejemplo de sesiones UDS: 3 estados × 4 eventos = 12 celdas → 6 transiciones válidas, 6 inválidas.

### 4. Modelado de ECUs y plantas para HIL
- Se modela: planta (dinámica física), ECU virtual (completa la red si falta hardware real), rest-bus simulation (desde DBC/ARXML), modelo de falla.
- La fidelidad debe ser "suficiente para el objetivo de prueba", no perfecta — se valida contra mediciones reales y se documenta el rango de validez.
- Se ejecuta en un objetivo de tiempo real (dSPACE/Speedgoat/ETAS) vigilando el tiempo de ejecución por paso para evitar overruns.

### 5. Generación de código
- Embedded Coder/TargetLink generan C/C++ desde el modelo, con soporte para código compatible con AUTOSAR (SWCs/ARXML).
- MISRA C aplica también al código generado; la versión de herramienta, generador y compilador forma parte de la línea base.
- Back-to-back: comparar MIL vs SIL/PIL con tolerancias definidas — causas típicas de discrepancia: precisión numérica, orden de evaluación, conversiones de tipo, desbordamiento.

### 6. Verificación y validación a nivel de modelo
- Model Advisor detecta lazos algebraicos, conversiones inseguras y bloques no recomendados — es el equivalente del análisis estático de código.
- La cobertura de modelo (decisión, condición, MC/DC) es análoga a la cobertura de código; Simulink Design Verifier hace verificación formal.
- El modelo es un producto de trabajo de SWE.2/SWE.3 (y evidencia para SWE.4 si se verifica a nivel de modelo); las herramientas de modelado/generación requieren calificación según el ASIL.

### 7. Buenas prácticas de modelado
- Arquitectura modular, parámetros fuera de los bloques (diccionario de datos, no números mágicos), tipos de datos explícitos, inicialización definida de estados/salidas.
- Evitar lazos algebraicos, `Goto`/`From` excesivos y jerarquías profundas; controlar explícitamente el orden de ejecución y las tasas de muestreo.
- Los archivos de modelo requieren control de versiones y herramientas de comparación/*merge* específicas.

### 8. Síntoma → causa (nivel modelo/HIL)
- La tabla de este capítulo (discrepancia MIL/SIL, overrun, comportamiento no determinista, chart atascado, cobertura incompleta) es el puente entre un síntoma de simulación y su causa en el modelo.