# Tema: HIL Testing (MIL, SIL, PIL, HIL, planta, fault injection, bancos)

> Material de estudio base: capítulo `02_hil_testing.md`. Este banco sigue el mismo formato que el de Protocolos de Comunicación: preguntas de escenario, troubleshooting y experiencia, organizadas por rol, con los puntos que un entrevistador espera escuchar. No memorices respuestas: construye la tuya con tus palabras y, cuando puedas, con un ejemplo real de Bosch, ALTEN, VW o Ford.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación o trade-off.
> - *Escenario o diagnóstico:* síntoma → hipótesis (de la más barata de descartar a la más cara) → cómo confirmas cada una → acción y evidencia que dejas.
> - *Experiencia:* método STAR (se desarrolla en el Módulo 3).

---

## A. Embedded Software Engineer

Este rol se evalúa en cómo el **software real** se comporta sobre el **hardware real** y en tu capacidad de depurar con evidencia.

**1.** "Un defecto aparece en HIL pero no en SIL con el mismo código fuente. ¿Qué diferencias entre ambos entornos podrían explicarlo?"
- *Puntos clave*: timing real (interrupciones, planificación, latencias), precisión numérica y tipos de dato (`double` en PC vs punto fijo en el objetivo), compilador y optimizaciones del objetivo, periféricos y caché, memoria y pila reales, comunicación real por bus, inicialización y reset. Confirmas con trazas y con PIL como paso intermedio.

**2.** "¿Cómo depurarías con una debug board un fallo que solo ocurre en cierto escenario de HIL?"
- *Puntos clave*: un breakpoint altera el timing, así que para problemas temporales prefieres **trazado** (ETM/trace) o instrumentación ligera; breakpoints condicionales; captura de registros y de la pila tras un fallo; correlación con el trace de CANoe mediante marcas de tiempo; observar variables con XCP sin detener la ejecución, usando el A2L del build exacto; reproducir el escenario de forma determinista.

**3.** "El simulador de tiempo real reporta *overruns* intermitentes. ¿Cómo distingues si el problema es del modelo de planta o de la ECU?"
- *Puntos clave*: el *overrun* es del **lado del simulador** (el cómputo excede el paso), no de la ECU. Medir el tiempo de ejecución por paso del modelo, revisar tamaño de paso, solver y otras tareas del objetivo de tiempo real. Si la ECU tiene problemas de plazos, se ve en sus propias trazas y en la respuesta del bus. Confirmar con mediciones separadas en ambos lados.

**4.** "¿Qué debe garantizar tu software para recuperarse bien de una caída de alimentación (*brown-out*) inyectada en HIL?"
- *Puntos clave*: arranque con inicialización definida, causa del último reset registrada, **consistencia de datos no volátiles** si la escritura se interrumpe (mecanismos de validación, doble copia), comportamiento del watchdog, persistencia correcta de DTC, estado seguro ante condiciones de alimentación fuera de rango, ausencia de corrupción de Flash/EEPROM.

**5.** "Estás desarrollando un *patch* para un defecto reportado en HIL. ¿Cómo lo validas antes de entregarlo?"
- *Puntos clave*: reproducir el defecto con el build original (línea base), aplicar el parche, **confirmar** que el caso falla antes y pasa después, ejecutar regresión dirigida por análisis de impacto, análisis estático y revisión de código, registrar versión exacta de software, modelo, DBC/A2L y banco, conservar evidencia (logs y trazas). Esto conecta con tu experiencia de desarrollo de parches y validación dirigida en Bosch.

**6.** "¿Cómo diseñarías el software de diagnóstico para facilitar la prueba de fallas en HIL sin dejar código de prueba en producción?"
- *Puntos clave*: umbrales y tiempos de antirrebote parametrizables (calibración), estado de DTC y *freeze frame* legibles por UDS, reacciones de falla bien definidas, NRC inequívocos, registros útiles. Los *hooks* de prueba se aíslan por configuración de compilación o calibración y se verifica que no estén activos en la versión liberada.

---

## B. Test Engineer

Este rol se evalúa en **diseño de casos, ejecución sistemática y análisis de resultados** en el banco.

**1.** "Explícame la diferencia entre MIL, SIL, PIL y HIL y qué detecta cada nivel."
- *Puntos clave*: MIL valida la lógica del algoritmo; SIL, la generación de código y errores de software (*back-to-back* con MIL); PIL, precisión numérica y timing en el procesador real; HIL, integración hardware + software, comunicación real y *fault management*. Ninguno reemplaza a los demás: cada uno detecta defectos distintos y cuesta más tiempo y dinero conforme se acerca al vehículo. HIL no detecta fenómenos puramente físicos (vibración, EMI real).

**2.** "¿Qué es la planta y por qué la fidelidad del modelo condiciona la validez de tus resultados?"
- *Puntos clave*: la planta es el sistema físico que reacciona al controlador (en HIL, la ECU real). Un modelo demasiado simple oculta defectos o produce falsos positivos; uno demasiado detallado no corre en tiempo real. Se valida contra mediciones reales y se documenta su rango de validez (ruido, retardos, saturaciones de sensores).

**3.** "Diseña la prueba del *timeout* de un mensaje CAN cíclico en una ECU."
- *Puntos clave*: estímulo (detener el mensaje desde CANoe), verificación del **tiempo de detección** contra el requisito (por ejemplo, 3 ciclos), DTC generado y su estado, valor sustituto y reacción de la función, recuperación al reanudar, pérdidas parciales y valores frontera de duración. Registrar trazas con timestamps y estado de DTC.

**4.** "¿Cómo inyectas y validas una falla de circuito abierto en un sensor de pedal con doble canal?"
- *Puntos clave*: con módulos de relés/cargas del VT System integrados con CANoe; casos por canal y combinados. Validar **plausibilidad** entre canales, DTC correcto, modo degradado, **tiempo de reacción frente al FTTI**, recuperación al retirar la falla y que no haya efectos en otras funciones. Incluir degradación gradual además de falla binaria.

**5.** "¿Cómo validas el flasheo UDS en HIL, incluidos los casos negativos?"
- *Puntos clave*: secuencia de sesión de programación, Security Access, descarga, transferencia, salida y verificación (0x10, 0x27, 0x34, 0x36, 0x37, 0x31, 0x11). Negativos: key inválida (NRC 0x35), exceso de intentos (0x36), error de secuencia (0x24), transferencia interrumpida por pérdida de alimentación o de comunicación, checksum incorrecto, timeouts de CAN-TP. Verificar rollback o recuperación y que 0x78 no se trate como error.

**6.** "Una prueba pasa 9 de 10 veces. ¿Qué haces?"
- *Puntos clave*: no repetirla hasta que pase. Revisar estado inicial no determinista, `sleep` fijos frente a esperas con condición, sesión o Tester Present, carga del bus, orden entre pruebas, salud del banco, versiones de DBC/A2L. Reproducir en bucle con registro detallado, aislar y clasificar (software, prueba o infraestructura).

**7.** "¿Cómo especificas y automatizas pruebas de *state management* (arranque, operación, degradado, apagado)?"
- *Puntos clave*: aplicar la técnica de transición de estado (estados, eventos, transiciones válidas e inválidas, tabla de estados), ciclos de encendido y apagado, *wake-up* por bus, temporizaciones, orden de eventos. Cobertura de estados < transiciones válidas < todas las transiciones.

**8.** "¿Qué mides y cómo en una prueba de *runtime performance* (latencias, carga de CPU, carga de bus)?"
- *Puntos clave*: latencias con timestamps del trace de CANoe, distribución (percentiles y peor caso, no solo promedio), generación de carga de bus controlada, carga de CPU con XCP o herramientas del SO, criterios de aceptación definidos, repetibilidad y condiciones del banco registradas.

---

## C. Embedded Software Test Function Lead

Este rol se evalúa en **estrategia, criterio y coordinación** entre bancos, equipos y sitios.

**1.** "¿Cómo decides qué se prueba en SIL, qué en HIL y qué en vehículo?"
- *Puntos clave*: dependencia de hardware real, necesidad de timing real, riesgo y ASIL, costo por ciclo, disponibilidad de muestras y fidelidad requerida. Algoritmos en MIL/SIL; integración, comunicación, diagnóstico y fallas eléctricas en HIL; fenómenos físicos y validación final en vehículo. Se documenta en el plan de V&V.

**2.** "Un caso pasa en HIL pero la falla aparece en vehículo. ¿Qué hipótesis consideras?"
- *Puntos clave*: fidelidad del modelo, sensores sin ruido ni retardo, calibración o variante distinta, condiciones ambientales (temperatura, EMI), carga real del bus, cableado y alimentación, otras ECUs no simuladas. Confirmas replicando condiciones y capturando trazas en vehículo; cierras actualizando el modelo y añadiendo un caso de regresión.

**3.** "Una prueba falla en el banco de un sitio y pasa en el de otro. ¿Cómo lo abordas como líder?"
- *Puntos clave*: comparación sistemática de firmware, DBC/A2L, modelo de planta, versión de CANoe y drivers, interfaces, cableado y terminación, alimentación. Estandarizar y registrar la configuración de cada banco, compartir logs comparables y una lista de verificación común. Conecta con tu experiencia coordinando EE. UU., India y México.

**4.** "¿Cómo reduces la inestabilidad (*flakiness*) de una suite HIL?"
- *Puntos clave*: estado inicial determinista, esperas con condición y tiempo límite, verificación de salud del banco antes de ejecutar, separación entre fallas de infraestructura y de software, cuarentena temporal con plazo, análisis de causa raíz, control de versiones de modelo, DBC y A2L junto con los scripts.

**5.** "¿Cómo defines el alcance y los criterios de aceptación de las pruebas HIL para un B-Sample?"
- *Puntos clave*: criterios de entrada (muestra disponible, build y modelo definidos, banco operativo), alcance priorizado por riesgo y requisitos, criterios de salida (cobertura de requisitos, defectos abiertos por severidad, estabilidad de la regresión), riesgos y plan para lo no cubierto. Alinear con los hitos del programa.

**6.** "Tres equipos compiten por tiempo de banco. ¿Cómo gestionas la capacidad?"
- *Puntos clave*: reservas y prioridad por hito y riesgo, ejecución nocturna automatizada, bloqueo de recursos en el pipeline, trasladar lo posible a SIL/vECU, métricas de utilización y tiempo en cola, acceso remoto. Comunicar el criterio de priorización de forma transparente.

**7.** "¿Cómo aseguras la calidad del modelo de planta?"
- *Puntos clave*: validación contra mediciones, rango de validez documentado, control de versiones y de cambios, responsable definido, revisión del modelo, regresión del propio modelo y trazabilidad de qué versión se usó en cada ejecución.

---

## D. Test Manager

Este rol se evalúa en **riesgo, recursos, inversión y relación con stakeholders**.

**1.** "¿Cómo justificarías la inversión en un banco HIL adicional?"
- *Puntos clave*: defectos encontrados antes, menor dependencia de vehículos, más ciclos por día, reducción de riesgo en hitos; costos de hardware, licencias, mantenimiento y del modelo de planta. Datos de utilización, cuello de botella actual y alternativas (ECUs virtuales, SIL en la nube). Presentar con números.

**2.** "La regresión completa son 600 pruebas de 36 s y hoy corre en un banco. ¿Qué capacidad necesitas para cerrarla en una ventana nocturna de 1 hora?"
- *Puntos clave*: serie = 600 × 36 s = 21 600 s = 6 h; con 3 bancos ≈ 2 h ideales; para 1 h harían falta ≈ 6 bancos (≈ 7 con 15 % de sobrecarga). Antes de comprar: priorizar por riesgo, mover casos a SIL/vECU, reducir tiempos de preparación.

**3.** "¿Cómo equilibras HIL y pruebas de vehículo para reducir costos sin elevar el riesgo?"
- *Puntos clave*: reservar el vehículo para lo que HIL no puede validar (fenómenos físicos, integración real, validación final), maximizar la cobertura en HIL, usar análisis de riesgo y ASIL, medir hallazgos tardíos que escaparon de HIL para ajustar el reparto.

**4.** "¿Qué indicadores de salud del banco HIL reportarías?"
- *Puntos clave*: disponibilidad, utilización, tiempo en cola, tasa de fallas de infraestructura frente a fallas de software, estabilidad de la suite, tiempo de recuperación del banco, costo por ejecución. Interpretarlos como tendencia y en contexto.

**5.** "Un proveedor entrega un B-Sample cuyo hardware no permite inyectar fallas en un pin clave. ¿Cuál es tu plan?"
- *Puntos clave*: cuantificar el riesgo y el impacto en el cronograma, alternativas (cableado adicional, cobertura parcial en HIL, SIL/vECU para la lógica), negociar con el proveedor con evidencia técnica concreta, comunicar el riesgo residual a stakeholders y pedir una decisión explícita.

---

## Preguntas transversales (cierre o "curveball")

**1.** "¿Cuál es la limitación de HIL que más te preocupa y cómo la mitigas?"
- *Puntos clave*: la fidelidad finita del modelo de planta; la mitigas validando contra datos reales, documentando el rango de validez y complementando con vehículo.

**2.** "Explícale HIL a alguien sin formación técnica."
- *Puntos clave*: "Probamos la computadora real del auto conectándola a un simulador que le hace creer que está dentro de un auto completo, para poder provocar fallas y situaciones peligrosas sin riesgo y repetirlas tantas veces como haga falta."

**3.** "Cuéntame de un defecto que encontraste en HIL y que habría sido mucho más caro encontrar en vehículo."
- *Puntos clave*: prepara un ejemplo real con estructura STAR: situación, qué defecto, cómo lo reproduciste y aislaste, qué evidencia entregaste, qué impacto evitaste. Este tipo de pregunta se profundiza en el Módulo 3.