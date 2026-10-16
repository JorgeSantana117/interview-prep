# Tema: Ingeniería de Sistemas

> Material de estudio base: capítulo `08_ingenieria_de_sistemas.md`. Mismo formato que los bancos anteriores. Tema muy valorado en roles de Test Function Lead y Test Manager, donde se espera visión de sistema, seguridad y programa.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* contexto → criterio → decisión → evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "¿Cómo se relacionan los requisitos de sistema con los de software que implementas?"
- *Puntos clave*: el sistema se descompone y asigna a hardware, software y mecánica; los requisitos de software derivan de los del sistema con trazabilidad; las interfaces con hardware u otras ECUs se definen explícitamente.

**2.** "¿Qué debe definir una interfaz bien especificada entre ECUs o entre software y hardware?"
- *Puntos clave*: señales/mensajes, tipos y unidades, rangos, resolución, temporización (ciclo, latencia), comportamiento ante fallo y valor inválido, protocolo, versionado y responsable de cada lado.

**3.** "¿Qué es un presupuesto de diseño y cuáles encuentras en software embebido?"
- *Puntos clave*: límites asignados a cada elemento: tiempo de CPU, memoria (Flash/RAM/pila), latencia, consumo, carga de bus. Se vigilan durante el desarrollo para evitar sorpresas en integración.

**4.** "Explica ASIL y qué cambia en tu código según el nivel."
- *Puntos clave*: ASIL A–D por gravedad, exposición y controlabilidad (HARA). A mayor nivel: más rigor (revisiones, cobertura MC/DC, métodos formales, independencia), mecanismos de seguridad (monitores, redundancia, watchdog), reglas de codificación.

**5.** "¿Qué es un estado seguro y cómo se implementa?"
- *Puntos clave*: condición del sistema sin riesgo inaceptable (por ejemplo, torque cero o modo degradado); lo alcanza el mecanismo de seguridad dentro del FTTI tras detectar la falla; debe ser verificable.

---

## B. Test Engineer

**1.** "Explica el V-Model y el W-Model, y su relación con las pruebas."
- *Puntos clave*: V: cada fase de definición tiene su verificación (requisitos ↔ calificación, arquitectura ↔ integración, diseño ↔ unidad). W: las actividades de prueba empiezan en paralelo con las de definición (revisión de requisitos y diseño de pruebas desde el inicio).

**2.** "¿Qué defectos típicos de interfaz buscas en pruebas de integración?"
- *Puntos clave*: unidades o factores de escala distintos, orden de bytes (endianness), rangos incompatibles, temporización diferente a la especificada, versiones desalineadas de DBC/ARXML, manejo de valores inválidos y de timeout distinto en cada lado.

**3.** "¿Cómo derivas pruebas a partir de un análisis HARA?"
- *Puntos clave*: de cada objetivo de seguridad se derivan requisitos de seguridad con FTTI y estado seguro; pruebas de inyección de fallas, tiempo de reacción, verificación del estado seguro, recuperación y de no ocurrencia de reacciones indebidas.

**4.** "¿Qué es FMEA y cómo te sirve para diseñar pruebas?"
- *Puntos clave*: análisis de modos de falla y efectos: por componente, causa, efecto, detección, severidad. Cada modo de falla relevante es un candidato para un caso de inyección de falla y para verificar la detección.

**5.** "Explica FTA frente a FMEA."
- *Puntos clave*: FTA es deductiva (de un evento superior hacia causas, con compuertas AND/OR); FMEA es inductiva (de fallas de componentes hacia efectos). Se complementan.

**6.** "¿Qué es SOTIF y cómo se prueba?"
- *Puntos clave*: ISO 21448: riesgos sin falla del sistema por limitaciones funcionales o mal uso previsible (por ejemplo, percepción en condiciones extremas). Se prueba con escenarios, simulación, pruebas en campo y análisis de cobertura de escenarios.

**7.** "¿Cómo gestionas las pruebas con múltiples variantes de producto?"
- *Puntos clave*: casos comunes más específicos por variante, parametrización con datos de configuración, matriz de cobertura por variante, priorizar las más representativas o de mayor volumen.

---

## C. Embedded Software Test Function Lead

**1.** "¿Cómo defines el plan de V&V de un sistema?"
- *Puntos clave*: qué se verifica en cada nivel y con qué método (IADT), entorno (SIL, HIL, vehículo), criterios de aceptación, recursos, hitos A/B/C/D y SOP, riesgos y dependencias; alineado con ASPICE SYS.4/SYS.5 y con el plan de seguridad.

**2.** "Explica los hitos A, B, C, D-Sample y SOP, y qué pruebas esperas en cada uno."
- *Puntos clave*: A: concepto y funciones básicas; B: funcionalidad mayormente completa; C: cercano a producción, validación amplia; D: producción en herramientas definitivas; SOP: inicio de producción. La madurez y cobertura de pruebas crece por hito.

**3.** "¿Cómo planificas una estrategia de integración de sistema (bottom-up, top-down, big-bang)?"
- *Puntos clave*: integración incremental reduce el riesgo y facilita localizar fallas; usar stubs, restbus y ECUs virtuales para lo que aún no está; elegir según disponibilidad de componentes y de proveedores.

**4.** "¿Qué haces cuando una arquitectura E/E nueva (dominios, zonal, centralizada) cambia tu estrategia de pruebas?"
- *Puntos clave*: más software en menos ECUs, Ethernet, servicios, virtualización; más pruebas de integración y de rendimiento de red; aumento de SIL/vECU; actualizaciones OTA y ciberseguridad.

**5.** "¿Cómo evalúas la evidencia de pruebas para un caso de seguridad (safety case)?"
- *Puntos clave*: trazabilidad de objetivo → requisito → prueba → resultado, cobertura, independencia, configuración exacta, defectos abiertos con análisis de riesgo, revisión y aprobación.

**6.** "¿Qué son los TPMs y cómo los usas?"
- *Puntos clave*: medidas técnicas de desempeño (latencia, memoria, consumo, precisión) con valor objetivo y tendencia a lo largo del proyecto; alertan temprano de desviaciones.

---

## D. Test Manager

**1.** "¿Cómo alineas el plan de pruebas con el calendario del programa y los hitos?"
- *Puntos clave*: mapear actividades de prueba a cada hito, dependencias con desarrollo y proveedores, hitos de entrada/salida, colchones, revisión periódica del plan y comunicación de riesgos.

**2.** "¿Cómo gestionas el riesgo de integración con muchos proveedores?"
- *Puntos clave*: contratos de interfaz claros, integración temprana y frecuente, calidad mínima de entregas, entornos compartidos, escalamiento definido, registro de riesgos con dueños.

**3.** "¿Cómo participas en un trade study (por ejemplo, SIL en la nube vs más HIL)?"
- *Puntos clave*: criterios ponderados (costo, cobertura, fidelidad, plazo, riesgo), alternativas, sensibilidad, recomendación y decisión documentada.

**4.** "¿Cómo se comunica el riesgo residual cuando no se completó toda la verificación de un sistema crítico?"
- *Puntos clave*: lista de lo no verificado, impacto potencial, mitigaciones, aceptación formal del riesgo por el responsable; nunca ocultar ni minimizar.

---

## Preguntas transversales (cierre o "curveball")

**1.** "¿Cuál es la diferencia entre verificar un sistema y validarlo?"
- *Puntos clave*: verificar = cumple los requisitos especificados; validar = cumple la necesidad del usuario en su entorno real. Ejemplo: HIL verifica; las pruebas en vehículo y en carretera validan.

**2.** "Explica MBSE a alguien que usa documentos de Word."
- *Puntos clave*: el modelo (SysML) es la fuente única de verdad, del que se generan vistas y documentos; mejora consistencia y análisis de impacto; exige inversión en herramientas y formación.

**3.** "Cuéntame de una vez que una visión de sistema te ayudó a encontrar la causa de un problema."
- *Puntos clave*: ejemplo STAR: síntoma en una ECU, causa en una interfaz o en otro componente, cómo lo aislaste y el cambio resultante. Se profundiza en el Módulo 3.
