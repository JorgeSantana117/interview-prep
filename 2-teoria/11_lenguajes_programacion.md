# Capítulo: Lenguajes de Programación

> Objetivo del capítulo: dominar los lenguajes que realmente usas (**Python** y **CAPL**), los que aparecen en tu entorno de modelado y análisis (**MATLAB**) y el que casi seguro te preguntarán en una entrevista de Embedded SW Engineer (**C**), junto con las **buenas prácticas** que separan un script que "funciona" de un código mantenible. En entrevistas de este tipo suelen pedir escribir o corregir fragmentos cortos, así que el capítulo incluye ejemplos y ejercicios.
>
> **Nota sobre el alcance:** tu estructura original incluía Python, CAPL, MATLAB y buenas prácticas. Agregué una sección de **C embebido** porque no aparece en tu CV pero es el lenguaje base de los sistemas que pruebas y de las preguntas de código de un Embedded SW Engineer. Si prefieres quitarla, se puede mover a un apéndice.
>
> **Límites con otros capítulos (para no repetir):**
> - Tipos de bloques de eventos de CAPL, tabla de funciones integradas y ejemplo de transacción UDS → ya en *Herramientas de Testing y Automatización*. Aquí: tipos de datos, objetos de mensaje, acceso a señales, temporizadores, estructura de un módulo de prueba y errores típicos.
> - Keywords, librerías y la integración Robot Framework ↔ CANoe → ya en *Herramientas*. Aquí: Python como lenguaje de esas librerías.
> - MISRA C, análisis estático y principios de diseño → ya en *Ingeniería de Software*. Aquí: el lenguaje C en sí.
> - Simulink/Stateflow y el modelado de ECUs → irán en *Diseño Basado en Modelos*. Aquí: MATLAB como lenguaje de scripting y análisis.

---

## 1. Panorama: qué lenguaje para qué

| Lenguaje | Naturaleza | Dónde lo usas | Fortaleza | Limitación |
|---|---|---|---|---|
| **Python** | Interpretado, tipado dinámico | Automatización de pruebas, librerías de Robot Framework, análisis de logs, herramientas internas, scripts de CI | Productividad, ecosistema enorme, legibilidad | No determinista ni apto para firmware de tiempo real; más lento |
| **CAPL** | Interpretado/compilado dentro de CANoe, orientado a eventos, sintaxis tipo C | Simulación de nodos, pruebas de bus, módulos de prueba en CANoe | Acceso nativo a mensajes, señales, diagnóstico y temporización del bus | Solo existe dentro del ecosistema Vector |
| **MATLAB** | Interpretado, orientado a matrices | Análisis numérico y de datos de medición, scripts de Simulink, procesamiento de señales | Cálculo y visualización rápidos | Licencia comercial; menos adecuado como lenguaje de automatización general |
| **C** | Compilado, de bajo nivel | Firmware de ECU, drivers, AUTOSAR BSW, código generado desde modelos | Control total del hardware, determinismo, eficiencia | Sin red de seguridad: errores de memoria y comportamiento indefinido |

**Idea para entrevista:** un buen ingeniero de prueba no se identifica con un lenguaje sino que elige el adecuado para cada capa: CAPL cerca del bus, Python para orquestar y analizar, MATLAB para datos numéricos y modelos, y entiende C para razonar sobre el software bajo prueba.

---

## 2. Python

### 2.1 Fundamentos que se preguntan
- **Tipos y mutabilidad**: `int`, `float`, `str`, `bytes`, `list`, `tuple`, `dict`, `set`. Son **inmutables** `int`, `float`, `str`, `bytes`, `tuple`; **mutables** `list`, `dict`, `set`.
- **Igualdad vs identidad**: `==` compara valores, `is` compara si es el mismo objeto.
- **Comprensiones y generadores**: `[x*2 for x in datos if x > 0]` crea una lista; `(x*2 for x in datos)` es un generador que produce valores bajo demanda (útil para logs grandes).
- **Manejo de excepciones**: `try / except / else / finally`; capturar excepciones **específicas**, no `except:` a secas.
- **Context managers** (`with`): garantizan liberación de recursos (archivos, conexiones, sesiones de medición).
- **Decoradores**: funciones que envuelven a otras (por ejemplo, para registrar tiempo o reintentar).
- **Programación orientada a objetos**: clases, herencia, composición, `@dataclass` para estructuras de datos simples.
- **Anotaciones de tipo (type hints)**: `def leer(id: int) -> bytes:` mejoran legibilidad y permiten verificación estática (por ejemplo, con mypy).
- **Alcance y *closures***, desempaquetado (`a, *resto = lista`), f-strings.

### 2.2 Biblioteca estándar que más usarás
`pathlib` (rutas), `subprocess` (lanzar herramientas externas), `logging`, `argparse` (CLI), `json`/`csv`, `re` (expresiones regulares para parsear logs), `struct` (empaquetar/desempaquetar bytes), `datetime`/`time`, `threading` y `asyncio` (concurrencia), `unittest`.

### 2.3 Ecosistema para pruebas y automotriz
- **pytest**: framework de pruebas con *fixtures*, `parametrize` y marcadores. Alternativa moderna a `unittest`.
- **python-can**: acceso a interfaces CAN (incluidas muchas de Vector) desde Python.
- **cantools**: lectura de archivos DBC/ARXML y codificación/decodificación de señales.
- **can-isotp / udsoncan**: implementación de CAN-TP (ISO-TP) y de clientes UDS.
- **pySerial**: puerto serie (consolas de depuración).
- **NumPy / pandas / matplotlib**: análisis y gráficas de datos de medición.
- **asammdf**: lectura de archivos MDF (los logs de CANoe en formato `.mf4`).
- **pywin32 (`win32com`)**: controlar CANoe por su interfaz COM (visto en el capítulo de Herramientas).
- **requests, paramiko**: HTTP y SSH hacia plataformas (por ejemplo, ejecutar comandos en una ECU con Linux/QNX).

### 2.4 Ejemplos prácticos

**a) Valores frontera con pytest** (conecta con el ejercicio de velocidad de Fundamentos de Testing):
```python
import pytest

def clasificar_velocidad(v: int) -> str:
    if v <= 50:
        return "OK"
    if v <= 55:
        return "ADVERTENCIA"
    if v <= 60:
        return "MULTA"
    return "SUSPENSION"

@pytest.mark.parametrize("v, esperado", [
    (50, "OK"), (51, "ADVERTENCIA"),
    (55, "ADVERTENCIA"), (56, "MULTA"),
    (60, "MULTA"), (61, "SUSPENSION"),
])
def test_valores_frontera(v, esperado):
    assert clasificar_velocidad(v) == esperado
```

**b) Esperar una condición con *polling* y tiempo límite** (en vez de `sleep` fijo):
```python
import time

def esperar_hasta(condicion, timeout_s=5.0, intervalo_s=0.05):
    inicio = time.monotonic()
    while time.monotonic() - inicio < timeout_s:
        if condicion():
            return True
        time.sleep(intervalo_s)
    return False

assert esperar_hasta(lambda: ecu.sesion() == "EXTENDED", timeout_s=2.0)
```

**c) *Context manager* para una medición de CANoe** (garantiza que se detenga aunque la prueba falle):
```python
from contextlib import contextmanager

@contextmanager
def medicion_canoe(app):
    app.Measurement.Start()
    try:
        yield
    finally:
        app.Measurement.Stop()

with medicion_canoe(app):
    ejecutar_prueba()
```

**d) Empaquetar bytes respetando el orden** (relevante por endianness):
```python
import struct
payload = struct.pack("<HBx", 3000, 1)   # little-endian: uint16, uint8, 1 byte de relleno
```

**e) Decodificar con un DBC**:
```python
import cantools
db = cantools.database.load_file("vehiculo.dbc")
msg = db.get_message_by_name("EngineStatus")
senales = msg.decode(bytes.fromhex("00 7D 00 00 00 00 00 00"))
```

### 2.4.1 Errores clásicos de Python (preguntas de "¿qué imprime esto?")
- **Argumento por defecto mutable**: `def f(x, lista=[])` comparte la misma lista entre llamadas. Se corrige con `lista=None` y creándola dentro.
- **Copia superficial vs profunda**: `copy.copy` copia el contenedor pero comparte los objetos internos; `copy.deepcopy` los duplica.
- **Comparar flotantes con `==`**: usar `math.isclose` o una tolerancia.
- **`bytes` vs `str`**: son tipos distintos; hay que codificar/decodificar explícitamente.
- **Enlace tardío en *closures*** dentro de bucles.
- **División**: `/` siempre da `float`; `//` es división entera.

### 2.5 Concurrencia
- **GIL (Global Interpreter Lock)**: en CPython, solo un hilo ejecuta bytecode a la vez, por lo que `threading` **no acelera** trabajo intensivo de CPU, pero sí sirve para tareas dominadas por E/S (esperar respuestas de un banco, lecturas de red o serie).
- **`multiprocessing`**: procesos separados, evita el GIL en cálculo pesado.
- **`asyncio`**: concurrencia cooperativa para muchas E/S simultáneas.
- Para pruebas de HIL, el uso típico es un hilo o proceso auxiliar que **registra datos** (logging de bus) mientras el hilo principal ejecuta la secuencia de prueba.

### 2.6 Entorno y distribución
`venv` para aislar dependencias, `pip` con `requirements.txt` o `pyproject.toml` con versiones fijadas (reproducibilidad en CI y en Docker), y separación entre código de prueba, librerías reutilizables y configuración.

---

## 3. CAPL en profundidad (complemento al capítulo de Herramientas)

### 3.1 Tipos de datos (¡cuidado con los tamaños!)
| Tipo | Tamaño | Comentario |
|---|---|---|
| `char` | 8 bits | Caracteres y enteros pequeños con signo |
| `byte` | 8 bits | Sin signo |
| `int` | **16 bits** con signo | Trampa clásica: en CAPL `int` NO es de 32 bits |
| `word` | 16 bits | Sin signo |
| `long` | 32 bits con signo | |
| `dword` | 32 bits | Sin signo |
| `int64` / `qword` | 64 bits | Con y sin signo |
| `float` / `double` | Punto flotante | |

Un contador declarado como `int` se desborda a 32767. Para tiempos largos o valores grandes conviene `long` o `dword`.

### 3.2 Objetos de mensaje y acceso a datos
```c
variables
{
  message 0x123 TxMsg;     // por ID
  message EngineStatus msgEngine;   // por nombre simbólico (requiere base de datos DBC)
}

on message EngineStatus
{
  byte primero = this.byte(0);       // acceso a bytes
  word rpmRaw  = this.word(1);       // acceso por palabra (respeta el orden configurado)
  long id      = this.id;
  long dlc     = this.dlc;
  double rpm   = $EngineSpeed;       // señal decodificada con '$'
}
```
- **`this`**: referencia al mensaje que disparó el evento (solo válido dentro de `on message`).
- **`$NombreSeñal`**: acceso al valor físico de una señal definida en la base de datos.
- **`@NombreVariableDeEntorno`** y **`@sysvar::Espacio::Variable`**: variables de entorno y de sistema (para paneles, y para intercambiar estado entre scripts y herramientas).
- Un mensaje se envía con `output(msg)`. Conviene **inicializar `dlc` y los bytes** antes de enviar.

### 3.3 Temporizadores
- `msTimer`: resolución en **milisegundos**. `timer`: en **segundos**.
- Un temporizador **no se repite solo**: hay que volver a armarlo dentro de su handler (`setTimer`) o usar un esquema cíclico propio.
- Cancelar con `cancelTimer`. Reiniciar un timer ya armado con `setTimer` lo reprograma.

### 3.4 Modelo de ejecución
Los handlers de eventos **se ejecutan uno a la vez y no se interrumpen entre sí**. Consecuencias: mantenerlos **cortos**, evitar bucles largos o esperas bloqueantes dentro de `on message`, y recordar que una espera activa congela la atención a otros eventos del mismo nodo.

### 3.5 Funciones, estructuras, archivos
- Funciones con tipo de retorno y parámetros (`int Suma(int a, int b)`).
- `struct` para agrupar datos relacionados, arreglos fijos y cadenas con funciones como `snprintf` (en versiones recientes).
- `includes { #include "Utilidades.cin" }` para reutilizar código (`.cin` son archivos de inclusión; `.can` los programas de nodo).

### 3.6 Ejemplos

**a) Mensaje cíclico con contador y *checksum*:**
```c
variables
{
  message 0x123 TxMsg;
  msTimer tCiclo;
  byte contador = 0;
}

on start
{
  TxMsg.dlc = 8;
  setTimer(tCiclo, 100);
}

on timer tCiclo
{
  int i;
  byte chk = 0;

  TxMsg.byte(0) = contador;
  for (i = 0; i < 7; i++)
  {
    chk ^= TxMsg.byte(i);
  }
  TxMsg.byte(7) = chk;

  output(TxMsg);
  contador = (contador + 1) & 0x0F;   // contador de 4 bits
  setTimer(tCiclo, 100);
}
```

**b) Monitoreo de timeout de un mensaje cíclico (cada 100 ms; se declara fallo tras 3 ciclos perdidos):**
```c
variables
{
  msTimer tRxTimeout;
  const int cTimeoutMs = 300;
  int conteoTimeout = 0;
}

on message EngineStatus
{
  cancelTimer(tRxTimeout);
  setTimer(tRxTimeout, cTimeoutMs);
}

on timer tRxTimeout
{
  conteoTimeout++;
  write("Timeout de EngineStatus (#%d)", conteoTimeout);
}
```

**c) Esqueleto de un módulo de prueba (las firmas exactas de las funciones de prueba varían entre versiones; confirma en la ayuda de tu CANoe):**
```c
testcase TC_Velocidad_En_Rango()
{
  testCaseTitle("TC_001", "La velocidad llega al rango esperado");
  // estímulo: fijar la entrada (por ejemplo, una variable de entorno o sysvar)
  // verificación: esperar con límite de tiempo
  if (testWaitForSignalInRange(VehicleSpeed, 49, 51, 2000) == 1)
    testStepPass("1", "Velocidad dentro del rango");
  else
    testStepFail("1", "Velocidad fuera del rango en 2 s");
}

void MainTest()
{
  TC_Velocidad_En_Rango();
}
```

### 3.7 Errores típicos en CAPL
- **`int` de 16 bits** que se desborda sin avisar.
- **Handler bloqueante** que provoca pérdida de eventos.
- **Olvidar rearmar el timer** o armarlo en `on start` pero no cancelarlo al detener.
- Confundir **señal** (`$`), **variable de entorno** (`@`) y **variable de sistema** (`@sysvar::`).
- **Orden de bytes** incorrecto al leer con `word(n)`/`dword(n)` sobre mensajes en formato Motorola frente a Intel.
- **Pruebas que dependen de `testWaitForTimeout` fijo** en vez de esperar una condición con tiempo límite.
- Enviar un mensaje **sin haber fijado `dlc`**.

---

## 4. MATLAB

### 4.1 Fundamentos
- Tipo de dato central: la **matriz**. Una variable escalar es una matriz de 1×1.
- **Indexación desde 1** (no desde 0, a diferencia de Python y C) y los rangos son inclusivos: `v(2:4)` devuelve los elementos 2, 3 y 4.
- **Vectorización**: operar sobre arreglos completos en lugar de usar bucles (más claro y más rápido).
- **Operaciones elemento a elemento** (`.*`, `./`, `.^`) frente a las **matriciales** (`*`, `/`, `^`). Confundirlas es un error típico.
- **Estructuras de datos**: arreglos numéricos, `struct`, **celdas** (`cell`, para contenidos heterogéneos), **tablas** (`table`, tipo hoja de cálculo) y cadenas.
- **Scripts vs funciones**: un script comparte el espacio de trabajo; una función tiene el suyo propio y recibe y devuelve argumentos.
- **Gráficas**: `plot`, `subplot`, `histogram`, etiquetas, ejes.

### 4.2 Uso típico en tu contexto
- **Análisis de mediciones**: leer logs (por ejemplo, archivos MDF con las herramientas del *Vehicle Network Toolbox*), filtrar, calcular estadísticas, comparar contra límites y graficar.
- **Scripting de Simulink**: automatizar simulaciones y parametrizar modelos.
- **Procesamiento de señales**, filtros, estimación de latencias y jitter a partir de registros de tiempo.
- **Pruebas**: el framework `matlab.unittest` y herramientas como Simulink Test, para modelos y funciones.
- **Generación de código**: de un modelo se genera C con herramientas como Embedded Coder (se verá en Diseño Basado en Modelos).

### 4.3 Ejemplos
**Vectorizado en lugar de bucle (misma regla de velocidades del ejemplo en Python):**
```matlab
v = [48 50 51 55 56 60 61];
estado = repmat("OK", size(v));
estado(v > 50 & v <= 55) = "ADVERTENCIA";
estado(v > 55 & v <= 60) = "MULTA";
estado(v > 60)           = "SUSPENSION";
```

**Automatizar una simulación:**
```matlab
load_system('mi_modelo');
set_param('mi_modelo/Ganancia', 'Gain', '2.5');
salida = sim('mi_modelo', 'StopTime', '10');
```

**Latencia a partir de timestamps (idea):**
```matlab
lat = t_respuesta - t_solicitud;      % vectores de tiempos emparejados
fprintf('Media = %.2f ms, Máx = %.2f ms\n', mean(lat)*1e3, max(lat)*1e3);
```

### 4.4 MATLAB vs Python (pregunta frecuente)
| Aspecto | MATLAB | Python |
|---|---|---|
| Costo | Licencia comercial | Gratuito |
| Indexación | Desde 1 | Desde 0 |
| Cálculo numérico y visualización | Muy integrado, con cajas de herramientas (toolboxes) | Con NumPy, SciPy, pandas, matplotlib |
| Integración con Simulink | Nativa | Mediante APIs externas |
| Automatización general, CI, redes, sistemas | Limitada | Excelente |
| Reproducibilidad en pipelines | Requiere licencias en el entorno | Sencilla (contenedores) |

Una respuesta madura: MATLAB sigue siendo la opción natural cuando el flujo gira en torno a Simulink y toolboxes; Python gana cuando hay que integrarse con CI, bancos de prueba, herramientas externas y análisis de datos sin depender de licencias.

---

## 5. C para sistemas embebidos

### 5.1 Fundamentos que se evalúan
- **Tipos de ancho fijo** (`stdint.h`): `uint8_t`, `int16_t`, `uint32_t`… Evitan ambigüedades de tamaño entre plataformas (un `int` puede ser de 16 o 32 bits).
- **Promoción de enteros y conversiones implícitas**: fuente de errores sutiles (por ejemplo, mezclar con y sin signo).
- **Punteros y aritmética de punteros**, relación entre arreglos y punteros, punteros a función, `const` en distintas posiciones (`const uint8_t *p` vs `uint8_t *const p`).
- **`volatile`**: el valor puede cambiar fuera del flujo del programa (registros, variables modificadas por una ISR); evita ciertas optimizaciones del compilador. **No** garantiza atomicidad ni sincronización.
- **`static`**: a nivel de archivo oculta el símbolo; dentro de una función conserva el valor entre llamadas.
- **`struct`, `union`, campos de bits, alineación y relleno (*padding*)**: el tamaño de una estructura puede ser mayor que la suma de sus campos. Los campos de bits y las uniones dependen de la implementación (endianness, orden), por lo que son peligrosos para mapear protocolos.
- **Preprocesador**: `#define`, macros (con paréntesis para evitar errores de precedencia), compilación condicional.
- **Memoria**: pila, montón, datos estáticos (ver capítulo de Sistemas Embebidos). En sistemas críticos se evita `malloc`.
- **Comportamiento indefinido (UB)**: desbordamiento de enteros con signo, acceso fuera de límites, uso de variables sin inicializar, dereferenciar `NULL`, violar *strict aliasing*. El compilador puede asumir que no ocurre y producir resultados inesperados.

### 5.2 Manipulación de bits (pregunta casi segura)
```c
#define BIT_SET(reg, n)   ((reg) |=  (1U << (n)))
#define BIT_CLR(reg, n)   ((reg) &= ~(1U << (n)))
#define BIT_TGL(reg, n)   ((reg) ^=  (1U << (n)))
#define BIT_CHK(reg, n)   (((reg) >> (n)) & 1U)

/* Contar bits en 1 (algoritmo de Kernighan) */
uint8_t contar_bits(uint32_t x)
{
    uint8_t c = 0U;
    while (x != 0U)
    {
        x &= (x - 1U);   /* elimina el bit 1 menos significativo */
        c++;
    }
    return c;
}
```

### 5.3 Otros fragmentos clásicos
```c
/* ¿Little-endian? */
bool es_little_endian(void)
{
    uint16_t v = 0x0001U;
    return (*(uint8_t *)&v) == 1U;
}

/* Máquina de estados con tabla de funciones */
typedef enum { ST_IDLE, ST_ACTIVE, ST_ERROR, ST_COUNT } estado_t;
typedef estado_t (*manejador_t)(void);

static estado_t on_idle(void)   { /* ... */ return ST_ACTIVE; }
static estado_t on_active(void) { /* ... */ return ST_ACTIVE; }
static estado_t on_error(void)  { /* ... */ return ST_IDLE;   }

static const manejador_t tabla[ST_COUNT] = { on_idle, on_active, on_error };

void paso(estado_t *s) { *s = tabla[*s](); }
```
También son habituales: **buffer circular** (con cuidado en la concurrencia entre ISR y tarea), invertir bits de un byte, *swap* de variables, determinar el tamaño de una estructura y reconocer los errores en un fragmento (desbordamiento de buffer, puntero colgante, uso de `==` con flotantes).

### 5.4 Preguntas típicas de entrevista (con la idea de la respuesta)
- *¿Para qué sirve `volatile` y cuándo se necesita?* Registros mapeados en memoria y variables compartidas con ISRs; no sustituye a la sincronización.
- *¿Qué diferencia hay entre `const` y `#define`?* `const` tiene tipo y ámbito y se ve en el depurador; `#define` es sustitución textual.
- *¿Qué pasa si una ISR modifica una variable que lee la tarea principal?* Condición de carrera; hay que proteger el acceso (sección crítica, acceso atómico, doble buffer).
- *¿Qué es un *stack overflow* y cómo lo detectarías?* Pila excedida; se detecta con marcas en la pila (*canaries*), análisis de uso máximo, protección por MPU.
- *¿Por qué evitar `malloc` en sistemas críticos?* Fragmentación, no determinismo y fallos de asignación.

---

## 6. Buenas prácticas de programación

### 6.1 Principios generales (aplican a cualquier lenguaje)
- **Legibilidad primero**: el código se lee muchas más veces de las que se escribe. Nombres descriptivos, funciones cortas con una sola responsabilidad.
- **DRY, KISS, YAGNI**: no repetir lógica, mantener la solución simple, no construir lo que aún no se necesita.
- **Evitar números mágicos**: constantes con nombre y unidades (`TIMEOUT_MS = 300`).
- **Comentar el *porqué***, no lo que el código ya dice. Documentar interfaces, supuestos y limitaciones.
- **Manejo explícito de errores**: no ignorar valores de retorno ni excepciones; mensajes con contexto.
- **Una función, un nivel de abstracción.**
- **Código determinista y con efectos secundarios controlados**, sobre todo en pruebas.

### 6.2 Buenas prácticas específicas por lenguaje
| Lenguaje | Estilo y herramientas |
|---|---|
| **Python** | PEP 8, *type hints*, docstrings, formateador (Black o Ruff), *linters* (Ruff, flake8, pylint), verificación de tipos (mypy), `venv` y dependencias fijadas |
| **CAPL** | Convenciones de nombres coherentes (prefijos para timers `t…`, constantes `c…`), handlers cortos, funciones reutilizables en `.cin`, constantes `const`, comentarios sobre IDs y unidades |
| **MATLAB** | Funciones en vez de scripts largos, vectorización, preasignar arreglos, nombres claros, análisis con Code Analyzer |
| **C** | MISRA C, tipos de ancho fijo, `const` y `static` cuando aplique, un solo punto de salida (según el estándar del proyecto), sin memoria dinámica, análisis estático en CI |

### 6.3 Código de prueba mantenible (lo que más importa en tu rol)
- **Arquitectura por capas**: *casos de prueba* (qué se verifica) → *palabras clave / acciones reutilizables* (cómo se hace) → *drivers de bajo nivel* (CANoe, bus, API). Cuando algo cambia, se toca una sola capa.
- **Independencia entre pruebas**: cada prueba prepara su estado inicial y limpia al terminar (*setup/teardown*); no deben depender del orden de ejecución.
- **Esperas con condición y tiempo límite** en lugar de `sleep` fijos (causa clásica de pruebas lentas e intermitentes).
- **Pruebas deterministas**: controlar o registrar semillas, tiempos y condiciones; una prueba intermitente (*flaky*) debe investigarse, no repetirse hasta que pase.
- **Datos separados del código** (*data-driven*): valores de entrada y esperados en tablas o archivos de configuración, con las particiones y fronteras de las técnicas de prueba.
- **Registro de evidencia**: *logging* con marcas de tiempo, versión del software bajo prueba y parámetros de la ejecución; imprescindible para reproducir y para auditorías.
- **Mensajes de fallo útiles**: valor esperado, valor obtenido y contexto.
- **Evitar la duplicación** de lógica de verificación; centralizarla en funciones de aserción propias.
- **Gestión de secretos y rutas**: nada de credenciales ni rutas absolutas fijas en el repositorio.

### 6.4 Control de versiones y revisión
- **Commits pequeños y atómicos**, con mensaje claro y referencia al ticket o requisito.
- **Revisión de código** también para scripts de prueba (son productos de trabajo y evidencia de proceso).
- **Integración continua**: *linters* y pruebas del propio código de prueba en cada cambio.
- **Versionado y trazabilidad**: asociar cada versión del código de prueba con la versión del software y del banco con los que se validó.

### 6.5 Rendimiento y recursos
Medir antes de optimizar. En Python, usar generadores y estructuras adecuadas para logs grandes; en MATLAB, vectorizar y preasignar; en C, priorizar claridad y determinismo, y optimizar solo con evidencia (perfilado).

---

## Ejercicios de práctica
1. **Python:** escribe una función `checksum_xor(datos: bytes) -> int` que calcule el XOR de todos los bytes, y una prueba `pytest` con tres casos (vacío, un byte, varios bytes).
   *Idea:* `functools.reduce(operator.xor, datos, 0)`; vacío → 0.
2. **Python:** dado un archivo de log con líneas `"[12.345] RX 0x123 8 00 7D ..."`, extrae con una expresión regular el tiempo y el ID, y calcula el periodo promedio entre tramas del ID `0x123`.
3. **CAPL:** modifica el ejemplo de monitoreo de timeout para que registre cuánto tiempo estuvo ausente el mensaje cuando éste reaparece. *(Pista: guardar `timeNow()` al expirar el timer y restarlo cuando vuelva a llegar.)*
4. **CAPL:** ¿qué ocurre con un contador `int` que se incrementa cada 10 ms durante 6 minutos? *(36 000 incrementos supera 32 767; se desborda a valores negativos. Usar `long` o `dword`.)*
5. **MATLAB:** dado un vector de latencias en segundos, calcula media, máximo y porcentaje que supera 50 ms sin usar bucles.
   *Idea:* `mean(l)`, `max(l)`, `100*mean(l > 0.05)`.
6. **C:** escribe una macro o función que extraiga un campo de `n` bits a partir del bit `pos` de un `uint32_t`.
   *Idea:* `(x >> pos) & ((1U << n) - 1U)` (con cuidado cuando `n = 32`).
7. **C:** ¿cuál es el tamaño probable de `struct { uint8_t a; uint32_t b; uint8_t c; }` y por qué? *(Con alineación típica de 4 bytes: 12 bytes por relleno; reordenar los campos reduce el tamaño.)*
8. **Revisión de código:** encuentra los problemas en este fragmento Python y corrígelos:
   ```python
   def leer(ids, resultados=[]):
       for i in range(len(ids)):
           try:
               resultados.append(consultar(ids[i]))
           except:
               pass
       return resultados
   ```
   *(Argumento mutable por defecto, `except` desnudo que oculta errores, iterar por índice en lugar de directamente.)*

## Preguntas de teoría
1. ¿Por qué elegirías CAPL, Python o MATLAB para cada una de estas tareas: simular una ECU en el bus, orquestar una regresión nocturna y analizar un log de 2 GB?
2. Explica la diferencia entre tipos mutables e inmutables en Python y da un ejemplo de error por mutabilidad.
3. ¿Qué es un *context manager* y por qué es útil para una sesión de medición?
4. ¿Qué es el GIL y cómo afecta al diseño de una herramienta que registra datos mientras ejecuta pruebas?
5. ¿Para qué sirven las *fixtures* y `parametrize` en pytest y cómo se relacionan con partición de equivalencia y valores frontera?
6. ¿Por qué `time.sleep` fijo es una mala práctica en pruebas y qué lo sustituye?
7. Menciona dos librerías de Python útiles para trabajar con CAN y qué hace cada una.
8. ¿Cuál es el tamaño de `int` en CAPL y qué problema práctico genera?
9. ¿Qué diferencia hay entre `$Señal`, `@VariableDeEntorno` y `@sysvar::…` en CAPL?
10. ¿Por qué los handlers de CAPL deben ser cortos?
11. ¿Cómo se comporta un `timer` de CAPL después de expirar y cómo se logra un envío cíclico?
12. ¿Qué diferencia hay entre `.*` y `*` en MATLAB y por qué la indexación desde 1 genera errores al migrar de Python?
13. ¿Qué es la vectorización y por qué se prefiere a los bucles en MATLAB?
14. Compara MATLAB y Python para análisis de datos de medición en un entorno con CI.
15. ¿Qué hace `volatile` y qué no hace?
16. ¿Por qué usar tipos de ancho fijo y qué riesgos tiene mapear un protocolo con campos de bits?
17. Explica qué es el comportamiento indefinido en C y da tres ejemplos.
18. ¿Cómo implementarías una máquina de estados en C y cómo la probarías con la técnica de transición de estado?
19. ¿Qué características tiene un script de prueba mantenible? Menciona cinco.
20. ¿Cómo estructurarías un framework de pruebas por capas (casos, acciones, drivers) y qué beneficio tiene al cambiar de herramienta?
21. ¿Qué es una prueba *flaky* y cómo la investigarías?
22. ¿Qué verificaciones automáticas añadirías al pipeline para el propio código de prueba?

## Errores comunes
- Describir Python como "solo para scripts": es un lenguaje con tipado, POO, concurrencia y un ecosistema de pruebas completo.
- Escribir `except:` sin tipo ni registro, ocultando fallos reales de la prueba.
- Usar valores por defecto mutables o comparar flotantes con `==`.
- En CAPL, asumir que `int` es de 32 bits o que el timer se repite solo.
- Bloquear un handler de CAPL con esperas largas y perder eventos.
- Confundir señal, variable de entorno y variable de sistema en CAPL.
- Leer bytes de un mensaje sin considerar el orden de bytes de la señal (Intel vs Motorola).
- En MATLAB, usar bucles donde bastaría vectorizar, o mezclar operaciones matriciales y elemento a elemento.
- Olvidar que MATLAB indexa desde 1 al trasladar lógica desde Python o C.
- En C, creer que `volatile` sincroniza o que un `struct` tiene el tamaño de la suma de sus campos.
- Ignorar el comportamiento indefinido porque "en mi compilador funciona".
- Incluir `sleep` fijos y rutas absolutas en las pruebas, lo que las vuelve lentas, frágiles e imposibles de reproducir en CI.
- Repetir una prueba intermitente hasta que pase en lugar de hallar la causa.
- No revisar ni versionar los scripts de prueba con el mismo rigor que el código de producción.
- Elegir un lenguaje por costumbre en vez de por su adecuación a la capa del problema.
