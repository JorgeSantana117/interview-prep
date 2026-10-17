# Capítulo: HIL Testing (Hardware-in-the-Loop)

> Objetivo del capítulo: al terminar podrás (1) distinguir MIL/SIL/PIL/HIL/Vehicle Testing por **qué tipo de defecto detecta cada uno**, no solo por cuánto hardware usan; (2) describir los componentes de un banco HIL y para qué sirve cada uno, incluida la debug board y el modelo de planta; (3) explicar por qué el simulador debe ser determinista en tiempo real y no solo "rápido"; y (4) dar ejemplos concretos de fault injection y del ciclo de vida completo de un caso de prueba HIL. Es el tema donde ya tienes más experiencia de primera mano (Ford, VW/ALTEN, Bosch) — el reto no es aprenderlo, sino **articularlo con precisión de arquitectura**, que es justo lo que separa una respuesta de **TE** de una de **TFL**.

---

## 1. Qué es HIL Testing

**Hardware-in-the-Loop (HIL)** es una técnica de validación en la que una **ECU real** (el hardware bajo prueba, también llamado DUT — Device Under Test) se conecta a un **simulador en tiempo real** que emula el resto del vehículo o sistema con el que esa ECU interactuaría normalmente: sensores, actuadores, otras ECUs, y el comportamiento físico del vehículo (motor, dinámica de manejo, temperatura, etc.).

La idea central: en vez de probar la ECU dentro de un vehículo físico completo (caro, lento de configurar, difícil de reproducir escenarios peligrosos o poco comunes), **se simula todo el entorno alrededor de la ECU**, mientras el hardware y software real de la ECU corren sin modificaciones — exactamente como se comportarían en el vehículo final. Esto te permite validar el comportamiento real del software/hardware bajo condiciones controladas, repetibles, y sin poner en riesgo un vehículo ni a personas.

## 2. Por qué se usa — HIL dentro del V-Model

HIL ocupa un lugar específico en el ciclo de desarrollo automotriz (V-Model), y es importante que sepas ubicarlo con precisión frente a otras técnicas de prueba. Vale la pena profundizar en cada nivel, porque la pregunta "explícame la diferencia entre MIL, SIL y HIL" es una de las más comunes en entrevistas de este dominio, y una respuesta superficial ("uno usa más hardware que otro") no es suficiente a nivel Test Engineer/Lead.

### 2.1 MIL (Model-in-the-Loop)

- **Qué se prueba**: el **modelo de control** en sí mismo (ej. un modelo de Simulink/Stateflow que implementa la lógica de un controlador PID, una máquina de estados de gestión de energía, un algoritmo de fusión de sensores) — **antes de que exista una sola línea de código C generado**.
- **Entorno**: todo corre dentro del entorno de modelado (Simulink/MATLAB), incluyendo tanto el modelo del controlador como el modelo de la planta (el "mundo" que ese controlador va a controlar) — ambos son modelos, ninguno es código real todavía.

> **¿Qué es la "planta" (plant) exactamente?** Es un término que viene de la teoría de control clásica, y se refiere al **sistema físico que está siendo controlado** — no al software que lo controla, sino a lo que ese software recibe como entrada y sobre lo que actúa. En términos simples: el **controlador** es la lógica que decide qué hacer; la **planta** es todo lo demás — el mundo físico real que reacciona a esas decisiones.
>
> **Ejemplo automotriz concreto — Cruise Control**: imagina que estás validando el algoritmo de Cruise Control de un vehículo.
> - El **controlador** es el algoritmo que decide cuánto abrir el acelerador o aplicar freno para mantener la velocidad deseada.
> - La **planta** es el modelo que representa **todo lo que el controlador no controla directamente pero que determina el resultado real**: la masa del vehículo, la resistencia aerodinámica, la fricción de las llantas contra el pavimento, la pendiente del camino, la respuesta del motor a una señal de aceleración (cuánto tiempo tarda realmente en generar más torque), e incluso el comportamiento del sensor de velocidad que le da retroalimentación al controlador.
> - En una simulación MIL, **ambos son modelos matemáticos**: el modelo del controlador (tu lógica de Cruise Control en Simulink) se conecta al modelo de la planta (ecuaciones de dinámica longitudinal del vehículo, también en Simulink), formando un **lazo cerrado** — el controlador manda una señal de aceleración, la planta calcula cómo reaccionaría el vehículo real a esa señal (nueva velocidad resultante), y esa nueva velocidad regresa como entrada al controlador, cerrando el ciclo — exactamente como pasaría en el vehículo real, pero completamente en simulación.
>
> **Otro ejemplo — ABS (Antilock Braking System)**: el controlador decide cuándo liberar/aplicar presión de freno en cada rueda; la planta modela la dinámica de la rueda (velocidad angular, deslizamiento contra el pavimento, coeficiente de fricción del neumático-carretera bajo distintas condiciones simuladas como lluvia o hielo) — es la planta la que le "dice" al controlador, en cada ciclo de simulación, qué tan cerca está la rueda de bloquearse, en función de las decisiones que el controlador tomó en el ciclo anterior.
>
> Este mismo concepto de "planta" es el que después reaparece en HIL (sección 4) — la diferencia es que en HIL el **controlador ya no es un modelo, es la ECU real**, pero la planta sigue siendo un modelo simulado que le da a esa ECU real las mismas señales que recibiría del mundo físico.

- **Objetivo principal**: validar que la **lógica de control sea correcta a nivel algorítmico/matemático** — ¿el controlador hace lo que el ingeniero de control diseñó que hiciera? Aquí se detectan errores de diseño del algoritmo, no errores de implementación de software.
- **Quién lo hace típicamente**: ingenieros de control/algoritmos, en las fases más tempranas del desarrollo — muchas veces antes de que el equipo de software embebido siquiera participe.
- **Ejemplo concreto**: validar en simulación pura que un algoritmo de ACC (Adaptive Cruise Control) mantiene la distancia de seguimiento correcta ante distintos perfiles de velocidad del vehículo delantero, sin que exista todavía ningún microcontrolador ni código C involucrado.
- **Ventaja clave**: iteración extremadamente rápida — cambiar un parámetro del modelo y volver a simular toma segundos, no hay que compilar ni flashear nada.
- **Limitación clave**: no valida absolutamente nada relacionado con cómo se comportará ese algoritmo una vez convertido a código real corriendo en un microcontrolador con recursos limitados (memoria, precisión numérica, tiempos de ejecución reales).

### 2.2 SIL (Software-in-the-Loop)

- **Qué se prueba**: el modelo se convierte a **código fuente** (típicamente C, generado automáticamente desde Simulink mediante herramientas de generación de código, o escrito manualmente) y ese código se **compila para correr en una PC** (no en el microcontrolador objetivo) — normalmente como una DLL o ejecutable que se conecta de vuelta al modelo de planta (que sigue corriendo en Simulink) o a un harness de prueba en Python/C.
- **Entorno**: el software ya es "real" (código C compilado), pero **el hardware sigue siendo la PC del ingeniero**, no el microcontrolador de la ECU. El modelo de planta puede seguir siendo el mismo de la fase MIL.
- **Objetivo principal**: validar que la **generación de código fue correcta** (¿el código C se comporta igual que el modelo original? — esto se llama *Back-to-Back Testing*, comparando resultados MIL vs SIL para el mismo escenario) y empezar a probar el software con herramientas de prueba de software convencionales (cobertura de código, análisis estático, debugging con breakpoints reales).
- **Quién lo hace típicamente**: empieza a involucrarse el equipo de software embebido, no solo control/algoritmos.
- **Ventaja clave**: sigue siendo rápido de ejecutar (corre en PC) y ahora sí permite usar herramientas de análisis de software real (cobertura de código MC/DC para ASPICE/ISO 26262, por ejemplo) que no aplican a un modelo Simulink.
- **Limitación clave**: la PC tiene arquitectura de procesador, precisión numérica (float vs fixed-point), velocidad y recursos de memoria **completamente distintos** al microcontrolador automotriz real — un bug relacionado con overflow de un tipo de dato de 8 bits, o un problema de timing real, simplemente no se puede detectar en SIL.

### 2.3 PIL (Processor-in-the-Loop) — el nivel intermedio que muchas veces se omite

Vale la pena mencionarlo aunque no esté en el nombre original de tu lista, porque mostrar que conoces este nivel intermedio demuestra profundidad real:
- El código C ya se compila **para el procesador objetivo real** (el mismo microcontrolador que usará la ECU) y corre en una placa de evaluación de ese procesador — pero **todavía sin el resto del hardware completo de la ECU** (sin el arnés completo, sin todos los periféricos de producción).
- Objetivo: detectar problemas específicos del procesador objetivo (precisión numérica real, tiempos de ejecución reales en ese hardware específico, uso real de memoria) **antes** de tener la ECU de producción completa disponible.
- Es un paso intermedio entre SIL y HIL que algunas organizaciones usan formalmente y otras omiten, integrando directamente esas validaciones dentro de HIL.

### 2.4 HIL (Hardware-in-the-Loop)

- **Qué se prueba**: el software corre **ya sobre el hardware real y completo de la ECU de producción** (el microcontrolador definitivo, los drivers reales, el bootloader real, los transceptores de comunicación reales), conectado a un simulador en tiempo real que emula el entorno (sensores, actuadores, otras ECUs, dinámica del vehículo).
- **Entorno**: es el primer nivel de toda la cadena MIL→SIL→HIL donde el **hardware real de la ECU** entra en juego — de ahí el nombre. Todo lo demás (el entorno alrededor de la ECU) sigue siendo simulado.
- **Objetivo principal**: validar el **sistema completo hardware + software** exactamente como se comportará en producción, incluyendo aspectos que ningún nivel anterior puede detectar: comportamiento real de comunicación en los buses físicos (CAN/LIN/Ethernet reales), timing real de interrupciones y tareas del RTOS, comportamiento térmico/eléctrico real del hardware, y la interacción real entre múltiples ECUs físicas si el banco es de tipo Subsystem/Full-Vehicle.
- **Quién lo hace típicamente**: el equipo de Test Engineering (tu rol) — este es exactamente el nivel donde vive la mayoría de tu experiencia profesional.
- **Ventaja clave**: es el nivel de mayor fidelidad antes de llegar a un vehículo físico completo — detecta la gran mayoría de defectos de integración real hardware/software.
- **Limitación clave**: como ya vimos en la sección 9 más adelante, la fidelidad del **modelo de planta** sigue siendo finita, y ciertos fenómenos puramente físicos (vibración mecánica real, interferencia electromagnética real del entorno del vehículo) no se pueden capturar completamente incluso con hardware real de la ECU.

### 2.5 Por qué cada nivel existe (no es solo "hacer lo mismo con más fidelidad")

Un error común es pensar que MIL, SIL y HIL prueban "lo mismo pero con distinto hardware" — en realidad, **cada nivel está diseñado para detectar tipos de defectos diferentes**, y por eso ninguno reemplaza completamente a los demás:

| Nivel | Detecta principalmente | NO puede detectar |
|---|---|---|
| **MIL** | Errores de lógica/diseño del algoritmo de control | Errores de generación de código, timing real, limitaciones de hardware |
| **SIL** | Errores de generación de código, bugs de software (lógica, cobertura) | Timing real, precisión numérica del hardware objetivo, comportamiento eléctrico |
| **PIL** | Problemas de precisión numérica y timing en el procesador real | Comportamiento del hardware periférico completo de la ECU, integración con otras ECUs |
| **HIL** | Integración hardware+software completa, timing real, comunicación real, fault management | Fenómenos puramente físicos del mundo real (vibración, EMI real, desgaste) |
| **Vehicle Testing** | Todo lo anterior en condiciones 100% reales | — (es el nivel de referencia final) |

### 2.6 Costo y velocidad de iteración a lo largo de la cadena

Otro eje que vale la pena poder explicar: conforme avanzas de MIL a Vehicle Testing, **el costo por ciclo de prueba sube y la velocidad de iteración baja**, de forma no lineal:
- MIL: cambios y re-pruebas en segundos/minutos.
- SIL: minutos (requiere compilar).
- PIL/HIL: minutos a horas (requiere flashear hardware real, configurar el banco).
- Vehicle Testing: horas a días (requiere disponibilidad de vehículo, condiciones de prueba, personal especializado, a veces condiciones climáticas específicas).

Esta es la razón estratégica real por la que el V-Model prioriza encontrar la mayor cantidad de defectos posible en los niveles más tempranos (MIL/SIL) — es exponencialmente más barato corregir un error de lógica de control en MIL que descubrir ese mismo error hasta la fase de Vehicle Testing.

**Por qué HIL es tan valioso en la práctica** (esto es lo que debes poder defender en entrevista):
- Permite probar **escenarios peligrosos o de baja probabilidad** (ej. falla de un sensor de frenos a alta velocidad) sin ningún riesgo real.
- Es **reproducible**: puedes ejecutar exactamente el mismo escenario cientos de veces, algo casi imposible en un vehículo real donde las condiciones ambientales varían.
- Permite empezar a validar el software **mucho antes** de que exista un vehículo prototipo completo — puedes tener un banco HIL funcionando con solo la ECU real y el resto simulado.
- Habilita la **automatización de pruebas** (conectando con tu experiencia en Robot Framework, CANoe, vTESTstudio) de una forma que sería inviable en un vehículo físico.
- Es significativamente **más económico** por ciclo de prueba que usar prototipos de vehículo completos.

## 3. Tipos de HIL

Existen distintas clasificaciones según qué tanto del sistema real está presente. Es común que te pregunten esta clasificación específicamente:

### Por nivel de fidelidad de la simulación
- **Signal-level HIL**: el simulador genera/recibe directamente las señales eléctricas que la ECU esperaría de sus sensores/actuadores (ej. un voltaje analógico simulando la señal de un sensor de temperatura). Es el nivel más común y el que probablemente usas más seguido.
- **Mechatronic HIL (Power HIL)**: además de señales eléctricas de bajo nivel, se simulan también las cargas de potencia reales (ej. un motor eléctrico real conectado al sistema para simular carga mecánica) — usado cuando el comportamiento eléctrico/mecánico de un actuador real es relevante para la prueba, no solo su señal de control.

### Por alcance del sistema bajo prueba
- **Component HIL**: se prueba una sola ECU de forma aislada, con todo su entorno simulado.
- **Subsystem HIL**: se prueban varias ECUs relacionadas en conjunto (ej. todo el subsistema de frenos: ABS + ESC), con las ECUs fuera de ese subsistema simuladas.
- **Full-Vehicle HIL (Vehicle-level HIL)**: se conecta una red completa de ECUs reales (o la mayoría de ellas) en un banco que simula el vehículo completo — el nivel más cercano a un vehículo real sin serlo físicamente.

*Nota directamente ligada a tu CV*: tu experiencia describe HIL a nivel de ECUs individuales de infotainment/ADAS (Component/Subsystem HIL) — si te preguntan por Full-Vehicle HIL y no tienes experiencia directa, es honesto decir que tu experiencia es más fuerte a nivel de componente/subsistema, y explicar que entiendes conceptualmente cómo escala hacia el nivel de vehículo completo.

## 4. Componentes de un banco HIL

Un banco HIL típico (el que armas y mantienes, según tu CV en ALTEN) se compone de:

- **ECU bajo prueba (DUT)**: el hardware real, con su software real cargado, sin modificaciones.
- **Simulador en tiempo real (Real-Time Simulator/Target)**: el corazón del banco — una computadora especializada que ejecuta el modelo de planta (plant model) con garantías de determinismo (debe completar cada ciclo de cómputo dentro de un tiempo fijo, sin excepciones, o la simulación deja de representar la realidad correctamente).
- **Modelo de planta (Plant Model)**: el modelo matemático/lógico (típicamente desarrollado en Simulink/Stateflow, conectando con el capítulo de Diseño Basado en Modelos) que representa el comportamiento físico del entorno: dinámica del vehículo, comportamiento térmico, respuesta de sensores/actuadores.
- **Interfaces de I/O**: hardware que convierte las señales digitales del simulador en señales eléctricas reales que la ECU puede leer (y viceversa) — conversores digital-analógico, generadores de PWM, simuladores de resistencia (para sensores tipo NTC), etc.
- **Load Boxes / Breakout Box**: hardware intermedio que permite, entre otras cosas, **inyectar fallas físicas** de forma controlada (ver sección 6) y acceder físicamente a las señales para medición/depuración sin desconectar cables.
- **Debug Board (tarjeta de depuración)**: hardware adicional, normalmente conectado directamente a los puertos de depuración del microcontrolador de la ECU (ej. JTAG/SWD), que permite a los ingenieros de software hacer **debugging a nivel de código** mientras la ECU está corriendo en el banco HIL — poner breakpoints, inspeccionar variables internas en memoria, hacer step-through del código en tiempo real. Es lo que conecta tu trabajo de "debugging a nivel de código" y "source-level debugging" (tal como lo describes en tu experiencia en Bosch) con el entorno HIL: sin una debug board, solo puedes observar el comportamiento externo de la ECU (señales, mensajes de bus); con ella, puedes ver exactamente qué está pasando dentro del software mientras el escenario simulado se ejecuta. También suele integrarse con herramientas de medición/calibración como XCP (del capítulo de Protocolos de Comunicación) para observar variables internas sin necesariamente detener la ejecución.
- **Interfaces de bus vehicular**: hardware para CAN/CAN FD/LIN/Ethernet (ej. Vector VN interfaces si trabajas con CANoe) que conecta el banco a los buses de comunicación reales de la ECU.
- **PC de control / Herramienta de automatización**: donde corre CANoe, vTESTstudio, o tus scripts de Robot Framework, orquestando la ejecución de las pruebas, el control del simulador, y el registro de resultados.

## 5. Simulación de planta y determinismo en tiempo real

Un punto que distingue una respuesta de nivel medio de una de nivel senior en entrevista: **por qué el simulador debe ser "tiempo real" y no solo "rápido"**.

- El objetivo no es que el modelo corra lo más rápido posible, sino que corra **exactamente al mismo ritmo que el tiempo real del mundo físico** (1 segundo de simulación = 1 segundo real) — porque la ECU real está operando en tiempo real y espera que las señales del entorno cambien de forma consistente con la física real, no más rápido ni más lento.
- Si el simulador no cumple su ciclo de cómputo a tiempo (un "overrun"), la simulación pierde sincronía con el tiempo real y los resultados dejan de ser válidos — por eso los sistemas HIL usan hardware/sistemas operativos especializados con garantías de determinismo (conectando con lo que ya sabes de RTOS vs sistemas operativos de propósito general del capítulo de Sistemas Operativos Embebidos).

## 6. Inyección de fallas (Fault Injection)

Una de las capacidades más valiosas de HIL — y una de las razones por las que HIL existe en primer lugar — es poder **simular fallas físicas de forma controlada y segura**, algo extremadamente difícil o peligroso de hacer en un vehículo real:

- **Circuito abierto (Open Circuit)**: simula un cable roto o un conector desconectado.
- **Corto a batería (Short to Battery / Short to VBatt)**: simula que una señal queda cortocircuitada a la línea de alimentación del vehículo.
- **Corto a tierra (Short to Ground)**: simula que una señal queda cortocircuitada a tierra.
- **Corto entre señales (Short to adjacent pin/signal)**: simula que dos señales adyacentes en un conector quedan en corto entre sí.
- **Degradación gradual**: simula una falla progresiva (ej. una resistencia de contacto que aumenta con el tiempo), no solo fallas binarias de todo/nada.

Esto conecta directamente con tu experiencia de "fault management" y "robustness" en las pruebas de sistema que describes en tu CV — es útil poder dar un ejemplo concreto de qué tipo de fallas has inyectado y qué comportamiento esperado validabas (ej. que se genere el DTC correcto, que el sistema entre en modo seguro/degradado).

## 7. Tipos de pruebas que se ejecutan en un banco HIL

Conectando con tu experiencia descrita en el CV (flashing, comunicación, diagnóstico, fault management, state management, robustez, runtime performance):

- **Pruebas funcionales**: validar que la ECU haga lo que debe hacer bajo condiciones normales (ej. que una función de ACC calcule correctamente la distancia de seguimiento).
- **Pruebas de comunicación**: validar los protocolos que ya dominas — timing y contenido correcto de mensajes CAN/CAN FD, comportamiento de UDS (sesiones, flashing, DTCs), cumplimiento de parámetros CAN-TP.
- **Pruebas de manejo de fallas (fault management)**: usando fault injection, validar que el sistema detecte, reporte (DTC) y reaccione correctamente ante condiciones anómalas.
- **Pruebas de gestión de estados (state management)**: validar transiciones correctas entre estados del sistema (ej. apagado → arranque → operación normal → modo degradado → apagado seguro).
- **Pruebas de robustez**: enviar entradas inesperadas, fuera de rango, o mal formadas, y validar que el sistema no falle de forma insegura.
- **Pruebas de desempeño en tiempo de ejecución (runtime performance)**: medir tiempos de respuesta, uso de CPU/memoria bajo carga, latencias de comunicación bajo distintas condiciones de estrés del bus.
- **Pruebas de regresión**: re-ejecutar el set completo (o un subconjunto priorizado) de pruebas automatizadas cada vez que hay un cambio de software, para asegurar que no se rompió nada que ya funcionaba — este es probablemente el tipo de ejecución que más automatizas con Robot Framework/CANoe en tu día a día.

## 8. Ciclo de vida de un caso de prueba HIL

Un flujo típico, útil para explicar en entrevista cómo trabajas de principio a fin:

1. **Análisis de requisitos**: identificar qué comportamiento específico se debe validar (conectando con tu experiencia en Requirements Engineering).
2. **Diseño del caso de prueba**: definir precondiciones, pasos, estímulos a inyectar (señales, mensajes, fallas) y resultados esperados — especificado en herramientas como vTESTstudio o documentado en DOORS/RQM.
3. **Implementación/automatización**: traducir el diseño a un script ejecutable (CAPL en CANoe, keywords en Robot Framework, Python).
4. **Ejecución**: correr el caso de prueba en el banco HIL, ya sea manualmente o como parte de una suite automatizada (potencialmente disparada por Jenkins como parte de tu experiencia en CI/CD).
5. **Análisis de resultados**: comparar el resultado real contra el esperado; si falla, hacer un primer análisis para determinar si es un defecto de la ECU o un problema del propio test/banco.
6. **Reporte de defectos**: documentar en Jira/RQM con evidencia (logs, traces) suficiente para que el equipo de desarrollo pueda reproducir el problema.
7. **Regresión continua**: el caso queda incorporado a la suite de regresión para validar automáticamente en cada nuevo build.

## 9. Ventajas y limitaciones de HIL

Ser capaz de discutir las **limitaciones** con honestidad es señal de madurez técnica en entrevista — nadie espera que digas que HIL resuelve todo.

**Ventajas** (resumen de lo ya cubierto): seguridad, repetibilidad, disponibilidad temprana, automatización, costo por ciclo.

**Limitaciones**:
- La **fidelidad del modelo de planta es finita** — el modelo simulado nunca captura el 100% de la complejidad física real; siempre existe la posibilidad de que un comportamiento solo se manifieste con hardware/vehículo real (ej. vibración mecánica real, interferencia electromagnética real de un entorno no simulado).
- No sustituye completamente las pruebas en vehículo — HIL es un **complemento** dentro del V-Model, no un reemplazo total; los defectos relacionados con integración física real, ergonomía, o condiciones ambientales genuinas (temperatura extrema real, vibración de carretera real) requieren validación en vehículo.
- Requiere **inversión inicial significativa** en la construcción y mantenimiento del banco (hardware de simulación, modelos de planta bien validados), y ese modelo debe mantenerse actualizado conforme cambia el diseño real del vehículo — un modelo de planta desactualizado puede dar una falsa sensación de confianza.

## 10. Comparación SIL vs HIL vs Vehicle Testing

| Aspecto | SIL | HIL | Vehicle Testing |
|---|---|---|---|
| Hardware real de la ECU | No | Sí | Sí |
| Entorno | 100% simulado | Simulado (con ECU real) | Real |
| Costo por ciclo | Muy bajo | Medio | Muy alto |
| Velocidad de iteración | Muy rápida | Rápida-Media | Lenta |
| Riesgo físico | Ninguno | Ninguno | Existe |
| Reproducibilidad de escenarios | Total | Total | Limitada (condiciones reales varían) |
| Detecta problemas de integración física real | No | Parcialmente | Sí |
| Momento típico en el V-Model | Desarrollo temprano | Desarrollo/Integración | Validación final |

## Preguntas de teoría (para autoevaluarte)
1. ¿Cuál es la diferencia fundamental entre MIL, SIL, HIL y Vehicle Testing?
2. ¿Qué tipo de defectos detecta MIL que SIL no puede detectar, y viceversa?
3. ¿Qué es Back-to-Back Testing entre MIL y SIL, y para qué sirve específicamente?
4. ¿Qué es PIL (Processor-in-the-Loop) y qué detecta que ni SIL ni HIL detectan por sí solos?
5. ¿Por qué el costo y tiempo de iteración aumentan de forma no lineal conforme avanzas de MIL a Vehicle Testing, y por qué esto justifica la estrategia del V-Model de detectar errores lo antes posible?
6. ¿Por qué es importante que el simulador de un banco HIL sea "tiempo real" y no solo "rápido"? ¿Qué pasa si no lo es?
7. Describe la diferencia entre Component HIL, Subsystem HIL y Full-Vehicle HIL.
8. ¿Qué es Signal-level HIL vs Mechatronic/Power HIL, y cuándo usarías cada uno?
9. Enumera al menos 3 tipos de fallas que se pueden inyectar en un banco HIL y qué comportamiento validarías en cada caso.
10. Describe el ciclo de vida completo de un caso de prueba HIL, desde el requisito hasta la regresión continua.
11. ¿Cuáles son las principales limitaciones de HIL Testing, y por qué no reemplaza completamente las pruebas en vehículo?
12. Si un caso de prueba falla en HIL pero el mismo escenario funciona correctamente en un vehículo real, ¿qué hipótesis considerarías?
13. ¿Qué componentes forman parte de un banco HIL típico, y qué función cumple cada uno?
14. ¿Por qué HIL habilita mejor la automatización de pruebas que las pruebas en vehículo real?

## Errores comunes / lo que un entrevistador espera que NO confundas
- Describir HIL como "simplemente probar la ECU sin el auto" sin mencionar el rol crítico del **determinismo en tiempo real** del simulador — es lo que técnicamente distingue HIL de una simulación offline cualquiera.
- Confundir SIL con HIL — en SIL no hay hardware real de la ECU involucrado; es un error común decir "SIL" cuando en realidad la persona se refiere a probar el software ya en el microcontrolador real (eso ya es HIL).
- Presentar HIL como si no tuviera limitaciones — un entrevistador experimentado valora que reconozcas honestamente qué tipo de defectos HIL no puede detectar.
- No distinguir entre los niveles de alcance (Component/Subsystem/Full-Vehicle) al hablar de tu experiencia — ser preciso sobre el nivel exacto en el que trabajaste muestra rigor técnico.
- Olvidar mencionar la fidelidad del modelo de planta como un factor crítico — un banco HIL es tan bueno como el modelo que simula el entorno; esto conecta con tu experiencia previa en Ford desarrollando modelos Simulink/Stateflow para HIL.

## Puntos clave

Repaso rápido antes de una entrevista — un bullet por idea, sin reexplicar.

- **Qué es HIL**: ECU real (DUT) + entorno simulado en tiempo real (sensores, actuadores, planta) — permite validar sin riesgo físico ni necesidad de vehículo completo.
- **Escalera del V-Model**: MIL (lógica de control, sin código) → SIL (código C en PC, Back-to-Back contra MIL) → PIL (código en el procesador real, sin ECU completa) → HIL (HW+SW real completo) → Vehicle Testing (100% real). Cada nivel detecta defectos distintos; ninguno reemplaza a los demás.
- **Costo vs velocidad**: sube el costo y baja la velocidad de iteración de forma no lineal conforme avanzas de MIL a Vehicle Testing — por eso el V-Model prioriza encontrar defectos temprano.
- **Tipos de HIL**: por fidelidad — Signal-level (señales eléctricas) vs Mechatronic/Power HIL (cargas de potencia reales); por alcance — Component / Subsystem / Full-Vehicle HIL.
- **Componentes del banco**: DUT, simulador de tiempo real, modelo de planta, interfaces de I/O, Load/Breakout Box (fault injection), Debug Board (JTAG/SWD, source-level debugging), interfaces de bus vehicular, PC de automatización.
- **Determinismo en tiempo real**: el simulador debe correr al mismo ritmo que el mundo físico (1 s sim = 1 s real), no solo "rápido" — un overrun rompe la validez de la simulación.
- **Fault injection**: open circuit, short to battery, short to ground, short entre señales adyacentes, degradación gradual.
- **Tipos de prueba en HIL**: funcionales, comunicación, fault management, state management, robustez, runtime performance, regresión.
- **Ciclo de vida de un caso de prueba**: requisito → diseño (vTESTstudio/DOORS) → automatización (CAPL/Robot Framework) → ejecución → análisis de resultados → reporte (Jira/RQM) → regresión continua (CI/CD).
- **Limitaciones de HIL**: fidelidad finita del modelo de planta; no sustituye vehículo real; requiere inversión y mantenimiento continuo del modelo conforme cambia el diseño.
