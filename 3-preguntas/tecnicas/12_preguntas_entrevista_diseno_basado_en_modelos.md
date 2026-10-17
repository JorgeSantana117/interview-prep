# Tema: Diseño Basado en Modelos (Simulink / Stateflow)

> Material de estudio base: capítulo `12_diseno_basado_en_modelos.md`. Mismo formato que los bancos anteriores. Tu conexión natural: los modelos de planta usados en HIL y las pruebas back-to-back MIL/SIL.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* síntoma → hipótesis → cómo confirmas → acción y evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "¿Qué es el diseño basado en modelos y qué ventajas y riesgos tiene?"
- *Puntos clave*: se diseña y valida el comportamiento en un modelo ejecutable y se genera código; ventajas: simulación temprana, trazabilidad, menos errores de codificación manual; riesgos: confianza ciega en el generador, modelos mal estructurados, dependencia de herramientas y licencias.

**2.** "Explica los solvers de Simulink: paso fijo vs variable."
- *Puntos clave*: paso fijo (obligatorio para código embebido y tiempo real; define el periodo base); paso variable para simulación de dinámicas continuas. Elegir el solver y el paso según la dinámica más rápida y el costo de cómputo.

**3.** "Punto fijo: ¿por qué se usa y qué errores comete?"
- *Puntos clave*: ECUs sin FPU o por determinismo y costo; tipos con escala y offset, rango y resolución; riesgos: saturación, desbordamiento, pérdida de precisión, redondeo; verificar con pruebas de límites y comparación con doble precisión.

**4.** "Describe la semántica de ejecución de Stateflow."
- *Puntos clave*: orden de evaluación de transiciones (por prioridad/orden geométrico), acciones de entrada, durante y salida, eventos, estados paralelos y jerárquicos, transiciones por defecto; ejemplos de transiciones en bucle y de acciones con efectos secundarios no esperados.

**5.** "¿Qué esperas del código generado y cómo lo integras en el proyecto?"
- *Puntos clave*: funciones de inicialización y paso (`step`), datos en estructuras, dependencia del compilador, cumplimiento de MISRA (con configuración), integración con RTE/AUTOSAR (SWC), revisión del código en puntos críticos.

**6.** "¿Cómo mantienes consistentes el modelo, el código y el A2L/ARXML?"
- *Puntos clave*: generación automatizada en el pipeline, versión única del modelo, artefactos etiquetados, verificación de que el A2L corresponde al build, evitar edición manual del código generado.

---

## B. Test Engineer

**1.** "Explica MIL, SIL y PIL y la prueba back-to-back."
- *Puntos clave*: comparar la salida del modelo (MIL) con el código generado (SIL/PIL) con las mismas entradas, dentro de tolerancias; detecta errores del generador, tipos, redondeos y diferencias de compilador/procesador.

**2.** "¿Cómo pruebas un modelo de Stateflow?"
- *Puntos clave*: transición de estados (válidas e inválidas), cobertura de estados, transiciones y condiciones (MC/DC en seguridad), eventos concurrentes, temporizaciones, casos desde requisitos; herramientas: Simulink Test, Design Verifier.

**3.** "¿Qué revisas en un modelo antes de probarlo?"
- *Puntos clave*: revisión estática: guías de modelado (MAB), nombres, tamaños de bloque, tipos, uso de bloques no recomendados, ciclos algebraicos, jerarquía, comentarios; herramientas de chequeo (Model Advisor).

**4.** "Un modelo de planta funciona en simulación pero causa overruns en HIL. ¿Qué haces?"
- *Puntos clave*: medir tiempo de ejecución por paso, simplificar dinámicas y lookup tables grandes, ajustar paso fijo, dividir tareas en múltiples tasas, mover partes a FPGA o a otro núcleo, verificar fidelidad tras simplificar.

**5.** "¿Cómo validas la fidelidad de un modelo de planta?"
- *Puntos clave*: comparar con mediciones reales de vehículo o banco (respuesta a escalón, rangos), documentar el rango de validez y las simplificaciones, revisar con el responsable del modelo, versionar.

**6.** "¿Cómo conviertes una especificación en casos de prueba sobre un modelo?"
- *Puntos clave*: analizar requisitos, definir entradas y salidas esperadas, valores límite y escenarios, trazabilidad, automatizar con scripts o Simulink Test y reportar cobertura del modelo.

---

## C. Embedded Software Test Function Lead

**1.** "¿Cómo defines la estrategia de pruebas en un flujo MBD (MIL → SIL → PIL → HIL)?"
- *Puntos clave*: reutilizar los mismos casos de prueba en todos los niveles, comparar resultados entre niveles, definir qué defecto se espera hallar en cada uno, criterios de paso por nivel y cobertura objetivo.

**2.** "¿Cómo se relaciona MBD con ASPICE e ISO 26262?"
- *Puntos clave*: el modelo se considera diseño y/o requisito ejecutable; se exige trazabilidad, revisión, verificación del modelo y del código; calificación de herramientas (generador, TCL); pautas de modelado.

**3.** "¿Qué haces si el equipo de modelado y el de pruebas interpretan distinto un requisito?"
- *Puntos clave*: revisión conjunta del requisito, aclaración documentada con el responsable, actualización de modelo/casos, y revisión temprana compartida para prevenir.

**4.** "¿Cómo gestionas versiones de modelos y de datos asociados?"
- *Puntos clave*: repositorio con modelos, diccionarios de datos, scripts de inicialización y configuraciones; etiquetas por release; evitar binarios sin revisión (usar formatos diffables) y flujo de merge definido.

**5.** "¿Cuál es el valor de ejecutar el modelo de planta en SIL/vECU en la nube?"
- *Puntos clave*: escala (muchas ejecuciones en paralelo), costo, ejecución temprana en CI; limitaciones: fidelidad de timing y hardware, necesidad de confirmar en HIL.

---

## D. Test Manager

**1.** "¿Qué inversión requiere adoptar MBD y cómo la justificas?"
- *Puntos clave*: licencias, formación, herramientas de verificación; beneficios: detección temprana, menos retrabajo, reutilización de pruebas entre niveles; medir con defectos hallados en MIL/SIL frente a HIL/vehículo.

**2.** "¿Cómo gestionas el riesgo de depender de un generador de código?"
- *Puntos clave*: calificación de herramienta, pruebas back-to-back, revisión de código en áreas críticas, versionado del generador, plan ante actualizaciones.

**3.** "¿Qué indicadores usarías para medir madurez de MBD en el programa?"
- *Puntos clave*: cobertura de modelo, cumplimiento de guías de modelado, número de defectos por nivel, tiempo de ciclo de simulación a prueba en banco, porcentaje de pruebas reutilizadas.

**4.** "Un cliente exige cobertura de modelo del 100 %. ¿Cómo respondes?"
- *Puntos clave*: aclarar el tipo de cobertura y el nivel de integridad, justificar cobertura no alcanzable (código muerto, restricciones), análisis y aceptación documentada.

---

## Preguntas transversales (cierre o "curveball")

**1.** "¿Por qué el código generado también necesita pruebas?"
- *Puntos clave*: el generador puede tener defectos o hacer suposiciones; el compilador y el hardware pueden diferir; la integración y la configuración introducen errores. Los resultados de MIL no garantizan el comportamiento del ejecutable.

**2.** "Explica un modelo de planta con un ejemplo automotriz."
- *Puntos clave*: modelo de motor o de pedal con sensores, dinámica del actuador y del vehículo que reacciona a las salidas de la ECU; en HIL entrega señales realistas y recibe las respuestas de la ECU.

**3.** "Cuéntame de una vez que un modelo o su simulación te ayudó a localizar un defecto."
- *Puntos clave*: ejemplo STAR: síntoma, simulación o comparación back-to-back, causa, corrección y verificación. Se profundiza en el Módulo 3.

