# Tema: Herramientas de Testing y Automatización (CANoe, VT System, CAPL, vTESTstudio, Robot Framework)

> Material de estudio base: capítulo `05_herramientas_testing_automatizacion.md`. Mismo formato que los bancos anteriores. Este es el tema donde más te conviene hablar con ejemplos propios de Bosch, ALTEN, VW y Ford.

> Estructura recomendada para responder:
> - *Concepto:* qué es → para qué se usa → ejemplo propio → limitación.
> - *Escenario:* síntoma → hipótesis → cómo confirmas → acción y evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

Aquí se evalúa cómo usas las herramientas para depurar y validar tu propio software.

**1.** "¿Cómo usarías CANoe para depurar un mensaje que tu ECU envía con periodo incorrecto?"
- *Puntos clave*: ventana Trace con timestamps, medir el periodo real, Statistics para la carga, comparar con el DBC (ciclo y delay), revisar tareas del scheduler de la ECU, descartar arbitraje y carga del bus. Verificar con una medición de histograma de periodos.

**2.** "¿Para qué sirve el archivo DBC y qué pasa si no coincide con el software de la ECU?"
- *Puntos clave*: define mensajes, señales, factor/offset, rangos, ciclos y nodos. Un DBC desactualizado produce valores mal interpretados o *timeouts* falsos. Control de versiones del DBC junto con el build.

**3.** "Escribe (o describe) un script CAPL que detecte un mensaje faltante y lo reporte."
- *Puntos clave*: `on message` actualiza un timestamp; `on timer` periódico compara con el tiempo actual; si excede el umbral, `write()` o registro de falla. Mencionar `setTimer`, `timeNow` y `testStep` si es un caso de prueba.

**4.** "¿Cómo observas variables internas de la ECU sin detenerla?"
- *Puntos clave*: XCP con el archivo A2L correspondiente al build exacto; medición con DAQ en eventos de tiempo; calibración en línea; impacto en la carga del bus.

**5.** "¿Cómo ayudas a un probador a automatizar una prueba de tu función?"
- *Puntos clave*: documentar interfaces (señales, estados, DTC), exponer variables observables por XCP, definir requisitos medibles y tiempos esperados, proveer un caso de ejemplo y una referencia de comportamiento esperado.

---

## B. Test Engineer

Este rol es el núcleo del tema.

**1.** "¿Qué es CANoe y cuáles son sus componentes principales?"
- *Puntos clave*: herramienta de desarrollo, simulación y prueba de redes y ECUs. Configuración, simulation setup, panels, Trace, Graphics, Measurement Setup, test modules, CAPL Browser, test configuration. Simula nodos y *restbus*.

**2.** "¿Qué hardware se usa con CANoe y cómo eliges entre interfaces?"
- *Puntos clave*: familia VN (VN1610/1630/1640/5640, etc.) según canales, CAN FD, LIN, Ethernet; seleccionar por protocolo, número de canales, velocidad, sincronización de tiempo. Instalar driver y configurar en Vector Hardware Config.

**3.** "Explica el rol del VT System y dónde lo usarías."
- *Puntos clave*: módulos para generar y medir señales eléctricas (entradas/salidas digitales y analógicas, PWM, carga, cortocircuito, circuito abierto) para HIL; integrado con CANoe; ideal para fault injection eléctrico.

**4.** "Diferencia entre CANoe y vTESTstudio. ¿Cuándo usas cada una?"
- *Puntos clave*: CANoe ejecuta y simula; vTESTstudio diseña/autoría de casos (CAPL, Python, tablas de prueba, *test patterns*) y genera los módulos que CANoe ejecuta. Diseñar en vTESTstudio, ejecutar y reportar en CANoe.

**5.** "Escribe un caso de prueba en CAPL para verificar que, tras enviar un mensaje de solicitud, la ECU responde en menos de 100 ms."
- *Puntos clave*: `testcase`, `testStep`, enviar `output(msg)`, `testWaitForMessage(id, 100)`, verificar resultado con `testStepPass`/`testStepFail`, evaluar señal con `getValue`. Considerar timeouts y limpieza al final.

**6.** "¿Qué es un restbus y por qué se simula?"
- *Puntos clave*: emulación de los demás nodos de la red para que la ECU bajo prueba vea tráfico realista; se genera desde el DBC con CANoe (Interaction Layer). Incluye mensajes cíclicos, alive counters y checksums (E2E).

**7.** "¿Cómo pruebas el manejo de errores del bus (por ejemplo, error frames o bus off)?"
- *Puntos clave*: usar interfaces con capacidad de inyección o el módulo de error de CANoe, validar contador de errores, bus off, recuperación, DTC y comportamiento degradado. Seguir con evidencia de trazas.

**8.** "Describe cómo se estructura un proyecto con Robot Framework."
- *Puntos clave*: archivos `.robot` con secciones Settings, Variables, Test Cases, Keywords; librerías (Python propias, BuiltIn, Collections); recursos compartidos; *setup/teardown*; etiquetas para selección; reportes `log.html` y `report.html`.

**9.** "¿Cómo integras Robot Framework con CANoe?"
- *Puntos clave*: librería en Python con la interfaz COM de CANoe (win32com): abrir configuración, iniciar medición, leer/escribir variables de entorno y de sistema, ejecutar test modules, recoger resultados. Robot orquesta, CANoe ejecuta.

**10.** "Una prueba automatizada falla de forma intermitente. ¿Cómo investigas?"
- *Puntos clave*: separar fallas de infraestructura, prueba y software; revisar esperas fijas, estado inicial, versiones de DBC/A2L, carga del bus, orden entre pruebas; reproducir en bucle con trazas; no relanzar sin analizar.

**11.** "¿Qué buenas prácticas sigues al escribir keywords reutilizables en Robot Framework?"
- *Puntos clave*: nivel de abstracción (keywords de negocio sobre keywords técnicas), nombres legibles, argumentos con valores por defecto, documentación, sin lógica compleja en el `.robot` (va a Python), manejo de errores y logs útiles.

---

## C. Embedded Software Test Function Lead

Aquí se evalúa arquitectura de automatización, estandarización y escalado.

**1.** "¿Cómo defines la arquitectura de un framework de automatización con CANoe, vTESTstudio y Robot Framework?"
- *Puntos clave*: capas: casos legibles (Robot) → keywords → librerías Python/CAPL → CANoe/hardware. Separar datos, configuración y lógica; control de versiones de todo; reportes unificados; ejecución por CI.

**2.** "¿Qué estándares pones para que varios ingenieros escriban pruebas consistentes?"
- *Puntos clave*: guía de estilo, plantillas, convenciones de nombres, revisión de código de pruebas, librería común, trazabilidad al requisito, ejemplos de referencia, métricas de calidad de la suite.

**3.** "¿Cómo versionas DBC, A2L, configuraciones de CANoe y scripts de manera coherente?"
- *Puntos clave*: repositorio (Git/Bitbucket) con etiquetas por release, dependencias fijadas, artefactos del build junto con el DBC/A2L correspondientes, pipeline que verifica coherencia antes de ejecutar.

**4.** "Los scripts de prueba se vuelven costosos de mantener. ¿Qué haces?"
- *Puntos clave*: refactor hacia keywords y librerías, eliminar duplicación, datos parametrizados, pruebas de la propia librería, retirar pruebas obsoletas, medir costo de mantenimiento antes de automatizar más.

**5.** "¿Cómo integras la ejecución en Jenkins con bancos físicos?"
- *Puntos clave*: agentes dedicados por banco con etiquetas, reserva de recurso (lock), verificación de salud previa, ejecución de Robot, publicación de reportes, almacenamiento de logs/trazas, notificaciones; Docker para entornos no dependientes de hardware.

**6.** "¿Qué automatizarías primero y qué dejarías manual?"
- *Puntos clave*: automatizar lo repetitivo, estable, de alto riesgo y regresión; mantener manual lo exploratorio, lo que cambia con frecuencia o tiene bajo retorno. Calcular el retorno antes de decidir.

**7.** "¿Cómo aseguras la calidad de tus propias herramientas de prueba?"
- *Puntos clave*: probar librerías y keywords, validar con casos conocidos (pasa/falla esperado), revisión, versionado, calificación de herramientas cuando el estándar lo exige (ISO 26262 Tool Confidence Level).

---

## D. Test Manager

Aquí se evalúa inversión, licencias y retorno.

**1.** "¿Cómo justificas el costo de licencias de CANoe y de hardware Vector frente a alternativas?"
- *Puntos clave*: productividad, estándar de la industria, soporte, integración con ECUs y OEMs, menor riesgo; comparar con opciones abiertas (python-can, SocketCAN) por costo total, mantenimiento y requisitos del cliente.

**2.** "¿Qué retorno esperas de la automatización y cómo lo mides?"
- *Puntos clave*: tiempo ahorrado en regresión, ciclos por día, defectos detectados pronto, reducción de errores manuales; restar costo de desarrollo y mantenimiento. Medir antes y después con datos.

**3.** "Un cliente exige una herramienta específica que tu equipo no domina. ¿Cómo lo gestionas?"
- *Puntos clave*: evaluar brecha de competencia, plan de formación o apoyo externo, piloto pequeño, riesgo en cronograma, comunicar costo y plazo al cliente.

**4.** "¿Cómo planificas la capacidad de licencias y bancos entre equipos y sitios?"
- *Puntos clave*: inventario y utilización, licencias flotantes, reservas, horario nocturno automatizado, métricas de cola, acceso remoto, priorización por hitos.

**5.** "¿Cómo reduces la dependencia de personas clave en la automatización?"
- *Puntos clave*: documentación, estándares, revisión por pares, rotación, formación cruzada, repositorio único y onboarding con ejemplos.

---

## Preguntas transversales (cierre o "curveball")

**1.** "¿Qué limitaciones tiene CANoe que debemos considerar?"
- *Puntos clave*: licencias y costo, dependencia de Windows, tiempos de CAPL no estrictamente de tiempo real para todo, necesidad de hardware para sincronía precisa, curva de aprendizaje; se mitigan con hardware adecuado y buenas prácticas.

**2.** "Si te quitan Robot Framework, ¿cómo seguirías automatizando?"
- *Puntos clave*: pytest con python-can y COM de CANoe, CAPL/vTESTstudio con ejecución desde CI. La arquitectura en capas hace reemplazable la capa de orquestación.

**3.** "Cuéntame de una automatización tuya que redujo tiempo o errores."
- *Puntos clave*: ejemplo STAR con números (antes/después), herramientas usadas, tu rol y aprendizaje. Se profundiza en el Módulo 3.
