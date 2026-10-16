# Capítulo: Sistemas Operativos Embebidos

> Objetivo del capítulo: entender **qué hace el sistema operativo dentro de una ECU o un controlador de dominio**, cómo se diferencian un RTOS, Linux embebido y QNX, cómo se planifican las tareas, cómo se comunican los procesos, cómo arranca el sistema y, sobre todo, **cómo se depura un problema a nivel de sistema**. Este capítulo es el más cercano a tu rol actual en Bosch (QNX, Qualcomm, debugging de infotainment).
>
> **Límites con otros capítulos (para no repetir):**
> - Hardware, interrupciones a nivel de CPU, memoria física, periféricos → ya en *Sistemas Embebidos*. Aquí se ve cómo el SO los usa y los abstrae.
> - Race conditions, deadlock e inversión de prioridades desde el **diseño de software** → ya en *Ingeniería de Software*. Aquí se ven los **mecanismos del SO** que las provocan o las mitigan.
> - AUTOSAR Classic/Adaptive como arquitectura → ya en *ASPICE/AUTOSAR*. Aquí el SO que está debajo (AUTOSAR OS, POSIX).
> - Flasheo por UDS → ya en *Protocolos de Comunicación*. Aquí el rol del bootloader dentro del arranque.
> - Docker, Jenkins, pipelines → irán en *CI/CD y Control de Versiones*.

---

## 1. Qué hace un sistema operativo embebido

Un sistema operativo (SO) administra los recursos del procesador y ofrece servicios al software de aplicación:
- **Planificación de tareas o procesos** (quién usa la CPU y cuándo).
- **Gestión de memoria** (asignación y, en algunos SO, aislamiento entre procesos).
- **Comunicación y sincronización** entre tareas (IPC).
- **Acceso a hardware** mediante drivers y abstracciones.
- **Gestión de tiempo** (temporizadores, alarmas).
- **Servicios de sistema** (archivos, red, registro de eventos).

No todo sistema embebido lleva SO. Hay tres niveles:

| Enfoque | Descripción | Cuándo se usa |
|---|---|---|
| **Bare-metal / super-loop** | Un bucle principal más interrupciones, sin SO | Funciones muy simples y de bajo costo |
| **RTOS** | Núcleo pequeño con planificación determinista | ECUs de control con plazos estrictos |
| **SO de propósito amplio (Linux, QNX, Android)** | Procesos aislados, memoria virtual, red, sistemas de archivos | Infotainment, cockpit, ADAS de alto desempeño, controladores de dominio |

## 2. Tiempo real y conceptos de planificación

### 2.1 Qué significa "tiempo real"
No significa "rápido": significa **predecible**. Un sistema es de tiempo real si la corrección depende no solo del resultado, sino de **cuándo** se produce.
- **Hard real-time (duro)**: perder un plazo es un fallo inaceptable (por ejemplo, el control de frenado).
- **Firm real-time**: un resultado tardío no sirve, pero ocasionalmente se tolera.
- **Soft real-time (blando)**: un resultado tardío degrada la calidad, sin ser catastrófico (por ejemplo, fluidez de una interfaz gráfica).

Métricas importantes: **latencia** (de interrupción y de planificación), **jitter** (variación de la latencia), **tiempo de respuesta** y **WCET (Worst-Case Execution Time)**, el tiempo de ejecución en el peor caso.

### 2.2 Estados de una tarea
Una tarea típicamente está en uno de estos estados: **lista (ready)**, **en ejecución (running)**, **bloqueada (blocked/waiting)** esperando un evento o recurso, o **suspendida**. El **cambio de contexto** (guardar y restaurar el estado de la CPU) tiene un costo que cuenta en el presupuesto de tiempo.

### 2.3 Algoritmos de planificación
- **Prioridades fijas con desalojo (preemptive fixed-priority)**: la tarea lista de mayor prioridad siempre se ejecuta, desalojando a las demás. Es el modelo dominante en RTOS automotrices.
- **Round-robin**: tareas de la misma prioridad comparten la CPU por turnos (rebanadas de tiempo).
- **Rate Monotonic (RM)**: asigna mayor prioridad a la tarea con **menor periodo**. Es óptimo entre los esquemas de prioridad fija para tareas periódicas.
- **EDF (Earliest Deadline First)**: prioridad dinámica según el plazo más cercano. Teóricamente permite hasta 100% de utilización en el caso ideal, pero es más complejo de analizar.
- **Cooperativo vs apropiativo**: en cooperativo, la tarea cede la CPU voluntariamente; en apropiativo, el SO puede quitársela.

### 2.4 Análisis de planificabilidad
- **Utilización**: U = Σ (Cᵢ / Tᵢ), donde C es el tiempo de ejecución y T el periodo.
- **Cota de Liu–Layland para RM**: con n tareas, si U ≤ n·(2^(1/n) − 1), el conjunto es planificable. Para n = 3 la cota es ≈ 0.78, y tiende a ≈ 0.69 cuando n crece. Es una condición **suficiente, no necesaria**: superarla no implica fallo, solo que hay que analizar con más detalle.
- **Análisis de tiempo de respuesta (RTA)**: calcula el peor tiempo de respuesta de cada tarea considerando la interferencia de las de mayor prioridad. Es más preciso que la cota.

### 2.5 Sincronización y comunicación entre tareas
| Mecanismo | Para qué sirve | Nota |
|---|---|---|
| **Mutex** | Exclusión mutua sobre un recurso; tiene **propietario** | Puede implementar herencia de prioridades |
| **Semáforo (binario/contador)** | Señalización o control de acceso a N recursos | No tiene propietario |
| **Eventos / banderas** | Notificar que ocurrió algo | Muy usado en AUTOSAR OS |
| **Cola de mensajes** | Pasar datos entre tareas | Desacopla productor y consumidor |
| **Memoria compartida** | Intercambio de datos rápido | Requiere sincronización explícita |
| **Señales, pipes, sockets** | IPC en SO tipo POSIX | Sockets permiten comunicación entre máquinas |

**Mutex vs semáforo:** el mutex lo libera quien lo tomó y sirve para proteger recursos; el semáforo puede ser liberado por otra tarea o por una ISR y sirve sobre todo para señalizar. Confundirlos es un error clásico de entrevista.

### 2.6 Problemas clásicos (los mecanismos que los causan o los mitigan)
- **Inversión de prioridades**: una tarea de alta prioridad espera un recurso tomado por una de baja, mientras una de prioridad media se ejecuta y retrasa indefinidamente a la baja (y con ello a la alta). Se mitiga con **herencia de prioridades** (la tarea de baja hereda temporalmente la prioridad de la que espera) o con **techo de prioridad (priority ceiling)**. Un caso famoso es el de la misión Mars Pathfinder, donde este fenómeno causó reinicios del sistema y se corrigió habilitando la herencia de prioridades.
- **Deadlock**: ciclo de espera entre tareas por recursos. Se previene con un orden global de adquisición y secciones críticas cortas.
- **Starvation (inanición)**: una tarea nunca obtiene CPU por culpa de las de mayor prioridad.
- **Overrun**: una tarea excede su plazo o su presupuesto de tiempo. Los SO de seguridad ofrecen **monitoreo de tiempo** para detectarlo.

## 3. Sistemas operativos en la ECU de control: OSEK/AUTOSAR OS

Es el SO de **AUTOSAR Classic**, derivado del estándar OSEK/VDX.
- **Configuración estática**: tareas, alarmas, recursos y prioridades se definen en tiempo de configuración y no cambian en ejecución. Resulta en un SO pequeño, determinista y analizable.
- **Tipos de tarea**: **básicas** (se ejecutan hasta terminar, sin esperar eventos) y **extendidas** (pueden esperar eventos).
- **ISRs por categoría**: categoría 1 (sin servicios del SO, muy rápidas) y categoría 2 (pueden usar servicios del SO).
- **Alarmas y contadores**: base de la ejecución periódica.
- **Recursos con protocolo de techo de prioridad inmediato**: evita inversión de prioridades y deadlocks.
- **Clases de escalabilidad (SC1–SC4)**: de funcionalidad básica a **protección de tiempo** y **protección de memoria** (en SC3/SC4 se apoya en la MPU), lo que habilita mezclar software de distintos ASIL en un mismo núcleo.
- No tiene procesos con memoria virtual ni sistema de archivos: es un planificador de tareas con servicios mínimos, sobre una MCU.

## 4. Linux embebido

### 4.1 Características
- Núcleo **monolítico** con módulos cargables, **memoria virtual (MMU)** y procesos aislados, amplio soporte de drivers y de red, ecosistema enorme.
- Distribuciones embebidas construidas a medida con herramientas como **Yocto** o **Buildroot**.
- **Device Tree**: describe el hardware de la placa al kernel sin recompilarlo para cada variante.
- Base de plataformas como **Android Automotive** (que usa el kernel de Linux) y de muchas pilas de infotainment.

### 4.2 Linux y tiempo real
Linux estándar está optimizado para desempeño promedio, no para determinismo. Para plazos más estrictos existe el conjunto de parches **PREEMPT_RT**, que hace el kernel mucho más apropiativo (en versiones recientes del kernel ya está integrado de forma nativa). Se mide con herramientas como `cyclictest`. Aun así, Linux **no está certificado nativamente para ASIL**; existen iniciativas y soluciones comerciales para seguridad funcional.

### 4.3 Herramientas habituales de análisis en Linux
`dmesg` y `journalctl` (mensajes del kernel y servicios), `top`/`htop` (CPU y memoria), `strace` (llamadas al sistema), `gdb`/`gdbserver` (depuración remota), `perf` y `ftrace` (perfilado y trazado del kernel), `/proc` y `/sys` (estado del sistema), archivos `core` (volcado de memoria tras un fallo) y, en Android, `logcat`.

## 5. QNX

QNX Neutrino es un SO de tiempo real **comercial**, basado en **microkernel** y compatible con **POSIX**, muy presente en infotainment, cluster y ADAS. Hay versiones certificadas para seguridad funcional (QNX OS for Safety), con calificación para ASIL alto.

### 5.1 Arquitectura de microkernel
- El **microkernel** contiene solo lo esencial: planificación de hilos, IPC, manejo de interrupciones y temporizadores.
- Todo lo demás corre como **procesos en espacio de usuario**: drivers, sistemas de archivos, pila de red, el *process manager* (`procnto`).
- **Ventaja**: **aislamiento y robustez**. Si un driver falla, es un proceso que se puede reiniciar sin tumbar el sistema; el kernel es pequeño y más fácil de verificar.
- **Costo**: más cambios de contexto y paso de mensajes que en un kernel monolítico (compensado por un IPC muy optimizado).

### 5.2 Paso de mensajes (IPC nativo)
- La comunicación se basa en **mensajes síncronos** (llamadas tipo `MsgSend`, `MsgReceive`, `MsgReply`): el cliente se bloquea hasta que el servidor responde.
- **Resource managers**: procesos que exponen un servicio (por ejemplo, un driver) como una entrada del espacio de nombres de archivos; las aplicaciones lo usan con `open/read/write`.
- **Herencia de prioridad en el IPC**: el servidor que atiende a un cliente de alta prioridad adopta temporalmente esa prioridad, lo que reduce la inversión de prioridades.
- **Pulsos**: notificaciones asíncronas cortas, útiles para eventos y temporizadores.

### 5.3 Planificación
Políticas POSIX como **FIFO** y **round-robin**, con prioridades por hilo, y opciones adicionales como **planificación esporádica** y el **Adaptive Partitioning Scheduler (APS)**, que reserva un porcentaje de CPU a grupos de procesos para impedir que uno acapare la CPU (aislamiento temporal).

### 5.4 Arranque en QNX (visión general)
Un esquema típico: **IPL** (carga inicial) → **startup** (inicializa hardware básico, pasa información al kernel) → **procnto** (kernel y gestor de procesos) → scripts/servicios de arranque → drivers y aplicaciones. El sistema suele empaquetarse en una **imagen** (IFS) que se carga en memoria.

### 5.5 Herramientas de análisis y depuración en QNX (ejemplos habituales)
- `pidin`: información de procesos, hilos, estados y uso de memoria.
- Registro de eventos con **slogger2** (`slog2info` para consultarlo).
- `top`/`hogs`: consumo de CPU por proceso.
- **Kernel trace** (trazado de eventos del kernel, con herramientas como el *System Profiler* del IDE de QNX): muestra planificación, interrupciones, IPC y latencias.
- `dumper` y **archivos core**: volcado de memoria cuando un proceso falla, para análisis post mortem con `gdb`.
- `gdb` para el objetivo (cross-debugging) y herramientas de perfilado de memoria.

Los nombres y opciones exactos pueden variar según la versión de QNX y del entorno, por lo que conviene validarlos en la documentación del proyecto.

## 6. Hipervisores y virtualización

En cockpits modernos es común un **hipervisor de tipo 1** que ejecuta varios sistemas operativos **invitados** sobre un mismo SoC, cada uno en una **partición**. Un ejemplo habitual de la industria: un invitado con SO de tiempo real/seguridad (para cluster y funciones críticas) y otro con Android (para infotainment), compartiendo hardware.
- **Beneficio**: consolida cómputo (menos ECUs), aísla funciones de distinta criticidad (*mixed criticality*) y permite combinar ecosistemas.
- **Retos**: aislamiento de tiempo y memoria (*freedom from interference*), reparto de periféricos (GPU, buses), comunicación entre particiones y mayor complejidad de depuración.
- Afecta a las pruebas: un síntoma puede originarse en un invitado, en el hipervisor, en el reparto de recursos o en el hardware.

## 7. Gestión de memoria en el SO

| Aspecto | RTOS en MCU (OSEK/AUTOSAR) | SO con MMU (Linux, QNX) |
|---|---|---|
| Espacio de direcciones | Plano, compartido (con MPU opcional) | **Virtual**, un espacio por proceso |
| Aislamiento | Por MPU si se configura | Por MMU, un fallo no corrompe a otros procesos |
| Memoria dinámica | Evitada o con *pools* de tamaño fijo | Disponible (`malloc`), con riesgo de fragmentación |
| Paginación a disco | No | Normalmente no en embebido (o limitada) |
| Fallos típicos | Corrupción por desbordamiento de pila o de buffer | Segmentation fault, fugas de memoria, OOM |

- **Memory pools**: bloques de tamaño fijo preasignados para obtener asignación determinista y sin fragmentación.
- **Fugas de memoria (memory leaks)**: reservas que nunca se liberan. En sistemas que operan por horas o días degradan el sistema y terminan en fallos o en el **OOM killer** de Linux, que termina procesos cuando se agota la memoria.
- **Stack por tarea**: cada tarea/hilo tiene su pila; dimensionarla mal causa desbordamientos difíciles de diagnosticar.

## 8. Drivers, BSP y capas de abstracción

- **Driver**: software que gestiona un periférico (CAN, Ethernet, GPU, almacenamiento) y expone una interfaz al resto del sistema.
- **BSP (Board Support Package)**: conjunto de bootloader, drivers, configuración y herramientas de una **placa o SoC concreto** que permite que un SO corra sobre ese hardware. Lo suele entregar el proveedor del SoC (por ejemplo, Qualcomm) o del SO (por ejemplo, QNX).
- **HAL**: capa que abstrae el hardware para que el código de aplicación no dependa de él.

*Conexión con tu rol:* cuando colaboras con equipos de QNX y Qualcomm para investigar defectos de plataforma, normalmente tratas con el **BSP, los drivers y el kernel**. Poder delimitar si un síntoma pertenece a la aplicación, al SO, al driver, al BSP o al hardware es justamente lo que hace útil un reporte técnico.

## 9. Proceso de arranque (boot)

### 9.1 ECU con MCU (AUTOSAR Classic)
1. **Reset / power-on**: la MCU empieza a ejecutar desde una dirección fija (vector de reset).
2. **Código de arranque (startup)**: inicializa pila, reloj y memoria, **copia `.data` de Flash a RAM** y **pone a cero `.bss`** (secciones del capítulo anterior).
3. **Bootloader** (si existe): decide si arranca la aplicación o entra a modo de reprogramación (flasheo por UDS). Normalmente reside en una región protegida de la Flash y verifica la integridad de la aplicación antes de cederle el control.
4. **Arranque de AUTOSAR**: el gestor de ECU (EcuM) y el de modos (BswM) inicializan el Basic Software y arrancan el SO y las tareas.
5. **Aplicación en operación**: se inicia la comunicación de red y las funciones.

### 9.2 SoC con SO de propósito amplio (Linux o QNX)
1. **ROM de arranque (boot ROM)**: código inmutable en el chip que localiza y carga la siguiente etapa.
2. **Bootloaders por etapas**: inicializan reloj, memoria DRAM y periféricos básicos (por ejemplo, SPL/U-Boot en Linux; IPL y startup en QNX).
3. **Kernel**: se carga y se inicializa el SO.
4. **Servicios y drivers**: procesos de inicio (por ejemplo, `systemd` o scripts en QNX).
5. **Aplicaciones/HMI**.

### 9.3 Arranque seguro y tiempo de arranque
- **Secure boot / cadena de confianza**: cada etapa verifica criptográficamente (con la ayuda de un HSM y claves protegidas) la firma de la siguiente antes de ejecutarla. Impide cargar software no autorizado.
- **Tiempo de arranque**: en infotainment es un requisito de calidad y regulatorio (por ejemplo, la imagen de la cámara de reversa debe aparecer en pocos segundos; verifica el valor exacto vigente en la norma o especificación aplicable). Se optimiza paralelizando servicios, retrasando los no críticos y, en algunos casos, con arranque dividido (un SO rápido para lo crítico y otro más lento para el resto).
- **Pruebas típicas en HIL**: medir el tiempo desde la ignición hasta que cada función está disponible, ciclos repetidos de arranque/apagado, arranque tras pérdida de alimentación y comportamiento ante un firmware corrupto.

## 10. Depuración y diagnóstico a nivel de sistema

### 10.1 Fuentes de información
- **Logs** del SO y de las aplicaciones (con marcas de tiempo y niveles de severidad).
- **Volcados (core dumps)** cuando un proceso termina de forma anormal. Para analizarlos se necesita el **binario con símbolos que corresponda exactamente al build** que falló, igual que ocurre con el A2L.
- **Trazas del kernel** (planificación, interrupciones, IPC) para problemas de timing y de latencia.
- **Métricas de recursos**: uso de CPU y memoria por proceso, estado de hilos, número de descriptores abiertos.
- **Estado del watchdog y causa del último reset** (ver capítulo anterior).
- **Trazas de bus** (CANoe) correlacionadas con las del SO mediante marcas de tiempo sincronizadas.

### 10.2 Un flujo de diagnóstico reutilizable
Ante un fallo en una prueba HIL (por ejemplo, un proceso se reinicia o el sistema se congela de forma intermitente):
1. **Describir el síntoma con precisión**: qué falla, cuándo, con qué frecuencia y bajo qué condiciones.
2. **Preservar la evidencia**: logs completos con marcas de tiempo, versión exacta del software y de la plataforma, y volcados si existen, antes de reiniciar o limpiar.
3. **Correlacionar tiempos**: alinear el instante del fallo en el log del SO con el trace de CAN/Ethernet y con las acciones del caso de prueba.
4. **Clasificar la naturaleza del fallo**: ¿crash de proceso (segmentation fault, aserción), congelamiento (deadlock, bucle, inversión de prioridades), degradación progresiva (fuga de memoria, saturación de CPU) o reinicio de la plataforma (watchdog, brown-out)?
5. **Analizar el volcado** con el depurador y la pila de llamadas (*backtrace*), cuando haya crash.
6. **Medir recursos** antes y durante el fallo (CPU, memoria, hilos bloqueados) y, si se sospecha de timing, **capturar un trace del kernel**.
7. **Intentar reproducirlo de forma determinista** (estrés, carga de bus, secuencia exacta), reduciendo variables.
8. **Formular una hipótesis y probarla** con un cambio mínimo (un parche, una configuración, una prueba dirigida).
9. **Escalar con evidencia**: reporte con pasos de reproducción, logs, volcado, versiones y tu hipótesis, para que el equipo de plataforma (QNX/Qualcomm/proveedor) pueda actuar sin repetir tu trabajo.

*Conexión con tu CV:* este flujo es la base de "source-level debugging, patch development, build generation y validación dirigida" y de "compartir hallazgos técnicos y validar correcciones" con los equipos de QNX y Qualcomm.

## 11. Comparación de plataformas

| Aspecto | AUTOSAR OS (OSEK) | RTOS de propósito general | Linux embebido | QNX | Android Automotive |
|---|---|---|---|---|---|
| Hardware típico | MCU | MCU / MPU | MPU / SoC | MPU / SoC | SoC (cockpit) |
| Arquitectura | Planificador estático | Kernel pequeño | Monolítico | Microkernel | Basado en kernel Linux, capa de aplicaciones |
| Determinismo | Muy alto | Alto | Medio (mejora con PREEMPT_RT) | Alto | Bajo (blando) |
| Aislamiento de procesos | Opcional (MPU) | Limitado | Sí (MMU) | Sí (MMU, a nivel de drivers también) | Sí (sandboxing de apps) |
| Seguridad funcional | Hay implementaciones certificadas | Hay versiones certificadas | Limitada (iniciativas y soluciones comerciales) | Versiones certificadas | No es el objetivo; suele aislarse de funciones críticas |
| Uso típico | ECUs de control | Funciones embebidas variadas | Infotainment, gateways, ADAS | Cluster, infotainment, ADAS | Infotainment con ecosistema de apps |
| Soporte POSIX | No | Parcial | Sí | Sí | Parcial (bionic) |

## 12. Del síntoma a la causa probable (nivel SO)

| Síntoma observado | Hipótesis a considerar |
|---|---|
| Un proceso se cae con "segmentation fault" | Acceso a puntero inválido, desbordamiento de buffer o de pila, uso tras liberar memoria |
| El sistema se vuelve lento con el tiempo | Fuga de memoria, saturación de CPU por un proceso, crecimiento de colas o logs |
| Congelamiento sin reinicio | Deadlock, inversión de prioridades, bucle sin ceder CPU, hilo de alta prioridad acaparando |
| Reinicio de la plataforma sin crash visible | Watchdog sin alimentar, brown-out, panic del kernel, falla de hardware |
| Tareas periódicas pierden plazos solo bajo carga | Presupuesto de tiempo excedido, latencia de interrupciones, inversión de prioridades, I/O bloqueante |
| Pérdida intermitente de mensajes de bus | ISR o hilo de recepción demasiado lento, buffers insuficientes, prioridad inadecuada |
| El arranque es más lento de lo requerido | Servicios secuenciales innecesarios, inicialización de drivers lenta, verificación de integridad costosa |
| Falla que no se reproduce al depurar con *breakpoints* | Problema de timing que el depurador altera; usar trazado en lugar de detener la ejecución |
| El comportamiento cambia entre builds sin cambios aparentes | Diferencias de configuración, versión de BSP/driver, orden de arranque, compilador |
| Un invitado de un hipervisor se comporta mal | Reparto de recursos, aislamiento de tiempo/memoria, driver virtualizado, comunicación entre particiones |

## Ejercicios de práctica
1. **Utilización**: tres tareas periódicas con (C, T) = (1 ms, 4 ms), (2 ms, 6 ms), (3 ms, 12 ms). Calcula U y compáralas con la cota de RM.
   *Respuesta:* U = 0.25 + 0.333 + 0.25 ≈ 0.833, mayor que la cota ≈ 0.78, así que la prueba suficiente no concluye. Con análisis de tiempo de respuesta (prioridad por periodo: T1 > T2 > T3) se obtiene R1 = 1 ms, R2 = 3 ms, R3 = 10 ms, y todos cumplen su plazo (≤ T). El conjunto **sí es planificable** aunque supere la cota.
2. **Inversión de prioridades**: una tarea A (alta) y una C (baja) comparten un mutex, y una tarea B (media) no lo usa. C toma el mutex, A lo solicita y se bloquea, B se vuelve lista. Describe qué ocurre sin herencia y con herencia.
   *Respuesta:* sin herencia, B desaloja a C y puede retrasar a A de forma indefinida; con herencia, C hereda la prioridad de A, termina su sección crítica, libera el mutex y A continúa.
3. **Presupuesto de CPU**: una ISR de 20 µs se dispara cada 200 µs en el peor caso. ¿Qué porcentaje de CPU consume? *(10%)*
4. **Diagnóstico**: una función de infotainment se congela una vez cada pocas horas durante una prueba larga en HIL, sin reinicio. Enumera cinco evidencias que recolectarías antes de reiniciar el sistema.

## Preguntas de teoría
1. ¿Qué significa "tiempo real" y cuál es la diferencia entre hard, firm y soft real-time?
2. Compara bare-metal, RTOS y SO de propósito amplio, con un caso automotriz de cada uno.
3. Describe los estados de una tarea y el costo de un cambio de contexto.
4. Explica la planificación apropiativa con prioridades fijas y por qué Rate Monotonic asigna mayor prioridad a menor periodo.
5. ¿Qué dice la cota de utilización de RM y por qué no es una condición necesaria?
6. Diferencia entre mutex y semáforo. ¿Cuándo usarías cada uno?
7. Explica la inversión de prioridades y las dos formas de mitigarla.
8. ¿Qué características hacen a OSEK/AUTOSAR OS adecuado para una ECU de control (configuración estática, categorías de ISR, clases de escalabilidad)?
9. ¿Por qué Linux estándar no es de tiempo real duro y qué aporta PREEMPT_RT?
10. Describe la arquitectura de microkernel de QNX y sus ventajas y costos frente a un kernel monolítico.
11. ¿Cómo funciona el paso de mensajes síncrono en QNX y qué es un resource manager?
12. ¿Qué es el Adaptive Partitioning Scheduler y qué problema resuelve?
13. ¿Qué es un hipervisor y qué retos de aislamiento trae para pruebas y seguridad funcional?
14. ¿Qué es un BSP y por qué importa en una colaboración con Qualcomm/QNX?
15. Describe la secuencia de arranque de una ECU AUTOSAR Classic, desde el reset hasta la aplicación.
16. Describe una cadena de arranque de un SoC con Linux o QNX y qué aporta el secure boot.
17. ¿Qué evidencias preservarías ante un crash de proceso y por qué el binario con símbolos debe coincidir con el build?
18. ¿Cómo distinguirías una fuga de memoria de un deadlock a partir de lo que observas en el sistema?
19. ¿Por qué un *breakpoint* puede ocultar un problema de timing y qué usarías en su lugar?
20. Un caso de prueba pasa manualmente pero falla de forma intermitente en la ejecución automatizada. ¿Qué hipótesis de SO considerarías?

## Errores comunes
- Equiparar "tiempo real" con "rápido": lo que importa es el **determinismo** y el peor caso.
- Confundir mutex con semáforo, o creer que cualquier mecanismo de sincronización evita la inversión de prioridades.
- Suponer que la cota de utilización de RM es una condición necesaria: superarla no implica automáticamente que el sistema falle.
- Presentar Linux como "tiempo real" sin matizar el uso de PREEMPT_RT, la medición de latencias y la falta de certificación nativa para ASIL.
- Describir QNX solo como "un SO de tiempo real" sin explicar su arquitectura de microkernel y el paso de mensajes.
- Tratar el hipervisor como un detalle de infraestructura, olvidando que introduce una capa más a considerar al diagnosticar y al verificar aislamiento.
- Atribuir cualquier fallo de plataforma a la aplicación (o al revés) sin delimitar la capa con evidencia.
- Reiniciar el sistema antes de recolectar logs, volcados y estado, perdiendo la evidencia del fallo.
- Analizar un core dump sin el binario y los símbolos del build exacto.
- Diagnosticar un fallo de timing con *breakpoints* en lugar de trazas.
- Ignorar la causa del último reset (watchdog, brown-out, panic) al investigar reinicios.
- Olvidar que la memoria dinámica y la fragmentación pueden romper el determinismo en sistemas de larga operación.