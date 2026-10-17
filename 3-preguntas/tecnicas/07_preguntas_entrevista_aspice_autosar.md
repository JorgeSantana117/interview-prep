# Tema: ASPICE y AUTOSAR

> Material de estudio base: capítulo `07_aspice_autosar.md`. Mismo formato que los bancos anteriores. Con tu experiencia en Bosch, ALTEN, VW y Ford, ASPICE es un tema donde puedes dar ejemplos reales de evidencias, auditorías y trazabilidad.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* contexto → criterio → decisión → evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "¿Qué es ASPICE y qué implica para tu trabajo diario como desarrollador?"
- *Puntos clave*: modelo de evaluación de capacidad de procesos de desarrollo automotriz. Exige requisitos, arquitectura, diseño detallado, pruebas unitarias e integración con trazabilidad bidireccional, revisiones y evidencia (work products).

**2.** "Explica el modelo V de software en ASPICE (SWE.1 a SWE.6)."
- *Puntos clave*: SWE.1 requisitos de software; SWE.2 arquitectura; SWE.3 diseño detallado y construcción de unidades; SWE.4 verificación de unidades; SWE.5 integración y prueba de integración; SWE.6 prueba de calificación del software. Cada lado izquierdo tiene su par de verificación.

**3.** "¿Qué evidencia generas para que un auditor confirme que seguiste el proceso?"
- *Puntos clave*: requisitos con ID, diseño, código en control de versiones, revisiones registradas, resultados de pruebas, matriz de trazabilidad, análisis estático, resolución de problemas. Consistencia entre los productos.

**4.** "¿Qué es AUTOSAR Classic y por qué se usa?"
- *Puntos clave*: arquitectura estándar por capas (aplicación, RTE, BSW, microcontrolador), separa software de aplicación del hardware, reutilización, intercambiabilidad y estándar entre proveedores y OEMs.

**5.** "Describe las capas y módulos de la BSW y dónde caería el stack de comunicación."
- *Puntos clave*: Servicios, ECU Abstraction, Microcontroller Abstraction (MCAL), Complex Drivers. Comunicación: COM → PduR → CanIf → CAN Driver; con CanTp para transporte; diagnóstico con DCM/DEM.

**6.** "¿Qué es el RTE y qué función cumple?"
- *Puntos clave*: capa generada que implementa el VFB para una ECU; conecta SWCs entre sí y con la BSW; oculta el mapeo de tareas y la comunicación intra e inter-ECU.

**7.** "¿Qué contiene un ARXML y quién lo modifica?"
- *Puntos clave*: descripción en XML de SWCs, interfaces, puertos, comunicación y configuración de la ECU; herramientas de configuración lo generan y consumen; versionado y revisión como cualquier artefacto.

---

## B. Test Engineer

**1.** "¿Cómo se ubican las pruebas en ASPICE?"
- *Puntos clave*: SWE.4 (unidad), SWE.5 (integración), SWE.6 (calificación del software), SYS.4 (integración de sistema), SYS.5 (calificación del sistema). Cada una verifica contra su nivel de requisitos o diseño, con especificación, ejecución y resultados.

**2.** "¿Qué significa trazabilidad bidireccional y consistencia en ASPICE?"
- *Puntos clave*: enlaces requisito ↔ diseño ↔ prueba ↔ resultado en ambas direcciones; consistencia = lo enlazado no se contradice. Detecta huecos y apoya el análisis de impacto.

**3.** "¿Qué debe tener una especificación de pruebas para satisfacer ASPICE?"
- *Puntos clave*: casos con precondiciones, pasos, resultado esperado, criterio de selección, trazabilidad al requisito, estrategia de regresión, resultados registrados, defectos vinculados.

**4.** "¿Cómo pruebas el stack de comunicación AUTOSAR?"
- *Puntos clave*: validar con CANoe contra el DBC/ARXML: ciclos, timeouts, valores iniciales y de fallback, E2E (CRC, alive counter), gestión de red (NM), CAN-TP y diagnóstico por UDS.

**5.** "¿Cómo conectas el DEM/DCM de AUTOSAR con tus pruebas de diagnóstico?"
- *Puntos clave*: DEM gestiona eventos, DTC, antirrebote y freeze frames; DCM atiende servicios UDS. Pruebas: generación y borrado de DTC, estados, lectura por 0x19, sesiones y seguridad.

**6.** "Diferencia entre AUTOSAR Classic y Adaptive desde la perspectiva de prueba."
- *Puntos clave*: Classic, estático, tiempo real duro, ECUs de control; Adaptive, POSIX, servicios (SOME/IP), actualizable, alto desempeño (ADAS, central compute). En Adaptive se prueban servicios, timing de comunicación, actualizaciones y aislamiento entre aplicaciones.

---

## C. Embedded Software Test Function Lead

**1.** "¿Qué son los niveles de capacidad de ASPICE y cuál buscas?"
- *Puntos clave*: 0 incompleto, 1 realizado, 2 gestionado, 3 establecido, 4 predecible, 5 innovador. Nivel 2 (gestionado: planificado, monitorizado, productos controlados) es el objetivo habitual de OEMs; 3 añade proceso estándar de la organización.

**2.** "¿Cómo te preparas para una auditoría ASPICE en el área de pruebas?"
- *Puntos clave*: autoevaluación contra prácticas base, revisión de evidencias y trazabilidad, muestreo de casos, plan de acciones para hallazgos, entrevistas ensayadas, evidencia coherente con el proceso descrito.

**3.** "El auditor encuentra pruebas sin requisito asociado. ¿Qué haces?"
- *Puntos clave*: determinar si falta el requisito o sobra la prueba, corregir el enlace o retirar la prueba, buscar la causa de proceso (revisiones, herramienta), y añadir una verificación periódica de trazabilidad.

**4.** "¿Cómo balanceas el cumplimiento de ASPICE con la agilidad?"
- *Puntos clave*: ASPICE pide resultados, no una metodología; ágil funciona si hay evidencias, trazabilidad y revisiones; automatizar la generación de evidencias en CI; evitar documentación redundante.

**5.** "¿Qué papel tiene la estrategia de pruebas de integración en AUTOSAR con múltiples proveedores?"
- *Puntos clave*: contratos de interfaz (ARXML, DBC), pruebas de integración tempranas con ECUs virtuales o restbus, responsabilidades claras, criterios de aceptación de entregas y gestión de configuración de versiones.

**6.** "¿Qué aportan las extensiones de ASPICE para ciberseguridad y seguridad funcional?"
- *Puntos clave*: procesos adicionales alineados con ISO/SAE 21434 y ISO 26262 (análisis de amenazas, requisitos de ciberseguridad, verificación); más trazabilidad y evidencias específicas; relevancia por UN R155.

---

## D. Test Manager

**1.** "¿Por qué los OEMs exigen ASPICE a sus proveedores y qué beneficios obtienes?"
- *Puntos clave*: reduce riesgo de calidad y de entrega, da transparencia, base común de evaluación. Beneficios: procesos repetibles, menos retrabajo, acceso a contratos.

**2.** "¿Cómo planificas recursos para mantener el nivel de capacidad objetivo?"
- *Puntos clave*: roles de calidad, formación, herramientas de trazabilidad, tiempo para revisiones y evidencias en la estimación, auditorías internas periódicas.

**3.** "¿Cómo reportas el cumplimiento de ASPICE?"
- *Puntos clave*: resultados de evaluación por proceso, hallazgos abiertos con plazo, cobertura de trazabilidad, tendencia; vinculado a riesgo y a fechas de auditoría del cliente.

**4.** "Un cliente exige AUTOSAR Adaptive y tu equipo es de Classic. ¿Qué haces?"
- *Puntos clave*: análisis de brechas, formación, proyecto piloto, perfiles externos, riesgo en el plan, comunicar costo y calendario con transparencia.

---

## Preguntas transversales (cierre o "curveball")

**1.** "Explica ASPICE a un directivo no técnico."
- *Puntos clave*: "Es un examen de cómo trabajamos, no del producto: demuestra que desarrollamos de manera ordenada, medible y repetible, para que el cliente confíe en las entregas."

**2.** "¿Qué relación hay entre ASPICE, AUTOSAR e ISO 26262?"
- *Puntos clave*: ASPICE evalúa procesos; AUTOSAR estandariza la arquitectura de software; ISO 26262 regula la seguridad funcional. Se complementan y se pueden aplicar en el mismo proyecto.

**3.** "Cuéntame una experiencia con una auditoría o evaluación de proceso."
- *Puntos clave*: ejemplo STAR: situación, tu responsabilidad, evidencia preparada, hallazgo, acción correctiva y resultado. Se profundiza en el Módulo 3.
