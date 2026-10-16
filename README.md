# Preparación de Entrevistas — Jorge Santana Mendoza

Material de estudio en Markdown para entrevistas en ingeniería de software embebido automotriz. Más adelante se convertirá en una aplicación.

**Roles objetivo:** Embedded Software Engineer (ESE), Test Engineer (TE), Embedded Software Test Function Lead (TFL), Test Manager (TM).

## Estructura

```
interview-prep/
├── README.md                          ← índice y estado del proyecto
├── CLAUDE.md                          ← contexto para Claude en VS Code
│
├── 1-perfil/                          ← materia prima: quién soy
│   ├── JorgeSantana_Resume.pdf
│   ├── presentacion.md                ← "háblame de ti", motivación, preguntas que yo hago
│   ├── historias-star.md              ← mis historias reales (Bosch, ALTEN, VW, Ford)
│   └── datos-hr.md                    ← sueldo, prestaciones, disponibilidad, modalidad
│
├── 2-teoria/                          ← estudiar (14 capítulos)
│   └── 01_protocolos_comunicacion.md … 14_devops_computo_nube.md
│
├── 3-preguntas/                       ← lo que me van a preguntar, por tipo de ronda
│   ├── tecnicas/                      ← 14 bancos; mismo número y tema que 2-teoria
│   │   └── 01_preguntas_entrevista_protocolos_comunicacion.md … 14_preguntas_entrevista_devops_nube.md
│   ├── comportamiento/
│   │   ├── guia-star.md
│   │   └── preguntas.md
│   └── hr/
│       └── preguntas.md
│
├── 4-simulacion/
│   ├── instrucciones.md               ← reglas del simulador y rúbrica
│   └── registros/                     ← una nota por sesión
│
├── 5-postulaciones/
│   ├── _historial.md                  ← tabla resumen
│   ├── _plantilla.md                  ← plantilla por vacante
│   ├── preguntas-reales.md            ← las que sí me han hecho
│   └── <empresa>-<puesto>.md
│
└── referencias/                       ← notas de cursos y material externo (solo lectura)
```

## Temas (teoría y preguntas técnicas comparten número y tema)

| N.º | Teoría (`2-teoria/`) | Preguntas técnicas (`3-preguntas/tecnicas/`) | Tema |
|---|---|---|---|
| 01 | 01_protocolos_comunicacion.md | 01_preguntas_entrevista_protocolos_comunicacion.md | Protocolos de comunicación (CAN, CAN FD, LIN, CAN-TP, UDS, XCP, Ethernet) |
| 02 | 02_hil_testing.md | 02_preguntas_entrevista_hil_testing.md | HIL Testing (MIL, SIL, PIL, HIL, planta, inyección de fallas) |
| 03 | 03_fundamentos_software_testing.md | 03_preguntas_entrevista_fundamentos_testing.md | Fundamentos de Software Testing (ISTQB CTFL 4.0) |
| 04 | 04_herramientas_testing_automatizacion.md | 04_preguntas_entrevista_herramientas_testing.md | Herramientas de testing y automatización (CANoe, VT System, CAPL, vTESTstudio, Robot Framework) |
| 05 | 05_requirements_engineering.md | 05_preguntas_entrevista_requirements_engineering.md | Requirements Engineering |
| 06 | 06_aspice_autosar.md | 06_preguntas_entrevista_aspice_autosar.md | ASPICE y AUTOSAR |
| 07 | 07_ingenieria_de_sistemas.md | 07_preguntas_entrevista_ingenieria_sistemas.md | Ingeniería de Sistemas |
| 08 | 08_ingenieria_de_software.md | 08_preguntas_entrevista_ingenieria_software.md | Ingeniería de Software |
| 09 | 09_sistemas_embebidos.md | 09_preguntas_entrevista_sistemas_embebidos.md | Sistemas Embebidos |
| 10 | 10_sistemas_operativos_embebidos.md | 10_preguntas_entrevista_so_embebidos.md | Sistemas Operativos Embebidos (RTOS, OSEK, Linux, QNX) |
| 11 | 11_lenguajes_programacion.md | 11_preguntas_entrevista_lenguajes_programacion.md | Lenguajes de Programación (C, Python, CAPL, MATLAB) |
| 12 | 12_diseno_basado_en_modelos.md | 12_preguntas_entrevista_diseno_basado_en_modelos.md | Diseño Basado en Modelos (Simulink, Stateflow) |
| 13 | 13_cicd_control_versiones.md | 13_preguntas_entrevista_cicd_control_versiones.md | CI/CD y Control de Versiones (Git, Gerrit, Jenkins, Docker) |
| 14 | 14_devops_computo_nube.md | 14_preguntas_entrevista_devops_nube.md | DevOps y Cómputo en la Nube |
