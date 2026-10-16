# Capítulo: Requirements Engineering

> Objetivo del capítulo: este es un tema donde ya tienes experiencia de herramienta (DOORS/RQM, Jira, Confluence) y ahora también una base formal de un curso de Requirements Engineering (con enfoque INCOSE/Systems Engineering). Este capítulo integra ambas cosas: el vocabulario formal de la disciplina (necesidades vs requisitos, los 4 procesos de RE, las 9 características de un buen requisito, los métodos de verificación) con tu experiencia práctica en automotriz. El objetivo es que puedas hablar de "análisis de requisitos" con el mismo rigor con el que ya hablas de protocolos o HIL.

---

## 0. Fundamentos de Systems Engineering (el marco donde vive Requirements Engineering)

Antes de entrar a Requirements Engineering en sí, vale la pena anclarlo dentro de la disciplina más amplia de la que forma parte, porque muchas entrevistas de nivel Test Function Lead/Manager empiezan preguntando por este contexto general antes de bajar al detalle de requisitos.

### 0.1 Qué es un Sistema y qué es Systems Engineering
- **Sistema**: un conjunto de elementos en interacción — no son las partes individuales lo que define un sistema, sino cómo interactúan entre sí para producir un comportamiento que ninguna parte podría lograr sola.
- **Systems Engineering (Ingeniería de Sistemas)**: la disciplina enfocada en asegurar que esas piezas **encajen y funcionen juntas** para cumplir los objetivos del sistema completo, a lo largo de todo su ciclo de vida — no solo durante el desarrollo, sino desde el concepto hasta el retiro del sistema.

*Conexión directa con tu experiencia*: cuando pruebas una ECU dentro de un banco HIL que simula el resto del vehículo, estás literalmente validando si esa pieza "encaja y funciona" dentro del sistema completo — Systems Engineering es el marco conceptual que explica por qué ese tipo de prueba (integración de sistema) es necesaria y no basta con probar la ECU de forma aislada (prueba de componente).

### 0.2 Etapas del Ciclo de Vida de Systems Engineering
Para gestionar el progreso de un sistema, su vida completa se divide en etapas:
1. **Concepto (Concept Stage)**: se identifica el problema, se exploran necesidades y posibles soluciones a alto nivel.
2. **Desarrollo (Development Stage)**: se diseña, construye y verifica el sistema — aquí vive la mayor parte de tu trabajo como probador.
3. **Producción (Production Stage)**: se fabrica el sistema a escala.
4. **Utilización/Soporte (Utilization/Support Stage)**: el sistema opera en campo y se mantiene — esto conecta directamente con la **Prueba de Mantenimiento** que ya estudiaste en Fundamentos de Testing.
5. **Retiro (Retirement Stage)**: el sistema se da de baja — conecta con el desencadenante de mantenimiento de "retirada" que también ya viste.

### 0.3 Dominio del Problema vs Dominio de la Solución
Un principio central de Systems Engineering, y posiblemente el error más común que cometen equipos técnicos (incluidos equipos de prueba): **saltar directamente a diseñar una solución antes de entender completamente el problema**.
- **Dominio del problema (Problem Domain)**: entender qué necesidad real existe, sin todavía comprometerse a una solución técnica específica.
- **Dominio de la solución (Solution Domain)**: una vez entendido el problema, se exploran y seleccionan soluciones técnicas concretas.

Establecer y entender bien el problema es **el primer paso** antes de hacer cualquier ingeniería de sistemas seria — un requisito mal concebido porque el equipo saltó directo a "la solución" sin entender el problema real es una causa raíz de proyectos completos que terminan resolviendo el problema equivocado (aunque lo resuelvan "bien" técnicamente).

### 0.4 Proyectos Greenfield vs Brownfield
- **Greenfield ("campo verde")**: un proyecto que inicia desde cero, sin sistema previo que condicione el diseño.
- **Brownfield ("campo marrón")**: un proyecto que se construye sobre un sistema ya existente — debe considerar restricciones, compatibilidad e integración con lo que ya está en producción.

*Conexión directa con tu experiencia*: la mayoría de tu trabajo en automotriz (evolucionar una plataforma de infotainment/ADAS ya existente, agregar soporte para un nuevo protocolo sobre una arquitectura ya establecida) es típicamente **Brownfield** — esto tiene implicaciones reales en cómo se escriben los requisitos: deben ser explícitos sobre qué debe **mantenerse compatible** con el sistema existente, no solo qué debe hacer la nueva función.

---

## 1. Qué es Requirements Engineering y por qué importa

**Requirements Engineering (Ingeniería de Requisitos)** es la disciplina que se encarga de **descubrir, analizar, comunicar y gestionar** las necesidades y condiciones que un sistema debe satisfacer, a lo largo de todo su ciclo de vida.

Los requisitos cumplen una función muy concreta y práctica: te dicen directamente **el criterio que debe cumplirse** — en la práctica, se convierten en la **lista de pendientes (to-do list)** de todo el proyecto. Si un elemento no está en los requisitos (o no se deriva trazablemente de ellos), formalmente no es parte del alcance del proyecto.

**Por qué es tan crítico** (esto conecta directamente con lo que ya viste en Fundamentos de Testing): un porcentaje enorme de los defectos que se encuentran en producción tienen su origen en requisitos ambiguos, incompletos o contradictorios — no en errores de código. Recuerda que en el capítulo de Prueba Estática vimos que las **ambigüedades, contradicciones y omisiones en requisitos** son de los defectos que más fácilmente se detectan con revisión (prueba estática), precisamente porque ocurren antes de que exista una sola línea de código.

### 1.1 El Proceso de Requirements Engineering (4 actividades formales)
El marco formal de Requirements Engineering (alineado con INCOSE) describe el proceso en **4 actividades**, que se solapan e iteran y son aplicables **en todos los niveles** del desarrollo del sistema (stakeholder, sistema, subsistema, software):

1. **Elicitación (Requirements Elicitation)**: descubrir las necesidades reales de los stakeholders.
2. **Análisis (Requirements Analysis)**: transformar esas necesidades en requisitos bien estructurados, resolver conflictos y ambigüedades.
3. **Comunicación (Requirements Communication)**: documentar y compartir los requisitos con todos los implicados de forma que puedan entenderlos, revisarlos y actuar sobre ellos — esto incluye tanto la **especificación formal** como la **validación** con los stakeholders de que lo documentado refleja lo que realmente necesitan.
4. **Gestión (Requirements Management)**: control de versiones, trazabilidad y gestión de cambios a lo largo de todo el ciclo de vida.

*Nota de reconciliación de vocabulario*: en la práctica (y en algunos glosarios, incluido ISTQB) verás esta misma idea descrita como 5 actividades separando Especificación y Validación — ambos enfoques son equivalentes, la diferencia es si "Comunicación" se trata como una sola actividad combinada o como dos actividades distintas. Lo importante para entrevista es que entiendas **qué pasa en cada paso**, no memorizar un número exacto de pasos como si fuera único y absoluto.

Estas 4 actividades **se aplican en cada nivel de desarrollo del sistema** — eliciteas, analizas, comunicas y gestionas requisitos de stakeholder, luego repites el ciclo para requisitos de sistema, luego para subsistema, y así sucesivamente — no es un proceso que ocurre una sola vez al inicio del proyecto.

---

## 2. Stakeholders, Necesidades y Requisitos (Module 2 del curso)

### 2.1 Terminología clave: Stakeholder, Necesidades vs Requisitos
- **Stakeholder (Implicado)**: cualquier persona, grupo u organización con un interés legítimo en el sistema — puede ser el usuario final, el OEM, un regulador, un equipo interno, un proveedor.
- **Necesidad (Need)**: lo que un stakeholder **quiere o espera** del sistema, expresado frecuentemente en su propio lenguaje, sin estructura formal — "quiero que el auto me avise si me voy a quedar sin frenos a tiempo".
- **Requisito (Requirement)**: la **transformación estructurada y verificable** de esa necesidad en una declaración formal que el sistema debe cumplir — "el sistema debe activar una alerta audible y visual cuando el nivel de líquido de frenos esté por debajo del umbral X".

**La distinción Necesidad vs Requisito es una de las preguntas más comunes de entrevista en esta disciplina**: una necesidad puede ser vaga, emocional, contradictoria o incompleta — es trabajo del ingeniero de requisitos (o del probador participando en la revisión) **transformarla** en uno o más requisitos que cumplan las características de calidad que veremos en el Módulo 3 (sección 4). Muchas veces una sola necesidad se descompone en varios requisitos, y a veces varias necesidades se consolidan en un solo requisito si apuntan al mismo criterio verificable.

### 2.2 El flujo completo de trabajo con Stakeholders
El proceso formal para pasar de "stakeholders" a "requisitos de stakeholder" bien definidos incluye:
1. **Determinar los stakeholders**: identificar quién tiene interés legítimo en el sistema — un error común es identificar solo a los stakeholders "obvios" (el cliente/OEM) y olvidar otros igualmente relevantes (reguladores, equipos de mantenimiento, equipos de otros sistemas que interactúan con el tuyo).
2. **Elicitar las necesidades de los stakeholders** (ver técnicas en 2.4).
3. **Transformar las necesidades en requisitos de stakeholder** — estructurados, pero todavía sin comprometerse a una solución técnica específica.
4. **Determinar la necesidad de sistemas, productos o servicios habilitantes (enabling systems)**: ¿qué más se necesita para que el sistema funcione, además del sistema mismo? (ej. herramientas de diagnóstico, infraestructura de prueba, documentación de servicio).
5. **Definir los criterios de validación**: ¿cómo vamos a confirmar que, al final, el sistema construido realmente satisface estas necesidades? (esto se retoma a fondo en el Módulo 4, sección 7).

### 2.3 Flujo de Elicitación (Elicitation Workflow)
El curso describe un flujo específico y repetible para la elicitación a nivel de stakeholder:
1. **Analizar los requisitos del cliente** (los que llegan ya expresados, frecuentemente de forma incompleta o informal, directamente del cliente/OEM).
2. **Realizar Análisis Operacional / Análisis de Casos de Uso**: entender cómo se usará el sistema en la práctica, a través de escenarios concretos (ver 2.5).
3. **Analizar a los Stakeholders y sus Necesidades**: profundizar más allá de lo que el cliente expresó inicialmente, incluyendo stakeholders que el cliente no mencionó.
4. **Analizar el Contexto**: el entorno operativo, restricciones externas, sistemas con los que debe interactuar (relevante para un proyecto Brownfield).
5. **Refinar los Requisitos del Cliente en Requisitos de Stakeholder**: el resultado final de este flujo — requisitos ya estructurados, trazables y lo suficientemente claros para avanzar al siguiente nivel (sistema).

### 2.4 Técnicas de Elicitación
- **Entrevistas**: conversación directa con stakeholders — estructuradas o abiertas.
- **Talleres (Workshops)**: sesiones grupales, a menudo con facilitación, para generar consenso entre stakeholders con intereses distintos.
- **Observación (Field Studies / Contextual Inquiry)**: observar a los usuarios en su contexto real de trabajo.
- **Prototipado**: construir una versión simplificada/visual del sistema para que los stakeholders reaccionen a algo concreto.
- **Casos de Uso (Use Cases)** y **Escenarios Operacionales (Operational Scenarios)**: ver 2.5.
- **ConOps (Concept of Operations)**: ver 2.5.
- **Historias de usuario**: ya las estudiaste a fondo en Fundamentos de Testing.
- **Análisis de documentación existente**: normativas, estándares de la industria (ISO 26262), sistemas legados, especificaciones de proveedores.

### 2.5 Casos de Uso, Escenarios Operacionales y ConOps
Tres herramientas específicas de elicitación/especificación que el curso trata en profundidad:
- **Casos de Uso (Use Cases)**: describen la interacción entre un **actor** (usuario, otro sistema) y el sistema para lograr un objetivo concreto — incluyen flujo principal y flujos alternativos/de excepción. Son especialmente útiles para capturar comportamientos con múltiples caminos posibles (ej. un flujo de diagnóstico con distintos NRCs posibles, que ya viste en Protocolos de Comunicación).
- **Escenarios Operacionales (Operational Scenarios)**: describen cómo el sistema se usará en un **contexto operativo real**, a menudo narrando una secuencia completa de eventos en el mundo real (ej. "el vehículo arranca en frío a -20°C, el conductor activa el control de crucero adaptativo en una autopista con tráfico moderado..."). Son más amplios que un caso de uso individual — pueden abarcar varios casos de uso en secuencia.
- **ConOps (Concept of Operations)**: un documento de alto nivel que describe **cómo se espera que el sistema sea usado** desde la perspectiva del usuario/operador, antes de entrar en el detalle de requisitos — responde "¿cómo se ve el éxito desde la perspectiva operativa?" y sirve como puente entre la necesidad del stakeholder y los requisitos formales que se derivarán después.

### 2.6 Caracterización de Necesidades: el Modelo de Kano
El **Modelo de Kano** es una técnica para **priorizar y categorizar** las necesidades de los stakeholders según su impacto en la satisfacción del usuario — muy útil en la fase de análisis para decidir qué necesidades realmente ameritan convertirse en requisitos obligatorios y cuáles son negociables:

- **Must-have (básicos/obligatorios)**: características que el usuario **da por sentadas** — su ausencia genera una insatisfacción severa, pero su presencia no genera satisfacción extra (el usuario simplemente las espera). Ejemplo automotriz: que los frenos funcionen.
- **Should-have (de desempeño/unidimensionales)**: a mayor cumplimiento, mayor satisfacción, de forma proporcional — ejemplo: la autonomía de la batería en un vehículo eléctrico, o el tiempo de respuesta de un sistema de diagnóstico.
- **Might-have (deleite/atractivos)**: características no esperadas por el usuario, pero que generan satisfacción desproporcionadamente alta cuando están presentes (y no generan insatisfacción si faltan, porque el usuario ni las esperaba). Ejemplo: una función de asistencia de estacionamiento que el usuario no pidió explícitamente pero que lo sorprende gratamente.

*Por qué esto importa para tu rol*: el Modelo de Kano ayuda a **priorizar el esfuerzo de prueba** también — un defecto en una característica "must-have" (ej. un fallo en el sistema de frenos) es catastrófico incluso si es poco frecuente, mientras que un defecto en una característica "might-have" puede tener una tolerancia de riesgo distinta. Esto conecta directamente con el **Análisis de Riesgo de Producto** que ya estudiaste en Gestión de Actividades de Prueba.

---

## 3. De Requisitos de Stakeholder a Requisitos de Sistema y Subsistema (Module 3)

### 3.1 La Jerarquía Completa de Requisitos
Retomando y formalizando la jerarquía:
- **Requisitos de Stakeholder**: lo que los implicados necesitan, ya estructurado pero aún centrado en el problema, no en una solución técnica específica.
- **Requisitos de Sistema**: lo que el sistema completo debe hacer para satisfacer los requisitos de stakeholder — aquí empieza el dominio de la solución.
- **Requisitos de Subsistema**: la descomposición de los requisitos de sistema hacia cada subsistema/componente que, en conjunto, conforman el sistema completo.
- **Requisitos de Software**: el nivel más específico, derivado de los requisitos de subsistema/sistema que corresponden a la parte de software.

Esta cadena de descomposición (stakeholder → sistema → subsistema → software) es exactamente la que luego se audita formalmente en ASPICE (SYS.2 → SYS.3 → SWE.1 → SWE.2), como ya vimos.

### 3.2 Las 9 Características de un Buen Requisito (marco INCOSE)
Este es probablemente el contenido más citado en cualquier entrevista o examen formal de Requirements Engineering — las **9 características** que debe cumplir un requisito bien escrito, cada una con sus propias reglas de redacción:

| # | Característica | Qué significa | Ejemplo de violación | Regla práctica de redacción |
|---|---|---|---|---|
| 1 | **Necesario (Necessary)** | Existe una razón real, trazable a una necesidad de stakeholder — si se eliminara, algo real se perdería | Un requisito "porque sí", sin trazabilidad a ninguna necesidad | Todo requisito debe trazar hacia atrás a al menos una necesidad/requisito de nivel superior |
| 2 | **Apropiado (Appropriate)** | El nivel de detalle es el correcto para el nivel de la jerarquía donde vive (ni demasiado abstracto, ni ya implementando una solución) | Un requisito de stakeholder que ya especifica qué microcontrolador usar | No mezclar niveles de abstracción — un requisito de stakeholder no debe prescribir diseño de solución |
| 3 | **No Ambiguo (Unambiguous)** | Solo puede interpretarse de una forma | "El sistema debe responder rápido" | Evitar adjetivos/adverbios vagos; usar valores numéricos y condiciones concretas |
| 4 | **Completo (Complete)** | No requiere información adicional para ser entendido o implementado | Un requisito de comunicación sin especificar qué pasa en caso de error | Especificar explícitamente casos normales y de excepción |
| 5 | **Singular (Singular)** | Expresa una sola idea/necesidad, no varias combinadas | "El sistema debe leer el sensor, validar el rango y generar un DTC si está fuera de rango" (son 3 requisitos en 1) | Un requisito = una sola declaración verificable; dividir requisitos compuestos |
| 6 | **Factible (Feasible)** | Es técnicamente y económicamente posible de implementar con la tecnología/tiempo/presupuesto disponibles | Un tiempo de respuesta que ningún hardware disponible puede cumplir | Validar factibilidad técnica con el equipo de desarrollo antes de formalizar el requisito |
| 7 | **Verificable/Comprobable (Verifiable)** | Se puede diseñar una prueba objetiva (inspección, análisis, demostración o prueba — ver Módulo 4) que confirme si se cumple | "El sistema debe ser fácil de usar" | Usar criterios medibles y un método de verificación identificable |
| 8 | **Correcto (Correct)** | Representa fielmente la necesidad real, sin errores de transcripción o de entendimiento | Un requisito que malinterpreta lo que el stakeholder realmente pidió | Validar el requisito directamente con el stakeholder que originó la necesidad |
| 9 | **Conforme (Conforming)** | Sigue las plantillas/estándares de redacción de requisitos de la organización (ej. EARS, plantilla de SRS) | Un requisito redactado en un formato distinto al resto del documento, dificultando el análisis automatizado/consistencia | Usar siempre la plantilla acordada por el equipo/organización |

*Nota de integración con lo que ya tenías*: estas 9 características **formalizan e incluyen** lo que ya habíamos cubierto (no ambiguo, completo, verificable, trazable=necesario, priorizado) pero agregan 4 conceptos nuevos clave que vale la pena resaltar en entrevista: **Apropiado** (nivel de abstracción correcto), **Singular** (un requisito = una idea), **Correcto** (fidelidad a la necesidad real, distinto de "no ambiguo") y **Conforme** (adherencia a plantilla/estándar).

### 3.3 "Requirement Smells" — patrones de ambigüedad que debes saber detectar
Esto sigue siendo extremadamente útil para entrevista porque muestra que sabes **revisar** requisitos activamente contra las 9 características, no solo leerlos:

- **Adverbios/adjetivos vagos**: "rápido", "fácil", "amigable", "eficiente", "robusto" — violan **No Ambiguo**.
- **Pronombres ambiguos**: "el sistema debe procesarlo y enviarlo" — ¿qué es "lo"? — violan **No Ambiguo**.
- **Voz pasiva sin sujeto claro**: "los datos serán validados" — ¿por quién/qué componente? — violan **Completo**.
- **Universales sin excepción explícita**: "el sistema siempre debe..." / "nunca debe..." — casi ningún "siempre" sobrevive un análisis de casos límite — violan **Correcto** o **Factible**.
- **Términos de comparación sin referencia**: "más rápido", "más seguro" — ¿comparado con qué línea base? — violan **No Ambiguo**.
- **Requisitos compuestos**: "el sistema debe leer el sensor y validar el rango y generar un DTC si está fuera de rango" — violan directamente **Singular**.

*Ejercicio mental útil para entrevista*: si te dan un requisito de ejemplo, practica identificar en voz alta **cuál(es) de las 9 características viola** y cómo lo reescribirías — es exactamente el tipo de ejercicio que un entrevistador de Test Function Lead podría pedirte en vivo.

### 3.4 Especificación: SRS, Casos de Uso e Historias de Usuario
- **SRS (Software Requirements Specification)**: documento formal y estructurado, típico de modelos secuenciales (V-Model) y de industrias reguladas.
- **Casos de Uso**: ya cubiertos en 2.5.
- **Historias de Usuario + Criterios de Aceptación**: el enfoque ágil, ya cubierto en Fundamentos de Testing — cada criterio de aceptación debe tener al menos un caso de prueba asociado.

### 3.5 Patrones de redacción (EARS)
Una buena práctica para cumplir específicamente la característica de **Conforme** y reducir violaciones de **No Ambiguo**:
- **Ubicuo**: *"El [sistema] debe [respuesta]"*.
- **Basado en evento**: *"Cuando [evento], el [sistema] debe [respuesta]"*.
- **Basado en estado**: *"Mientras [estado], el [sistema] debe [respuesta]"*.
- **Opcional**: *"Donde [característica], el [sistema] debe [respuesta]"*.
- **No deseado**: *"Si [condición no deseada], entonces el [sistema] debe [respuesta]"*.

*Ejemplo automotriz con EARS*: "Cuando la ECU reciba una solicitud UDS 0x27 con una key inválida, el sistema debe responder con NRC 0x35 (invalidKey) en menos de 100ms."

---

## 4. Trazabilidad de Requisitos

- **Trazabilidad hacia atrás (backward)**: desde un requisito de bajo nivel hacia el requisito de más alto nivel o necesidad que lo originó — responde "¿por qué existe esto?" y es la base de la característica **Necesario**.
- **Trazabilidad hacia adelante (forward)**: desde un requisito hacia los artefactos que lo implementan/verifican (diseño, código, casos de prueba) — responde "¿qué depende de esto?".
- **Matriz de Trazabilidad de Requisitos (RTM)**: vincula requisitos con casos de prueba (y opcionalmente diseño/código), permitiendo calcular cobertura de requisitos y hacer análisis de impacto.

**Por qué la trazabilidad es más que "burocracia"**: en automotriz, bajo ASPICE o ISO 26262, la trazabilidad **no es opcional** — es un requisito de proceso auditable, y su ausencia es, en sí misma, un hallazgo de no conformidad en un assessment ASPICE.

---

## 5. Herramientas: DOORS, RQM, Jira, Confluence

### 5.1 IBM DOORS / DOORS Next Generation (DNG)
- **DOORS** organiza los requisitos en **módulos**, donde cada requisito es un **objeto** con atributos configurables (ID, texto, estado, prioridad, responsable, nivel ASIL si aplica).
- **Enlaces (Links)**: permiten trazabilidad entre objetos de distintos módulos, y generar **matrices de trazabilidad** y reportes de cobertura (requisitos "huérfanos" sin enlaces hacia pruebas).
- **Líneas base (Baselines)**: una "fotografía" congelada del estado de un módulo — clave para análisis de impacto cuando un requisito cambia después de haber sido baseline.
- **DOORS Next Generation (DNG)**: evolución web-based, con mejor soporte para colaboración distribuida.

### 5.2 RQM (Rational Quality Manager)
Se integra con DOORS para heredar la trazabilidad: un caso de prueba en RQM se enlaza a un requisito en DOORS, cerrando el ciclo desde requisito hasta ejecución y resultado.

### 5.3 Jira y Confluence (Atlassian)
- **Jira**: gestión ágil de trabajo — historias de usuario, tareas, defectos, sprints.
- **Confluence**: documentación colaborativa — especificaciones, actas de talleres, ConOps, decisiones de diseño.

*Nota para entrevista*: es común que un proyecto real use ambos mundos simultáneamente — DOORS/RQM para trazabilidad formal exigida por ASPICE/seguridad, y Jira/Confluence para la gestión ágil del día a día.

---

## 6. Requirements Engineering en el Contexto Automotriz (ASPICE)

- **SYS.2 (System Requirements Analysis)**: análisis de requisitos a nivel de sistema completo.
- **SYS.3 (System Architectural Design)**: derivar la arquitectura del sistema.
- **SWE.1 (Software Requirements Analysis)**: análisis de requisitos de software.
- **SWE.2 (Software Architectural Design)**: arquitectura de software derivada de SWE.1.

La trazabilidad exigida por ASPICE conecta explícitamente estos niveles — es la cadena de la sección 4, formalizada como requisito de proceso auditable.

### Requisitos de Seguridad Funcional (ISO 26262)
- Cada requisito de seguridad lleva asignado un **nivel ASIL** (A, B, C, D), heredado de un análisis de riesgo previo (HARA).
- Un requisito de alto ASIL puede requerir **descomposición ASIL** en sub-requisitos distribuidos en elementos redundantes — cada sub-requisito necesita su propia evidencia de verificación.

---

## 7. Verificación y Validación de Requisitos (Module 4)

Esta sección formaliza algo que ya tocamos de pasada: **cómo confirmas, de manera objetiva, que un requisito se cumple** — y es distinto de simplemente "revisar" el requisito (eso ya lo vimos como prueba estática en Fundamentos de Testing).

### 7.1 Criterios de Validación
Definidos desde la etapa de elicitación de stakeholder (sección 2.2, paso 5): ¿cómo sabremos, al final, que el sistema construido satisface realmente la necesidad original? Esto se define **antes** de construir el sistema, no se improvisa al final.

### 7.2 Los 4 Métodos de Verificación (a veces recordados con el acrónimo "IADT")
Para cada requisito **verificable** (característica #7 de la sección 3.2), debe definirse **cuál** de estos 4 métodos se usará para confirmarlo:

1. **Inspección (Inspection)**: examen visual/manual del artefacto (documento, diseño, código) — equivalente a la **prueba estática/revisión** que ya estudiaste en Fundamentos de Testing. Se usa para requisitos donde basta verificar visualmente una característica (ej. "el conector debe tener 12 pines").
2. **Análisis (Analysis)**: uso de cálculos, modelos o simulaciones (no ejecución directa del sistema real) para confirmar el cumplimiento — ej. un análisis de timing worst-case para confirmar que un requisito de latencia es alcanzable, o una simulación MIL/SIL (que ya estudiaste en HIL Testing) para validar un algoritmo de control antes de tener hardware real.
3. **Demostración (Demonstration)**: operar el sistema (o una parte) para mostrar que cumple el requisito bajo condiciones específicas, sin necesariamente medir con instrumentación de precisión — ej. mostrar que una función de infotainment responde al presionar un botón.
4. **Prueba (Test)**: ejecutar el sistema bajo condiciones controladas, con instrumentación, para medir **objetivamente y con precisión** si se cumple el requisito — este es el método que más usas tú en HIL (ej. medir con precisión que un tiempo de respuesta UDS es menor a 50ms, usando el Trace de CANoe).

**Por qué esta clasificación importa en entrevista**: un error común es asumir que **todo** se verifica con "Prueba" (Test) en el sentido estricto de ejecutar el sistema con instrumentación — muchos requisitos se verifican más eficientemente con Inspección o Análisis, y elegir el método de verificación correcto para cada requisito es parte del trabajo de planificación de prueba (conectando con el **enfoque de prueba** que definiste en el Plan de Prueba, capítulo de Gestión de Actividades de Prueba).

*Ejemplo aplicado a tu dominio*: un requisito de ASPICE/ISO 26262 sobre el algoritmo de un ADAS podría verificarse así — el diseño del algoritmo se verifica por **Análisis** (simulación MIL), la implementación del hardware/conector se verifica por **Inspección**, una demo a un stakeholder de que la función "funciona" se hace por **Demostración**, y la medición precisa de los tiempos de respuesta bajo condiciones reales de bus se hace por **Prueba** en HIL.

### 7.3 Verificación vs Validación, aplicado específicamente a requisitos
Retomando la distinción de Fundamentos de Testing, pero aplicada aquí:
- **Verificar un requisito**: confirmar que el sistema **cumple** ese requisito tal como está escrito (usando uno de los 4 métodos IADT).
- **Validar un requisito**: confirmar que ese requisito, **incluso si se cumple perfectamente**, realmente refleja la necesidad real del stakeholder — puedes verificar exitosamente un requisito mal concebido (que no resuelve el problema real) y aun así fallar en validación. Esto conecta directamente con el principio de Fundamentos de Testing: "la verificación del software no asegura el éxito de un sistema" (Falacia de la ausencia de defectos).

---

## 8. Gestión de Cambios y Gestión de Requisitos (Module 5)

Los requisitos cambian — es inevitable. Un buen proceso de gestión de cambios incluye:
1. **Solicitud de cambio (Change Request)**: se documenta formalmente qué se quiere cambiar y por qué.
2. **Análisis de impacto**: usando la trazabilidad (sección 4), se determina qué otros requisitos, elementos de diseño, código y **casos de prueba** se ven afectados.
3. **Aprobación**: normalmente a través de un **Comité de Control de Cambios (Change Control Board)**.
4. **Implementación y actualización de trazabilidad**: se actualiza el requisito, se re-vinculan los artefactos afectados, y se genera una nueva línea base si corresponde.

**Estados típicos de un requisito a lo largo de su gestión** (status accounting): Propuesto → Analizado → Aprobado → Implementado → Verificado → Validado — cada cambio de estado debería quedar registrado y trazable, especialmente en DOORS, donde el atributo "estado" del objeto refleja exactamente en cuál de estas fases se encuentra.

---

## 9. Requirements-Based Testing (cerrando el círculo con lo que ya sabes)

- Cada requisito verificable debe tener **al menos un caso de prueba** que lo cubra.
- Las técnicas de caja negra que estudiaste en Fundamentos de Testing (partición de equivalencia, análisis de valor frontera, tabla de decisión, transición de estado) son las herramientas que usas para **derivar sistemáticamente** esos casos de prueba — un requisito que cumple las 9 características (especialmente Singular y No Ambiguo) es lo que hace posible aplicar estas técnicas de forma efectiva.
- La elección del **método de verificación** (sección 7.2) determina directamente qué tipo de "caso de prueba" se diseña — no todos los requisitos requieren un caso de prueba ejecutado en HIL; algunos se cierran con una inspección o un análisis.

## Preguntas de teoría
1. ¿Por qué entender el "dominio del problema" antes del "dominio de la solución" es tan importante, y qué puede salir mal si un equipo se salta este paso?
2. Da un ejemplo propio de la diferencia entre un proyecto Greenfield y uno Brownfield en tu experiencia automotriz.
3. Explica la diferencia entre una "necesidad" y un "requisito" con un ejemplo propio, distinto al de frenos usado en este capítulo.
4. Describe el flujo de 5 pasos para transformar requisitos de cliente en requisitos de stakeholder.
5. Usando el Modelo de Kano, clasifica 3 características de un sistema de infotainment en must-have, should-have y might-have.
6. ¿Cuál es la diferencia entre un Caso de Uso, un Escenario Operacional y un ConOps?
7. Explica las 9 características de un buen requisito (INCOSE) y da un ejemplo propio de violación para 3 de ellas distintas a las usadas en este capítulo.
8. Toma un requisito ambiguo real de tu experiencia e identifica qué característica(s) de las 9 viola.
9. ¿Cuáles son los 4 métodos de verificación (IADT), y da un ejemplo automotriz de cuándo usarías cada uno?
10. ¿Por qué puedes verificar exitosamente un requisito y aun así fallar en validarlo? Da un ejemplo.
11. ¿Qué es un "enabling system" (sistema habilitante) y por qué se identifica desde la fase de elicitación de stakeholder?
12. Describe los estados típicos por los que pasa un requisito desde que se propone hasta que se valida.

## Errores comunes
- Saltar directamente al "dominio de la solución" sin haber entendido completamente el "dominio del problema" — un error de Systems Engineering, no solo de Requirements Engineering.
- Confundir necesidad con requisito — una necesidad es la expresión informal del stakeholder; el requisito es su transformación estructurada y verificable.
- Memorizar "4 actividades" o "5 actividades" de RE como si fuera un número mágico absoluto — lo importante es entender qué ocurre en cada paso (elicitar, analizar, comunicar/especificar/validar, gestionar).
- Aplicar el Modelo de Kano solo a producto/marketing, sin conectar que también informa priorización de riesgo y esfuerzo de prueba.
- Confundir Caso de Uso con Escenario Operacional — el escenario operacional suele ser más amplio y puede abarcar varios casos de uso en secuencia narrativa.
- Reducir las características de un buen requisito a solo "no ambiguo y verificable" — las 9 características completas (especialmente Apropiado, Singular, Correcto y Conforme) son las que distinguen un análisis superficial de uno riguroso en entrevista.
- Pensar que todo requisito se verifica necesariamente con una "prueba" ejecutada en HIL — Inspección, Análisis y Demostración son métodos de verificación igualmente válidos y, a veces, más eficientes.
- Confundir verificación de un requisito (¿se cumple lo que está escrito?) con validación del requisito (¿lo que está escrito realmente resuelve el problema del stakeholder?).