# Capítulo: Ingeniería de Sistemas

> Objetivo del capítulo: cubrir la ingeniería de sistemas **tal como se practica en un programa automotriz**, con foco en lo que un Test Function Lead o Test Manager necesita dominar: arquitectura, interfaces, decisiones de diseño, integración y verificación a nivel sistema, seguridad funcional y gestión de variantes.
>
> **Límites con otros capítulos (para no repetir):**
> - Sistema, ciclo de vida, dominio del problema vs solución, Greenfield/Brownfield → ya en *Requirements Engineering* (sección 0).
> - Procesos SYS.1–SYS.5 y niveles de capacidad → ya en *ASPICE/AUTOSAR*.
> - Principios de diseño de software, MISRA, code review, ágil, configuración de software → irán en *Ingeniería de Software*.
> - Aquí: arquitectura de sistema, interfaces, trade studies, MBSE, V/W-Model, integración y verificación de sistema, hitos, análisis de seguridad y variantes.

---

## 1. La ingeniería de sistemas en la práctica

El **Systems Engineer** es quien garantiza que el sistema completo (hardware, software, mecánica, comunicaciones) cumpla las necesidades de los stakeholders. Sus actividades diarias típicas son: derivar requisitos de sistema, definir la arquitectura, asignar requisitos a subsistemas, controlar interfaces, planear la integración y la verificación, y gestionar los compromisos entre calidad, costo, tiempo y riesgo.

*Conexión con tu rol*: como probador de sistema en HIL, eres el principal "cliente" del trabajo del Systems Engineer. Los requisitos de sistema que verificas y las interfaces que ejercitas vienen de aquí. Entender cómo se produjeron te permite distinguir en un defecto si el problema está en el software, en una interfaz mal definida o en un requisito de sistema incompleto.

## 2. Procesos técnicos de la ingeniería de sistemas

El estándar **ISO/IEC/IEEE 15288** (y la guía INCOSE, que ya viste como base del curso de requisitos) agrupa los procesos técnicos así:

| Proceso | Qué produce | Dónde lo has visto |
|---|---|---|
| Análisis de misión/negocio | Definición del problema u oportunidad | RE, sección 0 |
| Definición de necesidades y requisitos de stakeholder | Requisitos de stakeholder | RE, sección 2 |
| Definición de requisitos de sistema | Requisitos de sistema | RE, sección 3 / ASPICE SYS.2 |
| Definición de arquitectura | Arquitectura de sistema | Sección 3 de este capítulo / SYS.3 |
| Definición de diseño | Diseño de elementos (HW, SW) | — |
| Análisis de sistema | Trade studies, simulaciones, análisis de desempeño | Secciones 5 y 11 |
| Implementación | Elementos construidos | — |
| Integración | Sistema ensamblado | Sección 7 / SYS.4 |
| Verificación | Evidencia de que cumple los requisitos | Sección 7 / SYS.5 |
| Transición | Entrega a producción/operación | Sección 8 |
| Validación | Evidencia de que satisface las necesidades | RE, sección 7 |
| Operación, mantenimiento, retiro | Soporte en campo | Fundamentos de Testing (prueba de mantenimiento) |

Estos procesos no son una secuencia rígida. Se solapan e iteran, y se aplican recursivamente a cada nivel (sistema, subsistema, componente).

## 3. Arquitectura de sistema y asignación de requisitos

### 3.1 Funcional vs física (lógica vs implementación)
- **Arquitectura funcional/lógica**: describe *qué funciones* realiza el sistema y cómo fluyen los datos entre ellas, sin decidir todavía qué componente físico las ejecuta. Ejemplo ADAS: "percibir el entorno → fusionar datos → decidir maniobra → actuar sobre el freno".
- **Arquitectura física**: asigna esas funciones a elementos concretos (sensores, ECUs, actuadores, buses).
- **Asignación (allocation)**: el acto de repartir funciones y requisitos de sistema a los elementos de la arquitectura física. De aquí surgen los **requisitos derivados**: requisitos nuevos que no estaban en el nivel superior pero son necesarios por la decisión de diseño (por ejemplo, "el bus CAN que une el radar y la ECU debe soportar X mensajes por segundo").

### 3.2 Descomposición y propiedades deseables
Una buena arquitectura de sistema busca **alta cohesión** (cada elemento tiene una responsabilidad clara), **bajo acoplamiento** (pocas dependencias entre elementos), interfaces simples y bien definidas, y margen de evolución.

### 3.3 Presupuestos de diseño (design budgets)
Los recursos críticos se reparten entre elementos como un "presupuesto", y la suma debe respetar el límite del sistema. Los más comunes:
- **Presupuesto de latencia/timing**: si un requisito de sistema exige "frenar en menos de 150 ms desde la detección", ese tiempo se reparte entre sensor, procesamiento, comunicación por bus y actuador. Si un elemento excede su parte, el sistema incumple aunque cada elemento "funcione".
- **Presupuesto de carga de bus**: porcentaje máximo de utilización de CAN/Ethernet (conecta con Protocolos de Comunicación).
- **Presupuesto de memoria y CPU, de consumo de energía y de masa/costo.**

Esto es muy relevante para pruebas: muchos requisitos no funcionales que verificas en HIL (latencias, tiempos de respuesta) son la verificación de un presupuesto asignado.

### 3.4 Arquitecturas E/E automotrices (evolución)
Un tema muy preguntado hoy en entrevistas:
- **Distribuida (clásica)**: muchas ECUs, cada una con una función específica, interconectadas por CAN/LIN, con un gateway central. Es el modelo típico de AUTOSAR Classic.
- **Por dominios**: las funciones se agrupan en dominios (powertrain, chasis, carrocería, ADAS, infotainment) con un controlador de dominio por área.
- **Zonal + cómputo central**: las ECUs se agrupan por **zona física** del vehículo (reduce cableado y peso), y funciones de alto desempeño se concentran en uno o pocos computadores centrales conectados por backbone **Ethernet**. Es el contexto natural de AUTOSAR Adaptive, SOME/IP y TSN.
- **Impacto en prueba**: menos ECUs pero más funciones por ECU, más software compartido y más dependencias entre funciones, lo que aumenta la importancia de la integración virtual y de las pruebas de interferencia entre funciones.

## 4. Gestión de interfaces

Las interfaces son la causa más frecuente de defectos de integración. Un buen proceso las trata como un elemento de primera clase.

### 4.1 Tipos de interfaz
- **Mecánica/física**: conectores, pinout, montaje.
- **Eléctrica**: niveles de voltaje, corriente, impedancia, alimentación.
- **De datos/comunicación**: señales, mensajes, protocolos, timing (CAN, LIN, Ethernet, UDS, XCP).
- **De software**: APIs, puertos AUTOSAR (Sender-Receiver, Client-Server).
- **Térmica y ambiental**, y **humano-máquina** (HMI).

### 4.2 Qué debe definir una interfaz
Qué se intercambia, en qué formato y unidades, con qué timing (periodicidad, latencias máximas), qué pasa ante errores o ausencia de datos (valores por defecto, timeouts, modos degradados), y quién es dueño de cada lado.

### 4.3 Artefactos
- **ICD (Interface Control Document)**: especificación formal de una interfaz entre dos elementos o sistemas, bajo control de cambios.
- **Matriz de comunicación (DBC/ARXML)**: la implementación de las interfaces de datos entre ECUs.
- **Diagrama N²**: matriz que muestra, para cada par de elementos, qué interfaz existe entre ellos. Sirve para detectar dependencias y huecos.
- **Propiedad y control de cambios**: un cambio en una interfaz afecta a ambos lados, por lo que requiere análisis de impacto y comunicación a todos los equipos (y a proveedores).

### 4.4 Defectos típicos de interfaz (para tus pruebas)
Unidades o escalas distintas, endianness, rangos o valores inválidos no definidos, timing no especificado, comportamiento ante ausencia de mensaje no definido, versiones de DBC/ICD desincronizadas entre emisor y receptor. Muchos fallos que ves en SYS.4 (integración de sistema) se resuelven corrigiendo la especificación de la interfaz, no el código.

## 5. Estudios de compromiso (Trade Studies)

Un **trade study** es un método estructurado para elegir entre alternativas de diseño comparándolas contra criterios definidos.

**Pasos:**
1. Definir el problema y las alternativas viables.
2. Definir **criterios** (costo, desempeño, riesgo, madurez tecnológica, tiempo de desarrollo, mantenibilidad, seguridad).
3. Asignar **pesos** a los criterios según prioridades de los stakeholders.
4. Calificar cada alternativa por criterio.
5. Calcular puntaje ponderado y hacer **análisis de sensibilidad** (¿cambia la decisión si cambian los pesos?).
6. Documentar la decisión y sus supuestos.

**Ejemplo ilustrativo** (los números son inventados, solo muestran el método): ¿CAN FD o Automotive Ethernet para un nuevo enlace de sensores?

| Criterio | Peso | CAN FD (1–5) | Ethernet (1–5) |
|---|---|---|---|
| Ancho de banda | 30% | 2 | 5 |
| Costo por nodo | 25% | 5 | 3 |
| Madurez en el equipo | 20% | 5 | 3 |
| Determinismo nativo | 15% | 5 | 3 (requiere TSN) |
| Escalabilidad futura | 10% | 2 | 5 |
| **Total ponderado** | | **3.85** | **3.70** |

Con estos números ganaría CAN FD por poco, pero si el peso del ancho de banda sube, la decisión se invierte. Eso es justamente lo que revela el análisis de sensibilidad, y es lo que un entrevistador espera que menciones.

## 6. MBSE y SysML

**MBSE (Model-Based Systems Engineering)** usa **modelos** como fuente principal de información del sistema, en lugar de documentos de texto dispersos.

**SysML** es el lenguaje más usado. Sus diagramas se agrupan en:
- **Requisitos**: diagrama de requisitos (relaciones *derive*, *satisfy*, *verify*).
- **Estructura**: diagrama de definición de bloques (BDD) e interno de bloques (IBD, muestra interfaces y flujos entre partes).
- **Comportamiento**: actividad, secuencia, máquina de estados, casos de uso.
- **Paramétrico**: restricciones matemáticas (por ejemplo, para presupuestos de latencia).

**Herramientas comunes**: Cameo Systems Modeler, Enterprise Architect, Capella (método Arcadia), entre otras.

**Ventajas**: trazabilidad nativa entre requisitos, arquitectura y verificación; análisis de impacto más confiable; consistencia entre vistas; reutilización.
**Retos**: curva de aprendizaje, costo de adopción y la necesidad de mantener el modelo actualizado.

*Relación con Diseño Basado en Modelos*: MBSE modela la **arquitectura y los requisitos del sistema** (el "qué" y "cómo se organiza"); Simulink/Stateflow modela el **comportamiento ejecutable** de una función o ECU (que luego se simula y genera código). Se complementan y a veces se vinculan.

## 7. V-Model, W-Model e integración y verificación de sistema

### 7.1 V-Model (resumen y relación con SYS)
- Lado izquierdo: descomposición (stakeholder → sistema → arquitectura → elementos).
- Lado derecho: integración y verificación ascendente, cada nivel contra su contraparte (SYS.4 integración contra arquitectura, SYS.5 calificación contra requisitos de sistema, validación contra necesidades del stakeholder).
- Limitaciones y mitigaciones: rigidez ante el cambio (se mitiga con V repetida e incrementos), prueba dinámica tardía (se mitiga con MIL/SIL y virtualización), validación tardía (prototipado y revisiones tempranas con stakeholders).

### 7.2 W-Model
Extiende la V con una segunda "V" de **actividades de verificación/prueba paralelas** al desarrollo: revisión de requisitos, de arquitectura y de diseño, además de diseño temprano de pruebas. Es la formalización de shift-left en el ciclo de vida.

### 7.3 Estrategias de integración
- **Bottom-up (ascendente)**: se integran primero los elementos de bajo nivel y se sube. Requiere *drivers* (simuladores de los niveles superiores).
- **Top-down (descendente)**: se empieza por lo alto con *stubs* para lo que falta.
- **Big-bang**: todo junto al final. Rápido de planear, muy difícil de depurar, y es la estrategia de mayor riesgo.
- **Sandwich/híbrida**: combina ambas.
- **Incremental por funcionalidad**: se integra función por función, lo más común hoy.

En sistemas con hardware, la estrategia está condicionada por la **disponibilidad de muestras**. Aquí HIL actúa como "stub inteligente": simula el resto del sistema mientras no existe físicamente (Component → Subsystem → Full-Vehicle HIL).

### 7.4 Integración virtual
Se integran y prueban elementos en simulación (MIL/SIL/virtual ECUs) **antes** del hardware, para detectar defectos de interfaz y arquitectura antes de los hitos físicos. Reduce riesgo y tiempo, pero no sustituye a HIL ni a pruebas en vehículo, por las limitaciones de fidelidad que ya revisaste.

### 7.5 Verificación a nivel sistema
- Cada requisito de sistema debe tener un **método de verificación** (Inspección, Análisis, Demostración o Prueba) y un **nivel/entorno** (SIL, HIL, vehículo, banco).
- La **matriz de verificación** (requisito → método → nivel → caso de prueba → resultado) es el artefacto que usa un assessor ASPICE o un auditor de seguridad para comprobar la cobertura.
- Criterios de cierre: cobertura de requisitos, defectos abiertos por severidad, evidencia completa y trazable.

## 8. Hitos del programa: A/B/C/D-Sample y SOP

Complementa lo visto en el capítulo anterior, ahora desde la perspectiva de sistemas. Los nombres exactos y los criterios varían entre OEMs; la lógica de madurez creciente es común.

| Hito | Madurez típica del sistema | Foco de ingeniería de sistemas | Foco de verificación |
|---|---|---|---|
| **A-Sample** | Prototipo funcional, concepto | Validar arquitectura y viabilidad | Pruebas de concepto, integración virtual, primeras pruebas HIL |
| **B-Sample** | Funcionalidad completa, diseño casi final | Congelar interfaces, cerrar requisitos | Verificación extensa en HIL, regresión sistemática |
| **C-Sample** | Hardware de diseño final, procesos de producción | Cerrar defectos, validar robustez | Calificación de sistema completa, pruebas ambientales, vehículo |
| **D-Sample / PPAP** | Producto de producción | Evidencia final de conformidad | Evidencias para aprobación del OEM |
| **SOP** | Producción en serie | Transición y soporte | Seguimiento de campo |

## 9. Análisis de seguridad y confiabilidad

Este bloque es especialmente importante en entrevistas de ADAS y funciones críticas. Se presenta a nivel conceptual; el detalle normativo está en ISO 26262.

### 9.1 Ciclo de vida de seguridad funcional (ISO 26262)
Parte del *item definition*, pasa por el análisis de riesgos, define objetivos de seguridad y requisitos de seguridad a nivel funcional, técnico, hardware y software, y se cierra con verificación, validación y evidencia en un *safety case*. Sigue la estructura del V-Model.

### 9.2 HARA (Hazard Analysis and Risk Assessment)
Identifica situaciones peligrosas de una función y clasifica cada riesgo con tres parámetros:
- **Severidad (S0–S3)**: gravedad del daño.
- **Exposición (E0–E4)**: probabilidad de encontrarse en la situación peligrosa.
- **Controlabilidad (C0–C3)**: capacidad del conductor u otros de evitar el daño.

La combinación determina el nivel **ASIL (A, B, C o D; D es el más exigente)** o **QM** (sin requisitos de seguridad especiales, solo gestión de calidad). De cada riesgo relevante se deriva un **objetivo de seguridad (safety goal)**, que se descompone en requisitos de seguridad (ya viste la descomposición ASIL en RE).

### 9.3 Conceptos clave de seguridad
- **Estado seguro (safe state)**: condición del sistema en la que el riesgo es aceptable (por ejemplo, desactivar la función y avisar al conductor).
- **FTTI (Fault Tolerant Time Interval)**: tiempo máximo entre la aparición de una falla y el momento en que debe alcanzarse el estado seguro, antes de que pueda ocurrir el evento peligroso. Es un requisito de timing que se verifica explícitamente con **fault injection en HIL** (conecta con la sección de fault injection del capítulo de HIL).
- **Mecanismos de seguridad**: detección y reacción ante fallas (monitores, redundancia, watchdogs, CRC y contadores en comunicación, comprobaciones de plausibilidad).

### 9.4 Técnicas de análisis
| Técnica | Enfoque | Para qué se usa |
|---|---|---|
| **FMEA / DFMEA** | Inductivo (de la falla al efecto) | Identificar modos de falla, efectos y su criticidad. En el método AIAG-VDA actual se usa **Action Priority (AP)** en lugar del RPN tradicional (severidad × ocurrencia × detección) |
| **FTA (Fault Tree Analysis)** | Deductivo (del evento no deseado hacia las causas), con compuertas AND/OR | Encontrar combinaciones de fallas que llevan a un evento peligroso |
| **FMEDA** | Cuantitativo, a nivel hardware | Calcular métricas de hardware (por ejemplo SPFM, LFM, PMHF) a partir de modos de falla y cobertura de diagnóstico |
| **Análisis de dependencia (DFA)** | Busca fallas de causa común | Validar independencia entre elementos redundantes |

### 9.5 SOTIF (ISO 21448)
Cubre riesgos que ocurren **sin que haya una falla**, por limitaciones de desempeño o por uso razonablemente previsible (por ejemplo, un radar que no distingue bien un objeto en cierta condición). Es muy relevante en ADAS y conducción automatizada y exige escenarios de prueba más amplios que las pruebas funcionales tradicionales.

### 9.6 Implicaciones para pruebas
Cada requisito de seguridad necesita verificación con evidencia trazable. La **inyección de fallas** valida los mecanismos de seguridad y el FTTI. La **independencia de la prueba** aumenta con el ASIL (concepto que viste en Fundamentos de Testing), y los resultados alimentan el *safety case*.

## 10. Gestión de variantes y líneas de producto

Un mismo producto base suele existir en muchas **variantes**: por mercado, nivel de equipamiento, motorización, hardware alternativo o cliente.

- **Ingeniería de líneas de producto (PLE)**: desarrollar una plataforma común reutilizable y gestionar sistemáticamente lo que varía.
- **Modelo de características (feature model)**: representa qué características existen y sus dependencias (obligatorias, opcionales, excluyentes).
- **Mecanismos de variación en ECU**: calibración/parametrización (conecta con XCP y A2L), codificación de variantes, configuración por software, hardware alternativo.
- **Gestión de configuración de variantes**: cada variante necesita su propia línea base de requisitos, DBC/A2L y casos de prueba aplicables.

**Impacto en pruebas**: el número de combinaciones crece de forma explosiva (ya conoces este problema por el principio de que la prueba exhaustiva es imposible). Estrategias típicas: priorizar por riesgo y volumen de mercado, probar a fondo una variante de referencia y las diferencias en las demás, y usar **pruebas combinatorias (pairwise)** para cubrir interacciones de parámetros con pocos casos.

## 11. Planificación de V&V a nivel sistema y medidas técnicas de desempeño

### 11.1 Plan de V&V del sistema
Debe definir: alcance y objetivos, requisitos a verificar y método para cada uno (matriz de verificación), niveles y entornos de prueba (SIL, HIL, vehículo, banco), recursos y herramientas, criterios de entrada/salida por hito, gestión de riesgos, trazabilidad esperada y reporte. Es el equivalente de sistema del plan de prueba que viste en Fundamentos de Testing.

### 11.2 Medidas técnicas de desempeño (TPMs)
Son parámetros críticos del sistema que se **siguen a lo largo del desarrollo** para anticipar si se cumplirá el requisito final: latencia extremo a extremo, consumo de energía, masa, uso de CPU o memoria, carga de bus, tasa de falsos positivos de una función. Se miden primero en modelos y simulaciones, luego en HIL y finalmente en vehículo, y se comparan contra el presupuesto asignado. Detectar una tendencia desfavorable temprano evita sorpresas en C-Sample.

## Preguntas de teoría
1. ¿Qué es la asignación (allocation) de requisitos y por qué genera requisitos derivados? Da un ejemplo automotriz.
2. Explica la diferencia entre arquitectura funcional y física, y por qué se definen en ese orden.
3. ¿Qué es un presupuesto de latencia y cómo se relaciona con un requisito que verificas en HIL?
4. Describe la evolución de arquitecturas E/E distribuida → dominios → zonal/cómputo central, y su impacto en el esfuerzo de prueba.
5. ¿Qué debe definir una especificación de interfaz además de las señales que se intercambian?
6. ¿Qué es un diagrama N² y para qué sirve?
7. Explica por qué muchos defectos de integración de sistema son defectos de interfaz y no de código. Da tres ejemplos concretos.
8. Describe los pasos de un trade study y qué aporta el análisis de sensibilidad.
9. ¿Qué diferencia hay entre MBSE/SysML y el modelado en Simulink/Stateflow?
10. Compara las estrategias de integración bottom-up, top-down, big-bang y sandwich. ¿Cómo encaja HIL en esa estrategia?
11. ¿Qué es una matriz de verificación y qué evidencia aporta en un assessment ASPICE?
12. Explica el HARA: parámetros S/E/C y cómo se llega a un ASIL.
13. ¿Qué es el FTTI y cómo lo verificarías en un banco HIL?
14. ¿Cuál es la diferencia entre FMEA y FTA (inductivo vs deductivo)? ¿Qué reemplazó al RPN en el método AIAG-VDA?
15. ¿Qué cubre SOTIF que ISO 26262 no cubre?
16. ¿Cómo abordarías la estrategia de pruebas de un producto con 40 variantes?
17. ¿Qué son las TPMs y por qué conviene seguirlas desde etapas tempranas?

## Errores comunes
- Pensar que ingeniería de sistemas es "documentación de arquitectura" y no un proceso de decisiones, compromisos y control de interfaces.
- Verificar cada componente por separado y asumir que el sistema cumplirá un presupuesto compartido (latencia, carga de bus) sin medir el total.
- Tratar las interfaces como un detalle de implementación en vez de como artefactos bajo control de cambios con dueño definido.
- Elegir una alternativa de diseño "por costumbre" sin un trade study documentado (y sin análisis de sensibilidad).
- Confundir MBSE con "dibujar diagramas": su valor está en que el modelo sea la fuente de verdad consistente y trazable.
- Usar integración big-bang con hardware escaso: maximiza el riesgo y complica el diagnóstico.
- Confundir ASIL con una medida de probabilidad de falla: el ASIL clasifica la exigencia de seguridad derivada del riesgo (severidad, exposición y controlabilidad).
- Ignorar SOTIF en funciones ADAS: un sistema sin fallas puede aun así ser inseguro por limitaciones de desempeño.
- Subestimar la gestión de variantes: probar solo una configuración y asumir que las demás se comportan igual.
- Presentar los hitos A/B/C/D como universales: son una convención común, pero los nombres y criterios los define cada OEM.