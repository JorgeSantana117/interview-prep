# Tema: Requirements Engineering

> Material de estudio base: capítulo `06_requirements_engineering.md`. Mismo formato que los bancos anteriores. Tu ventaja: has trabajado con DOORS, RQM, Jira y Confluence y con trazabilidad requisito ↔ prueba; úsalo.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* contexto → criterio → decisión → evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

Aquí se evalúa cómo interpretas y cuestionas los requisitos que implementas.

**1.** "Recibes un requisito ambiguo: 'el sistema debe responder rápidamente'. ¿Qué haces?"
- *Puntos clave*: no implementar suponiendo; identificar el *smell* (término vago), pedir un valor medible (por ejemplo, ≤ 50 ms bajo condiciones definidas), registrar la aclaración y actualizar el requisito y su trazabilidad.

**2.** "¿Qué hace que un requisito sea implementable y verificable?"
- *Puntos clave*: las características de un buen requisito: necesario, apropiado, inequívoco, completo, singular, factible, verificable, correcto, conforme. Medible, con condiciones y tolerancias, sin elegir la solución de diseño.

**3.** "Encuentras un conflicto entre dos requisitos. ¿Cómo lo manejas?"
- *Puntos clave*: documentarlo con evidencia, escalarlo al responsable del requisito, no resolverlo por cuenta propia en el código, seguir el proceso de cambio y confirmar la decisión por escrito.

**4.** "¿Cómo aseguras que tu código es trazable al requisito?"
- *Puntos clave*: identificadores de requisito en commits, comentarios y modelo; revisión de que cada requisito tenga implementación y cada función tenga requisito (sin código huérfano); enlace en la herramienta (DOORS/Jira).

**5.** "¿Qué diferencia hay entre un requisito funcional y uno no funcional? Da ejemplos de firmware."
- *Puntos clave*: funcional = qué hace (activar salida si la señal supera X); no funcional = cómo lo hace (tiempo de respuesta, memoria, disponibilidad, seguridad). Los no funcionales suelen causar más defectos si no se miden.

---

## B. Test Engineer

Este rol se evalúa en el análisis de requisitos para pruebas y la trazabilidad.

**1.** "¿Cómo revisas la testabilidad de un requisito?"
- *Puntos clave*: ¿se puede observar y medir?, ¿tiene criterio de aceptación?, ¿condiciones, entradas y salidas claras?, ¿tiempo y tolerancias? Si no, reportarlo como defecto del requisito antes de diseñar pruebas.

**2.** "Menciona patrones de ambigüedad (*requirement smells*) que detectas al revisar."
- *Puntos clave*: términos vagos ('rápido', 'adecuado', 'fácil'), cuantificadores sin valor ('algunos'), 'y/o', voz pasiva sin actor, comparativos sin referencia, 'etc.', múltiples requisitos en uno.

**3.** "Explica la matriz de trazabilidad y cómo la mantienes."
- *Puntos clave*: enlaza necesidad → requisito de sistema → requisito de software → diseño → caso de prueba → resultado; bidireccional (hacia adelante y hacia atrás); detecta requisitos sin prueba y pruebas sin requisito; analiza el impacto de cambios.

**4.** "Cuáles son los 4 métodos de verificación de requisitos y cuándo usas cada uno."
- *Puntos clave*: Inspección, Análisis, Demostración y Prueba (IADT). Prueba para medición cuantitativa (HIL); demostración para comportamiento observable; análisis para simulación o cálculo; inspección para atributos físicos o documentos.

**5.** "Un requisito cambió tarde. ¿Qué haces con tus casos de prueba?"
- *Puntos clave*: análisis de impacto vía trazabilidad, actualizar casos y resultados esperados, regresión dirigida, versionar el cambio, registrar en la herramienta y avisar al equipo.

**6.** "Escribe un requisito en formato EARS y deriva pruebas."
- *Puntos clave*: p. ej., "Cuando la señal de pedal exceda 95 %, el sistema deberá activar el modo X en menos de 20 ms". Pruebas: valor justo debajo, en el límite y arriba; medición del tiempo; condición negativa. Aplicar valores límite.

**7.** "Diferencia entre verificación y validación de requisitos."
- *Puntos clave*: verificar = el producto cumple el requisito; validar = el requisito expresa la necesidad real del stakeholder. Un requisito puede ser verificable y aun así inútil si no refleja la necesidad.

---

## C. Embedded Software Test Function Lead

Aquí se evalúa el proceso, la revisión y el vínculo con ASPICE.

**1.** "¿Cómo participas en la revisión de requisitos desde el lado de pruebas?"
- *Puntos clave*: revisión temprana con checklist de testabilidad, detección de ambigüedades y huecos, propuesta de criterios de aceptación, participación en la inspección. Shift-left aplicado a requisitos.

**2.** "¿Cómo se refleja Requirements Engineering en ASPICE?"
- *Puntos clave*: SYS.1 (elicitación), SYS.2 (análisis de requisitos del sistema), SWE.1 (requisitos del software); trazabilidad bidireccional y consistencia como prácticas base; verificación contra requisitos en SYS.5, SWE.6.

**3.** "Cobertura de requisitos: ¿cómo la defines y qué hacer si es baja?"
- *Puntos clave*: porcentaje de requisitos con al menos una prueba y con prueba ejecutada y aprobada; priorizar por riesgo; investigar huecos (requisito no verificable, falta de recursos); comunicar el riesgo residual.

**4.** "¿Cómo manejas requisitos de seguridad funcional en tus pruebas?"
- *Puntos clave*: derivados del HARA con ASIL; trazabilidad estricta a objetivos de seguridad; FTTI y estado seguro verificados; pruebas de inyección de fallas; independencia adecuada y evidencia auditable.

**5.** "Cambian los requisitos a mitad de sprint o de un hito. ¿Cómo proteges al equipo?"
- *Puntos clave*: proceso de control de cambios, análisis de impacto cuantificado, negociación de alcance, comunicar costo y riesgo, línea base y versionado.

**6.** "¿Qué haces cuando el cliente entrega requisitos incompletos?"
- *Puntos clave*: lista de preguntas y suposiciones documentadas, sesión de clarificación, registro de decisiones, prototipo o escenarios (ConOps) para validar, acuerdos por escrito.

---

## D. Test Manager

Aquí se evalúa la relación con stakeholders, la gestión del cambio y el riesgo.

**1.** "¿Cómo influyes en la calidad de los requisitos sin ser el dueño de ellos?"
- *Puntos clave*: datos de defectos originados en requisitos, participación temprana, criterios de aceptación acordados, checklist de revisión, métricas de retrabajo. Colaboración, no confrontación.

**2.** "¿Qué herramientas de gestión de requisitos y pruebas integrarías y por qué?"
- *Puntos clave*: DOORS/DNG para requisitos, RQM o Jira (Xray, Zephyr) para pruebas, Confluence para documentación; enlaces nativos, informes de trazabilidad, permisos, costo y adopción del equipo.

**3.** "El requisito del cliente es técnicamente inviable en el plazo. ¿Qué haces?"
- *Puntos clave*: evidencia técnica y estimaciones, alternativas (alcance reducido, entrega por fases, riesgo asumido), negociación con impacto en costo/tiempo/calidad, decisión documentada.

**4.** "¿Cómo reportas el estado de verificación de requisitos?"
- *Puntos clave*: porcentaje verificado/pendiente/fallido por prioridad o ASIL, huecos de trazabilidad, riesgos y decisiones requeridas, tendencia por hito.

**5.** "¿Qué impacto tienen los requisitos mal escritos en costo y calendario?"
- *Puntos clave*: los defectos de requisitos son los más caros si se descubren tarde; retrabajo en diseño, código y pruebas; invertir en revisión temprana reduce el costo total.

---

## Preguntas transversales (cierre o "curveball")

**1.** "Dame un ejemplo de requisito mal escrito y cómo lo mejorarías."
- *Puntos clave*: "El sistema responderá rápido ante fallas" → "Ante pérdida del mensaje X, el sistema deberá activar el DTC Y y el modo degradado en ≤ 300 ms." Explica qué características se corrigieron.

**2.** "¿Cómo explicas la diferencia entre necesidad, requisito y especificación?"
- *Puntos clave*: la necesidad es lo que el stakeholder quiere lograr (dominio del problema); el requisito es la declaración formal y verificable; la especificación detalla la solución y sus interfaces.

**3.** "Cuéntame una vez en que un problema de requisitos se descubrió durante las pruebas."
- *Puntos clave*: ejemplo STAR: requisito ambiguo o faltante, cómo lo detectaste, a quién escalaste, resultado y cambio de proceso. Se profundiza en el Módulo 3.

