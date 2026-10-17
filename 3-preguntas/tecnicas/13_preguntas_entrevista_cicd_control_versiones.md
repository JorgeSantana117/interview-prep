# Tema: CI/CD y Control de Versiones (Git, Gerrit, Bitbucket, Jenkins, Docker)

> Material de estudio base: capítulo `13_cicd_control_versiones.md`. Mismo formato que los bancos anteriores. Este es un tema donde tu CV es sólido (Docker, Jenkins, Git/Gerrit/Bitbucket): lleva ejemplos reales de pipelines y de problemas que resolviste.

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* síntoma → hipótesis (de la más barata de descartar a la más cara) → cómo confirmas → acción y evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "Explica el modelo mental de Git: working tree, staging area, commits y ramas."
- *Puntos clave*: los commits son instantáneas inmutables enlazadas en un grafo; las ramas son punteros a commits; `HEAD` apunta a la posición actual; el staging prepara qué entra al commit. Entender esto evita miedo a `reset`/`rebase`.

**2.** "Merge vs rebase: diferencias y cuándo usar cada uno."
- *Puntos clave*: merge conserva la historia real con un commit de unión; rebase reescribe commits sobre otra base para una historia lineal. Nunca reescribir historia ya compartida; en Gerrit se suele usar rebase y un commit por cambio.

**3.** "¿Cómo resuelves un conflicto de merge de forma segura?"
- *Puntos clave*: entender ambos cambios, hablar con el autor si hay duda, resolver, compilar y ejecutar pruebas, no aceptar ciegamente "ours/theirs", revisar el diff final.

**4.** "Buenas prácticas de commits."
- *Puntos clave*: commits pequeños y atómicos, mensaje con asunto imperativo y motivo, referencia al ticket/requisito, no mezclar refactor con funcionalidad, no subir binarios o secretos.

**5.** "Describe el flujo de revisión de código en Gerrit."
- *Puntos clave*: subir un cambio (`refs/for/<rama>`), Change-Id, revisores, votos Code-Review (±2) y Verified, CI como verificador, parches sucesivos (patch sets), submit tras aprobación. Distinto de PR en GitHub: se revisa cada commit.

**6.** "Un compañero hizo push de un secreto (contraseña) al repositorio. ¿Qué haces?"
- *Puntos clave*: revocar y rotar la credencial de inmediato, retirar del historial (con herramientas adecuadas) si se puede, avisar al responsable de seguridad, añadir detección previa (hooks, escáner). Retirar el commit no basta.

---

## B. Test Engineer

**1.** "¿Cómo usas `git bisect` para encontrar el commit que introdujo una regresión?"
- *Puntos clave*: marcar un commit malo (`git bisect bad`) y uno bueno, Git hace búsqueda binaria; se prueba cada commit (manual o con script `git bisect run`); en N commits requiere ~log₂N pasos. Requiere una prueba determinista.

**2.** "¿Qué debe versionarse junto con los scripts de prueba?"
- *Puntos clave*: DBC/ARXML/A2L, configuraciones de CANoe, modelos, datos de prueba, versiones de dependencias, scripts de CI, documentación de uso. Todo lo que influye en el resultado debe ser reproducible.

**3.** "Un pipeline falla, pero localmente la prueba pasa. ¿Cómo investigas?"
- *Puntos clave*: diferencias de entorno (versiones, rutas, variables, permisos), dependencias implícitas, datos o estado compartido, tiempos; reproducir con el mismo contenedor o agente; revisar el log completo.

**4.** "¿Cómo integras pruebas automáticas en un pipeline de Jenkins?"
- *Puntos clave*: stage de pruebas con comandos de Robot/pytest, publicación de resultados (JUnit/xUnit, plugin de Robot), archivado de logs y trazas, condiciones de fallo, notificaciones, paralelismo.

**5.** "¿Qué ventajas aporta Docker a las pruebas y cuáles son sus límites en tu contexto?"
- *Puntos clave*: entorno reproducible, aislamiento, versiones fijadas, rapidez de creación. Límites: acceso a hardware (interfaces CAN/USB, VT System), herramientas con licencias o solo Windows (CANoe), uso de drivers; para banco físico se usan agentes dedicados.

**6.** "¿Cómo gestionas archivos grandes o binarios en Git?"
- *Puntos clave*: Git LFS o un repositorio de artefactos (Artifactory/Nexus), no versionar artefactos de build, referencias por versión/hash.

---

## C. Embedded Software Test Function Lead

**1.** "Diseña un pipeline CI/CD para software embebido con SIL y HIL."
- *Puntos clave*: commit → build → análisis estático → pruebas unitarias → SIL/vECU → empaquetado de artefactos → HIL en bancos reservados → reportes y trazabilidad → liberación. Las etapas rápidas primero (fail fast), las caras (HIL) después y con selección por riesgo o nocturnas.

**2.** "¿Cómo manejas bancos HIL como recursos en Jenkins?"
- *Puntos clave*: agentes con etiquetas por tipo de banco, bloqueo de recurso (lockable resources) para evitar uso simultáneo, verificación de salud antes de ejecutar, reintentos controlados, limpieza al final, métricas de ocupación.

**3.** "¿Qué estrategia de ramificación recomiendas para un programa con varios clientes/variantes?"
- *Puntos clave*: trunk-based con ramas cortas y feature flags para integración continua; ramas de release para estabilizar y mantener; cherry-pick controlado de correcciones; evitar ramas de larga duración por variante cuando sea posible.

**4.** "Un pipeline tarda 5 horas. ¿Cómo lo reduces?"
- *Puntos clave*: perfilar etapas, paralelizar, cachear dependencias y capas Docker, pruebas incrementales según el cambio, mover pruebas a niveles más bajos, ejecución nocturna de las más costosas, más agentes.

**5.** "¿Cómo garantizas la trazabilidad entre un cambio, su revisión, su build y sus resultados?"
- *Puntos clave*: ID de ticket en el commit, Change-Id/PR, número de build, hash del commit en los artefactos y reportes, almacenamiento de resultados enlazados; auditoría ASPICE.

**6.** "¿Qué métricas de CI/CD sigues?"
- *Puntos clave*: frecuencia de integración, duración del pipeline, tasa de fallos y de flakiness, tiempo de recuperación, tiempo desde commit hasta resultado, lead time; interpretarlas como tendencia (DORA).

---

## D. Test Manager

**1.** "¿Cómo justificas la inversión en infraestructura de CI/CD?"
- *Puntos clave*: ciclos de retroalimentación más cortos, menos defectos tardíos, regresión automática, menos trabajo manual, trazabilidad; costos de servidores, licencias y mantenimiento; piloto con métricas.

**2.** "Los desarrolladores se quejan de que el pipeline es lento o inestable. ¿Qué haces?"
- *Puntos clave*: medir y mostrar datos, priorizar mejoras con el mayor retorno, tratar la flakiness como deuda, compromisos de servicio, comunicar avances.

**3.** "¿Cómo gestionas el acceso y la seguridad del repositorio y del pipeline?"
- *Puntos clave*: permisos por rol, ramas protegidas, revisión obligatoria, gestión de secretos fuera del código, auditoría, firma de artefactos, cumplimiento de ciberseguridad (ISO 21434).

**4.** "Un cliente quiere acceso a tu pipeline. ¿Qué consideras?"
- *Puntos clave*: separación de datos de otros clientes, permisos mínimos, propiedad intelectual, acuerdos contractuales, vistas de solo lectura de resultados.

---

## Preguntas transversales (cierre o "curveball")

**1.** "¿Qué es CI y en qué se diferencia de CD (entrega vs despliegue continuo)?"
- *Puntos clave*: CI integra y verifica cada cambio automáticamente; entrega continua mantiene el software siempre liberable; despliegue continuo lo despliega automáticamente. En automotriz, la liberación suele ser controlada por hitos y seguridad.

**2.** "Dime cómo revertirías un cambio ya integrado que rompe la regresión."
- *Puntos clave*: `git revert` (nuevo commit que lo deshace, seguro en historia compartida) frente a `reset` (reescribe); identificar el commit con bisect; comunicar y abrir un ticket para corregir bien.

**3.** "Cuéntame de un pipeline que construiste o mejoraste."
- *Puntos clave*: ejemplo STAR: situación, qué automatizaste (Docker, Jenkins), resultado medible (tiempo, estabilidad), aprendizaje. Se profundiza en el Módulo 3.

