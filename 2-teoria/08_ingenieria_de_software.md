# Capítulo: Ingeniería de Software

> Objetivo del capítulo: al terminar podrás (1) explicar un principio SOLID con un ejemplo concreto en C embebido y cómo facilita pruebas en SIL; (2) justificar una desviación MISRA C y distinguir qué detecta el análisis estático que la prueba dinámica no; (3) diagnosticar un fallo intermitente como posible condición de carrera, deadlock o inversión de prioridades, sabiendo qué sí y qué no resuelve `volatile`; (4) calcular cuántos casos mínimos implica una complejidad ciclomática dada; y (5) explicar *freedom from interference* y por qué las herramientas de desarrollo requieren calificación en proyectos de seguridad. Cubre la ingeniería de software **tal como se practica en software embebido automotriz**: cómo se diseña, se construye, se mantiene con calidad y se gestiona. Es el capítulo con más peso en entrevistas de **ESE**, y en la parte técnica de **TE/TFL** que revisa código, patches y cobertura.
>
> **Límites con otros capítulos (para no repetir):**
> - Requisitos y trazabilidad → *Requirements Engineering*.
> - Procesos SWE.1–SWE.6 y niveles de capacidad → *ASPICE/AUTOSAR*. Aquí solo se referencian.
> - V-Model completo, hitos A/B/C/D, interfaces de sistema, HARA/FMEA → *Ingeniería de Sistemas*.
> - Niveles y técnicas de prueba (caja blanca/negra, cobertura) → *Fundamentos de Software Testing*. Aquí se usan, no se redefinen.
> - Detalle de RTOS, planificación de tareas y boot → *Sistemas Operativos Embebidos*. Aquí solo la visión de ingeniería sobre concurrencia.
> - Pipelines, Docker, Jenkins y branching a fondo → *CI/CD y Control de Versiones*.
>
> **Nota sobre Software Architecture:** en el plan original era un capítulo propio. Por ahora incluyo aquí los estilos y principios de arquitectura (sección 4) para tener el panorama completo. Si prefieres un capítulo separado, esa sección se mueve tal cual.

---

## 1. Qué es la ingeniería de software y cómo se ubica en el desarrollo

**Ingeniería de software** es la aplicación de un enfoque sistemático, disciplinado y medible al desarrollo, operación y mantenimiento del software. No es "programar": incluye requisitos de software, arquitectura, diseño detallado, implementación, verificación, gestión de configuración, gestión de cambios y liberación.

Dentro de un sistema automotriz, el software recibe del sistema un subconjunto de requisitos asignados (la transición SYS.3 → SWE.1) y entrega software verificado que se integra de nuevo al sistema (SWE.6 → SYS.4).

**Particularidades del software embebido automotriz** (lo que lo distingue del software "de oficina"):
- Recursos limitados (memoria, CPU) y requisitos de **tiempo real**.
- Acoplamiento fuerte con el hardware (periféricos, buses, interrupciones).
- Larga vida del producto y actualización costosa (reflashing, OTA regulado).
- Requisitos de **seguridad funcional y ciberseguridad**.
- Evidencia formal exigida por **ASPICE e ISO 26262**.

## 2. Modelos de ciclo de vida desde el lado del software

El V-Model completo, con su estructura y limitaciones, está en *Ingeniería de Sistemas*. Aquí lo que importa es cómo se organiza el trabajo de software dentro de él.

### 2.1 Software dentro del V-Model (lado SWE)
Requisitos de software (SWE.1) → arquitectura (SWE.2) → diseño detallado y construcción de unidades (SWE.3) → verificación de unidades (SWE.4) → integración de software (SWE.5) → calificación de software (SWE.6). Cada nivel se verifica contra su artefacto de diseño correspondiente.

### 2.2 Iterativo, incremental y espiral
- **Iterativo/incremental**: el software crece en entregas funcionales, cada una probada en todos los niveles.
- **Espiral**: añade análisis de riesgo explícito por vuelta. Útil para tecnología nueva (por ejemplo, una plataforma con hipervisor o AUTOSAR Adaptive por primera vez).

### 2.3 Metodologías ágiles
- **Scrum**: sprints de duración fija (2–4 semanas). Roles: *Product Owner* (prioriza el backlog), *Scrum Master* (facilita y remueve impedimentos) y *Equipo de Desarrollo*. Eventos: Sprint Planning, Daily Scrum, Sprint Review y Retrospectiva. Artefactos: Product Backlog, Sprint Backlog e Incremento. Compromisos: *Definition of Done*, Meta del Sprint.
- **Kanban**: flujo continuo con **límites de trabajo en progreso (WIP)**, sin sprints. Métricas: *cycle time*, *throughput*.
- **XP**: prácticas técnicas (TDD, programación en pares, integración continua, refactorización, propiedad colectiva del código).
- **SAFe / LeSS**: coordinación de varios equipos. En SAFe, el trabajo se organiza en *Program Increments* (PI) de aproximadamente 8–12 semanas, con planeación conjunta.
- **Definition of Ready (DoR)** y **Definition of Done (DoD)**: la DoR dice cuándo un elemento está listo para entrar a un sprint; la DoD, cuándo está terminado. En automotriz, la DoD suele incluir: código revisado, análisis estático sin violaciones nuevas, pruebas unitarias y de integración ejecutadas con la cobertura requerida, trazabilidad actualizada y documentación mínima de diseño.

### 2.4 El enfoque híbrido y ASPICE
La mayoría de programas combina un marco de V-Model (por el hardware y la regulación) con ejecución ágil del software. Tensiones y soluciones habituales:
- **Documentación vs agilidad**: ASPICE exige productos de trabajo trazables. Se resuelve generando la evidencia desde las herramientas (trazabilidad Jira ↔ DOORS ↔ RQM, reportes automáticos) en lugar de documentos manuales extensos.
- **Hardware que no itera**: el software planifica contra la disponibilidad de muestras (A/B/C-Sample) y usa SIL, virtual ECUs y HIL para avanzar sin hardware definitivo.
- **Seguridad funcional**: la evidencia por ASIL se integra en el flujo del sprint (por ejemplo, en la DoD), no se deja para el final.

## 3. Principios de diseño de software

### 3.1 Abstracción y modularidad
Dividir el software en módulos con responsabilidades claras e interfaces bien definidas, ocultando detalles internos (*information hiding*). La arquitectura en capas de AUTOSAR es un ejemplo: la aplicación no conoce el hardware porque la MCAL y el RTE lo abstraen.

### 3.2 Cohesión y acoplamiento
- **Alta cohesión**: todo lo que hay en un módulo contribuye a un mismo propósito.
- **Bajo acoplamiento**: los módulos dependen poco unos de otros y solo a través de interfaces estables.
- Consecuencia práctica: un cambio local no debería obligar a modificar muchos módulos, y un módulo cohesivo y poco acoplado es mucho más fácil de **probar unitariamente**.

### 3.3 Principios SOLID (idea y aplicación conceptual en C embebido)
| Principio | Idea | Equivalente práctico en embebido |
|---|---|---|
| **S** – Responsabilidad única | Un módulo, una razón para cambiar | Separar la lógica de control del acceso a hardware |
| **O** – Abierto/Cerrado | Extender sin modificar código existente | Añadir variantes mediante configuración o punteros a función en vez de editar la lógica base |
| **L** – Sustitución de Liskov | Un sustituto debe respetar el contrato del original | Dos drivers distintos de un mismo sensor deben cumplir la misma interfaz |
| **I** – Segregación de interfaces | Interfaces pequeñas y específicas | No obligar a un módulo a depender de funciones que no usa |
| **D** – Inversión de dependencias | Depender de abstracciones, no de implementaciones | La lógica depende de una interfaz de "sensor", no de un registro concreto, lo que permite sustituirlo por un *mock* en SIL |

Son principios nacidos en orientación a objetos, pero la idea (interfaces estables, dependencias hacia abstracciones) se aplica también en C mediante cabeceras, capas y punteros a función.

### 3.4 Patrones habituales en software embebido
- **HAL (Hardware Abstraction Layer)**: aísla el hardware.
- **Máquina de estados**: lógica de modos y transiciones (se modela en Stateflow y se prueba con la técnica de transición de estado).
- **Observer / publish-subscribe**: notificación de eventos entre módulos.
- **Productor-consumidor con buffer**: desacopla recepción de datos (por ejemplo, tramas CAN en una ISR) de su procesamiento.
- **Watchdog**: supervisión de que el software sigue ejecutándose correctamente.
- **Doble buffer**: evita leer datos a medias mientras se actualizan.
- Precaución con patrones que asumen memoria dinámica o mucha abstracción: en sistemas críticos se evitan o se restringen.

### 3.5 UML a nivel práctico
Los diagramas que más se usan en diseño de software embebido: **de componentes** (estructura e interfaces), **de secuencia** (interacción en el tiempo, por ejemplo, una transacción UDS entre módulos), **de máquina de estados** (comportamiento por modos) y **de clases** (cuando se usa C++). Sirven como base de prueba para integración (los diagramas de secuencia son base de prueba de integración, como viste en Fundamentos de Testing).

### 3.6 Deuda técnica y refactorización
- **Deuda técnica**: decisiones que aceleran hoy pero encarecen el futuro (duplicación, módulos acoplados, falta de pruebas). Se gestiona de forma explícita: registrarla, estimarla y reservar capacidad para pagarla.
- **Code smells**: señales de problemas de diseño (funciones muy largas, duplicación, demasiados parámetros, módulos "dios").
- **Refactorización**: reestructurar el código **sin cambiar su comportamiento externo**. Solo es segura si existen **pruebas automatizadas** que detecten regresiones. Por eso la deuda técnica es también un problema del equipo de pruebas.

## 4. Arquitectura de software

### 4.1 Qué decide la arquitectura
Define la estructura de alto nivel: componentes, sus responsabilidades, interfaces y reglas de interacción. Las decisiones arquitectónicas son las más caras de cambiar y determinan los **atributos de calidad** del sistema (desempeño, seguridad, fiabilidad, mantenibilidad, portabilidad, testabilidad; la norma ISO 25010 los organiza).

### 4.2 Estilos de arquitectura
| Estilo | Idea | Ejemplo automotriz |
|---|---|---|
| **En capas** | Cada capa usa solo la inferior | AUTOSAR Classic (aplicación → RTE → BSW → MCAL) |
| **Basada en componentes** | Piezas con interfaces definidas, intercambiables | Software Components (SWC) de AUTOSAR |
| **Orientada a eventos / publish-subscribe** | Los productores publican eventos y los consumidores se suscriben | Eventos de SOME/IP; manejo de eventos de bus |
| **Cliente-servidor / orientada a servicios** | Un proveedor ofrece servicios que otros invocan | AUTOSAR Adaptive, SOME/IP, Client-Server de puertos AUTOSAR |
| **Pipeline (tubería y filtros)** | Datos transformados en etapas sucesivas | Cadena de percepción: adquisición → filtrado → fusión → decisión |
| **Microkernel** | Núcleo mínimo con servicios fuera del kernel | Arquitectura de QNX (ya vista en Sistemas Operativos Embebidos) |
| **Máquina de estados / controlador de modos** | El comportamiento depende del modo actual | Gestión de modos de una ECU: arranque, normal, degradado, apagado |

### 4.3 Vistas y documentación
Una arquitectura se describe con varias **vistas**: estructural (componentes y dependencias), de comportamiento (secuencias, estados), de despliegue (qué corre en qué ECU/procesador) y de datos. Las decisiones importantes se registran en **ADRs (Architecture Decision Records)**: contexto, decisión, alternativas y consecuencias. Muchos "porqués" que un probador necesita para entender un comportamiento viven aquí.

### 4.4 Evaluación de arquitectura
Se revisa contra los atributos de calidad con **revisiones de arquitectura** (por ejemplo, métodos tipo ATAM: escenarios de calidad, riesgos y puntos de compromiso) y con prototipos o simulaciones. Para ti: la **testabilidad** es un atributo arquitectónico. Una arquitectura que permite inyectar entradas, observar salidas internas e intercambiar dependencias por *mocks* facilita todo tu trabajo de verificación.

## 5. Calidad del código en software embebido

### 5.1 Estándares de codificación: MISRA C
- **MISRA C** es el estándar de codificación más extendido en automotriz para C (existe MISRA C++ para C++). Restringe construcciones propensas a error o de comportamiento no definido.
- Las reglas se clasifican por obligatoriedad (**mandatory, required, advisory**), y se acompañan de **directivas**. Las desviaciones (*deviations*) deben justificarse y documentarse formalmente.
- Ejemplos del tipo de cosas que restringe: uso de memoria dinámica, recursión, conversiones implícitas peligrosas, uso de `goto`, ausencia de manejo de valores de retorno, tipos de tamaño ambiguo (se prefieren tipos de ancho fijo como `uint8_t`, `uint32_t`).
- Los estándares de la organización (naming, estructura, comentarios) se suman a MISRA.

### 5.2 Análisis estático
Herramientas que examinan el código sin ejecutarlo para detectar violaciones de MISRA, errores de punteros, desbordamientos, variables sin inicializar, código inalcanzable y complejidad excesiva. Ejemplos de herramientas del mercado: Polyspace, Helix QAC, Coverity, Klocwork, PC-lint, cppcheck, SonarQube.
- Se integran en el pipeline de CI para que cada cambio se analice automáticamente (cumple el principio de "análisis estático antes de la prueba dinámica" de shift-left).
- Un riesgo real: el exceso de falsos positivos genera fatiga. Se gestiona ajustando la configuración y justificando supresiones.

### 5.3 Revisión de código (code review)
Es prueba estática aplicada a código y, además, transferencia de conocimiento.
- **Flujo con Gerrit** (de tu CV): el desarrollador sube un *patch set*, los revisores comentan y votan (por ejemplo, Code-Review +1/+2 y Verified), y el cambio se integra solo tras cumplir los criterios. Cada revisión es un **producto de trabajo** que evidencia SWE.3/SWE.4 en un assessment.
- **Buenas prácticas**: cambios pequeños, lista de verificación (MISRA, manejo de errores, concurrencia, casos límite), revisores con conocimiento del área, foco en defectos y diseño y no en estilo (el estilo lo automatizan las herramientas).
- Relaciona esto con los tipos de revisión que viste (informal, guiada, técnica, inspección): una revisión de código con lista de verificación y roles definidos se acerca a una inspección.

### 5.4 Programación defensiva en embebido
Validar entradas y rangos, comprobar valores de retorno, manejar explícitamente los casos de error y de ausencia de datos (timeouts de mensajes), usar *assertions* en desarrollo, inicializar todo, evitar comportamiento indefinido y definir valores por defecto seguros. Estas prácticas son las que luego verificas con pruebas de robustez y fault injection.

### 5.5 Pruebas desde el lado del desarrollador
- **Prueba unitaria** (SWE.4): se prueba cada unidad de forma aislada, reemplazando dependencias por **stubs/mocks** (*test doubles*). Herramientas habituales: VectorCAST, Tessy, Unity/CMock, GoogleTest.
- **TDD** en embebido: viable si el diseño separa lógica de hardware (principio de inversión de dependencias).
- **SIL**: se ejecuta el código real en PC con un entorno simulado (visto en HIL Testing).
- **Cobertura estructural**: la mide una herramienta durante las pruebas y se exige según el nivel de integridad (ver sección 8).

## 6. Métricas de software

| Métrica | Qué mide | Uso |
|---|---|---|
| **Complejidad ciclomática** V(G) = E − N + 2P (aristas, nodos, componentes conectados del grafo de flujo) | Número de caminos linealmente independientes | Indica dificultad de prueba y mantenimiento. Es común fijar límites (por ejemplo, 10–15 por función, según el proyecto). Una función con V(G) = 12 requiere al menos 12 casos para cubrir todos sus caminos independientes |
| **Acoplamiento (fan-in/fan-out)** | Dependencias entre módulos | Detectar módulos demasiado interdependientes |
| **Cohesión** | Qué tan relacionadas están las partes de un módulo | Detectar módulos con responsabilidades mezcladas |
| **Tamaño (LOC)** | Líneas de código | Normalizar otras métricas, estimar. Por sí sola es una métrica pobre de productividad |
| **Cobertura de código** | Sentencias, ramas o condiciones ejecutadas | Medir la completitud de pruebas |
| **Densidad de defectos** | Defectos por tamaño del código | Evaluar calidad y comparar versiones |
| **Violaciones de análisis estático** | Cantidad y tendencia | Control de calidad en CI |
| **Cobertura de revisión / hallazgos por revisión** | Cuánto código se revisa y qué se encuentra | Efectividad de las revisiones |
| **Deuda técnica estimada** | Esfuerzo para corregir problemas conocidos | Priorizar refactorizaciones |

Una advertencia que suma puntos en entrevista: **una métrica sin contexto se puede manipular** (por ejemplo, cobertura alta con pruebas sin aserciones). Las métricas se interpretan en conjunto y como tendencia, no como objetivo aislado.

## 7. Concurrencia y tiempo real: la visión del ingeniero de software

El detalle del RTOS (tareas, prioridades, planificadores) está en *Sistemas Operativos Embebidos*. Aquí lo esencial de cara al diseño y las pruebas:

- **Condición de carrera (race condition)**: el resultado depende del orden de ejecución de tareas/ISRs que acceden a datos compartidos. Un ejemplo típico: una ISR de recepción CAN escribe en un buffer mientras una tarea lo lee a medias.
- **Sincronización**: *mutex*, semáforos, secciones críticas, desactivación temporal de interrupciones, buffers de doble banco. Cada una tiene costos (bloqueo, latencia).
- **Deadlock**: dos tareas esperan mutuamente recursos que la otra mantiene. Se previene con orden global de adquisición y minimizando secciones bloqueantes.
- **Inversión de prioridades**: una tarea de alta prioridad queda bloqueada por una de baja que tiene un recurso, mientras una de prioridad media se ejecuta. Se mitiga con protocolos de herencia o techo de prioridad.
- **Variables compartidas con ISRs**: requieren `volatile` y acceso atómico o protegido. `volatile` solo evita optimizaciones del compilador, no garantiza atomicidad.
- **Tiempo de ejecución en el peor caso (WCET)** y análisis de planificabilidad: se verifica que las tareas cumplan sus plazos incluso en el peor escenario.

**Implicaciones para pruebas**: estos defectos son intermitentes y dependen del timing, por lo que a menudo no aparecen en pruebas funcionales normales. Se detectan con revisión de diseño y código, análisis estático especializado, pruebas de estrés y carga de bus en HIL, trazas con marcas de tiempo precisas y reproducción determinista. Un fallo intermitente en una prueba automatizada que "pasa manualmente" suele apuntar a un problema de timing o concurrencia.

## 8. Software y seguridad funcional (ISO 26262 – Parte 6)

La Parte 6 de ISO 26262 regula el desarrollo de software de seguridad y sigue un V-Model análogo a SWE.1–SWE.6. Ideas clave a nivel conceptual:
- Los **requisitos de seguridad de software** se derivan de los requisitos técnicos de seguridad, heredando el **ASIL**.
- Se exigen **métodos de diseño y codificación** más estrictos según el ASIL: estructura modular, interfaces restringidas, complejidad limitada, uso de estándares como MISRA, restricciones sobre memoria dinámica, recursión y punteros.
- **Mecanismos de seguridad en software**: comprobaciones de plausibilidad, monitoreo de flujo de programa, protección de la comunicación (CRC, contadores de mensaje, timeouts), detección de corrupción de memoria, watchdogs, reacciones a estado seguro.
- **Verificación**: revisiones, análisis estático, pruebas unitarias y de integración, y **cobertura estructural** con rigor creciente según el ASIL (cobertura de sentencia como base, cobertura de rama y MC/DC fuertemente recomendadas en los niveles altos). Los niveles exactos y su carácter de "recomendado" o "altamente recomendado" están en las tablas de la norma, así que conviene consultarlas para un proyecto concreto.
- **Independencia** de la verificación creciente con el ASIL (concepto de Fundamentos de Testing).
- **Interferencia entre elementos** (*freedom from interference*): software de distintos ASIL en un mismo procesador debe estar aislado (memoria, tiempo de ejecución, comunicación). Se logra con partición de memoria (MPU), monitores de tiempo y, en arquitecturas modernas, con hipervisores.
- **Software tools**: las herramientas que podrían introducir o dejar pasar errores (compiladores, generadores de código, herramientas de prueba) requieren **clasificación y calificación** según su impacto.

## 9. Gestión de configuración, cambios y liberaciones del software

El marco general (identificación, línea base, control de cambios) ya lo viste en Fundamentos de Testing y en ASPICE (SUP.8, SUP.10). Lo específico del software:
- **Control de versiones (Git)**: cada commit es trazable a un requisito o ticket. Se recomienda vincular commits con IDs de Jira.
- **Estrategias de ramificación**: *trunk-based* (ramas cortas, integración frecuente), *GitFlow* (ramas de feature, develop, release y hotfix), o variantes del programa. Se profundiza en el capítulo de CI/CD.
- **Línea base de software**: combinación exacta de código fuente, configuración, herramientas (versión del compilador) y bibliotecas usada para producir un entregable. Debe poder **reproducirse el build** a partir de ella.
- **Versionado**: esquemas como *semantic versioning* (mayor.menor.parche) o numeración propia del programa. Cada versión liberada se vincula a sus pruebas, defectos conocidos y notas de versión.
- **Gestión de liberaciones (release management)**: criterios de liberación (quality gates), notas de versión, lista de defectos conocidos y evidencia de pruebas.
- **Gestión de cambios**: solicitud, análisis de impacto (usando trazabilidad), aprobación y verificación del cambio. En un *patch* de software (como los que desarrollas en Bosch), el análisis de impacto determina el alcance de la regresión.
- **Puertas de calidad (quality gates)**: criterios medibles para avanzar (cobertura, violaciones de análisis estático, defectos abiertos por severidad, resultados de regresión, cumplimiento de ASPICE).

## 10. Roles y colaboración en el desarrollo de software

- **Software Architect**: define y custodia la arquitectura.
- **Software Developer / Embedded SW Engineer**: diseña, implementa y verifica unidades.
- **Software Test Engineer / Test Function Lead**: verificación de integración y calificación, estrategia y reporte.
- **Software Quality / Process roles**: aseguramiento de calidad (SUP.1), conformidad con ASPICE.
- **Safety Engineer** y **Cybersecurity Engineer**: requisitos y evidencia de seguridad.
- **Roles ágiles**: Product Owner, Scrum Master, equipo de desarrollo.
- **Proveedores y plataformas**: integración con software de terceros (como en tu experiencia con QNX/Qualcomm), que exige interfaces claras, criterios de aceptación de entregas y gestión de defectos entre organizaciones.

## Preguntas de teoría
1. ¿Qué distingue a la ingeniería de software embebido automotriz del software de propósito general?
2. Explica cómo se organiza el trabajo de software dentro del V-Model (SWE.1–SWE.6) y qué se verifica en cada nivel.
3. Describe los roles, eventos y artefactos de Scrum, y la diferencia entre Definition of Ready y Definition of Done.
4. ¿Cómo conciliarías la documentación exigida por ASPICE con un equipo que trabaja en sprints?
5. Define cohesión y acoplamiento y explica por qué un módulo cohesivo y poco acoplado es más fácil de probar.
6. Explica uno de los principios SOLID con un ejemplo en C embebido y cómo facilita pruebas en SIL.
7. ¿Qué es la deuda técnica y por qué la refactorización requiere pruebas automatizadas?
8. Compara tres estilos de arquitectura con un ejemplo automotriz de cada uno.
9. ¿Qué es un ADR y por qué le interesa a un probador?
10. ¿Qué es MISRA C, cómo se clasifican sus reglas y cómo se gestionan las desviaciones?
11. ¿Qué detecta el análisis estático que las pruebas dinámicas no detectan fácilmente? ¿Cuál es su riesgo principal?
12. Describe cómo funciona una revisión de código con Gerrit y por qué es evidencia de proceso.
13. ¿Qué mide la complejidad ciclomática y cuántos casos de prueba mínimos implica una función con V(G) = 8?
14. ¿Por qué una cobertura alta no garantiza buenas pruebas?
15. Explica condición de carrera, deadlock e inversión de prioridades, y cómo detectarías cada uno.
16. ¿Qué hace `volatile` y qué no hace?
17. ¿Qué es *freedom from interference* y cómo se logra?
18. ¿Por qué las herramientas de desarrollo y prueba pueden requerir calificación en un proyecto de seguridad?
19. ¿Qué debe incluir una línea base de software para poder reproducir un build?
20. Un test automatizado falla de forma intermitente. ¿Qué hipótesis de ingeniería de software considerarías?

## Errores comunes
- Reducir la ingeniería de software a "escribir código" y olvidar arquitectura, calidad, configuración, cambios y liberación.
- Presentar ágil como "sin documentación ni procesos": en contextos regulados significa evidencia ligera y automatizada, no ausente.
- Confundir cohesión con acoplamiento, o describir "bajo acoplamiento" como "módulos que no se comunican".
- Aplicar SOLID literalmente como si solo existiera en orientación a objetos, sin explicar la intención (interfaces estables, dependencias hacia abstracciones).
- Refactorizar sin red de pruebas y esperar no introducir regresiones.
- Tratar MISRA como "lista de estilo": es un estándar de seguridad del lenguaje, y las desviaciones requieren justificación formal.
- Dejar el análisis estático solo para el final en lugar de integrarlo continuamente en el pipeline.
- Usar una sola métrica (cobertura, LOC, número de defectos) como indicador único de calidad.
- Creer que `volatile` resuelve la sincronización entre ISR y tarea.
- Diagnosticar fallos intermitentes solo desde la capa de prueba, sin considerar timing, concurrencia o carga de bus.
- Ignorar la calificación de herramientas y la independencia de la verificación en proyectos con ASIL alto.
- Pensar que la arquitectura es solo un diagrama: las decisiones y sus razones (ADRs) son parte del producto.

## Puntos clave

Repaso rápido antes de una entrevista — un bullet por idea, sin reexplicar.

### 1. Qué es y cómo se ubica
- No es "programar": incluye requisitos, arquitectura, diseño, implementación, verificación, configuración, cambios y liberación.
- El software recibe requisitos asignados del sistema (SYS.3 → SWE.1) y entrega software verificado de vuelta (SWE.6 → SYS.4).
- Particularidades del embebido automotriz: recursos limitados + tiempo real, acoplamiento fuerte al hardware, larga vida/actualización costosa, seguridad funcional y ciberseguridad, evidencia formal ASPICE/ISO 26262.

### 2. Modelos de ciclo de vida
- Lado SWE del V-Model: SWE.1 → SWE.2 → SWE.3 → SWE.4 → SWE.5 → SWE.6, cada nivel verificado contra su artefacto de diseño correspondiente.
- Espiral añade análisis de riesgo explícito por vuelta — útil para tecnología nueva (hipervisor, AUTOSAR Adaptive por primera vez).
- Definition of Ready (listo para el sprint) ≠ Definition of Done (terminado) — en automotriz la DoD suele incluir cobertura, análisis estático limpio y trazabilidad actualizada.
- Enfoque híbrido: V-Model por hardware/regulación + ejecución ágil del software; la evidencia ASPICE se genera desde las herramientas (Jira ↔ DOORS ↔ RQM), no con documentos manuales.

### 3. Principios de diseño
- Alta cohesión + bajo acoplamiento = más fácil de probar unitariamente.
- SOLID en C embebido: **S** separa lógica de hardware, **O** extiende sin modificar (punteros a función), **L** mismo contrato entre drivers intercambiables, **I** interfaces pequeñas y específicas, **D** depender de abstracciones (habilita mocks en SIL).
- Patrones típicos: HAL, máquina de estados, observer/pub-sub, productor-consumidor con buffer, watchdog, doble buffer — cuidado con memoria dinámica en sistemas críticos.
- Refactorizar sin pruebas automatizadas que detecten regresiones no es seguro.

### 4. Arquitectura de software
- Las decisiones arquitectónicas son las más caras de cambiar y determinan los atributos de calidad (ISO 25010).
- Estilos y su ejemplo automotriz: capas (AUTOSAR Classic), basada en componentes (SWC), orientada a eventos (SOME/IP), cliente-servidor (Adaptive), pipeline (cadena de percepción), microkernel (QNX), máquina de estados (modos de ECU).
- Un ADR registra el porqué de una decisión — muchos "porqués" que un probador necesita para entender un comportamiento viven ahí.
- La **testabilidad** es un atributo arquitectónico: una arquitectura que permite inyectar entradas, observar salidas internas y sustituir dependencias por mocks facilita todo el trabajo de verificación.

### 5. Calidad del código
- MISRA C: reglas mandatory/required/advisory + directivas; las desviaciones se justifican y documentan formalmente — no es una "lista de estilo".
- El análisis estático detecta sin ejecutar (punteros, desbordamientos, complejidad); riesgo real: fatiga por exceso de falsos positivos.
- Code review (Gerrit): cada revisión es un Work Product que evidencia SWE.3/SWE.4 — se acerca a una inspección si tiene lista de verificación y roles definidos.
- Programación defensiva: validar entradas, manejar errores/timeouts, assertions, valores por defecto seguros — se verifica después con robustez y fault injection.
- Prueba unitaria (SWE.4) con stubs/mocks; TDD es viable si la lógica está separada del hardware.

### 6. Métricas de software
- Complejidad ciclomática `V(G) = E − N + 2P`: indica el número mínimo de casos para cubrir los caminos linealmente independientes.
- Una métrica sin contexto se manipula (ej. cobertura alta con pruebas sin aserciones) — se interpretan en conjunto y como tendencia, nunca aisladas.

### 7. Concurrencia y tiempo real
- Condición de carrera, deadlock, inversión de prioridades (mitigada con herencia o techo de prioridad) — son defectos intermitentes que dependen del timing.
- `volatile` solo evita optimizaciones del compilador — **no** garantiza atomicidad ni sincronización.
- Un fallo intermitente que "pasa manualmente" suele apuntar a timing/concurrencia, no a un error de lógica funcional.

### 8. Software y seguridad funcional (ISO 26262 – Parte 6)
- El rigor de diseño/codificación y de cobertura estructural (sentencia → rama → MC/DC) crece con el ASIL.
- *Freedom from interference*: software de distinto ASIL en el mismo procesador debe aislarse (MPU, monitores de tiempo, hipervisores).
- Las herramientas que podrían introducir o dejar pasar errores (compiladores, generadores de código) requieren clasificación y calificación según su impacto.

### 9. Gestión de configuración, cambios y liberaciones
- Línea base de software = código + configuración + herramientas (versión del compilador) + bibliotecas — debe poder reproducir el build exacto.
- Trunk-based (integración frecuente, ramas cortas) vs GitFlow (ramas de feature/develop/release/hotfix).
- Quality gates: cobertura, violaciones de análisis estático, defectos abiertos por severidad, resultados de regresión, cumplimiento ASPICE.

### 10. Roles y colaboración
- Software Architect (custodia la arquitectura), Developer/ESE (diseña y verifica unidades), Test Engineer/TFL (integración, calificación, estrategia), Safety/Cybersecurity Engineer (requisitos y evidencia de seguridad) — más los roles ágiles (PO, Scrum Master, equipo).