# Capítulo: Herramientas de Testing y Automatización (CANoe, vTESTstudio, Robot Framework)

> Objetivo del capítulo: al igual que HIL, este es un tema de experiencia directa y diaria para ti — las tres herramientas de tu CV. El objetivo no es aprender a usarlas desde cero, sino poder **explicar su arquitectura, propósito y diferencias** con precisión técnica, porque en entrevista es común que te pidan justificar *por qué* usarías una herramienta sobre otra en un escenario dado, no solo que confirmes que sabes usarlas.

---

## 1. CANoe (Vector Informatik)

### 1.1 Qué es
CANoe es la herramienta líder de la industria (desarrollada por Vector Informatik) para **desarrollo, simulación, prueba y análisis** de redes y ECUs de comunicación vehicular. Es una herramienta multipropósito: el mismo entorno se usa para simular una red completa antes de que existan ECUs reales, para hacer análisis de tráfico de bus en tiempo real, y para automatizar la ejecución de pruebas — es prácticamente el "sistema operativo" alrededor del cual se construye un banco HIL orientado a comunicación vehicular.

### 1.2 Arquitectura y componentes principales
- **Bus/Network Simulation**: CANoe puede simular nodos completos de red (ECUs "virtuales") que no existen físicamente todavía, lo que permite validar el comportamiento de una ECU real contra el resto de una red simulada — exactamente el escenario típico de un banco HIL de Component/Subsystem HIL que ya estudiaste.
- **CAPL (CAN Access Programming Language)**: el lenguaje de scripting propio de CANoe (que ya está en tu CV) — se usa para programar el comportamiento de nodos simulados, manejar eventos de bus (mensajes recibidos, timers, cambios de estado), e implementar lógica de prueba personalizada. CAPL está basado en sintaxis similar a C, orientado a eventos (event-driven): defines bloques `on message`, `on timer`, `on key` que se ejecutan automáticamente cuando ocurre el evento correspondiente.
- **Paneles (Panels)**: interfaces gráficas configurables (botones, gauges, indicadores) que permiten a un ingeniero interactuar manualmente con la simulación — por ejemplo, un panel con un botón que simule "abrir puerta" enviando el mensaje CAN correspondiente, útil para pruebas exploratorias o demos.
- **Trace Window**: la ventana de análisis de tráfico de bus en tiempo real — muestra cada trama CAN/CAN FD/LIN/Ethernet con timestamp, ID, datos, y permite filtrar/decodificar usando una base de datos de señales (DBC/ARXML/FIBEX). Es la herramienta que usas constantemente para debugging de comunicación.
- **Diagnostic Console / Test Feature Set**: módulo específico para trabajar con diagnóstico UDS — permite enviar servicios (0x10, 0x22, 0x27, etc.) de forma interactiva o programada, usando un archivo de descripción de diagnóstico (CDD/ODX) que define qué servicios/DIDs soporta la ECU, de forma análoga a como el A2L describe variables para XCP.
- **Test Module / Test Units**: contenedores de casos de prueba automatizados, escritos en CAPL o (en versiones más recientes) en .NET (C#) — aquí es donde vive la automatización real, no solo la simulación interactiva.
- **Simulation Setup**: la configuración que define qué nodos (reales y simulados), qué bases de datos de comunicación (DBC/ARXML) y qué interfaces de hardware (Vector VN devices) forman parte del escenario de prueba actual.

### 1.3 Hardware más común usado con CANoe

CANoe por sí mismo es software — necesita **interfaces de hardware Vector (familia VN)** para conectarse físicamente a los buses reales. Es una pregunta común en entrevista que sepas nombrar y diferenciar el hardware típico, no solo el software:

**Interfaces para CAN / CAN FD:**
- **VN1610 / VN1611**: interfaces compactas de 1-2 canales CAN/CAN FD por USB — típicas para trabajo de escritorio, debugging rápido o bancos pequeños.
- **VN1630 / VN1640**: interfaces de gama media, varios canales CAN/CAN FD (y LIN en algunos modelos), usadas comúnmente en bancos HIL de tamaño moderado.
- **VN8900 series (ej. VN8912, VN8970)**: dispositivos "standalone" más potentes, con más canales simultáneos (CAN, CAN FD, LIN, y a veces Ethernet en el mismo chasis), pensados para bancos HIL de mayor escala donde necesitas sincronizar muchos buses a la vez — puedes conectar varias ECUs y redes simultáneamente desde un solo dispositivo.

**Interfaces para Automotive Ethernet:**
- **VN5610 / VN5610A**: interfaces para Automotive Ethernet (100BASE-T1/1000BASE-T1) — te permiten capturar y generar tráfico Ethernet automotriz real, incluyendo soporte para SOME/IP y DoIP, conectando directamente con lo que ya viste en el capítulo de Protocolos de Comunicación.
- **VN7570 / VN7600**: interfaces más avanzadas, multi-puerto, que combinan Ethernet automotriz con capacidades de switching/enrutamiento — útiles cuando necesitas simular no solo un nodo sino parte de la topología de switches de una red Ethernet automotriz completa.
- **VT System con módulos de red** (ver 1.4): para escenarios de HIL a gran escala donde Ethernet automotriz es solo una parte de un banco con muchas más señales.

**Nota práctica para entrevista**: la elección del hardware específico depende del **número de canales simultáneos** que necesitas, si requieres **sincronización temporal precisa** entre buses (relevante si recuerdas TSN del capítulo de Ethernet), y si el escenario es de banco de escritorio (interfaces USB simples) o un banco HIL de producción (dispositivos standalone con más robustez y canales).

### 1.4 VT System — simulación de señales eléctricas reales

El **VT System** de Vector es un sistema de hardware **modular** diseñado específicamente para la estimulación y medición de señales eléctricas en bancos HIL — es la pieza de hardware que conecta directamente con lo que estudiaste en el capítulo de HIL Testing sobre **Signal-level HIL** e **interfaces de I/O**.

- **Arquitectura modular en chasis**: el VT System se compone de un chasis (rack) donde se insertan distintos **módulos VT** intercambiables, según qué tipo de señal necesites simular o medir. Esto permite configurar un banco HIL a la medida exacta de los pines/señales de la ECU bajo prueba, y escalarlo simplemente agregando o quitando módulos.
- **Módulos comunes** (ejemplos representativos, no una lista exhaustiva):
  - **VT2004 / VT2816**: módulos de I/O analógico/digital — generan voltajes analógicos (simulando, por ejemplo, la señal de un sensor de temperatura tipo NTC) y leen/generan señales digitales.
  - **VT6104 / VT6204**: módulos de carga electrónica (electronic load) — simulan cargas reales (ej. un motor o un actuador) para pruebas donde el comportamiento eléctrico de la carga importa, conectando con el concepto de **Mechatronic/Power HIL** del capítulo de HIL.
  - **VT7001 / VT7900**: módulos de fuente de alimentación programable — permiten simular condiciones de batería del vehículo (ej. bajo voltaje, sobrevoltaje, rizado) para pruebas de robustez eléctrica.
  - **VT2848 / módulos de relés**: usados específicamente para **fault injection física** — permiten conmutar conexiones para simular circuito abierto, corto a batería, corto a tierra, exactamente los escenarios que estudiaste en la sección de inyección de fallas de HIL Testing.
- **Software de configuración (VT System Designer / integración con CANoe)**: el VT System no se controla de forma aislada — se integra directamente con **CANoe** (o con vTESTstudio a través de CANoe), de modo que un caso de prueba puede tanto enviar un mensaje CAN como, en el mismo script, ordenarle al VT System que simule un circuito abierto en un pin específico, todo de forma sincronizada.

**Por qué esto es relevante para tu discurso en entrevista**: cuando en tu CV mencionas "HIL platforms" o "fault management", el VT System es probablemente el hardware físico detrás de esa inyección de fallas — poder nombrarlo específicamente (en vez de decir genéricamente "el banco HIL") es una señal concreta de profundidad técnica real, no solo de conocimiento de libro.

### 1.5 CAPL en profundidad

Ya mencionamos que CAPL es el lenguaje de scripting propio de CANoe, con sintaxis similar a C y orientado a eventos. Vale la pena profundizar porque "sé CAPL" es una afirmación muy amplia en tu CV, y un entrevistador técnico puede pedirte que expliques o incluso escribas fragmentos reales.

**Estructura básica de un programa CAPL**
Un archivo CAPL (.can) se organiza en bloques de eventos (event procedures) — no se ejecuta de forma secuencial de arriba a abajo como un script tradicional, sino que cada bloque se "despierta" cuando ocurre su evento correspondiente:

```c
variables
{
  msTimer myTimer;
  int counter = 0;
}

on start
{
  write("Simulación iniciada");
  setTimer(myTimer, 1000); // dispara en 1000 ms
}

on message EngineStatus
{
  if (this.RPM > 3000)
  {
    write("RPM alto detectado: %d", this.RPM);
  }
}

on timer myTimer
{
  counter++;
  output(EngineStatus); // reenvía el mensaje al bus
  setTimer(myTimer, 1000); // se reprograma para repetirse
}
```

**Tipos de bloques de eventos más comunes**
- `on start` / `on preStart` / `on stopMeasurement`: se ejecutan al iniciar/detener la medición/simulación — útiles para inicializar variables o cerrar recursos.
- `on message <NombreMensaje>` (o `on message *` para cualquier mensaje): se ejecuta cada vez que llega ese mensaje específico al bus — el bloque más usado para reaccionar a tráfico real.
- `on timer <nombreTimer>`: se ejecuta cuando expira un timer previamente armado con `setTimer()` — la base para lógica periódica o para simular retrasos/timeouts.
- `on key <tecla>`: se ejecuta al presionar una tecla definida — útil para pruebas exploratorias manuales controladas desde teclado.
- `on sysvar <variable>`: se ejecuta cuando cambia una variable de sistema (System Variable), un mecanismo de CANoe para compartir estado entre distintos scripts/paneles.
- `on diagRequest <servicio>`: específico para diagnóstico — se ejecuta cuando llega una solicitud UDS de un servicio determinado, permitiendo simular la respuesta de una ECU virtual a ese servicio (muy usado para simular ECUs que aún no existen físicamente).

**Funciones integradas (built-in functions) más comunes, con ejemplo de uso**

| Función | Qué hace | Ejemplo |
|---|---|---|
| `output(msg)` | Envía una trama al bus | `output(EngineStatus);` |
| `write(...)` | Imprime en la ventana Write (equivalente a un "print" para debugging) | `write("Valor actual: %d", velocidad);` |
| `setTimer(t, ms)` | Arma un timer para que dispare en X milisegundos | `setTimer(myTimer, 500);` |
| `cancelTimer(t)` | Cancela un timer armado previamente | `cancelTimer(myTimer);` |
| `getValue(sig)` / `setValue(sig, val)` | Lee/escribe el valor de una señal decodificada | `setValue(DoorStatus, 1);` |
| `envVarGetValue(ev)` / `envVarSetValue(ev, val)` | Lee/escribe una variable de entorno (Environment Variable), usada para simular entradas analógicas/digitales en un panel | `envVarSetValue(SensorTemp, 85.5);` |
| `testStep(estado, texto)` | Registra un paso de prueba con su resultado (dentro de un Test Module) | `testStep(1, "Sesión Extendida activada correctamente");` |
| `testCase(id, texto)` | Marca el inicio de un caso de prueba dentro de un Test Module | `testCase(1, "TC_SecurityAccess_Valido");` |
| `testWaitForTimeout(ms)` | Espera un tiempo determinado dentro de un caso de prueba automatizado | `testWaitForTimeout(2000);` |
| `diagSendRequest(req)` | Envía una solicitud de diagnóstico (UDS) usando la descripción CDD/ODX | `diagSendRequest(diagRequest.ReadDataByIdentifier);` |
| `diagGetLastResponseCode()` | Obtiene el código de la última respuesta de diagnóstico recibida | usado típicamente dentro de un `if` para validar un NRC esperado |

**Ejemplo integrador — simulando parte de una transacción UDS con CAPL** (ilustrativo, simplificado):
```c
on key 'a'
{
  diagRequest ReadEngineTemp req;
  diagSendRequest(req);
}

on diagResponse ReadEngineTemp
{
  if (this.RC == 0) // respuesta positiva
  {
    write("Temperatura del motor leída: %d", this.DID_EngineTemp);
  }
  else
  {
    write("Respuesta negativa recibida, NRC: %X", this.RC);
  }
}
```

Este tipo de ejemplo es justo el nivel de detalle que separa "he escuchado de CAPL" de "he trabajado con CAPL" en una entrevista técnica — poder describir el flujo evento → acción → verificación con soltura.

### 1.6 CANoe vs vTESTstudio — la pregunta de entrevista casi garantizada
Es muy común que un entrevistador pregunte directamente: *"¿cuál es la diferencia entre CANoe y vTESTstudio, si ambos son de Vector y ambos se usan para pruebas?"* — la respuesta correcta no es "hacen lo mismo", es una diferencia de **capa de abstracción y audiencia**:

- **CANoe** es la plataforma de **ejecución y simulación** — el motor que realmente interactúa con el bus, ejecuta la simulación, corre los scripts CAPL, y muestra los resultados en tiempo real. Requiere conocimiento más profundo de programación (CAPL) para crear pruebas complejas desde cero.
- **vTESTstudio** es una herramienta de **diseño y especificación de pruebas** de más alto nivel, pensada para que ingenieros de prueba puedan crear casos de prueba de forma más estructurada y con menor necesidad de programar directamente en CAPL — permite diseñar casos de prueba con una interfaz más cercana a diagramas de flujo/tablas, generando automáticamente el código de prueba que luego se ejecuta **dentro de CANoe**.
- **Relación entre ambas**: vTESTstudio **no reemplaza** a CANoe — los casos de prueba diseñados en vTESTstudio se exportan/vinculan para ejecutarse sobre el motor de CANoe. Es una relación de "diseño en una herramienta, ejecución en otra", similar en espíritu a cómo MIL/SIL usan Simulink para diseñar pero requieren un motor de ejecución/compilación distinto para cada etapa.

### 1.7 Por qué CANoe es tan central en tu flujo de trabajo
Conectando con lo que ya estudiaste en Protocolos de Comunicación, HIL y UDS: CANoe es literalmente la herramienta donde **conviven** todos esos conceptos en la práctica — decodificas tramas CAN/CAN FD reales en el Trace, ejecutas transacciones UDS completas (con CAN-TP segmentando por debajo, de forma transparente) desde la Diagnostic Console, simulas fault injection mediante scripts CAPL que alteran señales, y todo esto se integra con el banco HIL físico a través de las interfaces Vector VN y el VT System.

## Preguntas de teoría — CANoe
1. ¿Por qué CAPL es un lenguaje orientado a eventos, y qué ventaja tiene ese diseño para simular el comportamiento de una red vehicular?
2. ¿Qué diferencia hay entre usar el Trace Window para depurar comunicación y usar la Diagnostic Console para trabajar con UDS?
3. Explica la relación exacta entre CANoe y vTESTstudio — ¿por qué no son herramientas competidoras sino complementarias?
4. ¿Qué rol cumple una base de datos de comunicación (DBC/ARXML) dentro de una configuración de CANoe?
5. ¿Cómo usarías CANoe para simular una ECU que aún no existe físicamente, dentro de una red donde sí tienes hardware real de otras ECUs?
6. ¿Qué diferencia práctica hay entre elegir una interfaz VN16xx y una VN89xx para un banco HIL, más allá de "una tiene más canales"?
7. ¿Por qué necesitarías una interfaz VN56xx en vez de una VN16xx para cierto tipo de pruebas?
8. Explica qué es el VT System y da un ejemplo de cómo se usaría para simular un circuito abierto en el pin de un sensor.
9. ¿Qué diferencia hay entre un módulo VT2004 (I/O analógico/digital) y un módulo VT6104 (carga electrónica), y en qué escenario usarías cada uno?
10. Escribe (o describe en pseudocódigo) un bloque CAPL `on message` que valide si el valor de una señal excede un umbral y lo registre en la ventana Write.
11. ¿Cuál es la diferencia entre `testStep()` y `testCase()` en un Test Module de CANoe?
12. ¿Para qué sirve `envVarSetValue()` y en qué se diferencia de `setValue()` sobre una señal de bus?

## Errores comunes — CANoe
- Decir que CANoe y vTESTstudio "hacen lo mismo" — la diferencia de capa de abstracción (ejecución vs diseño de pruebas) es justamente lo que un entrevistador busca verificar.
- Olvidar mencionar que los Test Modules pueden escribirse tanto en CAPL como en .NET en versiones modernas — mostrar que conoces esta evolución da una señal de que tu conocimiento está actualizado.
- Confundir la simulación de nodos (para pruebas cuando falta hardware real) con la comunicación real de HIL — CANoe soporta ambos modos, y saber quándo estás en uno u otro es relevante para explicar tu trabajo con precisión.
- Hablar de "el banco HIL" de forma genérica sin poder nombrar el hardware específico (VN devices, VT System) — en una entrevista técnica, la especificidad es lo que distingue experiencia real de conocimiento superficial.
- Confundir una variable de entorno (Environment Variable, usada para simular entradas de panel/VT System) con una señal de bus decodificada — son mecanismos distintos dentro de CANoe, aunque ambos se leen/escriben con funciones similares (`envVarGetValue` vs `getValue`).

---

## 2. vTESTstudio (Vector Informatik)

### 2.1 Qué es y su propósito específico
vTESTstudio es la herramienta de Vector diseñada específicamente para **diseñar, especificar e implementar casos de prueba de forma reutilizable y mantenible**, con un enfoque en la trazabilidad hacia requisitos y en reducir la dependencia de programación directa en CAPL para escribir pruebas.

### 2.2 Conceptos clave de diseño en vTESTstudio
- **Test Cases y Test Steps**: un caso de prueba se descompone en pasos (steps) reutilizables — por ejemplo, un step "Enviar UDS Session Control a Extended Session" puede reutilizarse en decenas de casos de prueba distintos, en vez de reescribir la lógica cada vez.
- **Bibliotecas de pasos reutilizables (Reusable Test Steps / Building Blocks)**: esta es la característica que más diferencia a vTESTstudio de escribir CAPL puro — permite construir una **librería de acciones estandarizadas** (enviar un mensaje, verificar una respuesta, esperar un tiempo, cambiar de sesión de diagnóstico) que todo el equipo reutiliza, mejorando consistencia y reduciendo tiempo de mantenimiento cuando algo cambia (ej. si cambia el formato de un mensaje, se actualiza en un solo lugar, no en cada script individual).
- **Parametrización de casos de prueba**: los mismos pasos pueden ejecutarse con distintos valores de entrada (ej. probar Security Access con 5 keys incorrectas distintas) sin duplicar la lógica del caso de prueba — esto conecta directamente con la Partición de Equivalencia y Análisis de Valor Frontera que estudiaste en el capítulo de Fundamentos de Testing: vTESTstudio facilita ejecutar el mismo caso con múltiples valores de partición de forma sistemática.
- **Vinculación con requisitos**: permite establecer trazabilidad directa entre un caso de prueba y el requisito que valida (conectando con DOORS/RQM de tu stack), lo cual facilita generar reportes de cobertura de requisitos automáticamente.
- **Generación de código para CANoe**: al final, vTESTstudio traduce el diseño de alto nivel a algo ejecutable dentro del entorno de CANoe — el diseño y la ejecución están separados, pero conectados.

### 2.3 Ventaja concreta en la práctica (para justificar en entrevista por qué se usa)
Si el equipo solo usara CAPL puro en CANoe para todo, cada ingeniero probablemente escribiría la lógica de prueba de forma distinta, dificultando el mantenimiento a escala cuando el equipo crece o cuando cambian las especificaciones. vTESTstudio estandariza ese proceso — es la diferencia entre que cada ingeniero de prueba "programe" sus pruebas de forma libre, versus que el equipo **diseñe** pruebas de forma estructurada y reutilizable, con menor curva de entrada para ingenieros nuevos (conectando con tu experiencia real de onboarding de nuevos integrantes en Bosch).

## Preguntas de teoría — vTESTstudio
1. ¿Por qué la reutilización de Test Steps es una ventaja de mantenibilidad, no solo de velocidad de creación de pruebas?
2. ¿Cómo conecta la parametrización de casos de prueba en vTESTstudio con las técnicas de Partición de Equivalencia y Análisis de Valor Frontera?
3. ¿Qué ventaja de escalabilidad de equipo ofrece vTESTstudio frente a escribir CAPL puro para cada caso de prueba?
4. ¿Cómo se relaciona la trazabilidad de vTESTstudio hacia requisitos con la disciplina de Requirements Engineering en DOORS/RQM?

## Errores comunes — vTESTstudio
- Presentar vTESTstudio como una herramienta completamente independiente de CANoe — siempre depende del motor de CANoe para la ejecución real.
- No mencionar la reutilización de pasos como su principal valor diferencial — reducirlo solo a "una interfaz más amigable" pierde el punto de fondo (mantenibilidad y consistencia a escala de equipo).

---

## 3. Robot Framework

### 3.1 Qué es
Robot Framework es un framework de automatización de pruebas **genérico, open-source, basado en palabras clave (keyword-driven)** — a diferencia de CANoe/vTESTstudio (especializados en comunicación vehicular), Robot Framework no es específico de automotriz; se usa ampliamente también en pruebas web, API, y automatización general. Su valor en tu contexto es que actúa como **capa de orquestación** por encima de herramientas más especializadas.

### 3.2 Arquitectura keyword-driven
- **Palabras clave (Keywords)**: son las unidades reutilizables de acción — pueden ser palabras clave de más alto nivel construidas a partir de otras más básicas, o palabras clave de bajo nivel implementadas directamente en código (típicamente Python, conectando con tu CV).
- **Test Cases**: se escriben en un formato de **texto plano, tabular y legible por humanos no necesariamente programadores** (esto es una diferencia filosófica importante frente a CAPL) — un caso de prueba en Robot Framework se lee casi como una lista de pasos en lenguaje natural estructurado, lo que facilita que stakeholders no técnicos (ej. representantes de negocio) puedan entender o incluso contribuir a la especificación de pruebas — conectando con el enfoque de equipo completo y ATDD del capítulo de Fundamentos de Testing.
- **Librerías (Libraries)**: el mecanismo de extensibilidad de Robot Framework — una librería expone palabras clave implementadas en código (Python es la más común). Existen librerías estándar (ej. para manejo de archivos, procesos) y librerías personalizadas que tu equipo puede desarrollar para interactuar con herramientas específicas — **aquí es donde probablemente conectas Robot Framework con CANoe**: mediante una librería personalizada en Python que se comunica con la API/COM de CANoe, permitiendo que un caso de prueba escrito en Robot Framework dispare y controle una ejecución de prueba dentro de CANoe.
- **Test Suites**: agrupaciones de casos de prueba relacionados, organizados típicamente por archivo/carpeta.
- **Reportes y Logs automáticos**: Robot Framework genera automáticamente reportes HTML detallados de cada ejecución (qué pasó, qué falló, con capturas de log paso a paso) sin necesidad de configuración adicional — una ventaja operativa relevante para reportar resultados a stakeholders rápidamente.

### 3.3 Librerías y palabras clave integradas más comunes

Además de librerías personalizadas (como la que conecta con CANoe), Robot Framework incluye **librerías estándar** que ya traen palabras clave listas para usar — vale la pena conocerlas porque suelen ser la base sobre la cual se construyen los casos de prueba reales antes de necesitar nada personalizado:

- **BuiltIn** (se carga automáticamente, no requiere importarse): contiene las palabras clave más básicas y usadas.
  - `Log` — escribe un mensaje en el log de ejecución (equivalente conceptual a `write()` en CAPL).
  - `Should Be Equal` / `Should Not Be Equal` — la base de cualquier aserción/verificación.
  - `Should Contain` — verifica que un valor esté presente dentro de otro (ej. una cadena, una lista).
  - `Run Keyword If` — ejecuta una palabra clave condicionalmente.
  - `Sleep` — pausa la ejecución un tiempo determinado (equivalente a `testWaitForTimeout()` en CAPL).
  - `Set Variable` / `Set Global Variable` — define variables dentro del alcance de un test o de forma global.
- **Collections**: manejo de listas y diccionarios — `Append To List`, `Get From Dictionary`, `Length Should Be`.
- **OperatingSystem**: interacción con el sistema de archivos — `Create File`, `Remove File`, `Directory Should Exist` — útil para, por ejemplo, verificar que un archivo de log o reporte se generó correctamente tras una ejecución de prueba.
- **Process**: ejecutar procesos externos y capturar su salida — `Run Process`, `Start Process` — esto es clave para **disparar herramientas externas** (por ejemplo, lanzar un script que a su vez controla CANoe) desde un caso de prueba de Robot Framework.
- **DateTime**: manejo de fechas/tiempos — útil para timestamps en reportes o para calcular duraciones de prueba.

**Ejemplo de un caso de prueba simple en formato Robot Framework (.robot)**:
```
*** Settings ***
Library    OperatingSystem
Library    CANoeLibrary    # librería personalizada

*** Test Cases ***
Verificar Sesion Extendida UDS
    [Documentation]    Valida que la ECU acepte el cambio a Extended Diagnostic Session
    Iniciar Medicion CANoe
    Enviar Servicio UDS    10 03
    ${respuesta}=    Obtener Ultima Respuesta UDS
    Should Be Equal    ${respuesta}    50 03
    Log    Sesión extendida confirmada correctamente
    Detener Medicion CANoe
```

Nota cómo el caso de prueba se lee casi como una lista de instrucciones en lenguaje natural — `Enviar Servicio UDS`, `Obtener Ultima Respuesta UDS` y `Iniciar/Detener Medicion CANoe` no son palabras clave nativas de Robot Framework, sino **palabras clave personalizadas** definidas en una librería propia (`CANoeLibrary`) que el equipo desarrolla en Python.

**Ejemplo simplificado de cómo se vería esa librería personalizada por dentro (Python)**:
```python
# CANoeLibrary.py
import win32com.client  # librería estándar de Windows para interactuar vía COM

class CANoeLibrary:
    def __init__(self):
        self.app = win32com.client.Dispatch("CANoe.Application")

    def iniciar_medicion_canoe(self):
        self.app.Measurement.Start()

    def detener_medicion_canoe(self):
        self.app.Measurement.Stop()

    def enviar_servicio_uds(self, servicio_hex):
        # Aquí se invocaría la API de diagnóstico de CANoe (simplificado)
        self.app.Diagnostics.SendRequest(servicio_hex)

    def obtener_ultima_respuesta_uds(self):
        return self.app.Diagnostics.LastResponse
```

Este ejemplo (simplificado a propósito, la API real de CANoe.Application tiene más matices) ilustra el punto técnico central que un entrevistador busca: **Robot Framework no habla "CAN" de forma nativa** — la magia ocurre porque una librería Python usa la interfaz **COM de CANoe** (`win32com.client`) para controlar la aplicación programáticamente, y Robot Framework simplemente orquesta llamadas a esa librería con una sintaxis legible.

### 3.5 Por qué se usa Robot Framework junto con CANoe/vTESTstudio (y no en su lugar)
Esta es otra pregunta de entrevista con potencial de trampa: *"si ya tienes CANoe y vTESTstudio para probar comunicación, ¿para qué necesitas Robot Framework?"* — la respuesta clave es que **resuelven problemas en capas distintas**:
- CANoe/vTESTstudio están **especializados** en la interacción de bajo nivel con protocolos vehiculares (CAN, UDS, XCP) — son la herramienta correcta para *ese* dominio específico.
- Robot Framework brilla como **orquestador de alto nivel**: puede coordinar un escenario de prueba que involucre no solo CANoe (para la parte de comunicación vehicular), sino también otras herramientas, APIs, o pasos que no tienen nada que ver con buses vehiculares (ej. verificar un log en un sistema externo, interactuar con una base de datos de resultados, disparar un pipeline de Jenkins) — todo dentro del mismo caso de prueba legible.
- También aporta la ventaja de la **legibilidad para no-programadores** y la generación automática de reportes, que no son el foco principal de un script CAPL.

### 3.6 Integración con CI/CD (conectando con tu experiencia en Docker/Jenkins)
Robot Framework se integra naturalmente en pipelines de Jenkins: un pipeline puede disparar automáticamente una suite de Robot Framework en cada nuevo build/commit, y el resultado (pass/fail, reporte HTML) se recopila como parte del pipeline de CI — esto es lo que habilita la **prueba de regresión automatizada** de la que hablamos en Fundamentos de Testing, ejecutándose sin intervención manual cada vez que hay un cambio de software. Docker, por su parte, permite empaquetar el entorno de ejecución de estas pruebas (dependencias de Python, librerías) de forma consistente y reproducible entre distintas máquinas del equipo.

## Preguntas de teoría — Robot Framework
1. ¿Qué significa que Robot Framework sea "keyword-driven", y qué ventaja de legibilidad ofrece frente a escribir scripts en un lenguaje de programación tradicional?
2. ¿Cómo conectarías Robot Framework con CANoe técnicamente? (pista: piensa en el rol de una librería personalizada en Python)
3. ¿Por qué Robot Framework y CANoe/vTESTstudio no son herramientas competidoras, sino que operan en capas distintas del problema de automatización?
4. ¿Cómo encaja Robot Framework en un pipeline de Jenkins, y qué papel juega Docker en ese flujo?
5. ¿Qué ventaja ofrece Robot Framework para el enfoque de equipo completo (whole-team approach) que no ofrece un script CAPL puro?
6. ¿Qué diferencia hay entre una palabra clave de la librería `BuiltIn` (ej. `Should Be Equal`) y una palabra clave personalizada como `Enviar Servicio UDS`?
7. ¿Qué es `win32com.client` y qué rol cumple específicamente en la integración entre Python/Robot Framework y CANoe?
8. Si necesitaras verificar que un archivo de reporte se generó correctamente después de una ejecución, ¿qué librería estándar de Robot Framework usarías?
9. ¿Por qué diseñar bien los nombres de las palabras clave personalizadas (que sean legibles en lenguaje natural) importa para el enfoque de equipo completo?

## Errores comunes — Robot Framework
- Presentar Robot Framework como una herramienta "para probar CAN" — es un framework genérico de automatización; su rol específico en tu stack es de **orquestación**, no de interacción directa de bajo nivel con el bus (eso lo sigue haciendo CANoe por debajo).
- No mencionar el mecanismo de librerías como el punto de integración con Python/CANoe — es lo que explica técnicamente cómo encajan las piezas.
- Olvidar la generación automática de reportes como una ventaja operativa concreta, no solo una característica técnica menor.
- Pensar que Robot Framework "habla CAN nativamente" — la comunicación real ocurre a través de una librería Python que usa una interfaz como COM (`win32com.client`) para controlar CANoe; Robot Framework nunca interactúa con el bus directamente.
- Confundir palabras clave estándar (`BuiltIn`, `Collections`, `OperatingSystem`) con palabras clave personalizadas del dominio — un entrevistador puede notar si solo conoces las genéricas y no cómo se construyen las específicas del proyecto.

---

## 4. Cómo encajan las tres herramientas juntas (síntesis para entrevista)

Un entrevistador de nivel Test Function Lead puede pedirte explícitamente: *"Descríbeme tu stack de automatización de principio a fin"* — aquí tienes una forma clara de estructurar esa respuesta:

1. **Diseño del caso de prueba**: se especifica en **vTESTstudio**, con pasos reutilizables y trazabilidad a requisitos (DOORS/RQM).
2. **Ejecución de bajo nivel**: el caso diseñado se ejecuta dentro de **CANoe**, que interactúa realmente con el bus físico/simulado (CAN, CAN FD, UDS, XCP) a través del banco HIL.
3. **Orquestación de alto nivel**: **Robot Framework** puede envolver la ejecución completa, coordinando el caso de CANoe junto con otros pasos del escenario de prueba (verificaciones externas, generación de reportes, disparo desde CI/CD).
4. **Integración continua**: **Jenkins** dispara automáticamente esta cadena completa en cada build nuevo, con el entorno empaquetado en **Docker** para consistencia, generando reportes que alimentan las métricas de prueba (cobertura, defectos) que estudiaste en Gestión de Actividades de Prueba.
5. **Gestión de resultados**: los defectos encontrados se registran en **Jira/RQM**, cerrando el ciclo completo del proceso de prueba.

## Tabla comparativa rápida

| Herramienta | Capa | Propósito principal | Lenguaje/Formato |
|---|---|---|---|
| **CANoe** | Ejecución/Simulación | Motor de interacción real con el bus (CAN/LIN/Ethernet), simulación de nodos | CAPL, .NET (C#) |
| **vTESTstudio** | Diseño de pruebas | Especificación estructurada y reutilizable de casos de prueba | Interfaz gráfica → genera ejecución en CANoe |
| **Robot Framework** | Orquestación | Automatización genérica, legible, integración con CI/CD | Texto tabular + Python (librerías) |

## Preguntas de teoría — Síntesis
1. Describe tu stack de automatización de principio a fin, desde el diseño de un caso de prueba hasta el reporte de resultados en CI/CD.
2. Si tuvieras que eliminar una de las tres herramientas de tu flujo de trabajo, ¿cuál sería la más difícil de reemplazar y por qué?
3. ¿En qué escenario usarías directamente CAPL en CANoe en vez de diseñar el caso en vTESTstudio?

## Errores comunes — Síntesis
- Presentar las tres herramientas como opciones intercambiables entre sí, en vez de piezas complementarias de un mismo flujo — mostrar que entiendes la arquitectura completa (diseño → ejecución → orquestación → CI/CD) es lo que distingue una respuesta de nivel medio de una de Test Function Lead.