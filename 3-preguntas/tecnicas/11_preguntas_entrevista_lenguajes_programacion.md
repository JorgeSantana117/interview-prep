# Tema: Lenguajes de Programación (C, Python, CAPL, MATLAB)

> Material de estudio base: capítulo `12_lenguajes_programacion.md`. Mismo formato que los bancos anteriores. En este tema es común que te pidan escribir o leer código: practica a mano o en pizarra, sin autocompletado.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Código:* aclara supuestos, escribe una solución simple y correcta, luego prueba con casos límite en voz alta.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "Escribe macros para activar, borrar, invertir y consultar el bit N de un registro de 32 bits."
- *Puntos clave*: `reg |= (1u << n)`; `reg &= ~(1u << n)`; `reg ^= (1u << n)`; `(reg >> n) & 1u`. Usar literales sin signo, paréntesis en macros, comprobar `n < 32`.

**2.** "¿Qué hace `volatile` y cuándo se usa? ¿Basta para compartir datos con una ISR?"
- *Puntos clave*: impide que el compilador optimice accesos (registros de periféricos, variables modificadas en ISR). No garantiza atomicidad ni orden entre núcleos: para datos de varios bytes se necesitan secciones críticas.

**3.** "Diferencia entre `const`, `static` y `volatile`; ¿qué significa `const volatile`?"
- *Puntos clave*: `const` = no modificable por el programa; `static` = duración estática y visibilidad local al archivo o función; `const volatile` = registro de solo lectura que puede cambiar por hardware (ej. registro de estado).

**4.** "Explica punteros a función y un ejemplo de uso en embebido."
- *Puntos clave*: tablas de despacho de una máquina de estados, callbacks de drivers, inyección de dependencias para pruebas. Validar punteros nulos y usar `typedef`.

**5.** "¿Qué problemas hay con tipos enteros y conversiones en C?"
- *Puntos clave*: promoción entera, mezcla de signed/unsigned (comparaciones erróneas), desbordamiento (indefinido en signed), `char` con o sin signo, tamaños dependientes de plataforma: usar `stdint.h` (`uint8_t`, etc.).

**6.** "¿Qué es un desbordamiento de buffer y cómo lo evitas?"
- *Puntos clave*: escritura fuera de límites; usar tamaños explícitos, comprobar longitudes, evitar `strcpy`/`sprintf` sin límite, análisis estático, MISRA.

**7.** "Gestión de memoria dinámica en embebido: ¿por qué se evita?"
- *Puntos clave*: fragmentación, no determinismo, fallos de asignación en campo; alternativas: pools estáticos, asignación en arranque. MISRA lo prohíbe.

**8.** "¿Qué hace `static inline` frente a una macro y cuándo prefieres cada una?"
- *Puntos clave*: la función inline tiene verificación de tipos y evita doble evaluación de argumentos; las macros son útiles para constantes y compilación condicional pero propensas a errores.

---

## B. Test Engineer

**1.** "Escribe una función en Python que lea un archivo de log de CAN y cuente mensajes por ID."
- *Puntos clave*: leer línea a línea, parsear con split/regex, `collections.Counter` o `defaultdict(int)`, manejar líneas inválidas y archivos grandes (iteración, no cargar todo). Mostrar pruebas.

**2.** "Diferencias entre lista, tupla, conjunto y diccionario en Python."
- *Puntos clave*: lista mutable ordenada; tupla inmutable; conjunto sin duplicados y búsqueda O(1) promedio; diccionario clave-valor. Elegir por mutabilidad, orden y búsqueda.

**3.** "¿Qué imprime este código? Argumento mutable por defecto: `def f(x, l=[]): l.append(x); return l`."
- *Puntos clave*: el valor por defecto se evalúa una sola vez y se comparte entre llamadas: `f(1)` → `[1]`, `f(2)` → `[1, 2]`. Solución: `l=None` y crearlo dentro.

**4.** "Explica generadores, decoradores y context managers y un uso en pruebas."
- *Puntos clave*: generador = iteración perezosa (leer logs grandes); decorador = envolver funciones (reintentos, temporización); `with` = recursos con limpieza (abrir/cerrar conexión a CANoe).

**5.** "Escribe un caso de prueba en CAPL que verifique que un mensaje cíclico llega cada 100 ms ± 10 %."
- *Puntos clave*: variables de tiempo, `on message` guarda `timeNow()`, calcula diferencias, `testStepPass/Fail` según rango, tamaño de muestra, manejo del primer mensaje; separar medición de evaluación.

**6.** "Cuidado con tipos en CAPL: ¿qué errores comunes hay?"
- *Puntos clave*: `int` de 16 bits en CAPL, `dword`/`long` para tiempos, división entera, comparaciones signed/unsigned, `timer` vs `msTimer`, olvidar `cancelTimer`, uso de `this` en handlers.

**7.** "¿Cómo estructuras código de pruebas mantenible?"
- *Puntos clave*: funciones pequeñas, nombres descriptivos, datos separados de lógica, sin duplicación, parametrización (pytest), fixtures, esperas con condición, logs claros, revisión de código y control de versiones.

**8.** "Con pytest: fixtures, parametrización y marcadores. ¿Cuándo los usas?"
- *Puntos clave*: fixtures para preparar/limpiar el banco; `@pytest.mark.parametrize` para variar datos; marcadores para seleccionar suites (smoke, regresión); `conftest.py` compartido.

**9.** "¿Para qué usas MATLAB en pruebas?"
- *Puntos clave*: análisis de señales y gráficos, procesamiento de mediciones, generación de estímulos, ejecución de modelos Simulink; comparación con Python para scripting y CI (licencia vs gratuito).

---

## C. Embedded Software Test Function Lead

**1.** "¿Cómo eliges el lenguaje para un nuevo framework de pruebas?"
- *Puntos clave*: integración con herramientas (CANoe: CAPL/COM; CI: Python), habilidades del equipo, ecosistema, mantenimiento, licencias, rendimiento. Python como capa de orquestación y CAPL para lógica cercana al bus.

**2.** "¿Qué estándares de codificación impones para scripts de prueba?"
- *Puntos clave*: PEP 8 con linters (flake8/ruff), formato automático, tipado gradual, docstrings, revisión de código, estructura común del repositorio, pruebas de las librerías internas.

**3.** "¿Cómo gestionas dependencias y entornos en Python en un equipo?"
- *Puntos clave*: entornos virtuales, `requirements.txt` con versiones fijadas o lock files, contenedores Docker en CI, versión de Python definida, reproducibilidad.

**4.** "¿Cómo evitas que el código de pruebas se vuelva ilegible o frágil?"
- *Puntos clave*: capas de abstracción, revisión obligatoria, refactor periódico, métricas de duplicación, documentación mínima, retirada de pruebas obsoletas.

**5.** "Un ingeniero junior escribe scripts con esperas fijas (`sleep`). ¿Cómo lo guías?"
- *Puntos clave*: explicar el costo en tiempo y flakiness, mostrar esperas con condición y timeout, revisión en pareja, ejemplo en la plantilla del equipo.

---

## D. Test Manager

**1.** "¿Cómo evalúas la capacidad de programación de un candidato de pruebas?"
- *Puntos clave*: ejercicio práctico corto y realista (parsear un log, escribir una prueba), lectura de código con errores, criterios de claridad y corrección más que memorización.

**2.** "¿Cómo equilibras generalistas y especialistas en el equipo?"
- *Puntos clave*: núcleo de ingenieros con buena programación para el framework, especialistas en CAPL/herramientas, formación cruzada y plan de sucesión.

**3.** "¿Cómo justificas tiempo para refactorizar código de pruebas?"
- *Puntos clave*: medir costo de mantenimiento y flakiness, tiempo de depuración de scripts, impacto en ciclos de regresión; propuesta acotada con resultados esperados.

**4.** "¿Qué riesgos tiene depender de scripts personales sin estándares?"
- *Puntos clave*: dependencia de personas, difícil mantenimiento y auditoría, resultados no reproducibles; mitigación con repositorio central, revisión y documentación.

---

## Preguntas transversales (cierre o "curveball")

**1.** "Dame un bug sutil de C que hayas visto o conozcas."
- *Puntos clave*: comparación signed/unsigned, falta de `volatile`, desbordamiento de pila o de índice, `=` en lugar de `==`, uso de variable no inicializada; di cómo lo detectaste (análisis estático, trazas).

**2.** "¿Qué haces si tu script funciona en tu máquina pero falla en el servidor de CI?"
- *Puntos clave*: diferencias de entorno (versiones, rutas, variables, permisos, hardware), reproducir con el mismo contenedor, registrar versiones, eliminar dependencias implícitas.

**3.** "Cuéntame de una herramienta o script que escribiste y que ahorró trabajo al equipo."
- *Puntos clave*: ejemplo STAR: problema, solución técnica, adopción por el equipo, resultado medible. Se profundiza en el Módulo 3.
