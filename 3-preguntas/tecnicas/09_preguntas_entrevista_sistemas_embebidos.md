# Tema: Sistemas Embebidos

> Material de estudio base: capítulo `10_sistemas_embebidos.md`. Mismo formato que los bancos anteriores. Para el rol de Embedded Software Engineer este es un tema central; para los demás, se espera que entiendas el hardware lo suficiente para diagnosticar síntomas observados en el banco.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* síntoma → hipótesis (de la más barata de descartar a la más cara) → cómo confirmas → acción y evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "Describe la anatomía de una ECU automotriz."
- *Puntos clave*: microcontrolador (núcleo, memorias, periféricos), alimentación y supervisión (SBC), transceptores de bus (CAN, LIN), etapas de entrada (acondicionamiento, ADC) y de salida (drivers, etapas de potencia), conectores, protecciones. Menciona dónde vive la lógica de seguridad.

**2.** "Diferencia entre microcontrolador, microprocesador, SoC, FPGA y ASIC."
- *Puntos clave*: MCU integra CPU, memoria y periféricos en un chip (control en tiempo real); MPU requiere memoria externa y corre SO complejos; SoC integra múltiples bloques; FPGA es lógica reconfigurable; ASIC es silicio a medida. Elegir por costo, desempeño, determinismo y seguridad.

**3.** "Explica el mapa de memoria de un programa embebido: secciones `.text`, `.data`, `.bss`, pila y heap."
- *Puntos clave*: `.text`/`.rodata` en Flash; `.data` en Flash con copia a RAM al arranque; `.bss` inicializada a cero; pila y heap en RAM. Riesgos: desbordamiento de pila, fragmentación del heap (se evita en código crítico), variables sin inicializar.

**4.** "¿Qué es endianness y cuándo causa problemas?"
- *Puntos clave*: orden de bytes (little/big endian). Problemas al intercambiar datos entre ECUs o con herramientas, en DBC (Intel vs Motorola), en uniones y casts de punteros y en protocolos binarios.

**5.** "¿Cómo diseñas una buena ISR?"
- *Puntos clave*: corta, sin bloqueos ni llamadas pesadas, solo captura el evento y delega a la tarea mediante banderas o colas, cuidado con variables compartidas (`volatile`, secciones críticas), prioridades y anidamiento definidos, limpiar banderas correctamente.

**6.** "Compara polling, interrupciones y DMA."
- *Puntos clave*: polling = simple y determinista pero consume CPU; interrupciones = respuesta rápida a eventos con sobrecarga de contexto; DMA = mueve bloques sin CPU, ideal para ADC y comunicación. Elegir según latencia, carga y complejidad.

**7.** "¿Cómo se maneja un reset y qué fuentes existen?"
- *Puntos clave*: encendido (POR), watchdog, brown-out, reset por software, externo, falla de reloj o de memoria. Registrar la causa al arranque, inicializar de forma determinista y decidir qué datos persisten.

**8.** "Explica ADC: resolución, muestreo y errores."
- *Puntos clave*: resolución en bits y LSB, frecuencia de muestreo (Nyquist), referencia de tensión, ruido, aliasing, filtrado, tiempo de adquisición, calibración. Ejercicio típico: 12 bits y 5 V → LSB ≈ 1,22 mV.

**9.** "¿Cómo funciona un PWM y qué parámetros controlas?"
- *Puntos clave*: frecuencia, ciclo de trabajo, resolución; control de motores, solenoides, iluminación; efectos sobre ripple, pérdidas de conmutación y ruido; medición del ciclo de trabajo para diagnóstico.

**10.** "¿Qué mecanismos de seguridad y confiabilidad hay a nivel de hardware embebido?"
- *Puntos clave*: watchdog (ventana), ECC en memoria, lockstep de núcleos, MPU/MMU, BIST, monitores de reloj y de tensión, redundancia de sensores, diagnósticos de pines. Relación con ASIL.

---

## B. Test Engineer

**1.** "En HIL, una ECU se reinicia aleatoriamente. ¿Qué causas de hardware consideras?"
- *Puntos clave*: caídas de tensión (alimentación del banco, cableado), picos, watchdog por sobrecarga, reset externo por línea ruidosa, temperatura, ruido de masa. Medir alimentación con osciloscopio, revisar causa de reset registrada y comparar con otra unidad.

**2.** "Una entrada analógica lee valores ruidosos. ¿Cómo investigas?"
- *Puntos clave*: ruido del banco o de la señal, referencia, filtrado, cableado y blindaje, resolución, muestreo, divisor de tensión; medir con osciloscopio, comparar con otra ECU, revisar la configuración del VT System.

**3.** "¿Qué fallas eléctricas típicas inyectas con un banco HIL?"
- *Puntos clave*: circuito abierto, corto a masa, corto a batería, corto entre líneas, resistencia de fuga, sobretensión/subtensión, caída de tensión. Verificar diagnóstico, DTC y estado seguro.

**4.** "¿Cómo pruebas los modos de energía y el wake-up de una ECU?"
- *Puntos clave*: secuencias sleep/wake por bus, por entrada digital y temporizador; medir corriente de reposo; validar tiempos de arranque, pérdida de datos, reinicios y causa de wake-up registrada.

**5.** "¿Cómo diferencias un fallo de software de uno de hardware en el banco?"
- *Puntos clave*: reproducibilidad, dependencia de la unidad (probar otra ECU), de la temperatura o carga, mediciones eléctricas, comparación con SIL/PIL, logs de reset y diagnósticos. Descartar primero lo más barato (cableado, alimentación, configuración).

**6.** "Explica los tipos de memoria de una ECU y qué se prueba de cada una."
- *Puntos clave*: Flash (programa, borrado por sectores, ciclos de escritura), RAM (volátil), EEPROM/Data Flash (datos persistentes), registros. Pruebas: flasheo, persistencia tras apagado, integridad (CRC), corrupción por corte de energía.

---

## C. Embedded Software Test Function Lead

**1.** "¿Qué conocimientos de hardware esperas del equipo de pruebas y cómo los desarrollas?"
- *Puntos clave*: lectura de esquemas básicos, uso de osciloscopio y multímetro, comprensión de alimentación y buses, formación cruzada con hardware, guías de diagnóstico del banco y casos reales documentados.

**2.** "¿Cómo estructuras un procedimiento de aislamiento de fallas entre software, hardware y banco?"
- *Puntos clave*: lista de verificación ordenada por costo, reproducir con otra unidad y otro banco, medir señales, comparar versiones, registrar evidencias, clasificar la causa y alimentar un catálogo de fallas conocidas.

**3.** "¿Qué diferencias de hardware entre muestras (A/B/C-Sample) afectan tus pruebas?"
- *Puntos clave*: componentes no definitivos, tolerancias, revisiones de PCB, funciones deshabilitadas, errata del microcontrolador; mantener registro de revisión de hardware en cada resultado y criterios distintos por muestra.

**4.** "¿Cómo gestionas el riesgo de fallos intermitentes de hardware en un programa?"
- *Puntos clave*: pruebas de larga duración y de ciclo de encendido, registro automático de condiciones, correlación con temperatura y tensión, escalamiento temprano al proveedor con evidencia reproducible.

**5.** "¿Cómo aseguras la compatibilidad entre versiones de microcontrolador, bootloader y software de aplicación?"
- *Puntos clave*: matriz de compatibilidad, versionado conjunto, pruebas de flasheo cruzadas, gestión de configuración y criterios de liberación.

---

## D. Test Manager

**1.** "¿Cómo planificas la disponibilidad de muestras de hardware para pruebas?"
- *Puntos clave*: calendario de entregas del proveedor, cantidad por banco y reserva para fallas, trazabilidad de unidades, escalamiento por retrasos, plan alterno con simulación.

**2.** "Un defecto de hardware aparece tarde. ¿Cómo gestionas el impacto?"
- *Puntos clave*: análisis de causa raíz, evaluación de impacto en el calendario y la seguridad, mitigación por software si es posible, comunicación con el cliente y el proveedor, cambios de plan de prueba y regresión.

**3.** "¿Cómo justificas inversión en equipos de medición (osciloscopios, analizadores)?"
- *Puntos clave*: reducción del tiempo de diagnóstico, menos escalamientos, mejor evidencia hacia proveedores; costo frente a días de banco detenido.

**4.** "¿Cómo reportas problemas de hardware a un cliente sin culpar al proveedor?"
- *Puntos clave*: hechos y evidencias, impacto y riesgo, acciones propuestas, tono neutral, decisiones y responsables claros.

---

## Preguntas transversales (cierre o "curveball")

**1.** "Explica qué pasa desde que se enciende una ECU hasta que ejecuta `main()`."
- *Puntos clave*: reset, vector de reinicio, inicialización de reloj y memoria (copia de `.data`, cero de `.bss`), bootloader opcional, inicialización de pila, llamada a `main` y del SO/RTE.

**2.** "Si solo pudieras medir una señal en una ECU que falla, ¿cuál sería y por qué?"
- *Puntos clave*: normalmente la alimentación y el reset, porque muchos síntomas aleatorios se originan ahí; justifica según el contexto.

**3.** "Cuéntame de un problema que parecía de software y resultó ser de hardware, o al revés."
- *Puntos clave*: ejemplo STAR: síntoma, hipótesis, mediciones, causa real, acción y aprendizaje. Se profundiza en el Módulo 3.
