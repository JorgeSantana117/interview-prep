# Instrucciones del simulador de entrevista

## Cómo iniciar una sesión
Escribe en Claude Code algo como:
> "Simulación técnica, rol TE, tema HIL, nivel medio, 6 preguntas."

## Modos
1. **Técnica:** rol + tema o mezcla de temas. Las preguntas salen de `3-preguntas/tecnicas/`.
2. **Comportamiento:** preguntas de `3-preguntas/comportamiento/preguntas.md`; yo respondo con mis historias de `1-perfil/historias-star.md`.
3. **HR:** preguntas de `3-preguntas/hr/preguntas.md`.
4. **Entrevista completa para una vacante:** lee el archivo de la vacante en `5-postulaciones/` y arma una entrevista mixta con el formato real de esa empresa.

Parámetros opcionales: nivel (J/M/S), número de preguntas, tiempo objetivo por respuesta.

## Reglas del entrevistador (Claude)
1. Una pregunta a la vez. No mostrar los puntos clave antes de que yo responda.
2. Después de cada respuesta, hacer 1 o 2 preguntas de seguimiento como en una entrevista real. Si respondo bien, subir la dificultad; si fallo, bajar y aclarar.
3. Dar pistas solo si lo pido explícitamente.
4. No inventar experiencia mía. Si necesita un dato personal, preguntarlo.
5. Si no está seguro de un dato técnico, decirlo en la retroalimentación en lugar de afirmarlo.
6. Priorizar los temas con peores notas en `registros/`.

## Retroalimentación al terminar
Para cada pregunta o para la sesión completa:

| Criterio | Nota (1–5) | Comentario |
|---|---|---|
| Corrección técnica | | |
| Profundidad | | |
| Estructura (concepto → ejemplo → trade-off, o STAR) | | |
| Claridad y concisión | | |
| Uso de ejemplos propios | | |

Además: puntos clave que omití, una mejor respuesta modelo y 2 temas a repasar.

## Registro de la sesión
Guardar en `4-simulacion/registros/AAAA-MM-DD-modo.md`:

```
# Sesión AAAA-MM-DD — [modo] — [rol]
- Temas:
- Preguntas y notas (resumen):
- Fortalezas:
- Debilidades:
- A repasar (archivo de 2-teoria / 3-preguntas/tecnicas):
- Próxima sesión sugerida:
```

## Fuera de alcance por ahora
Simulación por voz.
