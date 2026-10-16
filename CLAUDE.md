# CLAUDE.md — Proyecto de preparación de entrevistas

## Propósito
Material de estudio en Markdown para que Jorge Santana Mendoza prepare entrevistas en ingeniería de software embebido automotriz. Más adelante se convertirá en una aplicación, por lo que la estructura y el formato deben ser consistentes.

## Perfil del usuario (resumen del CV)
- Ingeniero de software embebido. Experiencia: Bosch (Embedded SW Engineer, Embedded SW Test Function Lead, Embedded SW Test Engineer) y ALTEN México (proyectos para Volkswagen y Ford).
- Maestría en IA Aplicada (Tec de Monterrey).
- Herramientas y tecnologías: CANoe, vTESTstudio, Robot Framework, Linux/QNX, Python, CAPL, Docker, Jenkins, Git/Gerrit/Bitbucket, CAN/CAN-TP/CAN FD/XCP/Ethernet/UDS, Simulink/Stateflow, Jira/DOORS/RQM/Confluence; conocimientos de SDLC, ISTQB, ASPICE y AUTOSAR.
- El CV completo está en `1-perfil/JorgeSantana_Resume.pdf`. Si falta un dato, no lo inventes: pregunta o deja un marcador `[COMPLETAR]`.

## Roles objetivo
ESE (Embedded Software Engineer), TE (Test Engineer), TFL (Embedded Software Test Function Lead), TM (Test Manager).

## Estructura del repositorio
- `1-perfil/` — `JorgeSantana_Resume.pdf`, `presentacion.md`, `historias-star.md`, `datos-hr.md`. Es la fuente de verdad sobre el usuario.
- `2-teoria/NN-tema.md` — 14 capítulos de teoría.
- `3-preguntas/tecnicas/NN-tema.md` — 14 bancos de preguntas; mismo número y nombre que la teoría.
- `3-preguntas/comportamiento/` — `guia-star.md` y `preguntas.md`.
- `3-preguntas/hr/preguntas.md` — cómo responder las preguntas de HR; los datos están en `1-perfil/datos-hr.md`.
- `4-simulacion/` — `instrucciones.md` y `registros/` (una nota por sesión).
- `5-postulaciones/` — `_historial.md`, `_plantilla.md`, `preguntas-reales.md` y un archivo por vacante.
- `referencias/` — notas de cursos y material externo (solo lectura).
- `README.md` — índice y estado. Actualízalo cuando cambie algo.

## Formato obligatorio de los bancos de preguntas técnicas (`3-preguntas/tecnicas`)
1. Título: `# Tema: <nombre>`.
2. Nota inicial con el capítulo de teoría base (en `2-teoria/`) y una guía de cómo responder (concepto / escenario / experiencia con STAR).
3. Secciones por rol, en este orden: `## A. Embedded Software Engineer`, `## B. Test Engineer`, `## C. Embedded Software Test Function Lead`, `## D. Test Manager`.
4. Preguntas numeradas dentro de cada sección, la pregunta entre comillas y una línea `*Puntos clave*:`.
5. `## Preguntas transversales (cierre o "curveball")` con 3 preguntas; la última siempre de experiencia personal (STAR).
Numeración consecutiva, sin saltos, dentro de cada sección.

## Formato de la teoría (`2-teoria`)
Secciones numeradas sin saltos ni duplicados; al final, preguntas de teoría, y errores comunes. Evitar duplicar contenido entre capítulos: usar referencias cruzadas.

## Reglas para `1-perfil` y `3-preguntas/comportamiento`
- Las historias se construyen solo con hechos que el usuario cuente o que estén en su CV. Marca con `[COMPLETAR]` lo que falte (números, fechas, nombres). No inventes métricas.
- Cada historia sigue STAR: Situación, Tarea, Acción (primera persona, "yo"), Resultado (medible si es posible) y aprendizaje.
- Etiqueta cada historia con las competencias que demuestra y con los roles donde sirve.
- Las preguntas de comportamiento remiten a historias por número (H1, H2…).

## Reglas para `3-preguntas/hr` y `1-perfil/datos-hr.md`
- Los datos personales (sueldo, prestaciones, disponibilidad) los escribe el usuario en `datos-hr.md`; no los inventes ni los repitas en otros archivos.
- Recomendar veracidad en los datos actuales y un rango como expectativa salarial.
- Tratar estos datos como sensibles: no incluirlos en resúmenes ni en archivos fuera de `1-perfil/`.

## Reglas del simulador (`4-simulacion`)
- Una pregunta a la vez; no mostrar los puntos clave antes de la respuesta; hacer preguntas de seguimiento como en una entrevista real.
- Retroalimentación al final con criterios fijos: corrección técnica, profundidad, estructura, claridad y uso de ejemplos propios. Nota de 1 a 5 por criterio y puntos clave omitidos.
- Guardar un registro en `4-simulacion/registros/AAAA-MM-DD-modo.md`. La sesión siguiente empieza leyendo los registros recientes y prioriza los temas débiles.
- Las preguntas técnicas salen de `3-preguntas/tecnicas`; las de comportamiento y HR, de sus carpetas; la entrevista completa para una vacante se arma desde su archivo en `5-postulaciones`.

## Reglas para `5-postulaciones`
- Un archivo por vacante a partir de `_plantilla.md`. Después de cada ronda, registrar las preguntas que hicieron en el archivo de la vacante y copiarlas a `preguntas-reales.md`.
- Mantener `_historial.md` actualizado (empresa, puesto, etapa, resultado, fecha).

## Reglas de trabajo generales
- Idioma: español; términos técnicos en inglés cuando son los usados en la industria.
- Rigor técnico: si no estás seguro de un dato (versiones, comandos, normas), dilo y pide verificarlo. No lo presentes como seguro.
- Trabaja un tema o un archivo por sesión; busca encabezados y lee solo lo necesario en lugar de releer capítulos enteros.
- No reproduzcas materiales protegidos; los apuntes de cursos del usuario se pueden usar como fuente con redacción propia.
- Antes de borrar o renombrar archivos, confírmalo. Antes de reescribir un archivo, revisa `git diff` y no reviertas cambios del usuario.
- Al terminar, resume en una o dos frases lo hecho y propón un único siguiente paso.
