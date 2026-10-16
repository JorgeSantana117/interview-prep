# Tema: Fundamentos de Software Testing (ISTQB CTFL 4.0)

> Material de estudio base: capítulo `04_fundamentos_software_testing.md`. Mismo formato que los bancos anteriores: preguntas por rol, con los puntos que el entrevistador espera escuchar. Construye tu respuesta con tus palabras y, siempre que puedas, con un ejemplo real de Bosch, ALTEN, VW o Ford.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación o trade-off.
> - *Escenario:* contexto → criterio que aplicas → decisión → evidencia o métrica.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

Aquí se evalúa que entiendas la prueba como parte de tu trabajo de desarrollo, no como una actividad ajena.

**1.** "¿Cuál es la diferencia entre probar y depurar? ¿Quién hace cada cosa?"
- *Puntos clave*: probar muestra fallos (el efecto) y depurar encuentra y corrige el defecto (la causa). El probador reporta el fallo, el desarrollador depura y luego se hace prueba de confirmación. Un buen reporte (pasos, versión, logs) acelera la depuración.

**2.** "Explica error, defecto y fallo con un ejemplo de firmware."
- *Puntos clave*: el error lo comete una persona (malinterpretó un requisito de 10 ms); el defecto queda en el código (comparación con `<` en lugar de `<=`); el fallo ocurre en ejecución (la salida se activa tarde). No todo defecto produce fallo. Menciona el análisis de causa raíz.

**3.** "¿Qué aporta la prueba unitaria en un entorno embebido y qué dificultades tiene?"
- *Puntos clave*: aísla funciones con mocks o stubs del HAL, ejecuta en PC o en el objetivo, mide cobertura (sentencia, decisión), detecta defectos baratos pronto. Dificultades: dependencia de hardware, acceso a registros, memoria limitada, timing real que SIL no reproduce.

**4.** "¿Cuándo es útil la prueba de caja blanca y qué cobertura exigirías en código crítico?"
- *Puntos clave*: complementa la caja negra al revelar ramas no ejercitadas. Cobertura de sentencia < decisión < MC/DC en ASIL alto. 100 % de cobertura no garantiza ausencia de defectos ni valida requisitos faltantes.

**5.** "¿Qué es shift-left y cómo lo aplicas como desarrollador?"
- *Puntos clave*: mover la prueba a etapas tempranas: revisiones de código y de requisitos, análisis estático (MISRA), pruebas unitarias y SIL en CI, TDD cuando aplica. Cuanto antes se detecta un defecto, menos cuesta corregirlo.

**6.** "Un revisor te señala defectos en tu código. ¿Qué tipos de revisión conoces y cuál usarías?"
- *Puntos clave*: informal, recorrido (walkthrough), revisión técnica, inspección (la más formal). Para código crítico, inspección o revisión técnica con checklist; para cambios pequeños, revisión por pares en Gerrit. Factores de éxito: objetivos claros, tamaño acotado, ambiente sin juicio personal.

---

## B. Test Engineer

Este rol es el núcleo del temario: técnicas, diseño de casos y reporte.

**1.** "Dime los 7 principios de la prueba y elige dos que hayas vivido."
- *Puntos clave*: la prueba muestra la presencia de defectos, no su ausencia; la prueba exhaustiva es imposible; la prueba temprana ahorra tiempo y dinero; los defectos se agrupan; paradoja del pesticida; la prueba depende del contexto; la ausencia de defectos es una falacia. Ejemplo propio para cada uno elegido.

**2.** "Diferencia entre verificación y validación."
- *Puntos clave*: verificación = ¿construimos el producto correctamente? (contra especificación); validación = ¿construimos el producto correcto? (contra necesidad del usuario). Ejemplo: HIL verifica contra requisitos; la prueba en vehículo valida el uso real.

**3.** "Tienes un requisito: 'el sistema acepta velocidades de 0 a 250 km/h'. Diseña casos con partición de equivalencia y valores límite."
- *Puntos clave*: particiones: válida [0, 250], inválidas < 0 y > 250 (y no numérica). Límites con 2 valores: 0 y 250 ; con 3 valores: −1, 0, 1 , 249, 250, 251. Define resultado esperado de cada caso.

**4.** "¿Cuándo usarías tabla de decisión y cuándo transición de estado?"
- *Puntos clave*: tabla de decisión cuando el resultado depende de combinaciones de condiciones (reglas de activación de una función); transición de estado cuando importa la historia (modos de operación, gestor de estados). Cobertura: reglas / transiciones válidas / inválidas.

**5.** "Describe el proceso de prueba y qué producto sale de cada actividad."
- *Puntos clave*: planificación (plan), monitorización y control (informes), análisis (condiciones de prueba), diseño (casos), implementación (procedimientos, scripts, datos), ejecución (resultados, reportes de defectos), compleción (informe final, lecciones). Trazabilidad base de prueba ↔ casos ↔ resultados.

**6.** "Diferencia entre prueba de confirmación y de regresión. ¿Cómo eliges qué regresión ejecutar?"
- *Puntos clave*: confirmación verifica que el defecto corregido ya no ocurre; regresión detecta efectos colaterales. Selección por análisis de impacto, riesgo y criticidad; automatizar la regresión repetible. Conecta con tu experiencia en validación dirigida.

**7.** "Un probador encuentra un defecto. ¿Qué debe contener el reporte?"
- *Puntos clave*: título claro, versión/build, entorno (banco, DBC, modelo), pasos reproducibles, resultado esperado vs obtenido, severidad y prioridad, evidencias (logs, trazas). Sin acusaciones. Distinguir severidad (impacto) de prioridad (urgencia).

**8.** "¿Cómo se complementan técnicas de caja negra, caja blanca y basadas en la experiencia?"
- *Puntos clave*: caja negra deriva de requisitos; caja blanca mide la estructura y encuentra lo no ejercitado; experiencia (suposición de errores, exploratoria, checklist) cubre lo que las otras no previeron. Se usan en combinación según riesgo.

**9.** "Describe los niveles y tipos de prueba y dónde cae cada uno en un proyecto automotriz."
- *Puntos clave*: componente, integración de componentes, sistema, integración de sistemas, aceptación. Tipos: funcional, no funcional, caja negra/blanca, relacionados con cambio. Correspondencia con el modelo V y con SWE.4–SWE.6 / SYS.4–SYS.5 de ASPICE.

---

## C. Embedded Software Test Function Lead

Aquí se evalúa estrategia, planeación y criterio de equipo.

**1.** "¿Qué incluye un plan de prueba y cómo lo adaptas a un proyecto ágil?"
- *Puntos clave*: contexto, alcance, enfoque, riesgos, calendario, roles, criterios de entrada y salida, métricas, entorno. En ágil: plan liviano que evoluciona por iteración, con participación del probador en el refinamiento.

**2.** "Define criterios de entrada y salida para el sistema de un B-Sample."
- *Puntos clave*: entrada: build estable e identificado, banco calibrado, requisitos revisados, datos disponibles. Salida: cobertura de requisitos, defectos críticos cerrados, estabilidad de regresión, informe emitido. Si no se cumplen, se decide con el riesgo explícito.

**3.** "¿Cómo priorizas casos de prueba cuando el tiempo no alcanza?"
- *Puntos clave*: basada en riesgo (probabilidad × impacto, ASIL), en requisitos de alta prioridad, en cobertura, o en dependencias. Comunica qué queda fuera y el riesgo residual.

**4.** "Explica la pirámide de prueba y los cuadrantes ágiles y aplícalos a un sistema embebido."
- *Puntos clave*: muchas pruebas unitarias baratas, menos de integración, pocas de sistema; en embebido se traduce a SIL/vECU abundantes y HIL/vehículo selectivos. Cuadrantes: negocio vs tecnología, soportan al equipo vs critican al producto.

**5.** "¿Qué métricas reportas y cómo evitas que se malinterpreten?"
- *Puntos clave*: ejecutadas/aprobadas/falladas/bloqueadas, cobertura de requisitos, densidad y tendencia de defectos, tiempo de ejecución, estabilidad. Mostrar tendencia y contexto; evitar métricas aisladas o usadas como presión sobre personas.

**6.** "¿Cómo gestionas el riesgo de producto en la planeación de pruebas?"
- *Puntos clave*: identificar, analizar (probabilidad e impacto), controlar (mitigar con pruebas más profundas, más temprano, más independencia) y revisar. Vincular con HARA/ASIL y con FMEA.

**7.** "¿Qué nivel de independencia de la prueba aplicarías y por qué?"
- *Puntos clave*: desde el autor hasta organización externa; más independencia reduce sesgos pero puede aumentar la distancia al desarrollo. Proporcional al riesgo; ASIL alto exige independencia (ISO 26262).

**8.** "¿Cómo logras que la calidad sea responsabilidad de todo el equipo?"
- *Puntos clave*: enfoque de equipo completo, criterios de aceptación claros, pruebas en CI, revisiones cruzadas, retrospectivas con acciones medibles.

---

## D. Test Manager

Aquí se evalúa estimación, riesgo, comunicación y mejora.

**1.** "¿Cómo estimas el esfuerzo de prueba de un nuevo proyecto?"
- *Puntos clave*: técnicas basadas en métricas históricas, en juicio de expertos (estimación de tres puntos, Planning Poker), por ratios o por descomposición (WBS). Incluir riesgo, automatización, mantenimiento y buffers; revisar la estimación al avanzar.

**2.** "El desarrollo se retrasa y el tiempo de prueba se reduce. ¿Qué haces?"
- *Puntos clave*: no recortar a ciegas; re-priorizar por riesgo, negociar alcance o fechas con datos, automatizar lo repetitivo, comunicar riesgo residual y obtener decisión explícita del stakeholder.

**3.** "¿Cómo presentas el estado de calidad a stakeholders no técnicos?"
- *Puntos clave*: resumen ejecutivo, semáforo con criterios claros, riesgos y decisiones requeridas, tendencias, sin jerga. Honestidad sobre lo no probado.

**4.** "¿Cómo gestionas el ciclo de vida de los defectos y evitas la acumulación?"
- *Puntos clave*: flujo claro (nuevo, asignado, corregido, verificado, cerrado), triage regular con severidad y prioridad, SLA, métricas de edad, análisis de causa raíz de los defectos recurrentes.

**5.** "¿Qué herramientas de prueba justificarías adquirir y cómo medirías su retorno?"
- *Puntos clave*: gestión de pruebas, automatización, CI, cobertura. Beneficios: repetibilidad, rapidez; riesgos: costo de mantenimiento, curva de aprendizaje, expectativas irreales. Prueba de concepto, métricas antes y después.

**6.** "¿Cómo aplicas mejora de proceso a partir de retrospectivas y métricas?"
- *Puntos clave*: identificar problemas con datos, pocas acciones concretas con responsable y fecha, medir el efecto en la siguiente iteración; compartir lecciones aprendidas entre sitios.

---

## Preguntas transversales (cierre o "curveball")

**1.** "Si solo pudieras hacer una cosa para mejorar la calidad de un proyecto, ¿cuál sería?"
- *Puntos clave*: respuesta con criterio, por ejemplo revisión temprana de requisitos y testabilidad, justificada con costo de defectos tardíos. Reconoce que depende del contexto.

**2.** "¿Qué significa 'ausencia de defectos es una falacia' en tu día a día?"
- *Puntos clave*: un producto sin fallos conocidos puede no satisfacer las necesidades; hay que validar con el usuario o stakeholder, no solo contra la especificación.

**3.** "Cuéntame de una vez que una técnica de diseño de pruebas te permitió encontrar un defecto que otros no vieron."
- *Puntos clave*: ejemplo STAR concreto: requisito, técnica usada (valores límite, transición de estado), defecto hallado, impacto evitado. Se profundiza en el Módulo 3.
