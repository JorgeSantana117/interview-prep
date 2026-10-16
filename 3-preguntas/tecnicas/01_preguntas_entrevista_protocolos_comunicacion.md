# Tema: Protocolos de Comunicación (CAN, CAN FD, LIN, CAN-TP, UDS, XCP, Automotive Ethernet)

> Diferencia clave con el Módulo 1: ahí estudiaste **teoría** ("¿qué es X?"). Aquí lo que verás es cómo un entrevistador real **empaqueta esa teoría en preguntas de escenario, troubleshooting y experiencia**, que es como realmente se evalúa en campo. No memorices respuestas — usa los puntos clave para construir tu propia respuesta con tus palabras y, cuando puedas, con un ejemplo real de Bosch/ALTEN/VW/Ford.

> Tip general de estructura de respuesta: para preguntas de escenario/troubleshooting, usa un patrón tipo **"Qué observarías → Qué hipótesis descartarías primero → Cómo lo confirmarías → Qué acción tomarías"**. Muestra proceso de pensamiento, no solo la respuesta final.

---

## A. Embedded Software Engineer

Este rol se evalúa más a nivel de **implementación y debugging de bajo nivel**. Esperan que hables con soltura de código, registros, y comportamiento del sistema.

**1.** "Tienes un ECU que reporta datos incorrectos vía CAN, pero el CRC de las tramas es válido. ¿Dónde buscarías el problema?"
- *Puntos clave*: si el CRC es válido, el problema no está en la capa física/enlace — probablemente es un error de **interpretación en la capa de aplicación** (escalado incorrecto de una señal, endianness mal configurado, un DBC/A2L desactualizado). Menciona que revisarías primero la definición de la señal (factor de escala, offset, byte order) antes de sospechar del bus.

**2.** "Explícame qué pasaría a nivel de software si el buffer de recepción de CAN-TP se llena antes de que la capa de aplicación (UDS) procese los datos."
- *Puntos clave*: el Flow Control (WT o incluso pausar el envío de más Consecutive Frames) es el mecanismo que previene esto; a nivel software, hablarías de colas circulares, prioridad de tareas/interrupciones, y qué pasa si la tarea que consume el buffer tiene menor prioridad que la ISR de recepción CAN.

**3.** "¿Cómo depurarías, a nivel de código, por qué una ECU no está entrando correctamente a Extended Diagnostic Session?"
- *Puntos clave*: revisar si el servicio 0x10 con sub-función 0x03 realmente llega (trace/log a nivel driver CAN), verificar precondiciones de la ECU (conditionsNotCorrect, NRC 0x22), y confirmar si depende de un Security Access previo mal implementado.

**4.** "¿Qué consideraciones de firmware cambian cuando pasas de implementar comunicación sobre CAN clásico a CAN FD?"
- *Puntos clave*: manejo de buffers más grandes (hasta 64 bytes), timing de interrupciones si el controlador no tiene DMA suficiente, y compatibilidad del stack de comunicación (driver, RTOS) con el nuevo tamaño de trama y BRS.

**5.** "Estás debuggeando un patch de software y sospechas que hay una condición de carrera (race condition) relacionada con el procesamiento de mensajes CAN entrantes. ¿Cómo lo confirmarías?"
- *Puntos clave*: esto conecta directamente con tu experiencia de "software patch development" en Bosch — habla de logging con timestamps de alta resolución, uso de herramientas de trace (ej. las que usarías en QNX/Linux), y cómo reproducirías el escenario de forma determinista antes de aplicar un fix.

**6.** "¿Qué harías si el equipo de Qualcomm/QNX te reporta un defecto de plataforma que se manifiesta como pérdida intermitente de mensajes en el bus, pero tu análisis de CAN/CAN-TP no muestra ningún error de protocolo?"
- *Puntos clave*: esta pregunta está diseñada para tu experiencia real de colaboración cross-team — el punto clave es demostrar que sabes **delimitar la causa raíz entre capas** (aplicación vs plataforma/SO vs bus físico) antes de asumir de quién es la responsabilidad, y cómo compartirías evidencia técnica concreta (logs, traces) para validar la hipótesis con el otro equipo.

**7.** "¿Cómo asegurarías, desde el diseño de tu software, que un flashing vía UDS pueda recuperarse si se interrumpe a la mitad (ej. pérdida de energía)?"
- *Puntos clave*: hablar de mecanismos como un bootloader dual-bank, verificación de checksum antes de marcar el firmware como válido, y cómo el 0x31 (Routine Control) de verificación de integridad encaja en ese diseño.

---

## B. Test Engineer

Este rol se evalúa en **diseño de casos de prueba, automatización y ejecución sistemática**. Esperan que conectes el protocolo con cómo lo *pruebas*, no solo cómo funciona.

**1.** "Diseña, a alto nivel, un caso de prueba para validar el mecanismo de Security Access de una ECU."
- *Puntos clave*: casos positivos (key correcta → acceso concedido) y negativos (key incorrecta → NRC 0x35, exceso de intentos → NRC 0x36 y bloqueo temporal). Menciona que también probarías el comportamiento en distintas sesiones (¿se puede pedir Security Access en Default Session?).

**2.** "¿Cómo automatizarías, en CANoe/vTESTstudio, la validación de que una ECU respeta correctamente el STmin durante una transacción CAN-TP?"
- *Puntos clave*: necesitas medir el tiempo real entre Consecutive Frames en el trace y compararlo contra el STmin acordado en el Flow Control — habla de cómo extraerías timestamps del log y qué tolerancia definirías como criterio de pass/fail.

**3.** "Tienes que probar el comportamiento de una ECU cuando pierde comunicación CAN por 2 segundos y luego se recupera. ¿Qué validarías?"
- *Puntos clave*: esto conecta con "fault management" y "robustness" de tu CV — validarías detección del fallo (posible DTC generado), comportamiento del sistema durante la pérdida (modo degradado/fail-safe), y recuperación correcta al restablecerse la comunicación (¿vuelve a estado normal automáticamente o requiere reset?).

**4.** "¿Cómo diseñarías un caso de prueba negativo para el servicio de flashing (0x34/0x36/0x37) que valide el manejo de un requestSequenceError (NRC 0x24)?"
- *Puntos clave*: intencionalmente enviar Transfer Data (0x36) sin haber hecho Request Download (0x34) primero, y verificar que la ECU responda con el NRC correcto en vez de aceptar la transferencia o comportarse de forma indefinida.

**5.** "¿Qué diferencia hay, desde el punto de vista de testing, entre validar una señal transmitida por CAN tradicional (basada en ID fijo) y un servicio expuesto vía SOME/IP?"
- *Puntos clave*: en CAN pruebas contra una matriz de señales estática (DBC); en SOME/IP necesitas validar también el **Service Discovery** (¿se anuncia correctamente el servicio? ¿se descubre en el tiempo esperado?) además del contenido del payload — es una dimensión de prueba adicional que no existe en CAN clásico.

**6.** "Si un test automatizado en CANoe reporta fallas intermitentes al leer un DID vía 0x22, pero manualmente (con vTESTstudio paso a paso) siempre funciona, ¿qué investigarías?"
- *Puntos clave*: sospecha de timing — posible condición de carrera entre el test automatizado y el estado real de la ECU (¿está en la sesión correcta al momento exacto de la ejecución automatizada?), delays insuficientes entre pasos del script, o un Tester Present que no se está manteniendo correctamente durante la ejecución automatizada.

**7.** "¿Cómo estructurarías la trazabilidad entre un requisito de comunicación (ej. 'la ECU debe soportar CAN FD con BRS') y tus casos de prueba en DOORS/RQM?"
- *Puntos clave*: conecta con tu experiencia real de "Analizar requisitos, definir especificaciones de prueba" — habla de descomponer el requisito en criterios de aceptación verificables (ej. medir la velocidad real de la fase de datos en el trace) y vincular cada caso de prueba de vuelta al ID del requisito para reportes de cobertura.

**8.** "¿Qué considerarías al diseñar pruebas de robustez (robustness testing) para el manejo de tramas CAN malformadas?"
- *Puntos clave*: inyectar tramas con DLC incorrecto, CRC corrupto intencionalmente, IDs no reconocidos, y validar que la ECU no se cuelgue, no entre en un estado inconsistente, y genere los errores/DTCs esperados sin afectar otras funciones.

---

## C. Embedded Software Test Function Lead

Este rol se evalúa en **estrategia de verificación, coordinación técnica y visión de sistema**, no solo ejecución. Esperan respuestas que muestren cómo piensas a nivel de bloque/función completa, no solo un caso de prueba aislado.

**1.** "¿Cómo definirías la estrategia de verificación para el soporte de un nuevo protocolo (ej. migrar de UDS sobre CAN-TP a UDS sobre DoIP) en un módulo ADAS?"
- *Puntos clave*: habla de identificar qué cambia (capa de transporte) vs qué se mantiene (servicios UDS), priorizar pruebas de regresión sobre lo que no debería cambiar, y diseñar pruebas nuevas específicas para DoIP (descubrimiter de vehículo por IP, manejo de conexión TCP). Menciona cómo coordinarías con el equipo para dividir el alcance.

**2.** "Tu equipo reporta que las pruebas de CAN FD pasan en el banco HIL de México pero fallan intermitentemente en el banco de la India. ¿Cómo abordarías esto como líder del equipo?"
- *Puntos clave*: esta pregunta está diseñada para tu experiencia real de coordinar equipos en US/India/México — el punto clave es no asumir causa raíz sin evidencia: comparar configuración de hardware/firmware de ambos bancos, longitud de cable (afecta integridad de señal a alta velocidad de datos en FD), y establecer un proceso para que ambos equipos compartan logs de forma comparable.

**3.** "¿Cómo priorizarías qué protocolos/funciones probar primero si tienes recursos limitados y un milestone ajustado?"
- *Puntos clave*: conecta con "Definir y trackear actividades de prueba contra milestones" — habla de priorización basada en riesgo (funciones de seguridad primero), criticidad para el cliente/OEM, y qué áreas tienen mayor probabilidad de defectos por ser código nuevo vs. ya validado.

**4.** "¿Cómo le explicarías a un stakeholder no técnico por qué encontrar un defecto de timing en CAN-TP (ej. violación de STmin) es importante, si 'el mensaje al final sí llega completo'?"
- *Puntos clave*: mostrar que sabes traducir un problema técnico a impacto de negocio/seguridad — un STmin violado puede funcionar en el banco de pruebas pero fallar en campo con hardware más lento o bus más cargado, lo que se traduce en riesgo de recall o falla en campo, no solo "un detalle técnico".

**5.** "¿Cómo estructurarías el onboarding de un nuevo integrante del equipo que nunca ha trabajado con UDS, para que en 2-3 semanas pueda ser productivo en verificación de flashing?"
- *Puntos clave*: conecta directamente con "Onboarded and mentored new team members" de tu CV — menciona una progresión: primero teoría y trace-reading, luego ejecución de casos existentes, luego diseño de casos nuevos con supervisión, con checkpoints claros.

**6.** "¿Qué harías si detectas que la cobertura de pruebas de un módulo tiene un hueco importante en el manejo de Negative Response Codes, pero ya estás cerca del milestone de entrega?"
- *Puntos clave*: evalúa cómo manejas trade-offs de riesgo vs tiempo — habla de cuantificar el riesgo (¿qué tan crítico es ese NRC específico?), comunicarlo transparentemente a stakeholders con una recomendación clara, y proponer un plan (ej. cobertura mínima ahora + plan de seguimiento post-milestone) en vez de simplemente aceptar el hueco en silencio.

**7.** "¿Cómo defines y comunicas métricas de progreso de verificación a stakeholders que no son expertos en protocolos de comunicación?"
- *Puntos clave*: habla de traducir cobertura técnica (% de servicios UDS probados, % de escenarios de error cubiertos) a indicadores de riesgo/calidad entendibles (ej. semáforos, tendencias de defectos abiertos por severidad).

---

## D. Test Manager

Este rol se evalúa en **gestión de riesgo, recursos, procesos y relación con el negocio** — se espera menos profundidad de protocolo puro y más visión de cómo el conocimiento técnico impacta planeación, calidad y entrega.

**1.** "¿Cómo evaluarías el riesgo de introducir un nuevo protocolo (ej. Automotive Ethernet/SOME-IP) en el plan de verificación de un programa que históricamente solo ha usado CAN?"
- *Puntos clave*: habla de riesgo desde varias dimensiones: madurez del equipo en el nuevo protocolo (necesidad de training), disponibilidad de herramientas (¿el banco HIL actual soporta Ethernet?), y el impacto en el cronograma si se requieren nuevas herramientas o certificaciones.

**2.** "Si un proveedor entrega una ECU con soporte incompleto de DoIP y esto pone en riesgo el milestone, ¿cómo lo gestionarías?"
- *Puntos clave*: esto se conecta con tu experiencia colaborando con "software suppliers" — habla de escalar con evidencia técnica clara (no solo "no funciona", sino qué específicamente falla y su impacto), negociar un plan de mitigación con el proveedor, y comunicar el riesgo de cronograma a los stakeholders internos de forma proactiva, no reactiva.

**3.** "¿Cómo decides cuántos recursos asignar a pruebas de robustez de protocolos de comunicación vs. pruebas funcionales, con presupuesto limitado?"
- *Puntos clave*: evalúa pensamiento de gestión de riesgo basado en impacto — pruebas de robustez de protocolos suelen ser menos visibles para el cliente pero de altísimo impacto en seguridad/campo; habla de cómo justificarías esa inversión con datos (historial de defectos, criticidad ASIL si aplica).

**4.** "¿Cómo asegurarías que el equipo mantenga el conocimiento técnico actualizado conforme la industria migra de CAN a protocolos como Automotive Ethernet?"
- *Puntos clave*: habla de estrategia de desarrollo de talento a nivel de equipo, no solo individual — planes de capacitación, rotación de personal entre proyectos CAN/Ethernet, y cómo medirías que el equipo está realmente listo (no solo que tomó un curso).

**5.** "Un stakeholder pregunta por qué el ciclo de pruebas de una ECU con soporte multi-protocolo (CAN + LIN + Ethernet) toma más tiempo que uno solo con CAN. ¿Cómo lo explicarías y justificarías el plan?"
- *Puntos clave*: traduce complejidad técnica a impacto de planeación — más protocolos significan más combinaciones de escenarios de interacción entre buses (ej. gateway CAN-LIN, DoIP vs CAN-TP), lo que incrementa la matriz de pruebas de forma no lineal, no solo aditiva.

---

## Preguntas transversales (aplican a cualquier rol — suelen usarse como "curveball" al final)

**1.** "¿Cuál es el protocolo o tecnología de comunicación automotriz en el que te sientes menos fuerte, y qué has hecho o harías para mejorar en eso?"
- *Puntos clave*: sé honesto — dado tu perfil, protocolos como TSN o SOME/IP a fondo (si no los has tocado en producción) son respuestas razonables. Muestra curiosidad genuina y, si puedes, menciona que este mismo plan de estudio es parte de cómo te preparas activamente.

**2.** "Si tuvieras que explicarle a alguien sin background técnico qué haces en tu trabajo diario relacionado con protocolos de comunicación, ¿cómo lo dirías?"
- *Puntos clave*: evalúa comunicación, no conocimiento técnico — practica una explicación breve y clara sin jerga (ej. "me aseguro de que las distintas computadoras del auto se comuniquen correctamente entre sí y que, si algo falla, el sistema lo detecte a tiempo").

**3.** "Cuéntame de una vez en la que un defecto relacionado con un protocolo de comunicación (CAN, UDS, etc.) resultó ser más complejo de lo que parecía inicialmente."
- *Puntos clave*: prepara un ejemplo real de tu experiencia (Bosch/ALTEN/VW/Ford) con estructura STAR — este tipo de pregunta suele evaluarse junto con habilidades blandas, así que vale la pena tenerlo ya preparado desde ahora aunque lo desarrollemos a fondo en el módulo correspondiente.
