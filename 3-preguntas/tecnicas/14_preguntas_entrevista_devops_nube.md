# Tema: DevOps y Cómputo en la Nube

> Material de estudio base: capítulo `14_devops_computo_nube.md`. Mismo formato que los bancos anteriores. Aquí conecta tu experiencia en Docker/Jenkins con la nube, el vehículo definido por software (SDV) y tu maestría en IA (MLOps).

> Estructura recomendada para responder:
> - *Concepto:* definición breve → por qué importa → ejemplo propio → limitación.
> - *Escenario:* síntoma → hipótesis → cómo confirmas → acción y evidencia.
> - *Experiencia:* método STAR (Módulo 3).

---

## A. Embedded Software Engineer

**1.** "¿Qué es DevOps y qué no es?"
- *Puntos clave*: cultura y prácticas que unen desarrollo y operaciones para entregar valor de forma rápida y fiable (automatización, colaboración, retroalimentación). No es un rol, ni una herramienta, ni solo CI/CD.

**2.** "¿Qué prácticas de DevOps se adaptan bien al software embebido y cuáles no?"
- *Puntos clave*: sí: CI, automatización de pruebas, IaC para entornos de build, SIL en la nube, versionado, observabilidad de flota. Limitaciones: hardware físico, ciclos de validación y homologación, seguridad funcional, actualizaciones controladas del vehículo.

**3.** "¿Qué es un vehículo definido por software (SDV) y qué implica para el desarrollo?"
- *Puntos clave*: funciones definidas por software, actualizables en el ciclo de vida, arquitecturas centralizadas/zonales, servicios y APIs, desarrollo continuo; implica OTA, ciberseguridad, desacoplo de hardware/software y pruebas virtuales.

**4.** "Describe cómo funciona una actualización OTA segura."
- *Puntos clave*: paquete firmado, verificación en la ECU, descarga cifrada, instalación con *rollback* (A/B), condiciones de seguridad (vehículo estacionado, batería), informe de estado. Marcos como Uptane; normativa UN R156.

**5.** "¿Qué diferencia hay entre contenedores y máquinas virtuales?"
- *Puntos clave*: contenedores comparten el kernel del anfitrión (ligeros, arranque rápido, menos aislamiento); VMs incluyen SO completo (mayor aislamiento y costo). Elegir según aislamiento y rendimiento.

---

## B. Test Engineer

**1.** "¿Qué puede moverse a la nube en las pruebas automotrices y qué no?"
- *Puntos clave*: sí: SIL, vECU, pruebas unitarias, análisis estático, simulación a gran escala, almacenamiento de resultados. No (por ahora): HIL con hardware real, pruebas eléctricas, partes con latencia o determinismo estricto.

**2.** "¿Cómo aprovechas la elasticidad de la nube en pruebas?"
- *Puntos clave*: ejecutar miles de escenarios en paralelo con instancias efímeras, pagar solo por uso, reducir el tiempo de regresión; controlar costos con apagado automático y cuotas.

**3.** "¿Qué es infraestructura como código y qué ventajas tiene para pruebas?"
- *Puntos clave*: definir entornos con archivos versionados (Terraform, Ansible, Dockerfile); reproducibilidad, revisión, rapidez de recreación, menos configuración manual y trazabilidad de cambios.

**4.** "¿Cómo pruebas la comunicación vehículo ↔ nube?"
- *Puntos clave*: telemetría, comandos remotos, OTA; casos de conectividad intermitente, latencia, autenticación, reintentos, datos corruptos, modos degradados y seguridad (certificados, TLS).

**5.** "¿Cómo manejas datos de flota para análisis de pruebas?"
- *Puntos clave*: ingesta y almacenamiento (objetos), anonimización y privacidad (GDPR), calidad de datos, análisis para reproducir fallas en SIL/HIL y alimentar nuevos casos de regresión.

**6.** "¿Qué es el monitoreo y la observabilidad y cómo ayudan al probador?"
- *Puntos clave*: métricas, logs y trazas para entender el comportamiento del sistema en ejecución; permiten detectar degradaciones y reproducir fallos de campo.

---

## C. Embedded Software Test Function Lead

**1.** "Explica las métricas DORA y cómo las usarías."
- *Puntos clave*: frecuencia de despliegue, tiempo de entrega de cambios, tasa de fallo de cambios, tiempo de recuperación. Usarlas como tendencia para mejorar el proceso, no para presionar a personas.

**2.** "¿Cómo diseñas una estrategia híbrida de pruebas entre nube y bancos físicos?"
- *Puntos clave*: SIL/vECU en la nube para volumen y retroalimentación rápida; HIL para integración y fidelidad; criterios de promoción entre niveles; trazabilidad unificada de resultados.

**3.** "¿Qué consideraciones de ciberseguridad aplicas a pruebas y datos en la nube?"
- *Puntos clave*: responsabilidad compartida, control de acceso mínimo, cifrado en tránsito y reposo, gestión de secretos, segregación por cliente, cumplimiento de requisitos del OEM (TISAX), registros de auditoría.

**4.** "¿Cómo introducirías DevSecOps en el flujo de pruebas?"
- *Puntos clave*: escaneo de dependencias (SCA), análisis estático de seguridad (SAST), pruebas dinámicas y fuzzing, SBOM, firma de artefactos, revisión de amenazas (TARA); controles automáticos en el pipeline.

**5.** "Estrategias de liberación: blue-green, canary, feature flags. ¿Cuáles aplican a flotas de vehículos?"
- *Puntos clave*: despliegue gradual a subconjuntos de vehículos (canary), banderas para activar funciones remotamente, rollback seguro; limitado por seguridad funcional y regulaciones.

**6.** "¿Qué entiendes por ingeniería del caos y dónde aplicaría?"
- *Puntos clave*: introducir fallos controlados (caída de servicio, latencia, pérdida de red) para verificar resiliencia; en backends de flota y servicios; en vehículo se parece a la inyección de fallas en HIL.

---

## D. Test Manager

**1.** "¿Cómo gestionas costos de la nube (FinOps) en un equipo de pruebas?"
- *Puntos clave*: etiquetado de recursos, presupuestos y alertas, apagado automático, instancias reservadas o spot para cargas tolerantes, medir costo por ejecución, revisión periódica.

**2.** "¿Cómo justificas migrar pruebas SIL a la nube?"
- *Puntos clave*: ahorro de tiempo (paralelismo), menor inversión en hardware, escalabilidad; riesgos: costo variable, seguridad y datos, dependencia del proveedor; piloto con métricas.

**3.** "¿Qué es el modelo de responsabilidad compartida?"
- *Puntos clave*: el proveedor protege la infraestructura; el cliente protege sus datos, accesos, configuraciones y aplicaciones. Malentenderlo causa brechas.

**4.** "¿Cómo defines objetivos de fiabilidad (SLI/SLO/SLA) para servicios de pruebas o de flota?"
- *Puntos clave*: SLI = medida (disponibilidad, latencia); SLO = objetivo interno; SLA = compromiso contractual; presupuesto de error para equilibrar velocidad y estabilidad.

**5.** "¿Qué riesgos tiene la dependencia de un proveedor de nube y cómo los mitigas?"
- *Puntos clave*: dependencia (lock-in), disponibilidad, cambios de precio; mitigación con contenedores y estándares abiertos, arquitectura portable, respaldos y plan de salida.

---

## Preguntas transversales (cierre o "curveball")

**1.** "Explica MLOps y su relación con DevOps. ¿Cómo se conecta con tu maestría?"
- *Puntos clave*: aplica prácticas DevOps al ciclo de vida de modelos de ML (datos, entrenamiento, validación, despliegue, monitoreo y reentrenamiento); añade versionado de datos/modelos y detección de deriva. Conecta con proyectos de IA aplicada y validación de funciones basadas en datos.

**2.** "¿Por qué no todo el software automotriz puede aplicar 'despliega varias veces al día'?"
- *Puntos clave*: seguridad funcional, homologación, hardware físico, riesgo para el usuario, ciclos de validación; se adapta con CI continuo y liberaciones controladas.

**3.** "Cuéntame cómo la automatización o la nube cambió tu forma de trabajar."
- *Puntos clave*: ejemplo STAR con antes/después, tu aportación, resultado medible y aprendizaje. Se profundiza en el Módulo 3.
