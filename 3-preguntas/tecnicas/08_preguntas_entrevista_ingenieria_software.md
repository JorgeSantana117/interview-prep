# Tema: Ingeniería de Software

> Material de estudio base: capítulo `08_ingenieria_de_software.md`. Mismo formato que los bancos anteriores. Tema central para el rol de Embedded Software Engineer y, en menor profundidad, para quien lidera pruebas.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* contexto → criterio → decisión → evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "Explica cohesión y acoplamiento y cómo los aplicas en un módulo en C."
- *Puntos clave*: alta cohesión = el módulo hace una sola cosa bien; bajo acoplamiento = dependencias mínimas y a través de interfaces. En C: archivos `.h/.c` con API pública pequeña, variables `static`, sin globales compartidas, inyección de dependencias por punteros a función.

**2.** "Menciona los principios SOLID y cómo se traducen a C embebido."
- *Puntos clave*: responsabilidad única (un módulo, una razón de cambio); abierto/cerrado (extender sin modificar, con tablas de punteros a función); sustitución y segregación de interfaces (contratos pequeños); inversión de dependencias (la lógica depende de abstracciones, no del HAL concreto).

**3.** "¿Qué patrones usas con frecuencia en software embebido?"
- *Puntos clave*: máquina de estados, productor-consumidor con buffer circular, observer/callback, capa de abstracción de hardware (HAL), singleton controlado, *state pattern* para modos, *watchdog manager*. Explica un caso real y su trade-off.

**4.** "¿Qué es MISRA C y cómo lo manejas en tu flujo de trabajo?"
- *Puntos clave*: guía de codificación segura para C; reglas obligatorias, requeridas y asesoras; análisis estático automatizado en CI; desviaciones justificadas, aprobadas y documentadas; no cumplir ciegamente sin entender el motivo.

**5.** "¿Qué es programación defensiva y qué ejemplos aplicas?"
- *Puntos clave*: validar entradas y rangos, comprobar retornos, valores por defecto seguros, `assert` en desarrollo, manejo de punteros nulos, protección contra desbordamiento, *timeouts*, estados inválidos en `switch` (default). Cuidado con ocultar defectos.

**6.** "¿Cómo reduces la deuda técnica sin frenar entregas?"
- *Puntos clave*: refactorización incremental respaldada por pruebas, regla del boy scout, tiempo reservado por iteración, métricas (complejidad, duplicación), justificación en términos de riesgo y costo.

**7.** "Describe las métricas de software que usarías y sus límites."
- *Puntos clave*: complejidad ciclomática (umbrales como 10–15), tamaño, acoplamiento, cobertura, densidad de defectos. Sirven como alerta; no son objetivos en sí mismos ni miden calidad total.

**8.** "¿Cómo manejas la concurrencia y los datos compartidos entre ISR y tareas?"
- *Puntos clave*: secciones críticas cortas, variables `volatile` donde corresponda (no sustituyen sincronización), buffers de doble acceso, mutex y semáforos con herencia de prioridad, evitar inversión de prioridad y bloqueo mutuo, diseño de ISR mínimas.

**9.** "¿Qué exige ISO 26262 Parte 6 al software de un componente ASIL alto?"
- *Puntos clave*: arquitectura con mecanismos de seguridad, diseño modular y simple, restricciones de lenguaje (subconjunto MISRA), métodos de verificación (revisión, análisis estático, cobertura estructural MC/DC), análisis de interferencias, independencia de verificación.

---

## B. Test Engineer

**1.** "¿Cómo aprovechas la arquitectura del software para diseñar pruebas?"
- *Puntos clave*: identificar módulos, interfaces y dependencias; pruebas unitarias por módulo con stubs, de integración en las interfaces, de sistema en funciones completas; foco en puntos de acoplamiento y en componentes de mayor riesgo.

**2.** "¿Qué aporta el análisis estático y qué no detecta?"
- *Puntos clave*: detecta violaciones de reglas, variables sin inicializar, posibles desbordamientos y código muerto sin ejecutarlo. No detecta errores de lógica ni defectos que dependen de datos o de timing; produce falsos positivos que deben triarse.

**3.** "¿Cómo pruebas una máquina de estados implementada en C?"
- *Puntos clave*: derivar casos con la técnica de transición de estados (válidas e inválidas, cobertura de transiciones), eventos fuera de orden, tiempos de permanencia, entradas inválidas, estado tras reset; verificar acciones de entrada y salida.

**4.** "¿Qué revisarías en una revisión de código desde la perspectiva de pruebas?"
- *Puntos clave*: testabilidad (dependencias inyectables), manejo de errores y rangos, condiciones límite, valores mágicos, complejidad, casos no cubiertos por el diseño, trazabilidad al requisito.

**5.** "¿Cómo detectas problemas de concurrencia durante las pruebas?"
- *Puntos clave*: son no deterministas; estrés con carga y variación de timing, inyección de retardos, herramientas de trazado de tareas, revisión del diseño, pruebas de larga duración, detección de carreras con herramientas de análisis dinámico.

**6.** "¿Qué métricas de código usas para focalizar pruebas?"
- *Puntos clave*: módulos con alta complejidad o cambios frecuentes y defectos previos concentran riesgo (los defectos se agrupan); usar para priorizar, no para evaluar personas.

---

## C. Embedded Software Test Function Lead

**1.** "¿Cómo evalúas la calidad de la arquitectura de software de una entrega?"
- *Puntos clave*: atributos de calidad (rendimiento, mantenibilidad, testabilidad, seguridad), revisión por escenarios, análisis de dependencias, métricas, verificación de que el diseño cumple los requisitos y la asignación a tareas y memoria.

**2.** "Tu equipo debe adoptar un modelo ágil dentro de un proceso ASPICE. ¿Cómo lo haces?"
- *Puntos clave*: enfoque híbrido: planificación e hitos por iteraciones con evidencias, definición de hecho (DoD) que incluya trazabilidad, pruebas y revisiones, automatización de reportes, backlog con requisitos trazables.

**3.** "¿Qué defines como 'Definition of Done' para el software?"
- *Puntos clave*: código revisado, análisis estático sin hallazgos críticos, pruebas unitarias y de integración aprobadas con cobertura objetivo, trazabilidad actualizada, documentación mínima, integración en CI exitosa.

**4.** "¿Cómo equilibras la presión de entrega con la calidad del código?"
- *Puntos clave*: políticas de calidad no negociables (revisión, pruebas), deuda técnica visible y planificada, comunicar riesgo con datos, negociar alcance en lugar de calidad.

**5.** "¿Cómo gestionas la configuración y las liberaciones de software?"
- *Puntos clave*: línea base, ramas, etiquetas, notas de versión, artefactos inmutables, trazabilidad cambio ↔ requisito ↔ prueba, control de cambios con revisión, criterios de liberación.

**6.** "¿Cómo defines la estrategia de pruebas unitarias para código legado sin pruebas?"
- *Puntos clave*: pruebas de caracterización antes de modificar, cobertura en las áreas de cambio y de riesgo, refactorización pequeña con red de seguridad, introducir costuras (seams) para sustituir dependencias.

---

## D. Test Manager

**1.** "¿Cómo influye la calidad del diseño de software en el costo de pruebas?"
- *Puntos clave*: bajo acoplamiento y alta cohesión facilitan pruebas aisladas y reducen regresión; la deuda técnica incrementa tiempo de depuración y de pruebas; conviene incluir criterios de testabilidad en las revisiones.

**2.** "¿Cómo abogas por tiempo de refactorización o de pruebas automatizadas ante la dirección?"
- *Puntos clave*: cuantificar defectos tardíos, tiempo de depuración y retrabajo, riesgo para hitos, retorno de la inversión; propuesta incremental con medidas de éxito.

**3.** "¿Qué indicadores usarías para seguir la salud del software de un programa?"
- *Puntos clave*: tendencia de defectos y su antigüedad, cobertura, estabilidad de regresión, complejidad, hallazgos de análisis estático, velocidad de cierre de revisiones. Interpretar en conjunto.

**4.** "¿Cómo trabajas con desarrollo cuando la relación entre pruebas y desarrollo es tensa?"
- *Puntos clave*: objetivos compartidos de calidad, reportes objetivos, comunicación temprana, participación en refinamiento y revisiones, retrospectivas conjuntas, reconocimiento de logros comunes.

---

## Preguntas transversales (cierre o "curveball")

**1.** "¿Cuál es el principio de diseño más importante para ti en software embebido y por qué?"
- *Puntos clave*: respuesta propia con justificación (por ejemplo, simplicidad/modularidad por su efecto en pruebas y seguridad) y un ejemplo concreto.

**2.** "¿Qué diferencia hay entre un bug de lógica y uno de integración y cómo los atacas distinto?"
- *Puntos clave*: lógica = aislable con pruebas unitarias; integración = aparece en interfaces, timing o datos entre componentes y requiere pruebas de integración y trazas.

**3.** "Cuéntame de un cambio de diseño que mejoró la mantenibilidad o testabilidad de un módulo."
- *Puntos clave*: ejemplo STAR: problema, decisión de diseño, cómo lo validaste (regresión), resultado medible. Se profundiza en el Módulo 3.

