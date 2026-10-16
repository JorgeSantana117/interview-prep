# Capítulo: Fundamentos de Software Testing (ISTQB CTFL 4.0)

> Objetivo del capítulo: a diferencia de Protocolos y HIL (donde tu experiencia va por delante de la teoría), aquí es al revés — probablemente ya tienes la certificación ISTQB, pero este es el tema donde los entrevistadores hacen las preguntas más "tramposas" y sutiles, porque el syllabus está lleno de distinciones finas (verificación vs validación, prueba vs depuración, cobertura de sentencias vs de ramas) que se prestan a preguntas de opción múltiple con matices — el mismo tipo de pregunta que verás en una entrevista técnica bien diseñada. Basé este capítulo en tu documento CTFL-4.0 y agregué contexto adicional, ejemplos automotrices y las conexiones con lo que ya sabes de HIL y Protocolos.

---

## 1. Fundamentos de la Prueba

### 1.1 Por qué es necesaria la prueba
El software que no funciona correctamente puede generar pérdidas económicas, pérdida de tiempo, pérdida de reputación, lesiones o incluso muerte — en tu dominio (automotriz, ADAS, sistemas de seguridad) esta última consecuencia no es abstracta, es literal. La prueba de software es un **conjunto de actividades para descubrir defectos y evaluar la calidad de los artefactos de software**, y debe estar alineada con el ciclo de vida de desarrollo — no es solo ejecutar el software y comprobar resultados.

### 1.2 Verificación vs Validación (distinción clásica de entrevista)
- **Verificación**: ¿el sistema cumple los requisitos especificados? — *"hacer las cosas correctamente"* (doing the thing right).
- **Validación**: ¿el sistema satisface las necesidades reales de usuarios/stakeholders en su entorno operativo? — *"hacer las cosas correctas"* (doing the right thing).

*Ejemplo automotriz*: puedes verificar perfectamente que una ECU implementa el servicio UDS 0x22 exactamente como dice la especificación (verificación), pero si esa especificación no refleja lo que el usuario/OEM realmente necesita en campo, no estás validando. Un sistema puede pasar el 100% de sus pruebas de verificación y aun así fallar en validación.

La prueba puede ser:
- **Dinámica**: implica ejecutar el software (esto es lo que haces todo el día en HIL).
- **Estática**: revisiones y análisis estático, sin ejecutar el software.

### 1.3 Objetivos característicos de la prueba
- Evaluar productos de trabajo (requisitos, historias de usuario, diseños, código).
- Provocar fallos y encontrar defectos.
- Asegurar la cobertura necesaria del objeto de prueba.
- Reducir el riesgo de calidad inadecuada.
- Verificar cumplimiento de requisitos especificados, contractuales, legales o normativos (en automotriz esto conecta directamente con ASPICE/ISO 26262).
- Proporcionar información a los implicados para la toma de decisiones.
- Validar que el objeto de prueba funciona como esperan los implicados.
- Generar confianza en la calidad del objeto de prueba.

Los objetivos de prueba **varían según el contexto** (producto, nivel de prueba, riesgos, CVDS) — no hay un set fijo de objetivos válido para todos los proyectos.

### 1.4 Probar vs Depurar
- **Probar** puede desencadenar un fallo (prueba dinámica) o directamente encontrar un defecto (prueba estática).
- **Depurar es una actividad ajena a la prueba**. Cuando una prueba dinámica desencadena un fallo, la depuración se encarga de: **reproducir el fallo → diagnosticar la causa raíz → corregir la causa**.
- La **prueba de confirmación** posterior verifica que la corrección resolvió el problema (idealmente la hace la misma persona de la prueba inicial). La **prueba de regresión** verifica que la corrección no rompió otra parte del sistema.
- Cuando una prueba **estática** identifica un defecto, la depuración solo lo elimina — **no requiere reproducción ni diagnóstico**, porque la prueba estática encuentra el defecto directamente, sin provocar un fallo primero.

*Percepciones erróneas comunes que un entrevistador puede usar para evaluarte*: pensar que las pruebas son "solo para buscar errores y nada más", o que probar es "solo ejecutar casos de prueba y verificar el resultado" — ambas ideas son incompletas frente a todo lo que abarca realmente el proceso de prueba (planificación, análisis, diseño, gestión).

### 1.5 Prueba y Aseguramiento de la Calidad (AC vs CC)
- **Control de Calidad (CC)**: enfoque **correctivo, orientado al producto** — la prueba es una forma de CC.
- **Aseguramiento de la Calidad (AC)**: enfoque **preventivo, orientado a los procesos** — se concentra en implementar y mejorar procesos.
- Los resultados de la prueba alimentan a ambos: en CC se usan para **corregir defectos**; en AC dan **retroalimentación sobre el rendimiento de los procesos** de desarrollo y prueba (ej. si un proceso genera consistentemente muchos defectos, eso es información de AC, no solo de CC).
- La prueba también **forma parte del AC**, porque reduce el nivel de riesgo para la calidad del sistema.

### 1.6 Errores, Defectos, Fallos y Causas Raíz
Los seres humanos cometen **errores**, que producen **defectos**, que pueden dar lugar a **fallos**:
- Algunos defectos siempre producen un fallo si se ejecutan; otros solo en circunstancias específicas; algunos nunca llegan a manifestarse como fallo.
- Puede haber defectos en documentación, código fuente o cualquier artefacto de apoyo — no solo en código.
- **Los errores y defectos no son la única causa de fallos** — condiciones ambientales (radiación, campo electromagnético) también pueden causarlos. *Esto es directamente relevante para ti*: en automotriz, EMI (interferencia electromagnética) es una causa real de fallo que no tiene que ver con un defecto de código.
- **No todo resultado inesperado es un fallo** — ejemplo: un falso negativo.
- Una **causa raíz** es la razón fundamental por la que se produce un problema — no confundir causa raíz con el síntoma observado.

### 1.7 Los 7 Principios de la Prueba

Este es probablemente el punto más citado en cualquier examen o entrevista ISTQB — debes poder explicar cada uno con un ejemplo propio, no solo recitarlo:

1. **La prueba muestra la presencia de defectos, no su ausencia**: no se puede demostrar que un objeto de prueba está libre de defectos, solo que se encontraron (o no) defectos con las pruebas realizadas.
2. **La prueba exhaustiva es imposible**: excepto en casos triviales, no se puede probar todo (todas las combinaciones de entradas y precondiciones) — de ahí la necesidad de técnicas de prueba, priorización y pruebas basadas en riesgo.
3. **La prueba temprana ahorra tiempo y dinero**: tanto pruebas estáticas como dinámicas deben iniciarse lo antes posible (principio de "shift-left" — ver sección 2.4).
4. **Los defectos se agrupan**: un pequeño número de módulos suele concentrar la mayoría de los defectos detectados o son responsables de la mayoría de los fallos en producción.
5. **Las pruebas se desgastan (pesticide paradox)**: si se repiten las mismas pruebas muchas veces, se vuelven cada vez menos efectivas para encontrar defectos nuevos — sin embargo, las pruebas de regresión automatizadas siguen siendo valiosas para detectar regresiones, aunque no para encontrar defectos nuevos.
6. **La prueba depende del contexto**: no existe un enfoque de prueba universal — probar un sistema de infoentretenimiento no es igual a probar un sistema de frenos crítico de seguridad.
7. **Falacia de la ausencia de defectos**: encontrar y corregir defectos no garantiza el éxito del sistema — si el sistema no cumple las necesidades/expectativas de los usuarios, puede seguir siendo un fracaso aunque esté "libre de defectos" (conecta directamente con verificación vs validación).

### 1.8 Actividades y Tareas de Prueba (el proceso de prueba)
El proceso de prueba consiste en un conjunto de actividades, no necesariamente secuenciales, que pueden superponerse o realizarse de forma iterativa:

| Actividad | Qué responde / qué incluye |
|---|---|
| **Planificación de la prueba** | Define objetivos y enfoque de prueba, estimación de esfuerzo |
| **Monitorización y control** | Comprobación continua de las actividades vs el plan (monitorización) + medidas correctivas (control) |
| **Análisis de la prueba** | ¿**Qué** hay que probar? — análisis de la base de prueba |
| **Diseño de la prueba** | ¿**Cómo** llevar a cabo la prueba? — condiciones de prueba, datos de prueba, entorno de prueba |
| **Implementación de la prueba** | Creación de guiones manuales/automatizados, construcción del entorno de prueba |
| **Ejecución de la prueba** | Ejecución según calendario, registro de resultados, análisis de anomalías |
| **Compleción de la prueba** | Ocurre en hitos del proyecto — cierre formal, lecciones aprendidas |

### 1.9 El proceso de prueba en contexto
La prueba nunca ocurre de forma aislada — depende de factores contextuales: implicados (expectativas/necesidades), miembros del equipo (competencias, experiencia), dominio del negocio, factores técnicos, restricciones del proyecto, el CVDS elegido, y las herramientas disponibles. Estos factores influyen en la estrategia de prueba, las técnicas usadas, el grado de automatización, el nivel de cobertura requerido, y el detalle de la documentación.

### 1.10 Productos de prueba (Testware) por actividad
| Actividad | Productos de trabajo típicos |
|---|---|
| Planificación | Plan de prueba, calendario, registro de riesgos, criterios de entrada/salida |
| Monitorización y control | Informes de avance, directivas de control, información de riesgos |
| Análisis | Condiciones de prueba, informes de defectos de la base de prueba |
| Diseño | Casos de prueba, contratos de prueba, elementos de cobertura, requisitos de datos/entorno |
| Implementación | Procedimientos de prueba, guiones automatizados, juegos de prueba, calendario de ejecución |
| Ejecución | Registros en bitácora, informes de defectos |
| Compleción | Informe de compleción, elementos de acción, lecciones aprendidas |

**Distinciones que suelen confundirse (y aparecen en examen/entrevista):**
- **Contrato de prueba**: un pacto/instrucción de los objetivos de la prueba y posibles ideas de cómo realizarla.
- **Juego de pruebas (test suite)**: un grupo de guiones de prueba o un calendario de ejecución.
- **Caso de prueba**: contiene los **resultados esperados**.
- **Guion de prueba**: conjunto de **instrucciones** para la ejecución (manual o automatizada) de una prueba.

### 1.11 Trazabilidad entre base de prueba y productos de prueba
Mantener trazabilidad entre elementos de la base de prueba, productos de prueba, resultados y defectos permite: evaluar cobertura (ej. trazabilidad de casos de prueba a requisitos — algo que ya haces en DOORS/RQM), evaluar calidad del producto y capacidad del proceso, facilitar auditorías, facilitar la comprensión de informes de avance, y determinar el impacto de cambios.

### 1.12 Roles de prueba
- **Rol de Gestión de prueba**: responsabilidad general del proceso, equipo y dirección de actividades — se concentra en planificación, monitorización/control y compleción. Funciones: preparar/entregar informes, adaptar la planificación según avance, tomar acciones correctivas, apoyar selección de herramientas, decidir implementación de entornos.
- **Rol de prueba (Tester)**: responsabilidad del aspecto técnico/de ingeniería — se concentra en análisis, diseño, implementación y ejecución. Funciones: revisar/contribuir a planes de prueba, diseñar/preparar/verificar entornos, crear calendario de ejecución detallado, implementar casos/procedimientos, usar herramientas apropiadas.

*Nota directamente relevante para tu carrera*: tu progresión de Test Engineer → Test Function Lead es literalmente la transición del rol de "Tester" hacia responsabilidades cada vez más cercanas al rol de "Gestión de prueba" — vale la pena tener esta distinción clara para hablar de tu propia trayectoria en entrevista.

### 1.13 Competencias genéricas necesarias para probar
Conocimiento en materia de prueba, minuciosidad/cuidado/curiosidad/atención a detalles/ser metódico, buena comunicación y escucha activa, pensamiento analítico/crítico/creatividad, conocimientos técnicos, y conocimiento del dominio (industria) — en tu caso, dominio automotriz.

### 1.14 Enfoque de equipo completo (Whole-Team Approach)
Cualquier miembro del equipo con los conocimientos necesarios puede realizar cualquier tarea, y **todos son responsables de la calidad** — no solo los probadores. Los probadores colaboran estrechamente con representantes de negocio (para crear pruebas de aceptación) y con desarrolladores (para acordar estrategia de prueba y automatización). **No siempre es apropiado** — en situaciones críticas de seguridad puede requerirse independencia formal, algo relevante en tu dominio.

### 1.15 Independencia de la prueba
Existen 4 niveles, de menor a mayor independencia:
1. **Sin independencia**: el propio autor prueba su trabajo.
2. **Cierta independencia**: probadores del mismo equipo que el autor.
3. **Alta independencia**: probadores ajenos al equipo del autor, pero dentro de la organización.
4. **Independencia muy alta**: probadores externos a la organización.

**Ventajas de mayor independencia**: reconoce distintos tipos de fallos/defectos gracias a antecedentes y perspectivas diferentes, cuestiona/refuta suposiciones del equipo de desarrollo.
**Desventajas**: falta de colaboración, problemas de comunicación, relación de confrontación con desarrollo, pérdida de sentido de responsabilidad por la calidad, riesgo de ser vistos como "los culpables" de retrasos.

## Preguntas de teoría — Sección 1
1. Explica la diferencia entre verificación y validación con un ejemplo propio (no el de UDS que ya usamos).
2. ¿Por qué la depuración no forma parte de la prueba, y qué actividades sí forman parte de ella?
3. ¿Cuál es la diferencia entre Control de Calidad y Aseguramiento de la Calidad, y cómo se relacionan con la prueba?
4. Explica los 7 principios de la prueba, cada uno con un ejemplo de tu experiencia en HIL/automotriz.
5. ¿Por qué "los defectos se agrupan" es relevante para priorizar el esfuerzo de prueba?
6. ¿Cuál es la diferencia entre un caso de prueba, un guion de prueba, un juego de pruebas y un contrato de prueba?
7. ¿Qué ventajas y desventajas trae un alto nivel de independencia de la prueba?

## Errores comunes — Sección 1
- Confundir verificación con validación — verificación es contra la especificación, validación es contra la necesidad real del usuario; un sistema puede pasar una y fallar la otra.
- Decir que "depurar es parte de probar" — son actividades distintas y complementarias, no la misma cosa.
- Olvidar que un defecto no siempre existe solo en código — puede estar en requisitos, documentación o cualquier artefacto.
- Pensar que el objetivo único de la prueba es "encontrar errores" — es una percepción errónea explícitamente señalada por el syllabus.
- No distinguir bien caso de prueba (con resultados esperados) de guion de prueba (instrucciones de ejecución) — es una confusión común en examen.

---

## 2. La Prueba a lo largo del Ciclo de Vida de Desarrollo de Software

### 2.1 Impacto del CVDS en la prueba
La elección del modelo de CVDS repercute en: alcance y cronología de las actividades de prueba, nivel de detalle de la documentación, elección de técnicas y enfoque, alcance de la automatización, y rol/responsabilidades del probador.

- **Modelos secuenciales** (cascada, modelo V): siguen secuencia lineal. En fases iniciales los probadores participan en revisión de requisitos, análisis y diseño de prueba, pero la **prueba dinámica no puede realizarse hasta que exista código** — este es exactamente el modelo bajo el que probablemente operas en tu contexto automotriz (V-Model), y conecta directamente con lo que ya estudiaste en HIL Testing.
- **Modelos iterativos e incrementales** (espiral, prototipado): en cada iteración existe un prototipo funcional, por lo que se puede probar estática y dinámicamente en todos los niveles desde etapas más tempranas.
- **Modelos ágiles**: iterativos, asumen cambio constante — se favorece documentación ligera y **amplia automatización** para facilitar regresión (scrum, kanban, lean IT).

### 2.2 Buenas prácticas de prueba independientes del CVDS
- Cada actividad de desarrollo tiene su actividad de prueba correspondiente.
- Cada nivel de prueba tiene objetivos específicos y diferentes.
- El análisis y diseño de prueba de un nivel **comienza durante la fase de desarrollo correspondiente** del CVDS, no después.
- Los probadores participan en revisión de productos de trabajo tan pronto como hay borradores disponibles, apoyando la estrategia de **desplazamiento a la izquierda (shift-left)**.

### 2.3 La prueba como impulsor del desarrollo (test-first)
Todos estos enfoques aplican el principio de prueba temprana:
- **Desarrollo Guiado por Prueba (TDD)**: se escriben las pruebas primero, luego el código para satisfacerlas, luego se refactoriza. Casos de prueba creados principalmente en prueba de componente, orientados a código.
- **Desarrollo Guiado por Prueba de Aceptación (ATDD)**: las pruebas se escriben antes de desarrollar, basadas en criterios de aceptación, como parte del diseño de prueba.
- **Desarrollo Guiado por Comportamiento (BDD)**: casos de prueba traducidos automáticamente en pruebas ejecutables, expresando el comportamiento deseado — típicamente con formato dado/cuando/entonces (Given/When/Then).

### 2.4 DevOps y la prueba
DevOps busca sinergia entre desarrollo (incluida prueba) y operaciones, promoviendo: autonomía de equipos, retroalimentación rápida y constante, integración de herramientas, y técnicas como **IC (Integración Continua)** y **EC (Entrega Continua)**.

- **IC**: integrar regularmente los cambios de los desarrolladores en un repositorio compartido — esto conecta directamente con tu experiencia en Jenkins/Docker.
- **EC**: automatizar las entregas para que sean más rápidas y confiables.

**Ventajas desde la prueba**: retroalimentación rápida sobre calidad y regresiones, promueve shift-left, promueve procesos automatizados (CI/CD), reduce pruebas manuales, reduce riesgo de regresión.
**Riesgos**: mantenimiento de automatizaciones y herramientas, costo de implementación. **La prueba manual sigue siendo necesaria** incluso con alto nivel de automatización.

### 2.5 Enfoque de Desplazamiento a la Izquierda (Shift-Left)
Es el nombre que se da al principio de prueba temprana. Buenas prácticas concretas para lograrlo:
1. Revisar la especificación desde la perspectiva de prueba.
2. Redactar casos de prueba **antes** de escribir el código, o ejecutarlo en un arnés de prueba durante la implementación.
3. Realizar pruebas **no funcionales** desde el nivel de prueba de componente (usualmente se dejan para más adelante).
4. Utilizar IC/EC para retroalimentación rápida y pruebas de componente automatizadas.
5. Completar el análisis estático del código fuente **antes** de la prueba dinámica.

Shift-left puede implicar formación, esfuerzo y costes adicionales al inicio, pero se espera que ahorre esfuerzo/costes después — conecta directamente con el principio 3 (la prueba temprana ahorra tiempo y dinero).

### 2.6 Retrospectivas y mejora de proceso
Reuniones (fin de proyecto/iteración/hito) que responden: ¿qué tuvo éxito y debe conservarse?, ¿qué no tuvo éxito y puede mejorarse?, ¿cómo incorporar mejoras y conservar éxitos? Beneficios: aumento de efectividad y calidad, unión/aprendizaje en equipo, mejora de la base de prueba.

### 2.7 Niveles de Prueba (los 5 niveles — pregunta de examen/entrevista muy frecuente)

Los **niveles de prueba** son grupos de actividades organizadas y gestionadas conjuntamente, cada uno una instancia del proceso de prueba en una etapa determinada. Se distinguen por: objeto de prueba, objetivos, base de prueba, defectos/fallos encontrados, y enfoque/responsabilidades.

| Nivel | Base de prueba | Objeto de prueba | Defectos típicos | Responsable |
|---|---|---|---|---|
| **1. Prueba de Componente** (unitaria) | Diseño detallado, modelos de datos, especificaciones de componente | Unidades/módulos, código, estructuras de datos | Funcionamiento incorrecto, problemas de flujo de datos | Desarrollador |
| **2. Prueba de Integración de Componente** | Diagramas de secuencia, arquitectura a nivel de componente | Infraestructura, interfaces, APIs | Incompatibilidad de interfaces, fallos de comunicación entre componentes | Desarrollador |
| **3. Prueba de Sistema** | Especificaciones de requisitos de sistema/SW, historias de usuario | Aplicaciones, sistema completo (SUT) | Flujos de control/datos incorrectos, incapacidad de completar tareas end-to-end, fallo del sistema | Probador |
| **4. Prueba de Integración de Sistema** | Diagramas de secuencia, arquitectura a nivel de sistema | Infraestructura, interfaces, APIs entre sistemas | Incompatibilidad de interfaces, fallos de comunicación entre sistemas | Probador |
| **5. Prueba de Aceptación** | Procesos de negocio, requisitos de usuario/negocio, normativas, contratos | Sistema completo, formularios | Flujos de trabajo que no cumplen requisitos de negocio/usuario, incumplimiento contractual/regulatorio | Implicados/usuarios |

**Notas importantes que suelen ser trampa en examen**:
- La **base de prueba de aceptación** son las **necesidades de negocio del usuario**; la base de prueba de sistema son las **reglas de negocio** — son distintas, no lo mismo.
- Los **fallos en la lógica** de un módulo se encuentran típicamente en **prueba de componente**, no en niveles superiores.
- La prueba de aceptación detecta fallos en el comportamiento del sistema cuando **se desvía de las necesidades de negocio del usuario** — específicamente eso, no cualquier fallo funcional genérico.

*Conexión directa con tu experiencia*: tu trabajo en HIL vive principalmente en **Prueba de Sistema** y **Prueba de Integración de Sistema** — validas el comportamiento completo de una ECU (sistema) y sus interfaces con otras ECUs/sistemas externos (integración de sistema). Vale la pena tener esta ubicación exacta clara para cuando te pregunten "¿en qué nivel de prueba trabajas?".

### 2.8 Tipos de Prueba

A diferencia de los niveles (que son etapas), los **tipos de prueba** están relacionados con **características de calidad específicas**, y la mayoría puede aplicarse en varios niveles de prueba simultáneamente:

- **Prueba funcional**: evalúa **qué debe hacer** el sistema — completitud, corrección y pertinencia funcional.
- **Prueba no funcional**: evalúa **qué tan bien** funciona el sistema — usabilidad, compatibilidad, seguridad, eficiencia de desempeño, mantenibilidad, portabilidad, fiabilidad. El descubrimiento tardío de defectos no funcionales es una amenaza grave para el proyecto; requiere un entorno de prueba muy específico (esto es exactamente lo que haces en pruebas de "runtime performance" y "robustness" en HIL).
- **Prueba de caja negra**: basada en la especificación, sin conocimiento de la estructura interna — comprueba comportamiento contra especificaciones.
- **Prueba de caja blanca**: basada en la estructura/implementación interna — busca cobertura de la estructura subyacente.

### 2.9 Prueba de Confirmación y Prueba de Regresión (distinción clave)
- **Prueba de confirmación**: confirma que un defecto **original** se corrigió con éxito — ejecutando los casos que fallaron previamente, y/o agregando nuevas pruebas para el cambio.
- **Prueba de regresión**: confirma que **no se produjeron consecuencias adversas** por un cambio (incluida una corrección ya confirmada) — pueden afectar al mismo componente, a otros componentes del mismo sistema, o incluso a otros sistemas.
- Se recomienda un **análisis de impacto** primero, para optimizar el alcance de la regresión — mostrando qué partes del software podrían verse afectadas.
- La prueba de regresión es una **firme candidata a automatización** desde fases tempranas del proyecto — esto es exactamente lo que haces con tus suites de Robot Framework/CANoe en cada nuevo build.
- Se necesita confirmación y/o regresión en **todos los niveles de prueba** donde se corrijan defectos o se hagan cambios.

### 2.10 Prueba de Mantenimiento
Cuando el sistema entra en producción y sigue evolucionando, entra en fase de mantenimiento. El **análisis de impacto** puede hacerse **antes** de un cambio, para decidir si debe realizarse, basándose en consecuencias potenciales.

El alcance de las pruebas de mantenimiento depende de: grado de riesgo del cambio, tamaño del sistema existente, magnitud del cambio.

**Desencadenantes del mantenimiento**:
1. **Modificaciones**: mejoras planificadas, cambios correctivos, correcciones en caliente (hotfixes).
2. **Actualizaciones/migraciones del entorno de operación**: cambio de plataforma — requiere pruebas del nuevo entorno **y** del software modificado.
3. **Retirada**: fin de vida útil de una aplicación (ej. procedimientos de restauración/recuperación tras archivado prolongado).

## Preguntas de teoría — Sección 2
1. ¿Por qué en un modelo secuencial (V-Model) la prueba dinámica no puede realizarse al principio del CVDS, aunque otras actividades de prueba sí?
2. Explica la diferencia entre TDD, ATDD y BDD.
3. ¿Qué ventajas y riesgos trae DevOps desde el punto de vista de la prueba?
4. Menciona 3 buenas prácticas concretas para lograr shift-left en un proyecto.
5. Describe los 5 niveles de prueba con su base de prueba y responsable típico.
6. ¿Cuál es la diferencia entre la base de prueba de la prueba de sistema y la de la prueba de aceptación?
7. ¿En qué nivel(es) de prueba se ubica típicamente tu trabajo en HIL, y por qué?
8. Explica la diferencia entre prueba de confirmación y prueba de regresión con un ejemplo de flashing UDS.
9. ¿Cuáles son los 3 tipos de desencadenantes de mantenimiento, y qué tipo de prueba requiere cada uno?

## Errores comunes — Sección 2
- Confundir niveles de prueba (etapas: componente, integración, sistema, etc.) con tipos de prueba (funcional, no funcional, caja negra/blanca) — son dos clasificaciones distintas y ortogonales.
- Decir que la prueba de regresión y la prueba de confirmación son lo mismo — la de confirmación valida que **ese defecto específico** se corrigió; la de regresión valida que **nada más se rompió**.
- Asumir que en un modelo ágil no hay documentación de prueba — hay documentación **ligera**, no ausente.
- Olvidar que DevOps no elimina la necesidad de prueba manual, solo la reduce.

---

## 3. Prueba Estática

### 3.1 Fundamentos
A diferencia de la prueba dinámica, **la prueba estática no requiere ejecutar el software** — se evalúa código, especificaciones, arquitectura u otros productos de trabajo mediante examen manual o con herramientas. Puede aplicarse tanto a **verificación** como a **validación**.

**Objetivos**: mejora de la calidad, detección temprana de defectos, evaluación de características (legibilidad, completitud, corrección, capacidad de ser probado, consistencia). El análisis estático suele requerir menor esfuerzo porque no necesita casos de prueba y suele apoyarse en herramientas.

### 3.2 Productos susceptibles de prueba estática
Casi cualquier producto de trabajo **legible y estructurado**: documentos de especificación de requisitos, código fuente, planes de prueba, casos de prueba, elementos del backlog, contratos de prueba, documentación de proyecto.

**No apropiados**: productos difíciles de interpretar por humanos y que no deben ser analizados por herramientas — ej. código ejecutable de terceros por razones legales.

### 3.3 Valor de la prueba estática
Permite: detectar defectos en fases tempranas del CVDS, identificar defectos que **no pueden detectarse en pruebas dinámicas**, evaluar calidad y generar confianza, mejorar comunicación entre implicados, reducir costo y tiempo de desarrollo.

### 3.4 Prueba estática vs prueba dinámica (tabla comparativa — muy propensa a preguntas de examen)

| Prueba estática | Prueba dinámica |
|---|---|
| Encuentra defectos **directamente** | Provoca **fallos**, a partir de los cuales se determinan los defectos |
| Detecta fácilmente defectos en caminos de código **rara vez ejecutados** | Es difícil detectar errores en líneas de código difíciles de alcanzar |
| Se aplica a productos de trabajo **no ejecutables** | Se aplica a productos **ejecutables** |
| Mide características que **no dependen** de la ejecución (ej. mantenibilidad) | Mide características que **sí dependen** de la ejecución (ej. eficiencia de desempeño) |

**Defectos que se encuentran más fácilmente con prueba estática**: defectos de diseño, defectos en requisitos (ambigüedades, contradicciones, omisiones), defectos de codificación (valores indefinidos, código duplicado), especificaciones incorrectas de interfaz (tipo de parámetro), lagunas o imprecisiones en la cobertura de la base de prueba, y ciertas vulnerabilidades de seguridad (ej. desbordamiento de memoria — buffer overflow, muy relevante si en algún momento hablas de seguridad en sistemas embebidos).

*Nota de examen*: un **vacío en la trazabilidad o cobertura de la base de prueba** es un defecto típico que se encuentra fácilmente en prueba estática, no en dinámica.

### 3.5 Retroalimentación temprana y frecuente
- **Retroalimentación temprana**: permite comunicar posibles problemas de calidad a tiempo, y mantenerse alineado con la visión del implicado.
- **Retroalimentación frecuente**: evita malentendidos sobre requisitos, asegura que los cambios se entiendan e implementen rápido, permite concentrarse en las prestaciones de mayor valor.

### 3.6 El Proceso de Revisión — 5 actividades
1. **Planificación**: alcance, propósito, producto a revisar, características de calidad a evaluar, criterios de salida, esfuerzo y plazos.
2. **Inicio de la revisión**: asegurar que todos los implicados están preparados.
3. **Revisión individual**: cada revisor evalúa por su cuenta, registrando anomalías, recomendaciones y preguntas.
4. **Comunicación y análisis**: las anomalías identificadas **no son necesariamente defectos** — deben analizarse y discutirse (típicamente en reunión), decidiendo estado, propiedad y acciones para cada una.
5. **Corrección y suministro de información**: por cada defecto confirmado, se crea un informe de defecto para hacer seguimiento; se corrigen los defectos.

*Nota de vocabulario*: una **anomalía** es una desviación inesperada o inusual — no todas las anomalías terminan siendo defectos reales.

### 3.7 Roles en las revisiones
- **Gestor**: decide qué se revisa y aporta recursos (personal, tiempo).
- **Autor**: crea y corrige el producto de trabajo.
- **Moderador**: asegura el funcionamiento eficaz de las reuniones.
- **Escriba**: recopila anomalías de los revisores y registra la información.
- **Revisor**: alguien del proyecto, experto en la materia, o cualquier otro implicado.
- **Líder de la revisión**: responsabilidad general — decide quién participa y organiza cuándo/dónde.

### 3.8 Tipos de Revisión (ordenados de menos a más formal)

| Tipo | Formalidad | Dirigida por | Características |
|---|---|---|---|
| **Informal** | Ninguna | — | No sigue proceso ni requiere salida formal; solo busca encontrar defectos |
| **Guiada (walkthrough)** | Semi-formal | El **autor** | Evalúa calidad, educa a revisores, obtiene consenso, genera ideas, motiva/capacita al autor; se hace lluvia de ideas |
| **Técnica** | Formal | Un **moderador** | Realizada por revisores técnicamente calificados; el objetivo es tomar decisiones sobre un problema técnico, además de detectar anomalías |
| **Inspección** | La más formal | — | Sigue el proceso genérico completo; objetivo principal: encontrar el **máximo número de anomalías**; las **listas de comprobación son obligatorias** (en los demás tipos, no lo son) |

El nivel de formalidad depende de: el CVDS seguido, madurez del proceso de desarrollo, criticidad/complejidad del producto, requisitos legales, necesidad de rastro de auditoría — factores que en tu dominio automotriz (ASPICE, funciones de seguridad) empujan naturalmente hacia niveles de formalidad más altos que en software de consumo genérico.

### 3.9 Factores de éxito de las revisiones
Objetivos claros y criterios de salida medibles, elegir el tipo de revisión adecuado, revisar en fragmentos pequeños (mantener concentración), dar retroalimentación a los implicados, dar tiempo suficiente para prepararse, hacer de las revisiones parte de la cultura organizacional, formación adecuada para todos los roles, y facilitar bien las reuniones.

## Preguntas de teoría — Sección 3
1. ¿Por qué la prueba estática puede aplicarse tanto a verificación como a validación?
2. Menciona 3 tipos de defectos que se detectan más fácilmente con prueba estática que con dinámica.
3. Explica la diferencia entre una anomalía y un defecto confirmado.
4. Describe las 5 actividades del proceso de revisión.
5. ¿Cuál es la diferencia entre una revisión guiada y una revisión técnica, en términos de quién la dirige y su propósito?
6. ¿Por qué la inspección es el tipo de revisión más formal, y qué la distingue específicamente de los demás tipos?
7. ¿Por qué crees que en un dominio como el automotriz (ASPICE, funciones de seguridad) suele preferirse un mayor nivel de formalidad en las revisiones?

## Errores comunes — Sección 3
- Decir que la prueba estática "no encuentra defectos reales, solo posibles problemas" — sí encuentra defectos directamente, sin necesidad de provocar un fallo.
- Confundir anomalía con defecto — toda anomalía debe analizarse antes de confirmarse como defecto.
- No saber diferenciar quién dirige cada tipo de revisión (autor en guiada, moderador en técnica) — es una pregunta de examen muy común.
- Olvidar que las listas de comprobación son obligatorias únicamente en inspección, no en los demás tipos de revisión.

---

## 4. Análisis y Diseño de Prueba (Técnicas de Prueba)

> Esta sección es la que más "preguntas trampa" tiene en cualquier examen ISTQB o entrevista técnica seria — las técnicas de caja negra en particular tienen fórmulas de cobertura y reglas de conteo muy específicas que vale la pena dominar con ejemplos numéricos, no solo de memoria.

### 4.1 Introducción a las técnicas de prueba
Las técnicas ayudan al probador en: análisis de prueba (qué probar), diseño de prueba (cómo probar), desarrollar casos de prueba de forma sistemática, definir condiciones de prueba, identificar elementos de cobertura, e identificar datos de prueba.

- **Caja negra**: basada en el comportamiento especificado, sin referencia a la estructura interna — los casos de prueba son independientes de cómo se implemente el software; si la implementación cambia pero el comportamiento se mantiene, los casos siguen siendo válidos. Se basa en entradas/salidas; las historias de usuario se usan como base de prueba.
- **Caja blanca**: basada en la estructura interna y el procesamiento — como depende de cómo está diseñado el software, solo puede crearse **después** del diseño o implementación.
- **Basada en la experiencia**: usa el conocimiento y experiencia del probador — puede detectar defectos que ni caja negra ni caja blanca detectan, por lo que **se complementan**. Se basa en la probabilidad de defectos y su distribución; su base de prueba **no** son los requisitos, sino la experiencia del probador.

### 4.2 Técnicas de Caja Negra

Las 4 técnicas de caja negra más comunes se aplican **a todos los niveles de prueba**:

#### a) Partición de Equivalencia (PE)
Divide los datos (entradas, salidas, elementos de configuración, valores internos, valores relacionados con el tiempo, etc.) en particiones, bajo la premisa de que **todos los elementos de una partición serán procesados de la misma manera** por el objeto de prueba. Si un caso de prueba de una partición detecta un defecto, se espera que cualquier otro valor de esa misma partición también lo detecte.

**Características de las particiones**: continuas o discretas, ordenadas o desordenadas, finitas o infinitas, **no deben sobreponerse**, **no puede haber conjuntos vacíos** (toda partición debe tener al menos un elemento).

- **Partición válida**: contiene valores válidos.
- **Partición no válida**: contiene valores no válidos.
- Cuando no se especifica válida/inválida, se refiere a ambas.

**Fórmula de cobertura**: `Coverage = No. de particiones probadas / No. total de particiones`

**Cada Elección**: criterio de cobertura que exige que los casos de prueba usen **cada partición de cada conjunto de particiones al menos una vez** — pero **no** se combinan exhaustivamente todas las particiones de un campo con todas las de otro, porque el número de combinaciones se volvería exponencial.

*Ejemplo automotriz*: si estás probando el rango válido de un DID (Data Identifier) leído vía UDS 0x22 que representa temperatura del motor en un rango de -40°C a 150°C, tendrías 3 particiones: menor a -40 (inválida), -40 a 150 (válida), mayor a 150 (inválida) — con "Cada Elección" necesitas al menos 3 casos de prueba, uno por partición, no necesariamente combinados con otras variables del sistema.

#### b) Análisis de Valor Frontera (AVF)
Se basa en practicar las **fronteras** de las particiones de equivalencia — por eso **solo puede usarse en particiones ordenadas**. Los valores mínimo y máximo de una partición son su frontera. Los defectos típicos que detecta AVF ocurren cuando las fronteras están mal ubicadas (por encima o por debajo de lo previsto) o se omiten por completo — un clásico error "off-by-one" en el código.

Existen 2 versiones:
- **AVF de 2 valores**: 2 elementos de cobertura por frontera — el valor frontera y su vecino más cercano de la partición adyacente.
- **AVF de 3 valores**: 3 elementos de cobertura por frontera — el valor frontera y sus **2** vecinos (uno de cada lado).

**Fórmula de cobertura**: `Coverage = No. total de valores frontera + vecinos probados / No. total de VF + vecinos`

*Ejemplo de examen tipo (como el de tu documento)*: un sistema de velocidad tiene reglas: ≤50 km/h nada, >50 y ≤55 advertencia, >55 y ≤60 multa, >60 suspensión. Con AVF de 2 puntos, las fronteras son 50/51, 55/56, 60/61 → el conjunto de valores sería **{50, 51, 55, 56, 60, 61}**. Este tipo de ejercicio numérico es exactamente el tipo de pregunta que puede aparecer en una entrevista técnica bien preparada — vale la pena practicar construir estos conjuntos a mano.

#### c) Prueba de Tabla de Decisión
Se usa para probar cómo **diferentes combinaciones de condiciones** dan lugar a diferentes resultados — muy efectiva para documentar **reglas de negocio complejas** (o, en tu contexto, lógica de decisión de un sistema, ej. cuándo activar un DTC según combinaciones de condiciones de sensores).

- **Filas**: condiciones y acciones resultantes.
- **Columnas**: reglas de decisión — cada una define una combinación única de condiciones.
- **Tablas limitadas**: condiciones/acciones con valores booleanos (V/F).
- **Tablas de entrada ampliada**: condiciones/acciones con valores múltiples (discretos, rangos, etc.).

**Notación estándar**: V/1 = verdadero, F/0 = falso, guión (-) = "no importa" (don't care), V/1/X en acciones = se ejecuta, F/0/blanco = no se ejecuta.

**Fórmula de cobertura**: `Coverage = No. de columnas practicadas / No. de columnas factibles` — para cobertura completa se necesita **al menos un caso de prueba por columna**.

**Valor de esta técnica**: identifica sistemáticamente combinaciones que de otro modo podrían pasarse por alto; documenta reglas de negocio complejas; ayuda a encontrar **lagunas o contradicciones en los requisitos** — algo especialmente valioso al analizar especificaciones de lógica de diagnóstico o fault management.

*Nota importante de examen*: algunas columnas de una tabla de decisión pueden **eliminarse** si representan combinaciones que **no ocurrirían en una situación real** (combinaciones lógicamente imposibles dado el contexto del negocio) — esto no reduce la validez de la técnica, es parte de aplicarla con criterio.

#### d) Prueba de Transición de Estado
Modela el comportamiento de un sistema mediante sus **posibles estados** y las **transiciones válidas** entre ellos, usando un **diagrama de transición de estados**. Se asume que las transiciones son instantáneas.

- **Tabla de estados**: modelo equivalente al diagrama — filas = estados, columnas = eventos, celdas = transiciones. A diferencia del diagrama, la tabla **muestra explícitamente las transiciones no válidas** (como celdas vacías) — el diagrama solo muestra lo que sí es válido.
- Un caso de prueba normalmente cubre **varias transiciones**, no solo una.

**Criterios de cobertura** (de más débil a más fuerte):
1. **Cobertura de todos los estados**: `Coverage = No. de estados visitados / No. total de estados` — **la más débil** de las tres.
2. **Cobertura de transiciones válidas** (0-switch): `Coverage = No. de transiciones válidas practicadas / No. total de transiciones válidas`.
3. **Cobertura de todas las transiciones** (válidas + inválidas): `Coverage = No. de trans. válidas + inválidas practicadas / No. total de trans. válidas + inválidas` — **la más fuerte**.

*Conexión directa con tu experiencia*: esta técnica mapea prácticamente 1:1 con lo que ya sabes de **"state management"** en tus pruebas de sistema, y con las sesiones de diagnóstico UDS (Default → Programming → Extended) que estudiamos en el capítulo de Protocolos — es literalmente una máquina de estados con transiciones válidas/inválidas que puedes probar con esta técnica.

### 4.3 Técnicas de Caja Blanca

Las 2 técnicas de caja blanca más comunes:

#### a) Prueba de Sentencia (Statement Testing)
Los elementos de cobertura son **sentencias ejecutables** (cualquier instrucción: asignación de variable, invocación de método, mostrar mensaje, etc. — **no** estructuras de control como tal).

**Fórmula**: `Coverage = No. de sentencias practicadas / No. total de sentencias ejecutables`

**Limitación clave**: alcanzar 100% de cobertura de sentencias **no garantiza** que se haya practicado toda la lógica de decisión — pueden quedar ramas del código sin practicar.

#### b) Prueba de Rama (Branch Testing)
Una **rama** es una transferencia de control entre dos nodos del grafo de flujo de control — puede ser **incondicional** (código en línea recta) o **condicional** (resultado de una decisión). Los elementos de cobertura son las ramas; **no implica conocimiento del dominio** (es puramente estructural).

**Fórmula**: `Coverage = No. de ramas practicadas / No. total de ramas`

**Relación clave**: la **cobertura de rama subsume la cobertura de sentencia** — cualquier conjunto de casos de prueba que logre 100% de cobertura de rama también logra 100% de cobertura de sentencia (lo inverso **no** es cierto).

*Ejemplo del propio documento (vale la pena memorizarlo exactamente así, porque es un ejemplo de examen clásico)*:
```
If (x > y):
    hacer_tal_cosa()
```
- **Cobertura de sentencias**: necesita **1** caso de prueba para 100% (con x > y verdadero, se ejecutan ambas sentencias — la condición y la llamada).
- **Cobertura de rama**: necesita **2** casos de prueba para 100% — uno donde la condición es verdadera (rama condicional tomada) y uno donde es falsa (rama incondicional / "salto" de la condición).

**Nota de examen sutil**: si el código tiene **una única sentencia `if` sin bucles ni sentencias CASE**, la afirmación correcta es: **"cualquier caso de prueba individual hace que el resultado de la sentencia `if` sea verdadero o falso"** — no que automáticamente dé 50% de cobertura de decisión de forma garantizada por defecto; depende de cuál resultado se ejecute.

**100% de cobertura de rama implica 100% de cobertura de decisión** — un *decision outcome* es literalmente una rama condicional, por eso están directamente relacionados (para condiciones binarias simples).

### 4.4 Valor de la prueba de caja blanca
**Ventajas**: considera toda la implementación real, facilitando detectar defectos incluso cuando la especificación es vaga, obsoleta o incompleta.
**Desventajas**: si el software **no implementa** uno o más requisitos (defecto de omisión), la caja blanca **no lo detecta** — porque no hay código para esa funcionalidad faltante, no hay nada que cubrir.

- Las técnicas de caja blanca también pueden usarse en **prueba estática** (ej. simulacros de código) — son adecuadas para revisar código que aún no está listo para ejecutarse.
- La caja negra por sí sola **no da una medida real de cobertura de código**; las medidas de caja blanca sí dan una medición objetiva.
- Si hay un **bucle** en el código, puede haber un número infinito de caminos posibles — por lo tanto **no es posible ejecutar todos los caminos** (conecta con el principio 2: la prueba exhaustiva es imposible).

### 4.5 Técnicas Basadas en la Experiencia

#### a) Predicción de Errores
Anticipa errores/defectos/fallos basándose en: cómo ha funcionado la aplicación en el pasado, tipos de errores típicos de los desarrolladores y sus defectos resultantes, y tipos de fallos ocurridos en aplicaciones similares.

Categorías típicas de errores/defectos/fallos: entrada (parámetros erróneos/ausentes), salida (formato/resultado incorrecto), lógica (casos ausentes, operador erróneo), cálculo (operando/cálculo incorrecto), interfaces (desajuste de parámetros), datos (inicialización/tipo incorrecto).

Los **ataques de defecto** son un enfoque metódico de predicción de errores: el probador crea/adquiere una lista de posibles errores/defectos/fallos y diseña pruebas para identificarlos o provocarlos.

#### b) Prueba Exploratoria
Las pruebas se diseñan, ejecutan y evalúan **simultáneamente** mientras el probador aprende sobre el objeto de prueba — útil para explorar más a fondo o cubrir áreas no probadas.

- A veces se estructura como **prueba basada en sesión**: dentro de un marco temporal definido, usando un **contrato de prueba** con los objetivos que guían la sesión, seguida de una recapitulación con los implicados.
- Es especialmente útil cuando las especificaciones son escasas/inadecuadas, o hay presión de tiempo importante.
- **Nota de examen importante**: la prueba exploratoria son **pruebas dinámicas y NO son pruebas aleatorias** — sí hay documentación (el contrato de prueba y los hallazgos registrados).

#### c) Prueba Basada en Lista de Comprobación
El probador diseña/implementa/ejecuta pruebas para cubrir condiciones de una lista de comprobación construida con base en la experiencia.

**Reglas de buena práctica para las listas**: no deben contener elementos comprobables automáticamente por otras vías, ni elementos más adecuados como criterios de entrada/salida, ni elementos demasiado generales. Suelen formularse como preguntas, y cada elemento debe poder comprobarse por separado y directamente. Deben **actualizarse periódicamente** (los desarrolladores aprenden a evitar errores pasados, así que ciertos elementos van perdiendo efectividad) y **evitar que la lista se haga excesivamente larga**.

### 4.6 Enfoques de Prueba Basados en la Colaboración

A diferencia de las técnicas anteriores (enfocadas en detección de defectos), los enfoques colaborativos se concentran en **prevenir** defectos mediante colaboración y comunicación.

#### a) Redacción Colaborativa de Historias de Usuario
Una historia de usuario tiene 3 aspectos críticos (las "3 C"):
- **Cuartilla (Card)**: el medio que describe la historia.
- **Conversación (Conversation)**: explica cómo se usará el software (documentada o verbal).
- **Confirmación (Confirmation)**: los criterios de aceptación.

**Formato habitual**: *"Como [rol], quiero que se cumpla [objetivo], para que pueda [valor de negocio para el rol]"*, seguido de criterios de aceptación.

**Buenas historias de usuario son INVEST**: Independiente, Negociable, Valioso/a, Estimable, Pequeño (Small), Capacidad de Prueba (Testable). Escritas por el dueño del producto con ayuda de usuarios/clientes/testing; las prioridades las determina el representante de negocio junto con el equipo de desarrollo.

#### b) Criterios de Aceptación
Condiciones que debe cumplir la implementación para ser aceptada por los implicados — suelen surgir de la "Conversación". **Mínimo un caso de prueba por criterio de aceptación**.

**Se usan para**: definir el alcance de la historia, alcanzar consenso entre implicados, describir escenarios positivos y negativos, servir de base para la prueba de aceptación de usuario, permitir planificación/estimación precisa.

**Formas de redactarlos**:
1. **Orientado al escenario** (usando BDD — dado/cuando/entonces).
2. **Orientado a reglas** (lista de verificación, tabla entrada/salida).

#### c) Desarrollo Guiado por Prueba de Aceptación (ATDD) — proceso completo
Enfoque "probar primero": los casos de prueba se crean **antes** de implementar la historia de usuario, por miembros del equipo con **diferentes perspectivas**.
1. **Taller de especificación**: el equipo analiza, debate y redacta la historia de usuario y sus criterios de aceptación.
2. **Creación de casos de prueba**: en equipo o individualmente, basados en los criterios de aceptación (funcionan como ejemplos de cómo debe comportarse el software). **Primero se crean los casos positivos, luego los negativos.**
3. Finalmente, el equipo cubre también las **características no funcionales**.

Los casos de prueba deben cubrir **todas** las características de la historia sin ir más allá de ella, y **no debe haber dos casos de prueba que describan la misma característica**.

## Preguntas de teoría — Sección 4
1. ¿Por qué las técnicas basadas en experiencia se consideran complementarias a las de caja negra y caja blanca, y no sustitutas?
2. Con el ejemplo de un DID de temperatura de motor entre -40°C y 150°C, identifica las particiones válidas/inválidas y calcula cuántos casos mínimos necesitas con "Cada Elección".
3. Explica la diferencia entre AVF de 2 valores y AVF de 3 valores, y construye el conjunto de valores frontera para un rango de velocidad de 0 a 200 km/h con una frontera en 120 km/h, usando AVF de 2 puntos.
4. ¿Por qué en una tabla de decisión pueden eliminarse ciertas columnas sin afectar la validez de la técnica?
5. Ordena, de más débil a más fuerte, los 3 criterios de cobertura de prueba de transición de estado, y explica por qué el orden es ese.
6. Con el ejemplo `If (x > y): hacer_tal_cosa()`, explica por qué la cobertura de sentencia necesita 1 caso y la de rama necesita 2.
7. ¿Por qué 100% de cobertura de sentencia NO garantiza 100% de cobertura de rama, pero 100% de cobertura de rama SÍ garantiza 100% de cobertura de sentencia?
8. ¿Por qué la prueba de caja blanca no puede detectar defectos de omisión (requisitos no implementados)?
9. ¿Por qué la prueba exploratoria no debe considerarse "prueba aleatoria sin documentación"?
10. ¿Qué significa que una historia de usuario sea INVEST, y por qué "Testable" es parte de esos criterios?
11. En ATDD, ¿por qué se crean primero los casos de prueba positivos y después los negativos?

## Errores comunes — Sección 4
- Aplicar "Cada Elección" combinando exhaustivamente todas las particiones de todos los campos — el criterio exige cubrir cada partición **al menos una vez**, no todas las combinaciones (eso sería exponencial e innecesario).
- Confundir AVF de 2 valores con AVF de 3 valores al construir el conjunto de valores frontera — 2 valores = frontera + 1 vecino; 3 valores = frontera + 2 vecinos (uno de cada lado).
- Pensar que la cobertura de todos los estados es la más fuerte de las 3 en prueba de transición de estado — es la **más débil**; la de todas las transiciones (válidas + inválidas) es la más fuerte.
- Asumir que 100% de cobertura de sentencia es suficiente para considerar el código bien probado — no garantiza cobertura de decisión/rama.
- Decir que la prueba exploratoria es informal y sin documentación — sí tiene documentación (contrato de prueba, hallazgos, recapitulación), lo que la distingue de simplemente "usar el sistema al azar".
- Olvidar que la base de prueba de las técnicas basadas en experiencia **no** son los requisitos formales, a diferencia de caja negra.

---

## 5. Gestión de las Actividades de Prueba

### 5.1 Propósito y contenido de un Plan de Prueba
Un plan de prueba describe objetivos, recursos y procesos de un proyecto de prueba. Documenta medios y calendario para alcanzar los objetivos, ayuda a asegurar que las actividades cumplirán los criterios establecidos, sirve como medio de comunicación, y demuestra alineación con la política y estrategia de prueba existentes.

**Contenido habitual**: implicados (roles, responsabilidades, necesidades de contratación/formación), contexto de la prueba (alcance, objetivos, restricciones, base de prueba), supuestos y restricciones, comunicación (formas/frecuencia, plantillas), **enfoque de prueba** (niveles, tipos, técnicas, entregables, criterios de entrada/salida, independencia, requisitos de entorno), registro de riesgos, presupuesto y calendario.

### 5.2 Contribución del probador a la planificación (contexto Ágil)
- **Planificación de entrega (release)**: se anticipa al lanzamiento. Contribución del probador: redactar historias de usuario, definir criterios de aceptación comprobables, participar en análisis de riesgo, estimar esfuerzo de prueba, determinar enfoque de prueba, planificar la prueba de la entrega.
- **Planificación de iteración**: se enfoca en una única iteración. Contribución del probador: análisis de riesgo de la iteración, determinar la capacidad de ser probadas de las historias, desglosar historias en tareas, estimar esfuerzo, identificar/refinar aspectos funcionales y no funcionales.

### 5.3 Criterios de Entrada y Criterios de Salida
- **Criterios de entrada**: precondiciones para emprender una actividad — si no se cumplen, la actividad se vuelve más difícil, lenta, costosa y arriesgada. Ejemplos: disponibilidad de recursos (personas, herramientas, entornos, presupuesto, tiempo), disponibilidad de material de prueba (requisitos comprobables, casos de prueba), y nivel inicial de calidad (ej. pruebas de humo completadas satisfactoriamente).
- **Criterios de salida**: lo que debe lograrse para declarar completada una actividad — en Ágil se conoce como **"Definición de Hecho" (Definition of Done)**. Ejemplos: medidas de completitud (cobertura, defectos sin resolver, densidad de defectos), criterios de compleción (pruebas planificadas ejecutadas, regresión automatizada), y **quedarse sin tiempo/presupuesto también es un criterio de salida válido**.
- Deben definirse para **cada nivel de prueba**, y diferirán según los objetivos.

### 5.4 Técnicas de Estimación del Esfuerzo
Predecir cuánto trabajo de prueba se necesita — siempre sujeto a errores de estimación. **Descomponer tareas grandes en pequeñas mejora la precisión**.

- **Basada en proporciones**: usa métricas históricas de proyectos anteriores de la organización.
- **Extrapolación**: mide lo antes posible en el proyecto actual y extrapola (con un modelo matemático) el esfuerzo restante — adecuada para CVDS iterativo.
- **Delphi Banda Ancha**: técnica iterativa basada en expertos — cada uno estima aislado, se discuten resultados, se vuelve a estimar aislado, hasta llegar a consenso. El **poker de planificación** es la variante ágil.
- **Tres puntos**: expertos dan 3 estimaciones (optimista Po, más probable Pm, pesimista Pp):
  - `E = (Po + 4Pm + Pp) / 6`
  - `SD = (Pp - Po) / 6`
  - `Estimación final = E ± SD`

### 5.5 Priorización de Casos de Prueba
Una vez organizados en juegos de prueba, se ordenan en un calendario de ejecución:
- **Basada en riesgo**: se ejecutan primero los casos que cubren los riesgos más importantes.
- **Basada en cobertura**: se ejecutan primero los que logran mayor cobertura (variante: **priorización de cobertura adicional** — el siguiente caso es siempre el que logra la mayor cobertura *adicional*, no la mayor cobertura absoluta).
- **Basada en requisitos**: se ejecutan primero los relacionados con los requisitos más importantes (prioridad definida por los implicados).

Si un caso de prioridad alta **depende** de uno de prioridad baja, el de prioridad baja debe ejecutarse primero. También debe considerarse la disponibilidad de recursos (herramientas, entornos, personas con disponibilidad limitada).

### 5.6 Pirámide de Prueba
Modelo que muestra que distintas pruebas tienen distinta **granularidad**. A mayor altura en la pirámide: menor granularidad, menor aislamiento, mayor tiempo de ejecución.
- **Capa inferior**: pruebas pequeñas, aisladas, rápidas — se necesitan **muchas** para cobertura razonable.
- **Capa superior**: pruebas complejas, de alto nivel, end-to-end — se necesitan **pocas** para cobertura razonable, pero son más lentas.

Ejemplos de nomenclatura de 3 capas: (unitarias, servicio, interfaz de usuario) o (unitarias, integración, extremo a extremo) — el número y nombres de capas puede variar entre organizaciones.

### 5.7 Cuadrantes de Prueba (contexto Ágil)
Agrupan niveles de prueba con tipos, actividades, técnicas y productos de trabajo apropiados en desarrollo ágil. Dos ejes: orientado al negocio vs a la tecnología; apoya al equipo (guía el desarrollo) vs critica el producto (mide comportamiento vs expectativas).

| Cuadrante | Orientación | Función | Ejemplos |
|---|---|---|---|
| **Q1** | Tecnología | Apoya al equipo | Prueba de componente e integración de componentes — deberían automatizarse e integrarse en IC |
| **Q2** | Negocio | Apoya al equipo | Pruebas funcionales (usuario, API) — comprueban criterios de aceptación, manuales o automatizadas |
| **Q3** | Negocio | Critica al producto | Prueba exploratoria, usabilidad, aceptación de usuario — orientadas al usuario, manuales |
| **Q4** | Tecnología | Critica al producto | Pruebas de humo y no funcionales (excepto usabilidad) — automatizadas |

### 5.8 Gestión de Riesgo

**Actividades principales**: **Análisis de riesgo** (identificación + evaluación) y **Control del riesgo** (mitigación + monitorización).

Un **riesgo** es un acontecimiento/peligro/amenaza/situación potencial cuya ocurrencia causa un efecto adverso, caracterizado por:
- **Probabilidad** (entre 0 y 1, sin llegar a ninguno de los extremos).
- **Impacto** (daño/consecuencias).
- Ambos definen el **nivel de riesgo** — a mayor nivel, más importante su tratamiento.

**Riesgos de Proyecto** (gestión y control del proyecto): problemas de organización (retrasos, estimaciones inexactas), de personal (competencias insuficientes, conflictos, escasez), técnicos (corrupción de alcance, soporte deficiente de herramientas), y con proveedores (fallo de entrega de terceros, quiebra) — este último es **directamente relevante para tu experiencia colaborando con proveedores de software**.

**Riesgos de Producto** (características de calidad del producto): falta de funcionalidad o funcionalidad incorrecta, cálculos erróneos, errores de ejecución, arquitectura deficiente, tiempo de respuesta inadecuado, mala experiencia de usuario. Consecuencias: insatisfacción del usuario, pérdida de ingresos/confianza/reputación, daños a terceros, altos costes de mantenimiento, **sanciones penales**, y **daños físicos o muerte** — de nuevo, no abstracto en tu dominio.

### 5.9 Análisis del Riesgo de Producto
Objetivo: proporcionar conciencia del riesgo para concentrar el esfuerzo de prueba, minimizando el riesgo residual. Idealmente comienza **temprano** en el CVDS.

- **Identificación del riesgo**: generar lista exhaustiva — técnicas como tormenta de ideas, talleres, entrevistas, diagramas causa-efecto.
- **Evaluación del riesgo**: categorizar, determinar probabilidad/impacto/nivel, priorizar, proponer forma de gestión.
  - **Enfoque cuantitativo**: `Riesgo = Probabilidad × Impacto`.
  - **Enfoque cualitativo**: matriz de riesgo.

**Los resultados se usan para**: determinar alcance de la prueba, determinar niveles/tipos de prueba, determinar técnicas y cobertura necesaria, estimar esfuerzo, priorizar para encontrar defectos críticos antes, y determinar si se necesitan actividades adicionales a la prueba para reducir riesgo.

### 5.10 Control del Riesgo de Producto
- **Mitigación**: implementar las acciones propuestas para reducir el nivel de riesgo.
- **Monitorización**: asegurar que las acciones son efectivas, mejorar la evaluación, identificar riesgos emergentes.

**Opciones de respuesta a un riesgo**: mitigación mediante prueba, aceptación del riesgo (cuando está fuera de control, ej. preferencias del cliente), transferencia del riesgo (ej. comprar un seguro en vez de mitigar directamente), o plan de contingencia.

**Medidas de mitigación mediante la prueba**: seleccionar probadores con el nivel adecuado de experiencia, aplicar independencia de prueba adecuada, realizar revisiones y análisis estático, aplicar técnicas y niveles de cobertura adecuados, aplicar los tipos de prueba adecuados, realizar pruebas dinámicas incluyendo regresión.

### 5.11 Monitorización, Control y Compleción de la Prueba
- **Monitorización de la prueba**: recopila información para evaluar avance y medir si se cumplen criterios de salida (cobertura de riesgos de producto, requisitos, criterios de aceptación).
- **Control de la prueba**: usa esa información para dar **directivas de control** — acciones correctivas. Ejemplos: re-priorizar pruebas cuando un riesgo se convierte en problema, reevaluar criterios de entrada/salida por repetición de trabajo, ajustar calendario por retraso en el entorno, añadir recursos donde se necesiten.
- **Compleción de la prueba**: recopila datos de actividades finalizadas — ocurre en hitos del proyecto (fin de prueba, liberación de un sistema, fin de una entrega de mantenimiento).

### 5.12 Métricas de Prueba
Se recopilan para mostrar avance vs calendario/presupuesto, calidad actual, y efectividad de las actividades:
- **Métricas de avance del proyecto**: compleción de tareas, uso de recursos, esfuerzo.
- **Métricas de avance de la prueba**: implementación de casos, preparación del entorno.
- **Métricas de calidad del producto**: tiempo de respuesta, disponibilidad.
- **Métricas de defectos**: densidad de defectos, número/prioridad.
- **Métricas de riesgo**: nivel de riesgo residual.
- **Métricas de cobertura**: cobertura de requisitos, cobertura de código.
- **Métricas de coste**: coste de la prueba, coste organizativo de la calidad.

### 5.13 Informes de Prueba

- **Informes de avance de prueba** (durante monitorización/control): apoyan el control continuo; deben tener suficiente información para modificar calendario/recursos/plan cuando haya desviación. Incluyen: periodo de prueba, avance, obstáculos y soluciones, métricas, riesgos nuevos/modificados, y prueba planificada para el siguiente periodo.
- **Informes de compleción de prueba** (durante compleción): resumen una etapa específica, informan sobre pruebas posteriores. Incluyen: resumen de la prueba, evaluación de la prueba y calidad del producto, desviaciones del plan, obstáculos y soluciones, métricas basadas en informes de avance, riesgos no mitigados, lecciones aprendidas.

*Nota de examen*: **no** se incluye en un informe de compleción la definición de criterios de paso/fallo ni objetivos de prueba (eso pertenece a la fase de planificación, no a la de cierre).

**Comunicación del estado de prueba**: verbal, cuadros de mando/tableros de tareas, canales electrónicos, documentación en línea, informes formales (ideales cuando la comunicación no es directa).

### 5.14 Gestión de la Configuración (GC)
Disciplina para **identificación, control y seguimiento** de productos de trabajo (planes, estrategias, condiciones y casos de prueba). Registra elementos, sus relaciones y versiones. Un elemento aprobado para prueba se convierte en **línea base**, y solo puede modificarse mediante un proceso formal de control de cambios. La GC mantiene registro de elementos modificados al crear una nueva línea base, permitiendo **volver a una línea base anterior** para reproducir resultados previos.

**Asegura**: identificación única de todos los elementos de configuración (incluidos elementos de prueba), control de versión, seguimiento de cambios, relación entre elementos para mantener trazabilidad, y referencia inequívoca de elementos de documentación/software en la documentación de prueba.

### 5.15 Gestión de Defectos
Las anomalías reportadas pueden resultar defectos reales u otra cosa — se resuelve durante el tratamiento del informe. Pueden reportarse en cualquier fase del CVDS.

**El proceso mínimo incluye**: un flujo de trabajo desde descubrimiento hasta cierre, y reglas de clasificación. Actividades: registrar, analizar/clasificar, decidir respuesta (arreglar o mantener tal cual), y cerrar el informe. Los defectos de pruebas estáticas también deben tratarse de forma similar.

**Un informe de defecto (de prueba dinámica) típicamente incluye**:
- Identificador único
- Título con breve resumen
- Fecha, organización emisora, autor (y su rol)
- Identificación del objeto de prueba y entorno de prueba
- Contexto del defecto (fase del CVDS, datos de prueba usados)
- Descripción del fallo (para permitir reproducción) y registros de prueba
- Resultados esperados y resultados reales
- Severidad del defecto
- Prioridad de corrección
- Estado del defecto
- Referencias

*Nota importante de examen*: **Severidad** y **Prioridad** son campos distintos — severidad mide el impacto técnico/de negocio del defecto, prioridad determina el orden en que debe corregirse; no siempre van de la mano (un defecto de baja severidad puede tener alta prioridad por razones contractuales, por ejemplo).

## Preguntas de teoría — Sección 5
1. ¿Cuál es la diferencia entre criterios de entrada y criterios de salida, y por qué "quedarse sin tiempo" puede ser un criterio de salida válido?
2. Aplica la técnica de Tres Puntos con Po=5, Pm=8, Pp=15 y calcula E y SD.
3. ¿Cuál es la diferencia entre priorización basada en riesgo, en cobertura y en requisitos?
4. Describe la pirámide de prueba y por qué la capa superior necesita menos pruebas que la inferior para lograr cobertura razonable.
5. Ubica en los 4 cuadrantes de prueba: pruebas de componente automatizadas, prueba exploratoria, pruebas de humo, y pruebas de aceptación de usuario.
6. ¿Cuál es la fórmula del enfoque cuantitativo de evaluación de riesgo, y qué alternativa cualitativa existe?
7. Menciona las 4 opciones de respuesta a un riesgo identificado, con un ejemplo de cada una.
8. ¿Qué diferencia hay entre un informe de avance de prueba y un informe de compleción de prueba?
9. ¿Por qué la gestión de configuración permite "volver a una línea base anterior", y por qué es valioso en un contexto de regresión de HIL?
10. ¿Cuál es la diferencia entre severidad y prioridad de un defecto? Da un ejemplo donde no coincidan.

## Errores comunes — Sección 5
- Confundir severidad con prioridad de un defecto — no son intercambiables ni siempre correlacionados.
- Pensar que un informe de compleción de prueba define criterios de paso/fallo — esos se definen en planificación, no en el cierre.
- Aplicar la fórmula de Tres Puntos incorrectamente (olvidar el peso 4 en el valor más probable, o invertir el orden de Po/Pp en el cálculo de SD).
- Creer que "priorización basada en cobertura" siempre ejecuta primero el caso de mayor cobertura absoluta — la variante de cobertura *adicional* prioriza distinto tras el primer caso.
- No distinguir riesgos de proyecto (afectan calendario/presupuesto/alcance) de riesgos de producto (afectan la calidad del producto en sí) — son categorías con implicaciones de gestión distintas.

---

## 6. Herramientas de Prueba

### 6.1 Clasificación de herramientas
- **Herramientas de gestión**: gestión del CVDS, requisitos, pruebas, defectos y configuración.
- **Herramientas de prueba estática**: revisiones y análisis estático.
- **Herramientas de diseño e implementación**: generación de casos y procedimientos de prueba.
- **Herramientas de ejecución y cobertura**.
- **Herramientas de pruebas no funcionales**.
- **Herramientas DevOps**: apoyan el pipeline de entrega, seguimiento de flujo de trabajo, procesos de build automatizados, CI/CD — aquí caen directamente Jenkins/Docker de tu stack.
- **Herramientas de colaboración**.
- **Herramientas de apoyo a la escalabilidad**: máquinas virtuales, contenerización.
- Cualquier otra herramienta que apoye la prueba (en tu caso: CANoe, vTESTstudio, Robot Framework caerían principalmente en diseño/implementación y ejecución).

### 6.2 Automatización de Prueba
El uso de software para realizar o apoyar actividades de prueba.

**Ventajas**: ahorro de tiempo al reducir trabajo manual repetitivo, evaluación más objetiva (y medidas demasiado complejas para humanos), retroalimentación más rápida, y más tiempo para que los probadores diseñen pruebas nuevas, profundas y efectivas.

**Riesgos**: estimaciones imprecisas del tiempo de automatización, usar una herramienta cuando la prueba manual sería más apropiada, confiar demasiado en la herramienta e ignorar el pensamiento crítico, dependencia del proveedor de la herramienta, y que la herramienta no sea compatible con la plataforma de desarrollo.

*Nota directamente relevante para tu experiencia*: este es exactamente el balance que manejas a diario entre Robot Framework/CANoe (automatización) y las pruebas exploratorias/manuales que aún requieren tu juicio como probador — el syllabus es explícito en que la automatización **no elimina** la necesidad de pensamiento crítico humano.

## Preguntas de teoría — Sección 6
1. ¿En qué categoría de herramientas ubicarías Jenkins, y por qué?
2. Menciona 3 riesgos de la automatización de prueba y cómo los mitigarías en un entorno HIL real.
3. ¿Por qué "confiar demasiado en la herramienta" se considera un riesgo, y no solo una ventaja de eficiencia?

## Errores comunes — Sección 6
- Pensar que automatizar es siempre mejor que probar manualmente — el syllabus señala explícitamente que usar una herramienta cuando la prueba manual es más apropiada es un riesgo real.
- Ignorar la dependencia del proveedor como un riesgo estratégico, no solo técnico.