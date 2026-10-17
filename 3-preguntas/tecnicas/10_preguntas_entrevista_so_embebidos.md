# Tema: Sistemas Operativos Embebidos (RTOS, OSEK/AUTOSAR OS, Linux, QNX)

> Material de estudio base: capítulo `10_sistemas_operativos_embebidos.md`. Mismo formato que los bancos anteriores. Tu perfil incluye Linux y QNX; prepara ejemplos propios de comandos y análisis que hayas usado.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* síntoma → hipótesis (de la más barata de descartar a la más cara) → cómo confirmas → acción y evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "¿Qué diferencia a un RTOS de un SO de propósito general?"
- *Puntos clave*: determinismo y latencia acotada, planificación por prioridades con apropiación, huella pequeña, sin garantías de equidad. "Tiempo real" significa cumplir plazos, no ser rápido. Distingue duro, firme y suave.

**2.** "Explica los estados de una tarea y las transiciones."
- *Puntos clave*: suspendida/dormida, lista (ready), en ejecución, en espera (bloqueada). Transiciones por activación, planificación, apropiación, bloqueo por evento o recurso y liberación.

**3.** "Compara planificación por prioridades fijas, Round Robin, Rate Monotonic y EDF."
- *Puntos clave*: prioridades fijas con apropiación (típico en automoción); Round Robin reparte tiempo entre iguales; RM asigna prioridad según frecuencia (menor periodo, mayor prioridad) y es óptimo entre prioridades fijas; EDF asigna por plazo más cercano, utilización hasta 100 % en teoría.

**4.** "Calcula la planificabilidad: tareas con periodos 10, 20, 50 ms y tiempos de ejecución 2, 4, 10 ms."
- *Puntos clave*: utilización U = 2/10 + 4/20 + 10/50 = 0,2 + 0,2 + 0,2 = 0,6. Para 3 tareas el límite RM es 3(2^(1/3) − 1) ≈ 0,78; como 0,6 < 0,78, es planificable. Mencionar el análisis de tiempo de respuesta como método más exacto.

**5.** "¿Qué es la inversión de prioridad y cómo se mitiga?"
- *Puntos clave*: una tarea de alta prioridad espera a una de baja que tiene un recurso, mientras una de prioridad media se ejecuta. Mitigación: herencia de prioridad o protocolo de techo de prioridad (el que usa OSEK para recursos), secciones críticas cortas.

**6.** "Diferencia entre mutex, semáforo y cola de mensajes."
- *Puntos clave*: mutex = exclusión mutua con propietario; semáforo = conteo/señalización; cola = transferencia de datos entre tareas. Elegir según sincronización o intercambio de información; riesgo de bloqueo mutuo.

**7.** "¿Cómo funciona OSEK/AUTOSAR OS? ¿Qué tipos de tareas hay?"
- *Puntos clave*: configuración estática, tareas básicas y extendidas (con eventos), ISR de categoría 1 y 2, alarmas y contadores, recursos con techo de prioridad, hooks, protección de tiempo y memoria en AUTOSAR OS (ScalabilityClass).

**8.** "Explica la gestión de memoria: MMU, MPU, pila y fragmentación."
- *Puntos clave*: MPU protege regiones en MCU; MMU ofrece memoria virtual en procesadores mayores; pila por tarea con dimensionamiento y detección de desbordamiento; evitar asignación dinámica en código crítico por fragmentación y no determinismo.

---

## B. Test Engineer

**1.** "Una ECU con Linux responde lento intermitentemente. ¿Cómo analizas?"
- *Puntos clave*: `top`/`htop` (CPU, procesos), `vmstat`/`free` (memoria, swap), `iostat` (E/S), `dmesg` (kernel), `strace` (llamadas al sistema), `perf` o `ftrace` para latencias; correlacionar con la carga; reproducir y registrar.

**2.** "En QNX, ¿qué herramientas usas para ver procesos y uso de CPU?"
- *Puntos clave*: `pidin` (procesos, hilos, memoria), `hogs` (consumo de CPU), `slog2info` (logs), `tracelogger` y System Profiler/ Momentics para trazas del kernel. Usar los que ofrezca la versión disponible.

**3.** "¿Cómo pruebas el tiempo de arranque de una ECU con SoC?"
- *Puntos clave*: medir desde el encendido hasta el servicio listo (ej. mensaje de red o interfaz), instrumentar con marcas de tiempo del log y del bus, repetir en frío y caliente, varios ciclos, comparar con el requisito y analizar etapas (bootloader, kernel, servicios).

**4.** "¿Qué verificarías sobre el comportamiento de un watchdog?"
- *Puntos clave*: reset al dejar de alimentarlo, ventana válida (demasiado pronto o tarde), causa de reset registrada, estado tras el reinicio, DTC, y que no se alimente desde una ISR que oculte bloqueos de tareas.

**5.** "Una tarea periódica pierde plazos bajo carga. ¿Cómo lo investigas?"
- *Puntos clave*: medir tiempos de ejecución y jitter con trazas, comprobar prioridades e inversión de prioridad, ISR largas, bloqueos por recursos, carga de bus; reproducir con carga controlada.

**6.** "¿Qué pruebas diseñarías para el manejo de datos compartidos entre tareas?"
- *Puntos clave*: estrés con diferentes tiempos y prioridades, escenarios de apropiación en puntos críticos, verificación de consistencia de datos, pruebas de larga duración y revisión del diseño de sincronización.

---

## C. Embedded Software Test Function Lead

**1.** "¿Cómo eliges entre AUTOSAR Classic, Linux y QNX para una nueva ECU?"
- *Puntos clave*: requisitos de determinismo, seguridad (ASIL), complejidad de aplicación, costo y soporte, ecosistema, certificaciones (QNX certificado ISO 26262), actualizaciones y ciberseguridad. Documentar con un trade study.

**2.** "¿Cómo adaptas tu estrategia de pruebas a un sistema con hipervisor y varias VMs?"
- *Puntos clave*: aislamiento y asignación de recursos, interferencia entre dominios, pruebas de temporización por VM, comunicación entre VMs, fallo de una VM sin afectar a las demás, arranque y recuperación.

**3.** "¿Qué evidencia pides para demostrar determinismo del SO?"
- *Puntos clave*: medición de latencia de interrupción y de planificación bajo carga, WCET y análisis de planificabilidad, trazas con histograma de jitter, pruebas de estrés prolongadas.

**4.** "¿Cómo estandarizas la recolección de logs y trazas del SO en todos los bancos?"
- *Puntos clave*: lista fija de herramientas y comandos, scripts automáticos tras cada fallo, formato único, almacenamiento junto con el resultado de la prueba y la versión del software.

**5.** "Linux con PREEMPT_RT, ¿cuándo es suficiente?"
- *Puntos clave*: mejora latencias y determinismo con kernel apropiativo, suficiente para tiempo real firme o suave según requisito; no tiene la certificación de seguridad de un RTOS comercial; se valida con medición (cyclictest).

---

## D. Test Manager

**1.** "¿Qué habilidades de SO requieres en el equipo de pruebas y cómo las cubres?"
- *Puntos clave*: Linux/QNX básico para diagnóstico, lectura de logs y trazas, scripting; formación, guías, mentoría y soporte de desarrollo para casos complejos.

**2.** "Un cliente cambia de plataforma de SO a mitad del programa. ¿Cuál es el impacto?"
- *Puntos clave*: reescritura de scripts de entorno, nuevas herramientas, regresión completa, riesgo de calendario, formación y presupuesto; presentar opciones y costos antes de aceptar.

**3.** "¿Cómo justificas pruebas de larga duración (soak tests)?"
- *Puntos clave*: detectan fugas de memoria, degradación y errores que aparecen tras horas; costo de banco frente al riesgo de campo; ejecución nocturna o de fin de semana automatizada.

**4.** "¿Qué métricas reportas sobre estabilidad del sistema?"
- *Puntos clave*: tiempo medio entre fallos, reinicios no planificados, uso de memoria con el tiempo, jitter y pérdidas de plazo, tendencia entre versiones.

---

## Preguntas transversales (cierre o "curveball")

**1.** "¿Por qué Linux estándar no se considera tiempo real duro?"
- *Puntos clave*: el kernel no es completamente apropiativo ni tiene latencias acotadas; interrupciones y secciones no apropiables; soluciones: PREEMPT_RT, co-kernel (Xenomai) o un RTOS.

**2.** "Describe un microkernel y por qué QNX se usa en sistemas críticos."
- *Puntos clave*: el núcleo mínimo (planificación, IPC, interrupciones) y el resto (drivers, sistemas de archivos) como procesos en espacio de usuario; un fallo de driver no tumba el sistema; paso de mensajes nativo; certificaciones de seguridad.

**3.** "Cuéntame de un problema de sistema operativo (memoria, CPU, planificación) que depuraste."
- *Puntos clave*: ejemplo STAR: síntoma, herramientas y comandos, causa raíz, solución y prevención. Se profundiza en el Módulo 3.
