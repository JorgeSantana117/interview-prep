# Capítulo: DevOps y Cómputo en la Nube

> Objetivo del capítulo: completar tu perfil con la **cultura y las prácticas DevOps** y con los **conceptos de nube** que hoy atraviesan la industria automotriz (vehículo definido por software, OTA, datos de flota, simulación a escala). No están en tu CV, así que el enfoque es **conceptual y conectado con lo que ya haces**: en entrevista conviene decir con honestidad "mi experiencia directa es en CI/CD con Jenkins y Docker, y conozco estos conceptos de nube" en lugar de aparentar experiencia práctica que no tienes. Aun así, entender el panorama te posiciona para roles con más exposición a infraestructura.
>
> **Límites con otros capítulos (para no repetir):**
> - Git, Gerrit, Jenkins, Docker y diseño de pipelines → ya en *CI/CD y Control de Versiones*. Aquí: la cultura y las prácticas que los rodean, y lo que viene después (nube, orquestación, observabilidad).
> - Shift-left, IC/EC como concepto y DevOps desde el punto de vista de la prueba → ya en *Fundamentos de Software Testing*.
> - SOME/IP, Automotive Ethernet y arquitecturas zonales → ya en *Protocolos* e *Ingeniería de Sistemas*. Aquí: la conexión vehículo ↔ nube.
> - Ciberseguridad como requisito de proceso (ASPICE for Cybersecurity) → mencionada en *ASPICE/AUTOSAR*. Aquí: seguridad en el pipeline y en la nube.

---

## 1. DevOps: qué es y qué no es

**DevOps** es un enfoque cultural y técnico que busca **acortar el ciclo entre una idea y su operación en producción**, uniendo desarrollo (incluida la prueba) y operaciones con responsabilidad compartida, automatización y **retroalimentación rápida**. No es un rol ni una herramienta: las herramientas (Jenkins, Docker, etc.) son un medio.

### 1.1 Relación entre DevOps y CI/CD
| Concepto | Qué es | Alcance |
|---|---|---|
| **CI/CD** | Conjunto de **prácticas técnicas** de integración, verificación y entrega automatizadas | El pipeline |
| **DevOps** | **Cultura + prácticas + herramientas** que abarcan todo el ciclo: planeación, desarrollo, pruebas, entrega, operación y monitoreo | Toda la organización |

CI/CD es una parte esencial de DevOps, pero DevOps incluye además colaboración, propiedad compartida, infraestructura como código, observabilidad, seguridad integrada y aprendizaje continuo.

### 1.2 Marcos conceptuales
- **CALMS**: **C**ulture (cultura), **A**utomation (automatización), **L**ean (flujo de valor, lotes pequeños), **M**easurement (medición), **S**haring (compartir conocimiento).
- **Las "Tres Vías"**: **Flujo** (acelerar el trabajo de izquierda a derecha, de desarrollo a operación), **Retroalimentación** (señales rápidas de derecha a izquierda) y **Aprendizaje y experimentación continuos**.
- **Ciclo infinito DevOps**: planear → codificar → construir → probar → liberar → desplegar → operar → monitorear → de nuevo a planear.

### 1.3 Prácticas clave
- **Lotes pequeños y frecuentes**: cambios pequeños son más fáciles de probar, revisar y revertir.
- **Todo como código**: aplicaciones, pruebas, pipelines, infraestructura y configuración, versionados y revisados.
- **Automatización de pruebas y despliegues**, con puertas de calidad.
- **Responsabilidad compartida** ("you build it, you run it" en su versión más estricta): quien desarrolla también se preocupa por cómo opera.
- **Retrospectivas sin culpas (blameless postmortems)**: analizar incidentes para mejorar el sistema, no para buscar culpables.
- **Observabilidad y monitoreo** en producción como fuente de retroalimentación.

### 1.4 Métricas DORA
Cuatro métricas clásicas para medir el desempeño de entrega de software:
| Métrica | Qué mide |
|---|---|
| **Frecuencia de despliegue** | Con qué frecuencia se libera a producción |
| **Lead time de cambios** | Tiempo desde el commit hasta que está en producción |
| **Tasa de fallo de cambios** | Porcentaje de liberaciones que causan un fallo o requieren corrección |
| **Tiempo de recuperación** | Tiempo en restaurar el servicio tras un fallo |

Las primeras dos miden **velocidad** y las otras dos **estabilidad**; los equipos de mejor desempeño suelen destacar en ambas, lo que desmiente la idea de que velocidad y calidad se oponen. Versiones recientes del estudio DORA han añadido otras medidas (por ejemplo, retrabajo), conviene verificar la definición vigente.

**Cuidado al aplicarlas a software embebido:** una ECU de producción no se despliega cada día, así que "frecuencia de despliegue" debe adaptarse (por ejemplo, frecuencia de liberación a banco de pruebas o a flota de validación). Las métricas son para **mejorar el flujo**, no para comparar equipos de forma ciega.

## 2. DevOps aplicado a software embebido y automotriz

### 2.1 Qué cambia respecto al software "de servicio"
| Aspecto | Software de servicio (web/backend) | Software embebido automotriz |
|---|---|---|
| Despliegue | Continuo, reversible en minutos | Controlado, regulado, con hitos (A/B/C-Sample, SOP) |
| Hardware | Estándar y abundante | Específico y escaso (bancos HIL) |
| Seguridad funcional | Raramente aplica | Central (ISO 26262, ASIL) |
| Retroalimentación de producción | Inmediata | Lenta, por flota o por campo |
| Reversión | Simple | Costosa (reflasheo, campañas OTA) |
| Evidencia | Ligera | Formal (ASPICE, auditorías) |

### 2.2 Prácticas que sí se adaptan bien
- **CI con análisis estático, SIL y regresión automatizada** (visto en el capítulo anterior).
- **Validación virtual**: ejecutar el software de la ECU sobre **ECUs virtuales (vECU)** y simuladores, sin hardware, en grandes cantidades (ver 6.3).
- **Pipelines con evidencia automática** para ASPICE.
- **Infraestructura como código** para bancos y entornos de prueba reproducibles.
- **Actualizaciones OTA** como mecanismo de entrega continua hacia el vehículo, con rigor de seguridad (ver 7).
- **Telemetría de flota** como retroalimentación de operación.

### 2.3 Vehículo definido por software (SDV)
Tendencia en la que la funcionalidad del vehículo se define cada vez más por software que puede **actualizarse a lo largo de la vida del vehículo**, apoyada en arquitecturas centralizadas/zonales, plataformas de servicios (SOME/IP, AUTOSAR Adaptive) y una **infraestructura de backend en la nube**. Exige procesos DevOps maduros adaptados a la seguridad funcional y a la ciberseguridad.

## 3. Prácticas DevOps en profundidad

### 3.1 Infraestructura como código (IaC)
Definir y gestionar la infraestructura mediante **archivos versionados** en lugar de configuraciones manuales.
- **Declarativa**: describes el **estado deseado** y la herramienta calcula cómo alcanzarlo (por ejemplo, Terraform, CloudFormation, Bicep). **Imperativa**: describes los **pasos** (scripts).
- **Idempotencia**: aplicar la misma configuración varias veces produce el mismo resultado. Es una propiedad fundamental.
- **Deriva de configuración (drift)**: diferencia entre el estado real y el declarado; la IaC y la revisión periódica la reducen.
- **Infraestructura inmutable**: en lugar de modificar servidores en uso, se reemplazan por nuevas instancias creadas desde una imagen/definición.
- **Herramientas**: **Terraform** (aprovisionamiento multi-nube; flujo `init → plan → apply`, con un archivo de **estado**), **Ansible** (gestión de configuración sin agente, *playbooks* en YAML), plantillas nativas de cada nube.

Ejemplo ilustrativo (Terraform):
```hcl
resource "aws_s3_bucket" "hil_artifacts" {
  bucket = "equipo-hil-artefactos"
  tags   = { proyecto = "adas", entorno = "test" }
}
```
Ejemplo ilustrativo (Ansible, preparar un agente de pruebas):
```yaml
- hosts: agentes_hil
  tasks:
    - name: Asegurar Python instalado
      ansible.builtin.package:
        name: python3
        state: present
```

### 3.2 Estrategias de liberación
| Estrategia | Idea | Ventaja | Consideración |
|---|---|---|---|
| **Rolling** | Se actualiza gradualmente instancia por instancia | Sin parada total | Conviven versiones antiguas y nuevas un tiempo |
| **Blue/Green** | Dos entornos idénticos; se cambia el tráfico de uno a otro | Reversión casi instantánea | Duplica recursos temporalmente |
| **Canary** | Se expone la nueva versión a un pequeño porcentaje y se amplía si todo va bien | Limita el impacto de un fallo | Requiere buena observabilidad |
| **Feature flags** | Se despliega el código desactivado y se activa por configuración | Separa **desplegar** de **liberar** | Gestión de flags y deuda asociada |

*Paralelo con OTA:* una **campaña OTA por fases** (primero un pequeño grupo de vehículos o región, luego más) es un despliegue *canary*; el **dual-bank de Flash** (capítulo de Sistemas Embebidos) es la versión embebida de *blue/green*, porque permite volver a la partición anterior si la nueva falla.

### 3.3 Observabilidad y monitoreo
- **Monitoreo**: saber **si** algo está mal (métricas y alertas conocidas). **Observabilidad**: poder preguntar **por qué** (explorar el estado interno del sistema a partir de sus salidas).
- **Tres pilares**: **logs** (eventos), **métricas** (series numéricas) y **trazas distribuidas** (el recorrido de una petición por varios servicios).
- **Cuatro señales doradas (SRE)**: **latencia**, **tráfico**, **errores** y **saturación**.
- **Herramientas habituales**: Prometheus y Grafana (métricas y paneles), pilas de logs (ELK/EFK), OpenTelemetry (instrumentación estándar), sistemas de trazas.
- **Alertas útiles**: basadas en síntomas que afectan al usuario, con umbrales razonables, para evitar fatiga de alertas.

### 3.4 Fiabilidad: SLI, SLO, SLA y presupuesto de error
- **SLI** (indicador): una medida (por ejemplo, % de peticiones exitosas).
- **SLO** (objetivo): el valor al que se aspira para ese indicador (por ejemplo, 99.9% mensual).
- **SLA** (acuerdo): compromiso contractual con consecuencias si no se cumple.
- **Presupuesto de error** = 1 − SLO: la cantidad de fallo tolerable. Mientras no se agote, el equipo puede liberar con más ritmo; si se agota, se prioriza la estabilidad.
- Ejemplo: un SLO de 99.9% en 30 días permite ≈ 43.2 minutos de indisponibilidad (30 × 24 × 60 × 0.001).

### 3.5 DevSecOps: seguridad integrada
Mover la seguridad **a la izquierda** y automatizarla en el pipeline:
- **SAST** (análisis estático de seguridad del código), **DAST** (pruebas dinámicas sobre la aplicación en ejecución), **SCA** (análisis de dependencias y licencias), **escaneo de imágenes de contenedores** y **detección de secretos** en el repositorio.
- **SBOM (Software Bill of Materials)**: inventario de componentes y versiones del software (formatos como SPDX o CycloneDX). Permite saber rápidamente si una vulnerabilidad nueva afecta a tus productos.
- **Seguridad de la cadena de suministro**: firmar artefactos, verificar procedencia, fijar y auditar dependencias.
- **Gestión de secretos**: almacenes dedicados, rotación y mínimo privilegio.
- **Conexión automotriz**: ISO/SAE 21434 (ingeniería de ciberseguridad), UN R155 (sistema de gestión de ciberseguridad) y ASPICE for Cybersecurity exigen procesos y evidencia que el pipeline puede automatizar.

### 3.6 Ingeniería del caos y pruebas de resiliencia
Introducir fallas controladas (apagar un servicio, aumentar latencia) para verificar que el sistema se degrada de forma segura. Es el equivalente en servicios de la **inyección de fallas** que ya haces en HIL.

## 4. Cómputo en la nube: conceptos fundamentales

### 4.1 Qué es y características esenciales
Acceso **bajo demanda** a recursos informáticos (cómputo, almacenamiento, red, servicios) a través de internet, con pago por uso. Características clásicas:
- **Autoservicio bajo demanda**: aprovisionas sin intervención humana del proveedor.
- **Acceso amplio por red.**
- **Agrupación de recursos (multi-tenant)**.
- **Elasticidad rápida**: escalar hacia arriba y hacia abajo según la demanda.
- **Servicio medido**: pagas lo que consumes.

**Escalabilidad**: **vertical** (más potencia a una máquina) vs **horizontal** (más máquinas). **Elasticidad**: ajustarse automáticamente a la carga.

### 4.2 Modelos de servicio
| Modelo | Qué gestionas tú | Ejemplos de tipo de servicio |
|---|---|---|
| **IaaS** (Infraestructura) | SO, middleware, aplicación, datos | Máquinas virtuales, redes virtuales, discos |
| **PaaS** (Plataforma) | Aplicación y datos | Plataformas de aplicaciones, bases de datos gestionadas |
| **FaaS / Serverless** | Solo la función y su configuración | Funciones que se ejecutan por evento |
| **SaaS** (Software) | Solo el uso de la aplicación | Correo, CRM, herramientas de colaboración |

Mientras más "arriba" en la pila, **menos controlas y menos operas**, y más depende de la plataforma.

### 4.3 Modelos de despliegue
- **Nube pública**: recursos compartidos de un proveedor (AWS, Azure, Google Cloud).
- **Nube privada**: infraestructura dedicada a una organización, propia o alojada.
- **Híbrida**: combinación de nube pública y privada/local, con integración entre ellas.
- **Multi-nube**: uso de varios proveedores (evita dependencia, añade complejidad).
- **Edge (borde)**: cómputo cerca del origen de los datos o del usuario (incluido el propio vehículo), para reducir latencia y tráfico.

### 4.4 Modelo de responsabilidad compartida
El proveedor es responsable de la seguridad **de** la nube (centros de datos, hardware, infraestructura base); el cliente, de la seguridad **en** la nube (configuración, identidades, datos, aplicaciones). El reparto cambia según el modelo (en IaaS el cliente tiene más responsabilidad que en SaaS). Muchas brechas ocurren por **malas configuraciones del cliente**, no por fallos del proveedor.

### 4.5 Regiones, zonas y alta disponibilidad
- **Región**: ubicación geográfica con varios centros de datos.
- **Zona de disponibilidad**: centro(s) de datos aislados dentro de una región. Distribuir una aplicación en varias zonas da tolerancia a fallos.
- **Redundancia, balanceo de carga y recuperación ante desastres** (RPO: cuánta pérdida de datos se tolera; RTO: cuánto tiempo se tolera estar caído).
- **Soberanía y localización de datos**: normativa como GDPR puede exigir dónde se almacenan y procesan datos personales (por ejemplo, datos de conductores o ubicación).

### 4.6 Servicios básicos (equivalencias entre proveedores)
| Categoría | AWS | Azure | Google Cloud |
|---|---|---|---|
| Máquinas virtuales | EC2 | Virtual Machines | Compute Engine |
| Contenedores gestionados / Kubernetes | EKS / ECS | AKS | GKE |
| Funciones serverless | Lambda | Functions | Cloud Functions |
| Almacenamiento de objetos | S3 | Blob Storage | Cloud Storage |
| Almacenamiento en bloque | EBS | Managed Disks | Persistent Disk |
| Red virtual | VPC | Virtual Network | VPC |
| Identidad y acceso | IAM | Entra ID / RBAC | IAM |
| Colas y mensajería | SQS / SNS | Service Bus | Pub/Sub |
| Bases de datos gestionadas | RDS / DynamoDB | SQL Database / Cosmos DB | Cloud SQL / Firestore |

No hace falta memorizar nombres: sí conviene entender las **categorías** y saber buscar la equivalencia.

### 4.7 Almacenamiento: objeto, bloque y archivo
- **Objeto**: archivos con metadatos accesibles por API, muy escalables (ideal para logs, artefactos, datos de flota).
- **Bloque**: discos virtuales para una máquina (bases de datos, sistemas de archivos).
- **Archivo**: sistema de archivos compartido por varios clientes.

### 4.8 Seguridad en la nube
- **IAM y mínimo privilegio**: cada identidad solo con los permisos necesarios; autenticación multifactor.
- **Cifrado** en tránsito y en reposo, con gestión de claves.
- **Segmentación de red** y reglas de acceso (grupos de seguridad, firewalls).
- **Registros de auditoría** y monitoreo continuo.
- **Gestión de secretos** centralizada y rotación.
- **Modelo de confianza cero (Zero Trust)**: no se confía por defecto en ninguna red o identidad; se verifica cada acceso.

### 4.9 Costos y FinOps
Modelo de **pago por uso** puede abaratar o disparar el gasto. Prácticas: **etiquetar recursos** para atribuir costos, dimensionar correctamente, usar **autoescalado**, apagar entornos no usados, aprovechar descuentos por compromiso o capacidad interrumpible (*spot*) en cargas tolerantes a interrupciones, y vigilar costos ocultos como la **transferencia de datos de salida (egress)**.

## 5. Contenedores y orquestación (cloud-native)

Ya conoces Docker. A escala, se necesita un **orquestador**.

### 5.1 Kubernetes (K8s) en lo esencial
- **Clúster**: conjunto de máquinas (**nodos**) gestionadas por un **plano de control** (servidor de API, `etcd` que guarda el estado, planificador y controladores).
- **Pod**: la unidad mínima desplegable (uno o varios contenedores que comparten red y almacenamiento).
- **Deployment**: declara cuántas réplicas de un pod deben existir y gestiona actualizaciones (*rolling update*) y reversiones.
- **Service**: punto de acceso estable (y balanceo de carga) a un conjunto de pods. **Ingress**: acceso HTTP(S) desde fuera.
- **ConfigMap / Secret**: configuración y datos sensibles separados de la imagen.
- **Namespace**: separación lógica de recursos.
- **StatefulSet / Job / CronJob**: cargas con estado, tareas puntuales y programadas (útiles, por ejemplo, para ejecutar regresiones SIL por lotes).
- **Autoescalado horizontal (HPA)**: ajusta el número de réplicas según la carga.
- **Modelo declarativo**: describes el **estado deseado** y el sistema trabaja para alcanzarlo y mantenerlo (se reinicia lo que falla).

Ejemplo ilustrativo de un `Deployment`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: servicio-reportes
spec:
  replicas: 3
  selector:
    matchLabels: { app: servicio-reportes }
  template:
    metadata:
      labels: { app: servicio-reportes }
    spec:
      containers:
        - name: app
          image: registry.interno/servicio-reportes:2.3.1
          resources:
            requests: { cpu: "250m", memory: "256Mi" }
            limits:   { cpu: "500m", memory: "512Mi" }
```
Comandos típicos: `kubectl apply -f`, `kubectl get pods`, `kubectl describe pod <p>`, `kubectl logs <p>`, `kubectl rollout status deployment/<d>`, `kubectl rollout undo deployment/<d>`.

### 5.2 Microservicios vs monolito
| Aspecto | Monolito | Microservicios |
|---|---|---|
| Despliegue | Una unidad | Servicios independientes |
| Escalado | Todo junto | Por servicio |
| Complejidad operativa | Menor | Mayor (red, observabilidad, versiones) |
| Autonomía de equipos | Menor | Mayor |
| Pruebas | Más simples de integrar | Requieren pruebas de contrato e integración entre servicios |

No hay una opción universalmente mejor; depende del tamaño del producto y del equipo.

### 5.3 Principios cloud-native y "12 factores"
Configuración en el entorno (no en el código), **procesos sin estado**, dependencias declaradas explícitamente, logs como flujo de eventos, paridad entre entornos, **desechabilidad** (arranque rápido y apagado limpio). Son la base de aplicaciones que escalan y se recuperan bien.

## 6. La nube en el flujo de trabajo de pruebas automotrices

### 6.1 Qué puede y qué no puede moverse a la nube
| Elemento | ¿En la nube? | Razón |
|---|---|---|
| **Bancos HIL** (ECU física, VT System, interfaces) | **No** | Requieren hardware real y tiempo real local |
| Orquestación, reportes, dashboards, repositorio de artefactos | **Sí** | Son software, no dependen de hardware |
| Pruebas **SIL** y de **vECU** | **Sí**, a gran escala | Se ejecutan en CPU estándar |
| Simulación de escenarios ADAS | **Sí** | Muy paralelizable |
| Almacenamiento y análisis de logs/mediciones | **Sí** | Volúmenes grandes y análisis distribuido |
| Entornos de pruebas efímeros | **Sí** | Se crean y destruyen bajo demanda |

**Modelo habitual:** un enfoque **híbrido**: los bancos HIL quedan en laboratorio, conectados al pipeline y a la plataforma de datos; todo lo demás se aprovecha de la elasticidad.

### 6.2 Aprovechar elasticidad para pruebas
- **Paralelizar** miles de simulaciones o casos SIL y reducir semanas a horas.
- **Entornos efímeros** por rama o por cambio, para pruebas aisladas.
- **Control de costos**: apagar recursos al terminar, usar capacidad interrumpible para cargas tolerantes, y medir costo por ejecución.
- **Reproducibilidad**: imágenes y definiciones versionadas, para repetir un entorno exacto.

### 6.3 Validación virtual y simulación escalable
- **ECUs virtuales (vECU)**: el software de producción corre sobre una representación virtual del hardware (ejemplos del ecosistema: dSPACE VEOS, Vector vVIRTUALtarget, plataformas basadas en QEMU o en modelos virtuales de procesador). Permite probar antes de que existan muestras de hardware y ejecutar en la nube.
- **Pruebas basadas en escenarios para ADAS**: bibliotecas de escenarios definidos con estándares como **ASAM OpenSCENARIO** y **OpenDRIVE**, con simuladores de conducción (por ejemplo, el de código abierto CARLA o los comerciales). Se ejecutan miles de variaciones en paralelo.
- **Limitaciones**: la fidelidad del modelo, la correlación con el hardware real y la validación de la propia herramienta de simulación (conecta con las limitaciones de HIL y con SOTIF).

### 6.4 Datos de pruebas y de flota
- **Ingesta** de logs y mediciones (por ejemplo, archivos MDF) a **almacenamiento de objetos**, con metadatos que identifican versión, banco, vehículo y condiciones.
- **Plataformas de datos** (*data lake*) y análisis distribuido para detectar tendencias, anomalías y regresiones.
- **Dashboards** de resultados de regresión, cobertura de requisitos y estabilidad de bancos.
- **Privacidad y gobernanza**: anonimización, retención, consentimiento y localización (GDPR).

## 7. La conexión vehículo ↔ nube

### 7.1 Componentes habituales
- **Unidad de comunicaciones del vehículo (TCU/telemática)** que conecta con el backend por redes celulares.
- **Protocolos de mensajería** ligeros como **MQTT** (publicación/suscripción) y HTTP/gRPC; **brokers** de mensajes (por ejemplo, Kafka) para ingesta masiva.
- **Backend de vehículos**: gestión de dispositivos, identidades, campañas de actualización, ingesta y análisis de telemetría.
- **Gemelos digitales (digital twins)**: representación virtual de un vehículo o componente alimentada por datos reales, útil para análisis y predicción.
- **Computación en el borde**: procesamiento en el vehículo para latencia y privacidad; la nube para agregación y entrenamiento.

### 7.2 Actualizaciones OTA (Over-The-Air)
- **Beneficios**: corregir defectos y añadir funciones sin llevar el vehículo al taller; entrega continua hacia el parque instalado.
- **Retos**: seguridad (autenticidad e integridad del paquete), **seguridad funcional** (no degradar funciones críticas), conectividad variable, energía suficiente durante la actualización, **reversión segura** (dual-bank), compatibilidad entre versiones de ECUs y consentimiento del usuario.
- **Marcos y regulación**: **Uptane** (marco de seguridad específico para actualizaciones automotrices, con repositorios y verificaciones separadas) y **UN R156** (sistema de gestión de actualizaciones de software), junto con **UN R155** (gestión de ciberseguridad).
- **Estrategia de despliegue**: campañas por fases (canary), con monitoreo de flota y capacidad de detener la campaña.
- **Impacto en pruebas**: hay que verificar la **actualización en sí** (flasheo, reintentos, pérdida de energía, rollback, compatibilidad), no solo la nueva funcionalidad. Es una extensión de las pruebas de flasheo UDS que ya conoces.

### 7.3 Ciberseguridad extremo a extremo
Autenticación mutua vehículo ↔ backend, cifrado, gestión de claves y certificados, segmentación de red dentro del vehículo (gateways, firewalls), arranque seguro con HSM, monitoreo de intrusiones y respuesta a incidentes. Los requisitos se trazan con ISO/SAE 21434 y UN R155.

## 8. MLOps: el puente con tu maestría en IA

Como tu perfil incluye una maestría en IA aplicada, vale la pena mencionar cómo se aplican las ideas de DevOps a modelos de aprendizaje automático:
- **MLOps** extiende CI/CD con **CT (entrenamiento continuo)** y gestión de **datos y modelos**.
- **Ciclo**: recolección y versionado de datos → entrenamiento y experimentación → validación → registro del modelo → despliegue → monitoreo → reentrenamiento.
- **Elementos clave**: **versionado de datos y modelos** (herramientas como DVC o MLflow, a modo de ejemplo), **reproducibilidad** (semillas, entornos fijados), **registro de modelos**, **monitoreo de deriva** (*data drift* y *concept drift*) y pruebas del modelo (precisión, robustez, sesgos).
- **Automotriz**: modelos de percepción y de asistencia requieren **curación de datos, validación basada en escenarios y evidencia de seguridad** (SOTIF; también existe normativa emergente sobre IA en seguridad vial, como ISO/PAS 8800). El entrenamiento y la simulación a escala se apoyan en la nube; la inferencia suele ocurrir en el borde (el vehículo).
- **Para tu rol:** probar un sistema con IA añade preguntas nuevas: ¿qué datos cubren el dominio operativo?, ¿cómo se detecta la degradación en campo?, ¿cómo se trazan versión de modelo, datos y pruebas?

## 9. Del síntoma a la causa probable (DevOps y nube)

| Síntoma | Hipótesis a considerar |
|---|---|
| Despliegues frecuentes pero muchas fallas | Falta de pruebas automatizadas o de puertas de calidad, lotes demasiado grandes, entornos distintos entre pruebas y producción |
| Entornos "que funcionan en uno y no en otro" | Deriva de configuración, falta de IaC, dependencias no fijadas |
| Costos de nube que se disparan | Recursos sin apagar, dimensionamiento excesivo, falta de etiquetas, transferencia de datos de salida, ausencia de autoescalado |
| Fuga de datos en almacenamiento en la nube | Permisos públicos o excesivos, credenciales expuestas, falta de cifrado o auditoría |
| El servicio es lento bajo carga | Falta de escalado horizontal, cuellos de botella en base de datos, latencia entre zonas, límites de recursos de contenedores |
| Un pod se reinicia constantemente | Límites de memoria excedidos, falla de arranque, sondas de salud mal configuradas, dependencia no disponible |
| Alertas constantes que se ignoran | Umbrales mal definidos, alertas por causas en lugar de síntomas |
| Una vulnerabilidad nueva y no se sabe qué productos afecta | Ausencia de SBOM o de inventario de dependencias |
| Una campaña OTA deja vehículos inconsistentes | Compatibilidad entre versiones no verificada, falta de reversión, interrupciones de energía o conectividad |
| El pipeline en la nube es lento o inestable | Cuellos de botella en agentes, pruebas intermitentes, colas, caché ausente |
| Pruebas SIL en la nube no reproducen el fallo del banco | Diferencias de fidelidad entre vECU y hardware real, temporización distinta, configuración |

## Ejercicios de práctica
1. **SLO:** un servicio tiene SLO de 99.95% mensual (30 días). ¿Cuántos minutos de indisponibilidad permite?
   *Respuesta:* 43 200 min × 0.0005 = **21.6 min**.
2. **Escalado:** una suite SIL de 12 000 casos de 30 s cada uno se ejecuta en serie. ¿Cuánto tarda y cuánto con 100 ejecutores en paralelo (caso ideal)?
   *Respuesta:* 12 000 × 30 s = 360 000 s = **100 h**; con 100 en paralelo ≈ **1 h** (más la sobrecarga de arranque).
3. **Costo:** un clúster de 20 nodos a 0.20 USD/h opera 8 h al día, 22 días al mes. ¿Costo mensual? ¿Y si se deja encendido 24 × 30?
   *Respuesta:* 20 × 0.20 × 8 × 22 = **704 USD**; encendido siempre: 20 × 0.20 × 720 = **2 880 USD**. Ilustra el valor de apagar entornos.
4. **Estrategia:** ¿canary, blue/green o rolling para una actualización OTA de infotainment con riesgo moderado y mala conectividad en algunos mercados? Justifica.
   *Idea:* campañas por fases (canary) con monitoreo y reversión por dual-bank; el rolling puro es demasiado ciego y el blue/green completo no aplica a vehículos individuales.
5. **Diseño:** propón una arquitectura híbrida para ejecutar regresión nocturna (SIL en la nube + HIL en laboratorio) con reporte unificado. Indica qué componentes van a cada lado.
6. **Seguridad:** lista cinco controles que aplicarías para proteger el almacenamiento de logs de pruebas que contienen datos de vehículos.
7. **IaC:** ¿qué ventajas aporta definir en código los bancos de prueba (agentes, dependencias, configuración) y qué riesgo se reduce?

## Preguntas de teoría
1. ¿Qué es DevOps y en qué se diferencia de CI/CD?
2. Explica CALMS y las Tres Vías.
3. Describe las cuatro métricas DORA y qué indican sobre velocidad y estabilidad. ¿Cómo adaptarías la "frecuencia de despliegue" a software embebido?
4. ¿Qué cambia al aplicar DevOps a una ECU frente a un servicio web?
5. ¿Qué es la infraestructura como código, qué es idempotencia y qué es deriva de configuración?
6. Compara declarativo vs imperativo y menciona un ejemplo de herramienta de cada tipo.
7. Compara rolling, blue/green, canary y feature flags. ¿Con cuál se parece una campaña OTA por fases?
8. ¿Qué diferencia hay entre monitoreo y observabilidad? ¿Cuáles son los tres pilares y las cuatro señales doradas?
9. Define SLI, SLO, SLA y presupuesto de error.
10. ¿Qué incluye DevSecOps y qué es un SBOM?
11. ¿Qué diferencia hay entre IaaS, PaaS, FaaS y SaaS? Da un ejemplo de cada uno.
12. Explica nube pública, privada, híbrida, multi-nube y edge.
13. ¿Qué dice el modelo de responsabilidad compartida y por qué las malas configuraciones son una causa frecuente de brechas?
14. ¿Qué es una región y una zona de disponibilidad? ¿Qué significan RPO y RTO?
15. Compara almacenamiento de objeto, bloque y archivo, con un caso de uso para cada uno.
16. ¿Qué es Zero Trust y cómo se aplica el principio de mínimo privilegio?
17. ¿Qué es FinOps y qué prácticas reducen el costo en la nube?
18. Explica los objetos básicos de Kubernetes (Pod, Deployment, Service, ConfigMap, Secret, Namespace).
19. ¿Qué significa que Kubernetes sea declarativo y qué hace el autoescalado horizontal?
20. Compara monolito y microservicios.
21. ¿Qué partes de tu flujo de pruebas (HIL, SIL, reportes, datos) pueden ir a la nube y cuáles no?
22. ¿Qué es una ECU virtual y qué ventajas y límites tiene frente a HIL?
23. Describe una arquitectura vehículo ↔ nube para telemetría y OTA.
24. ¿Qué riesgos tiene una actualización OTA y cómo se mitigan (seguridad, funcional, reversión)? Menciona Uptane, UN R155 y UN R156.
25. ¿Qué cambia al probar una actualización OTA frente a probar una funcionalidad nueva?
26. ¿Cómo se aplican las ideas de DevOps a modelos de ML (MLOps) y qué retos añade en automotriz?
27. Si te preguntan por experiencia en nube y solo has trabajado con Jenkins y Docker, ¿cómo responderías con honestidad y valor?

## Errores comunes
- Reducir DevOps a "usar Jenkins y Docker": es sobre todo cultura, flujo, retroalimentación y responsabilidad compartida.
- Creer que DevOps significa "desplegar a producción sin control", sin adaptarlo a la seguridad funcional y a la regulación automotriz.
- Interpretar velocidad y estabilidad como opuestas, o usar las métricas DORA como una herramienta para comparar y presionar a equipos.
- Confundir escalabilidad con elasticidad, o IaaS con PaaS con SaaS.
- Asumir que "la nube es segura por defecto": la mayoría de las brechas vienen de configuraciones del cliente.
- Dar permisos amplios "para que funcione" y no aplicar mínimo privilegio, ni gestionar secretos de forma adecuada.
- Migrar todo a la nube sin considerar costos, egress, latencia, soberanía de datos o el hecho de que el HIL requiere hardware físico.
- Tratar los contenedores como máquinas virtuales ligeras olvidando el kernel compartido y sus implicaciones de seguridad.
- Usar Kubernetes para todo, incluso donde un servicio simple bastaría, sin considerar la complejidad operativa.
- Ignorar la observabilidad hasta que ocurre un incidente.
- Tratar una actualización OTA como "otro despliegue": exige seguridad, reversión, compatibilidad y verificación de seguridad funcional.
- Presentar experiencia en nube que no se tiene en lugar de explicar con honestidad lo que sí se domina y cómo se relaciona.
- Olvidar que los modelos de ML requieren versionado de datos, monitoreo de deriva y evidencia de seguridad, no solo precisión en un conjunto de prueba.