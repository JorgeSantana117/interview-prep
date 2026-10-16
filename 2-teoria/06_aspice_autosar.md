# Capítulo: Estándares y Procesos Automotrices (ASPICE / AUTOSAR)

> Objetivo del capítulo: ambos términos están explícitamente en tu CV ("Automotive Standards & Processes: SDLC, ISTQB, ASPICE, AUTOSAR"), pero son dos cosas de naturaleza completamente distinta que conviene no mezclar: **ASPICE es un modelo de evaluación de procesos** (qué tan bien gestionas el desarrollo), mientras que **AUTOSAR es una arquitectura de software** (cómo se estructura el código dentro de la ECU). Este capítulo retoma y formaliza las menciones de SYS.2/SWE.1 que ya aparecieron en Requirements Engineering y HIL Testing.

---

## Parte 1: Automotive SPICE (ASPICE)

### 1.1 Qué es
**Automotive SPICE** (Software Process Improvement and Capability dEtermination) es un **modelo de evaluación y mejora de procesos** específico para la industria automotriz, basado en el estándar internacional **ISO/IEC 33002** (evolución de ISO/IEC 15504). No es un estándar de producto ni de software en sí — es un marco para evaluar **qué tan bien gestionada y controlada está la forma en que un proveedor desarrolla software/sistemas** para un OEM.

**Por qué existe**: los OEMs (fabricantes de vehículos) necesitan confiar en que sus proveedores (como Bosch, donde trabajas) tienen procesos de desarrollo predecibles, repetibles y de calidad — no solo que "el software funcione", sino que el **proceso** que lo produjo sea confiable. ASPICE es, en esencia, el lenguaje común que la industria usa para medir y comunicar esa confianza.

**Estructura del modelo**: ASPICE se compone de dos partes:
- **PRM (Process Reference Model)**: define **qué** procesos deben existir (ej. "debe haber un proceso de análisis de requisitos de sistema").
- **PAM (Process Assessment Model)**: define **cómo se evalúa** qué tan bien se ejecuta cada proceso, mediante **Prácticas Base (Base Practices)**, **Productos de Trabajo (Work Products)** y **Atributos de Proceso (Process Attributes)**.

### 1.2 Categorías de Procesos

ASPICE organiza los procesos en categorías. Las más relevantes para tu trabajo diario:

**Procesos Primarios del Ciclo de Vida (Primary Life Cycle Processes)**:
- **ACQ (Acquisition)**: procesos de adquisición — relevante cuando el OEM "adquiere" un sistema del proveedor.
- **SPL (Supply)**: procesos de suministro — el lado del proveedor (tu lado, trabajando para Bosch/ALTEN como parte de la cadena de suministro automotriz).
- **SYS (System Engineering)**: ingeniería de sistemas — SYS.1 a SYS.5.
- **SWE (Software Engineering)**: ingeniería de software — SWE.1 a SWE.6.

**Procesos de Soporte (Supporting Process Group - SUP)**:
- Incluye procesos como **SUP.1 (Quality Assurance)**, **SUP.8 (Configuration Management)**, **SUP.9 (Problem Resolution Management)**, **SUP.10 (Change Request Management)** — todos ellos conectan directamente con lo que ya estudiaste en Gestión de Configuración y Gestión de Defectos del capítulo de Fundamentos de Testing.

**Procesos de Gestión (Management Process Group - MAN)**:
- **MAN.3 (Project Management)**, **MAN.5 (Risk Management)** — conecta con Gestión de Riesgo que ya viste.

**Procesos de Mejora de Proceso y Reutilización (PIM, REU)**: menos relevantes día a día para un Test Engineer, pero forman parte del modelo completo.

### 1.3 Los procesos SYS y SWE en detalle (la columna vertebral del V-Model en ASPICE)

Esta es la parte más citada en cualquier entrevista automotriz — ASPICE formaliza exactamente el V-Model que ya conoces, asignando un código a cada etapa:

| Proceso | Nombre | Qué cubre | Conexión con lo que ya sabes |
|---|---|---|---|
| **SYS.1** | Requirements Elicitation | Elicitación de requisitos a nivel de sistema completo | Módulo 2 de Requirements Engineering |
| **SYS.2** | System Requirements Analysis | Análisis y estructuración de los requisitos de sistema | Ya mencionado en Requirements Engineering |
| **SYS.3** | System Architectural Design | Derivar la arquitectura del sistema (hardware + software) a partir de SYS.2 | — |
| **SYS.4** | System Integration and Integration Test | Integrar los componentes del sistema y probar esa integración | **Prueba de Integración de Sistema** de Fundamentos de Testing |
| **SYS.5** | System Qualification Test | Prueba de calificación del sistema completo contra los requisitos de SYS.2 | **Prueba de Sistema** — este es probablemente tu nivel de trabajo principal en HIL |
| **SWE.1** | Software Requirements Analysis | Análisis de requisitos específicos de software, derivados de SYS.3 | Ya mencionado en Requirements Engineering |
| **SWE.2** | Software Architectural Design | Arquitectura de software (módulos, interfaces) derivada de SWE.1 | Conecta con AUTOSAR (Parte 2 de este capítulo) — AUTOSAR es, en la práctica, una forma estandarizada de hacer SWE.2 |
| **SWE.3** | Software Detailed Design and Unit Construction | Diseño detallado y construcción de unidades de código | **Prueba de Componente** |
| **SWE.4** | Software Unit Verification | Verificación de las unidades de software (pruebas unitarias, análisis estático) | **Prueba de Componente** + Prueba Estática |
| **SWE.5** | Software Integration and Integration Test | Integrar unidades de software y probar esa integración | **Prueba de Integración de Componente** |
| **SWE.6** | Software Qualification Test | Prueba de calificación del software completo contra SWE.1 | Donde vive gran parte de tu trabajo de verificación en Bosch |

*Observación clave para entrevista*: nota el patrón **espejo** — SYS.2 (requisitos) se verifica en SYS.5 (calificación), SWE.1 (requisitos) se verifica en SWE.6 (calificación) — esto es literalmente el V-Model con nombres formales de proceso, y es exactamente lo que los assessors de ASPICE auditan: que exista **trazabilidad bidireccional** entre cada lado de la "V".

### 1.4 Niveles de Capacidad (Capability Levels)

ASPICE no evalúa solo "si existe" un proceso, sino **qué tan bien se ejecuta**, en una escala de 6 niveles (0 a 5):

| Nivel | Nombre | Qué significa |
|---|---|---|
| **0** | Incompleto (Incomplete) | El proceso no se realiza, o no logra su propósito |
| **1** | Realizado (Performed) | El proceso se ejecuta y logra su propósito básico, pero de forma no gestionada/ad-hoc |
| **2** | Gestionado (Managed) | El proceso se planifica, monitoriza y ajusta; los productos de trabajo se gestionan adecuadamente |
| **3** | Establecido (Established) | El proceso sigue un proceso estándar definido a nivel organizacional, no solo a nivel de proyecto individual |
| **4** | Predecible (Predictable) | El proceso se mide cuantitativamente y su desempeño es predecible dentro de límites definidos |
| **5** | En Optimización (Optimizing) | El proceso se mejora continuamente basándose en objetivos de negocio cuantitativos |

**La gran mayoría de los proyectos automotrices apuntan a Nivel 2 o 3** — llegar a Nivel 4/5 es raro y costoso; la mayoría de los OEMs exigen contractualmente un mínimo de Nivel 2, y muchos exigen Nivel 3 en los procesos más críticos (SYS.2, SWE.1, SWE.6).

**Atributos de Proceso (Process Attributes - PA)**: cada nivel de capacidad se evalúa mediante atributos específicos — por ejemplo, el Nivel 2 se evalúa con **PA 2.1 (Gestión del desempeño)** y **PA 2.2 (Gestión de los productos de trabajo)**. Cada atributo se califica en una escala (N - No alcanzado, P - Parcialmente alcanzado, L - Ampliamente alcanzado, F - Completamente alcanzado) — no hace falta memorizar cada PA en detalle, pero sí saber que existe esta estructura granular detrás de "nivel 2" o "nivel 3".

### 1.5 Prácticas Base y Productos de Trabajo
Cada proceso (ej. SYS.2) define:
- **Prácticas Base (Base Practices - BP)**: las actividades específicas esperadas — ej. para SYS.2: "analizar los requisitos de sistema", "estructurarlos", "analizar su impacto", "determinar su consistencia", etc.
- **Productos de Trabajo (Work Products - WP)**: la evidencia tangible de que el proceso se ejecutó — ej. para SYS.2, el producto de trabajo esperado es la "Especificación de Requisitos de Sistema" (que probablemente vive en DOORS).

Un **Assessment de ASPICE** consiste en que un assessor certificado revise evidencia real (entrevistas al equipo + revisión de productos de trabajo en las herramientas, como DOORS/RQM/Jira) para calificar cada proceso en su nivel de capacidad correspondiente.

### 1.6 Extensiones recientes: ASPICE for Cybersecurity y for Safety
El modelo base de ASPICE se ha extendido con guías específicas:
- **Automotive SPICE for Cybersecurity**: añade procesos específicos de ciberseguridad (alineados con ISO/SAE 21434), relevante dado el crecimiento de Automotive Ethernet/SOME-IP que ya estudiaste — más superficie de ataque significa más necesidad de procesos formales de ciberseguridad.
- **Automotive SPICE for Safety**: alinea los procesos de ASPICE con los requisitos de **ISO 26262** (funcional safety) — conectando con la descomposición ASIL que ya viste en Requirements Engineering.

### 1.7 Por qué ASPICE importa directamente para tu carrera
Cuando tu CV menciona experiencia con ASPICE, lo que un entrevistador quiere confirmar es que entiendes que **tu trabajo diario de testing es, literalmente, evidencia de assessment** — cada caso de prueba bien trazado en RQM, cada defecto correctamente documentado en Jira, cada revisión de requisitos que haces, es un **Work Product** que, en un assessment real, puede ser solicitado como evidencia de que SYS.5/SWE.6 se ejecuta al nivel de capacidad requerido contractualmente con el OEM.

## Preguntas de teoría — ASPICE
1. ¿Cuál es la diferencia entre el PRM y el PAM de ASPICE?
2. Explica el patrón "espejo" entre los procesos SYS/SWE de requisitos y los de prueba/calificación.
3. ¿En qué proceso(s) específico(s) de ASPICE ubicarías tu trabajo diario de HIL Testing?
4. ¿Cuál es la diferencia entre Nivel de Capacidad 1 (Realizado) y Nivel 2 (Gestionado)? Da un ejemplo concreto de qué cambiaría en la práctica.
5. ¿Por qué la mayoría de los proyectos automotrices apuntan a Nivel 2 o 3, y no a Nivel 4 o 5?
6. ¿Qué es un Work Product, y qué Work Product esperarías como evidencia de SWE.6?
7. ¿Por qué un hueco de trazabilidad (que ya estudiaste en Requirements Engineering) afectaría negativamente una evaluación de ASPICE?

## Errores comunes — ASPICE
- Confundir ASPICE con un estándar de calidad del producto de software — es un modelo de evaluación de **procesos**, no del producto en sí.
- Pensar que ASPICE es exclusivo de software — los procesos SYS (ingeniería de sistemas) son igual de centrales que los SWE.
- Asumir que "Nivel 2" significa que el proceso es deficiente — es, de hecho, el nivel objetivo típico y aceptable contractualmente en la mayoría de los proyectos.
- No poder ubicar tu propio trabajo (HIL, revisión de requisitos, gestión de defectos) dentro de procesos ASPICE específicos — es una de las preguntas más directas que puede hacer un entrevistador senior.

---

## Parte 2: AUTOSAR

### 2.1 Qué es
**AUTOSAR** (AUTomotive Open System ARchitecture) es un **estándar abierto de arquitectura de software** para sistemas embebidos automotrices, desarrollado por un consorcio de OEMs, proveedores y empresas de tecnología (incluyendo Bosch). A diferencia de ASPICE (que evalúa *procesos*), AUTOSAR define **cómo se estructura el software dentro de una ECU** — una arquitectura estandarizada para que el software de aplicación sea más portable e independiente del hardware específico, y para que distintos proveedores puedan integrar su software de forma más predecible.

**Por qué existe**: antes de AUTOSAR, cada fabricante/proveedor desarrollaba su propia arquitectura de software propietaria para cada ECU, lo que dificultaba la reutilización de código entre proyectos y encarecía la integración de software de distintos proveedores en un mismo vehículo. AUTOSAR estandariza las capas de software para que la lógica de aplicación esté **desacoplada** del hardware y de los detalles de comunicación específicos.

### 2.2 AUTOSAR Classic Platform — Arquitectura en capas

La arquitectura Classic (pensada para microcontroladores tradicionales, no para sistemas operativos de propósito general) se organiza en capas, de arriba hacia abajo:

- **Capa de Aplicación (Application Layer)**: contiene los **SWCs (Software Components)** — los bloques de lógica de negocio/funcionalidad real (ej. el algoritmo de control de un ADAS). Los SWCs están diseñados para ser **independientes del hardware** — no saben directamente sobre qué microcontrolador corren ni sobre qué bus se comunican.
- **RTE (Runtime Environment)**: la capa que **conecta** los SWCs entre sí y con el Basic Software — es generada automáticamente (a partir de la configuración ARXML) específicamente para cada ECU, actuando como "pegamento" entre la aplicación y el resto del sistema. El RTE es lo que hace posible que un SWC "no sepa" si se está comunicando con otro SWC en el mismo microcontrolador o con una ECU distinta vía CAN — esa complejidad queda oculta detrás del RTE.
- **Basic Software (BSW)**: el conjunto de módulos de software estandarizados que proveen servicios comunes, organizados a su vez en sub-capas:
  - **Services Layer**: servicios de sistema operativo (AUTOSAR OS, basado conceptualmente en OSEK), gestión de memoria (NVRAM Manager), gestión de diagnóstico (DCM - Diagnostic Communication Manager, DEM - Diagnostic Event Manager — directamente relacionado con todo lo que ya sabes de UDS y DTCs).
  - **ECU Abstraction Layer**: abstrae las diferencias específicas de la placa/ECU concreta (ej. qué pin físico corresponde a qué periférico).
  - **Microcontroller Abstraction Layer (MCAL)**: la capa más baja, abstrae directamente el hardware del microcontrolador (drivers de bajo nivel para ADC, PWM, comunicación serial, etc.).
  - **Complex Drivers**: una capa especial que permite acceso directo al hardware quando ninguna abstracción estándar es suficiente (ej. para sensores muy específicos con requisitos de timing extremos).

### 2.3 El Stack de Comunicación (muy relevante dado todo lo que ya sabes de Protocolos)

Dentro del Basic Software, el stack de comunicación es el que implementa en software real todo lo que estudiaste en el capítulo de Protocolos de Comunicación:
- **CAN Driver**: la capa más baja, interactúa directamente con el controlador CAN del microcontrolador.
- **CAN Interface (CanIf)**: abstrae el driver específico, normalizando el acceso.
- **PDU Router (PduR)**: enruta Protocol Data Units entre los distintos módulos del stack (ej. decide si un mensaje va hacia el stack de diagnóstico o hacia el stack de comunicación normal).
- **COM (Communication Module)**: maneja el empaquetado/desempaquetado de señales dentro de las tramas — es el módulo que traduce entre "señales con nombre" (las que ve la aplicación) y los bytes crudos de una trama CAN.
- **Diagnostic stack (DCM/DEM)**: implementa los servicios UDS que ya conoces a fondo — el DCM maneja la lógica de sesiones/servicios de diagnóstico, el DEM gestiona el ciclo de vida de los DTCs.

*Esto te da una pieza clave para entrevista*: cuando alguien te pregunta "¿dónde vive UDS dentro del software de una ECU AUTOSAR?", la respuesta específica es **DCM (Diagnostic Communication Manager)** para el manejo de servicios de sesión/comunicación, y **DEM (Diagnostic Event Manager)** para el manejo de eventos/DTCs — no es una respuesta genérica de "en el software", es un módulo específico y nombrado.

### 2.4 VFB (Virtual Functional Bus)
Un concepto de diseño clave de AUTOSAR: el **VFB** es una abstracción conceptual que permite diseñar la interacción entre SWCs **sin saber todavía** en qué ECU física terminará cada uno, ni sobre qué bus se comunicarán. El diseño de la arquitectura de aplicación ocurre primero a nivel VFB (puramente lógico), y **después** se hace el **mapeo** de cada SWC a una ECU física específica — este paso posterior es lo que determina si la comunicación entre dos SWCs terminará siendo una llamada de función local (si quedan en el mismo microcontrolador) o un mensaje CAN real (si quedan en ECUs distintas), todo gestionado de forma transparente por el RTE generado.

### 2.5 Puertos e Interfaces de los SWCs
- Cada SWC se comunica con el exterior únicamente a través de **puertos (Ports)**: **Provided Ports (P-Ports)** ofrecen un servicio/dato, **Required Ports (R-Ports)** consumen un servicio/dato de otro SWC.
- Los puertos se tipan mediante **interfaces**, que pueden ser de distintos tipos: **Sender-Receiver** (comunicación de datos, ej. una señal de velocidad) o **Client-Server** (invocación de un servicio con respuesta, de forma análoga a una llamada de función remota).

### 2.6 ARXML — el formato de configuración
**ARXML (AUTOSAR XML)** es el formato estándar basado en XML que describe toda la configuración de un proyecto AUTOSAR: la definición de los SWCs, sus puertos e interfaces, el mapeo de señales a tramas de comunicación, la configuración del RTE, etc. Ya mencionaste ARXML en el capítulo de Protocolos de Comunicación como una de las bases de datos de comunicación que CANoe puede importar — ahora tienes el contexto completo de **por qué** ese archivo existe y qué describe realmente.

### 2.7 AUTOSAR Adaptive Platform — la evolución para sistemas de alto desempeño

A diferencia de Classic (pensada para microcontroladores con recursos limitados y ejecución determinista basada en un RTOS tipo OSEK), **AUTOSAR Adaptive** está diseñada para los microprocesadores de alto desempeño que requieren ADAS avanzado, conducción autónoma e infotainment moderno — exactamente el dominio donde tú trabajas con QNX/Qualcomm en Bosch.

**Diferencias clave Classic vs Adaptive**:

| Aspecto | AUTOSAR Classic | AUTOSAR Adaptive |
|---|---|---|
| Hardware objetivo | Microcontroladores de recursos limitados | Microprocesadores de alto desempeño |
| Sistema operativo | AUTOSAR OS (tipo OSEK, estático) | Basado en **POSIX** (ej. Linux, QNX) — conecta directamente con tu capítulo de Sistemas Operativos Embebidos |
| Comunicación | Señal-céntrica (CAN, basada en el COM Stack) | **Orientada a servicios (SOME/IP)** — exactamente lo que ya estudiaste a fondo en Automotive Ethernet |
| Configuración | Estática, determinada en tiempo de compilación | Dinámica — los servicios pueden descubrirse y configurarse en tiempo de ejecución |
| Actualización de software | Requiere reflashing completo (vía UDS, como ya sabes) | Soporta actualizaciones más flexibles, incluyendo OTA |
| Caso de uso típico | ECUs de control tradicional (motor, frenos, carrocería) | ADAS de alto desempeño, infotainment, dominios centrales (domain controllers) |

**Conexión directa con tu rol actual**: tu experiencia en Bosch trabajando sobre **QNX** y colaborando con equipos de plataforma (Qualcomm), validando infotainment, es precisamente el dominio de **AUTOSAR Adaptive** (o, en algunos casos, plataformas no estrictamente AUTOSAR pero con la misma filosofía POSIX/orientada a servicios) — mientras que tu experiencia anterior en VW/Ford validando funciones más tradicionales de ADAS/confort sobre HIL es más representativa de **AUTOSAR Classic**. Poder articular en qué plataforma exacta se ubica cada una de tus experiencias pasadas es una señal fuerte de madurez técnica.

## Preguntas de teoría — AUTOSAR
1. ¿Cuál es la diferencia fundamental de propósito entre ASPICE y AUTOSAR? (pregunta de apertura casi garantizada)
2. Describe las capas de la arquitectura AUTOSAR Classic, de la aplicación hasta el microcontrolador.
3. ¿Qué rol cumple el RTE, y por qué permite que un SWC "no sepa" si se comunica localmente o vía bus?
4. ¿En qué módulos específicos del Basic Software vive la implementación de UDS, y qué hace cada uno?
5. ¿Qué es el VFB y por qué el diseño a ese nivel ocurre antes del mapeo a una ECU física?
6. ¿Cuál es la diferencia entre un Provided Port y un Required Port, y entre una interfaz Sender-Receiver y Client-Server?
7. ¿Qué describe un archivo ARXML, y cómo se conecta con herramientas como CANoe?
8. Explica las diferencias clave entre AUTOSAR Classic y Adaptive, en términos de sistema operativo y modelo de comunicación.
9. ¿En qué plataforma (Classic o Adaptive) ubicarías tu experiencia trabajando con QNX en Bosch, y por qué?
10. ¿Por qué AUTOSAR Adaptive usa SOME/IP en vez del enfoque señal-céntrico de Classic?

## Errores comunes — AUTOSAR
- Confundir AUTOSAR con un sistema operativo — es una arquitectura de software completa (que puede incluir un SO específico como parte del Basic Software, pero no se reduce a eso).
- Decir que todos los SWCs "saben" cómo se comunican entre sí — justamente el punto del RTE/VFB es que esa información queda abstraída del diseño de aplicación.
- Ubicar UDS genéricamente "en el software" sin poder nombrar DCM/DEM específicamente.
- Tratar Classic y Adaptive como simplemente "una versión más nueva reemplaza a la anterior" — conviven, porque resuelven necesidades de hardware y dominios de aplicación distintos (ECUs tradicionales vs dominios de alto desempeño).
- No conectar AUTOSAR Adaptive con SOME/IP y POSIX, que ya estudiaste a fondo en capítulos anteriores — es la misma tecnología vista desde ángulos distintos (protocolo de comunicación vs arquitectura de software que lo implementa).

---

## Síntesis: cómo se relacionan ASPICE y AUTOSAR entre sí

Una buena forma de cerrar este capítulo en entrevista si te piden "relaciona estos dos términos": **ASPICE te dice qué procesos debes seguir y con qué rigor** (ej. SWE.2 — Software Architectural Design, debe ejecutarse de forma gestionada y con productos de trabajo trazables) — **AUTOSAR es, en la práctica, una de las formas más comunes de la industria de *ejecutar* ese proceso SWE.2**, proveyendo una arquitectura de referencia ya estandarizada en vez de que cada proveedor diseñe su propia arquitectura desde cero. Uno es el "cómo se mide si lo hiciste bien" (ASPICE), el otro es "una forma estandarizada de hacerlo" (AUTOSAR) — no son competidores ni sustitutos, son complementarios y, de hecho, es extremadamente común que un proyecto automotriz use ambos simultáneamente.
