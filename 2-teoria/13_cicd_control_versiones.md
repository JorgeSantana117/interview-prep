# Capítulo: CI/CD y Control de Versiones (Git, Gerrit, GitHub, Bitbucket, Jenkins, Docker)

> Objetivo del capítulo: dominar las herramientas y el razonamiento detrás de los **pipelines de integración y entrega continua** y del **control de versiones**, aplicados a software embebido y a **automatización de HIL**. Todas estas herramientas están en tu CV, así que en entrevista se espera que expliques no solo qué comandos usas sino **por qué un pipeline está diseñado de cierta forma** y cómo resuelves los problemas típicos.
>
> **Límites con otros capítulos (para no repetir):**
> - Principios de DevOps (cultura, retroalimentación, métricas DORA a fondo), nube y contenedores *cloud-native* → irán en *DevOps y Cómputo en la Nube* (Fase 5). Aquí se usan las herramientas concretas en el contexto automotriz.
> - Shift-left, IC/EC como concepto y regresión automatizada → ya en *Fundamentos de Software Testing*.
> - Gestión de configuración, líneas base y gestión de cambios (SUP.8, SUP.10) → ya en *ASPICE/AUTOSAR* e *Ingeniería de Software*. Aquí: cómo se materializan con Git y pipelines.
> - Robot Framework, CANoe y su integración con Jenkins → ya en *Herramientas de Testing y Automatización*. Aquí: el pipeline que los orquesta.
> - Revisión de código, MISRA y análisis estático → ya en *Ingeniería de Software*. Aquí: cómo se automatizan en el pipeline.

---

## 1. Control de versiones con Git

### 1.1 Por qué importa
Un sistema de control de versiones (VCS) registra la historia de los cambios, permite colaborar sin pisarse y hace posible **reproducir** cualquier versión pasada. En automotriz es además **evidencia de proceso** (ASPICE SUP.8): qué se cambió, quién, cuándo, por qué y a qué requisito o defecto estaba asociado.

### 1.2 Modelo mental de Git
- Git es **distribuido**: cada clon tiene la historia completa.
- Un **commit** es una *instantánea* del proyecto, identificada por un hash, que apunta a su(s) commit(s) padre. La historia es un **grafo dirigido acíclico**.
- Una **rama (branch)** es solo un **puntero** móvil a un commit. **HEAD** apunta a la rama o commit actual.
- Tres áreas: **directorio de trabajo** (archivos que editas), **área de preparación o *index*** (lo que irá en el próximo commit) y **repositorio local**. Además, uno o más **remotos** (por ejemplo, `origin`).
- Una **etiqueta (tag)** marca un commit de forma estable (por ejemplo, una versión liberada).

### 1.3 Comandos esenciales
| Comando | Qué hace |
|---|---|
| `git clone`, `git status`, `git log --oneline --graph` | Obtener el repositorio, ver estado e historia |
| `git add`, `git commit` | Preparar y confirmar cambios |
| `git diff` | Ver diferencias (`--staged` para lo preparado) |
| `git branch`, `git switch` / `git checkout` | Crear y cambiar de rama |
| `git fetch` | Descargar cambios del remoto **sin** modificar tu rama |
| `git pull` | `fetch` + integrar (merge o rebase) |
| `git push` | Publicar commits en el remoto |
| `git merge` / `git rebase` | Integrar cambios de otra rama |
| `git stash` | Guardar temporalmente cambios sin confirmar |
| `git cherry-pick <hash>` | Aplicar un commit concreto en otra rama (típico para llevar un *hotfix* a una rama de liberación) |
| `git revert <hash>` | Crear un commit que **deshace** otro (seguro en historia compartida) |
| `git reset --soft/--mixed/--hard` | Mover la rama (y opcionalmente el *index* y el directorio de trabajo); `--hard` descarta cambios |
| `git reflog` | Historial de movimientos de HEAD; salvavidas para recuperar commits "perdidos" |
| `git tag -a v1.2.0 -m "..."` | Crear una etiqueta anotada |
| `git blame` | Quién cambió cada línea y en qué commit |
| `git bisect` | Búsqueda binaria del commit que introdujo un defecto |

### 1.4 Merge vs rebase
| Aspecto | Merge | Rebase |
|---|---|---|
| Qué hace | Une dos historias con un commit de fusión | Re-aplica tus commits sobre otra base, creando commits nuevos |
| Historia | Conserva la historia real (con bifurcaciones) | Historia lineal y limpia |
| Riesgo | Historia más ruidosa | **Reescribe commits**; peligroso si ya se compartieron |
| Regla de oro | — | **No hagas rebase de ramas públicas/compartidas** |

En flujos con Gerrit es habitual trabajar con *rebase* para mantener la historia lineal, porque cada cambio es un solo commit.

### 1.5 Conflictos
Ocurren cuando dos cambios tocan las mismas líneas. Pasos: identificar los archivos en conflicto, resolver los marcadores (`<<<<<<<`, `=======`, `>>>>>>>`) entendiendo la **intención de ambos cambios**, compilar y ejecutar pruebas, y continuar (`git add` + `git rebase --continue` o `git commit`). Se reducen con **integración frecuente**, cambios pequeños y buena modularidad del código.

### 1.6 `git bisect`: la herramienta del probador
Cuando una regresión aparece en un build y no sabes en qué commit, `bisect` encuentra el cambio culpable en un número logarítmico de pasos:
```bash
git bisect start
git bisect bad                 # el commit actual falla
git bisect good v1.4.0         # esta versión estaba bien
# Git te lleva a un commit intermedio; compilas/pruebas y marcas:
git bisect good    # o: git bisect bad
# ... hasta que identifica el primer commit malo
git bisect reset
```
Con un script de prueba automatizado se puede lanzar `git bisect run ./prueba.sh`. Con 1 000 commits entre ambos puntos, bastan unos 10 pasos (⌈log₂ 1000⌉ = 10). Es una forma muy eficaz de colaborar con desarrollo al reportar una regresión.

### 1.7 Buenas prácticas de commits
- **Commits pequeños y atómicos**: un cambio lógico por commit, que compile y pase pruebas.
- **Mensaje claro**: una línea de resumen (imperativo, ~50–72 caracteres), línea en blanco y un cuerpo que explique el **porqué**. Incluir la referencia al ticket o requisito (por ejemplo, `PROJ-1234`) para la trazabilidad.
- **No versionar** artefactos generados, credenciales ni archivos locales (`.gitignore`).
- **Firmar** commits/etiquetas cuando el proceso lo exige.

### 1.8 Archivos grandes y binarios
Los modelos (`.slx`), logs, binarios y archivos de calibración engordan el repositorio. Estrategias: **Git LFS** para binarios necesarios, un **repositorio de artefactos** (Artifactory, Nexus) para binarios generados por el pipeline, y herramientas específicas de comparación y *merge* para modelos.

### 1.9 Organización de repositorios
- **Monorepo**: un repositorio con todo. Facilita cambios atómicos entre componentes y la reutilización; exige herramientas de build escalables y control de acceso fino.
- **Multi-repo**: un repositorio por componente. Aísla equipos, pero complica coordinar versiones entre repos.
- **Submódulos** (`git submodule`) o la herramienta **`repo`** (con *manifest*, muy usada en Android y en muchos BSP de plataforma) permiten componer un proyecto a partir de varios repositorios con versiones fijadas. Tiene relación directa con la gestión de **líneas base** de plataforma que usas al trabajar con equipos de QNX/Qualcomm.

---

## 2. Estrategias de ramificación (branching)

| Estrategia | Idea | Cuándo encaja | Riesgos |
|---|---|---|---|
| **Trunk-based development** | Todos integran con frecuencia en una rama principal; ramas muy cortas; funciones incompletas ocultas con *feature flags* | Equipos maduros con CI sólido y alta automatización de pruebas | Exige disciplina y pruebas fiables; requiere gestionar flags |
| **GitHub Flow** | Rama `main` siempre desplegable; ramas de funcionalidad con *pull request* | Productos con despliegue continuo | Menos estructura para múltiples versiones en paralelo |
| **GitFlow** | Ramas `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` | Productos con liberaciones planificadas y varias versiones mantenidas | Ramas de larga vida, integraciones tardías y más conflictos |
| **Ramas de liberación por cliente/muestra** | Una rama estable por A/B/C-Sample, cliente o variante, con *cherry-pick* de correcciones | Programas automotrices con varios hitos y variantes en paralelo | Divergencia entre ramas; costo de mantener y probar cada una |

**Realidad automotriz:** suele haber una rama principal de desarrollo y **ramas de liberación de larga vida** por hito o cliente. Las correcciones se hacen en la rama principal y se **trasladan (cherry-pick/backport)** a las ramas de liberación, con **pruebas de regresión por rama**. La clave es limitar la divergencia y mantener trazabilidad de qué corrección está en qué versión.

**Etiquetado y versionado:** cada entrega se **etiqueta** (por ejemplo, con *semantic versioning* `MAYOR.MENOR.PARCHE` o la numeración del programa) y se vincula a sus pruebas, defectos conocidos y notas de versión. La **línea base** debe permitir **reproducir el build** exacto (código, configuración, versión de herramientas y dependencias).

---

## 3. Revisión de código en plataformas: Gerrit, GitHub y Bitbucket

| Aspecto | Gerrit | GitHub (Pull Requests) | Bitbucket (Pull Requests) |
|---|---|---|---|
| Unidad de revisión | **Cambio (change)**: un commit con `Change-Id`; cada actualización genera un **patch set** | Rama completa con varios commits | Rama completa con varios commits |
| Cómo se envía | `git push origin HEAD:refs/for/<rama>` | `git push` de la rama + abrir PR | `git push` de la rama + abrir PR |
| Actualizar tras comentarios | Modificar el commit (`git commit --amend`) y volver a subir → nuevo patch set | Añadir commits a la rama | Añadir commits a la rama |
| Votos | **Code-Review** (típicamente −2 a +2) y **Verified** (−1 a +1) | Aprobaciones y comprobaciones de estado | Aprobaciones y comprobaciones de estado |
| Integración (*submit*) | Cuando se cumplen las reglas de votos; puede reordenar y hacer *rebase* | Merge del PR (merge, squash o rebase) | Merge del PR |
| Integración con CI | Jenkins vota **Verified** automáticamente | GitHub Actions u otras como *status checks* | Bitbucket Pipelines o Jenkins |

**Flujo con Gerrit (el de tu CV):**
1. Creas una rama local, haces el cambio y confirmas **un commit** con mensaje claro y referencia al ticket.
2. Subes el cambio a `refs/for/main`; Gerrit crea el *change* y asigna un `Change-Id`.
3. **Jenkins** (mediante el *Gerrit Trigger*) compila, ejecuta análisis estático y pruebas, y publica el voto **Verified +1/−1**.
4. Los revisores comentan y votan **Code-Review**.
5. Atiendes los comentarios con `commit --amend` y vuelves a subir → **nuevo patch set**, que dispara de nuevo el CI.
6. Con los votos requeridos, se hace *submit* y el cambio se integra.

**Protecciones de rama y reglas:** exigir revisión y CI verde antes de integrar, prohibir *push* directo a ramas protegidas, definir responsables de código (por ejemplo, `CODEOWNERS` en GitHub) y limitar quién puede reescribir historia. Esto es **evidencia de proceso** para ASPICE.

---

## 4. Integración y entrega continuas (CI/CD)

### 4.1 Conceptos
- **Integración continua (CI)**: integrar cambios en la rama compartida con frecuencia y **verificarlos automáticamente** (compilación, análisis estático, pruebas) en cada cambio.
- **Entrega continua (Continuous Delivery)**: el software siempre está en un estado **liberable**; el despliegue a producción es una decisión manual.
- **Despliegue continuo (Continuous Deployment)**: cada cambio que pasa las verificaciones se despliega automáticamente. En automotriz **raramente** se aplica a ECUs de producción por razones de seguridad y regulación; sí se aplica a entornos de prueba y herramientas internas.
- **Pipeline como código**: la definición del pipeline vive en el repositorio (por ejemplo, un `Jenkinsfile`), versionada y revisada como cualquier código.

### 4.2 Etapas típicas de un pipeline embebido
```
Cambio → Checkout → Análisis estático → Build (compilación cruzada) → Pruebas unitarias (SIL)
      → Empaquetado de artefactos → Despliegue al banco (flasheo) → Smoke test en HIL
      → Regresión (nocturna/por hito) → Reportes y puertas de calidad → Archivo de evidencia
```
- **Rápido primero, lento después**: las etapas baratas (análisis estático, unitarias) van al inicio para dar retroalimentación en minutos; las costosas (regresión completa en HIL) se ejecutan con menor frecuencia (por ejemplo, de forma nocturna o en cambios de determinadas ramas).
- **Fallar rápido**: si una etapa temprana falla, no se desperdician bancos de HIL.
- **Puertas de calidad (quality gates)**: criterios medibles (cobertura, violaciones, resultados de pruebas) que determinan si el pipeline continúa.
- **Artefactos con trazabilidad**: binarios, A2L, DBC usados, logs, reportes; todo asociado a un identificador de build y al commit.

### 4.3 HIL en el pipeline (lo específico de tu entorno)
- **Los bancos HIL son recursos escasos y físicos.** El pipeline debe **reservarlos** (bloqueo de recursos) para que dos ejecuciones no usen el mismo banco a la vez.
- **Agentes dedicados**: el PC del banco (con CANoe, que corre en **Windows**, y las interfaces de hardware) se configura como **agente de Jenkins** con una etiqueta (por ejemplo, `hil-bench-01`).
- **Pasos típicos**: preparar el banco (alimentación, estado conocido), **flashear** la versión bajo prueba, ejecutar *smoke tests*, lanzar la regresión (por ejemplo, Robot Framework → CANoe), recoger logs/trazas y **dejar el banco en estado seguro**.
- **Paralelización**: partir la suite en lotes y repartirlos entre varios bancos (*sharding*). Ejemplo: 600 pruebas de 36 s promedio ≈ 6 h en un solo banco; con 3 bancos en paralelo ≈ 2 h en el caso ideal (algo más con la sobrecarga de preparación).
- **Estabilidad**: las pruebas intermitentes (*flaky*) y las fallas de infraestructura (banco caído, licencia) deben distinguirse de los fallos reales del software; de lo contrario, el equipo pierde confianza en el pipeline.
- **Regresión por hito**: en B/C-Sample se ejecuta la suite completa; en el día a día, un subconjunto priorizado por riesgo y por análisis de impacto.

### 4.4 Ejemplo de `Jenkinsfile` declarativo (ilustrativo)
```groovy
pipeline {
  agent none
  options { timestamps(); timeout(time: 90, unit: 'MINUTES') }

  stages {
    stage('Build y análisis') {
      agent { docker { image 'registry.interno/embedded-build:1.4.2' } }
      steps {
        sh 'cmake -B build -G Ninja && cmake --build build'
        sh './tools/run_static_analysis.sh'
        sh 'ctest --test-dir build --output-on-failure'   // pruebas unitarias / SIL
      }
      post {
        always { junit 'build/test-results/**/*.xml' }
        success { archiveArtifacts artifacts: 'build/out/*.hex, build/out/*.a2l', fingerprint: true }
      }
    }

    stage('HIL smoke + regresión') {
      agent { label 'hil-bench-01' }          // agente Windows del banco
      steps {
        lock('bench-01') {                     // plugin Lockable Resources
          bat 'python tools\\flash.py --image build\\out\\app.hex'
          bat 'robot --outputdir results --xunit xunit.xml --variable BENCH:bench01 tests\\smoke'
          bat 'robot --outputdir results --variable BENCH:bench01 tests\\regression'
        }
      }
      post {
        always {
          junit 'results/xunit.xml'
          archiveArtifacts artifacts: 'results/**', allowEmptyArchive: true
        }
      }
    }
  }

  post {
    failure { echo 'Pipeline fallido: revisar etapa y logs' }
  }
}
```
Los nombres de plugins y pasos (`lock`, `junit`, publicación de reportes de Robot Framework) dependen de la instalación; conviene verificarlos en tu Jenkins.

---

## 5. Jenkins

### 5.1 Arquitectura
- **Controlador (controller)**: gestiona la configuración, la cola de trabajos y la interfaz. Antes se llamaba *master*.
- **Agentes (agents/nodes)**: máquinas que ejecutan los trabajos. Se seleccionan por **etiquetas** (`label`). Pueden ser permanentes (el PC de un banco HIL) o efímeros (contenedores).
- **Ejecutores (executors)**: cuántos trabajos simultáneos admite un agente.
- **Plugins**: extienden Jenkins (Gerrit Trigger, Lockable Resources, Docker, Robot Framework, JUnit, Credentials, entre otros).

### 5.2 Tipos de trabajo
- **Freestyle**: configurado desde la interfaz; simple pero difícil de versionar.
- **Pipeline** (`Jenkinsfile`): definido como código. Dos sintaxis: **declarativa** (estructura fija: `pipeline`, `agent`, `stages`, `steps`, `post`) y **scripted** (Groovy más libre). La declarativa es preferible por legibilidad y validación.
- **Multibranch Pipeline**: descubre ramas (y cambios) y crea un pipeline por cada una usando el `Jenkinsfile` del repositorio.
- **Bibliotecas compartidas (Shared Libraries)**: código común de pipeline reutilizable entre proyectos.

### 5.3 Elementos habituales
- **Disparadores (triggers)**: por cambio (webhook, Gerrit Trigger), programados (`cron`), por otro trabajo o manuales con **parámetros**.
- **Credenciales**: se guardan en el almacén de Jenkins y se inyectan en tiempo de ejecución; nunca en el código del repositorio.
- **Artefactos y reportes**: `archiveArtifacts`, `junit` (resultados en XML), reportes de Robot Framework, cobertura.
- **Sección `post`**: acciones según el resultado (`always`, `success`, `failure`, `unstable`, `cleanup`).
- **Paralelismo**: bloque `parallel` para ejecutar etapas simultáneamente (por ejemplo, varios bancos).
- **Recursos bloqueables y colas**: evitan colisiones en recursos físicos.

### 5.4 Buenas prácticas
Pipeline como código y revisado, builds **reproducibles** (entorno fijado en una imagen), agentes **sin estado** cuando sea posible, limpieza del espacio de trabajo, tiempos límite (*timeouts*), notificaciones útiles, **no ejecutar trabajo pesado en el controlador**, copias de seguridad de configuración y mantenimiento de plugins (son una fuente habitual de fallos y de vulnerabilidades).

---

## 6. Docker

### 6.1 Conceptos
- **Imagen**: plantilla inmutable y en capas con el sistema de archivos y la configuración necesaria. **Contenedor**: instancia en ejecución de una imagen.
- **Dockerfile**: receta para construir una imagen. Cada instrucción crea una **capa** que se guarda en caché.
- **Registro (registry)**: repositorio de imágenes (Docker Hub, o uno interno). Las imágenes se identifican por nombre y **etiqueta** (`embedded-build:1.4.2`).
- **Volúmenes y *bind mounts***: datos persistentes o compartidos con el anfitrión.
- **Redes**: comunicación entre contenedores y con el exterior.
- **Docker Compose**: define y levanta varios contenedores relacionados con un archivo.

### 6.2 Contenedores vs máquinas virtuales
| Aspecto | Contenedor | Máquina virtual |
|---|---|---|
| Aislamiento | Procesos aislados que **comparten el kernel** del anfitrión | SO completo con su propio kernel (hipervisor) |
| Tamaño y arranque | Ligero, segundos | Pesado, minutos |
| Densidad | Alta | Menor |
| Aislamiento de seguridad | Menor que una VM | Mayor |
| Uso típico | Entornos de build y pruebas reproducibles, microservicios | Aislamiento fuerte, otro SO completo |

### 6.3 Ejemplo de Dockerfile para un entorno de build
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y --no-install-recommends \
        gcc-arm-none-eabi cmake ninja-build python3 python3-pip git \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt /tmp/requirements.txt
RUN pip3 install --no-cache-dir -r /tmp/requirements.txt

WORKDIR /workspace
```
Comandos habituales:
```bash
docker build -t embedded-build:1.4.2 .
docker run --rm -v "$PWD":/workspace embedded-build:1.4.2 cmake -B build -G Ninja
docker images ; docker ps ; docker logs <contenedor> ; docker exec -it <contenedor> bash
```

### 6.4 Buenas prácticas
- **Fijar versiones** (imagen base, paquetes, dependencias de Python) para builds reproducibles; evitar `latest`.
- **Ordenar las capas** para aprovechar la caché: primero lo que cambia poco (paquetes del sistema, dependencias), al final el código.
- **Imágenes pequeñas**, un propósito por imagen, usuario no root cuando sea posible, sin secretos dentro de la imagen.
- **Versionar y publicar** las imágenes de build con etiquetas inmutables, de modo que un build antiguo pueda reproducirse con la misma imagen.
- **Escaneo de vulnerabilidades** de imágenes.

### 6.5 Límites de Docker en tu contexto (punto de madurez para entrevista)
- **CANoe y el ecosistema Vector corren en Windows** y dependen de controladores y hardware (interfaces VN, VT System); no se contenerizan de forma práctica. Por eso los bancos HIL usan **agentes Windows dedicados**.
- Docker es ideal para lo que sí es portable: **entornos de compilación cruzada**, análisis estático, pruebas **SIL**, ejecución de scripts de Python y Robot Framework que no dependen de hardware específico, y herramientas auxiliares.
- El acceso a dispositivos físicos (USB, serie) desde contenedores es posible en Linux con opciones específicas (`--device`), pero añade complejidad y riesgos.
- El kernel compartido implica que un contenedor Linux no reproduce comportamientos específicos del kernel del objetivo (QNX, por ejemplo); para eso se usan el objetivo real o simuladores.

---

## 7. Qué hace bueno a un pipeline (y cómo medirlo)

**Características:** rápido para los cambios frecuentes, **determinista** (mismo commit → mismo resultado), con **retroalimentación clara** (qué falló y dónde), **trazable** (build ↔ commit ↔ pruebas ↔ artefactos), seguro (credenciales protegidas) y mantenible (como código).

**Métricas útiles:**
- Tiempo de build y de pipeline total, y tiempo en cola.
- Tasa de fallos y **porcentaje de fallos no atribuibles al código** (infraestructura, pruebas intermitentes).
- Tiempo medio de recuperación de un pipeline roto.
- **Métricas DORA** (se profundizan en el capítulo de DevOps): frecuencia de despliegue, *lead time* de cambios, tasa de fallo de cambios y tiempo de recuperación.
- Cobertura y tendencia de defectos por build.

**CI/CD como evidencia de proceso:** los registros de pipeline, revisiones y resultados enlazados a commits y tickets sirven como productos de trabajo para ASPICE (SWE.4–SWE.6, SUP.8, SUP.9, SUP.10). Las herramientas de CI también pueden requerir evaluación en proyectos de seguridad, porque deciden si el software "pasa" o "falla".

---

## 8. Del síntoma a la causa probable

| Síntoma | Hipótesis a considerar |
|---|---|
| Compila en mi máquina pero falla en CI | Dependencias o versiones de herramientas distintas, archivos locales no versionados, variables de entorno, rutas absolutas |
| Un build antiguo no se puede reproducir | Imagen o dependencias no fijadas, herramientas sin versionar, artefactos externos cambiados |
| Pruebas HIL intermitentes en CI | Banco en estado inconsistente, tiempos de espera fijos, falta de reinicio entre pruebas, interferencia entre ejecuciones, infraestructura inestable |
| Dos ejecuciones usan el mismo banco a la vez | Falta de bloqueo de recursos, etiquetas de agente mal definidas |
| El pipeline tarda demasiado | Etapas lentas al inicio, ausencia de caché, falta de paralelismo, pruebas redundantes |
| Muchos conflictos de integración | Ramas de larga vida, cambios grandes, falta de integración frecuente, módulos muy acoplados |
| Una corrección está en una versión pero no en otra | Backport/cherry-pick omitido, trazabilidad insuficiente entre ramas |
| El agente del banco aparece desconectado | Red, servicio del agente, actualización del sistema, licencia o hardware |
| El repositorio crece sin control | Binarios versionados, falta de LFS o de repositorio de artefactos |
| Cambios sin revisar llegan a la rama principal | Falta de protección de rama, permisos excesivos |
| El pipeline pasa pero el software falla en el banco real | Entornos distintos entre SIL y HIL, cobertura insuficiente, datos de prueba no representativos |
| Credenciales visibles en logs o repositorio | Secretos en el código o en variables impresas, falta de uso del almacén de credenciales |

## Ejercicios de práctica
1. **Gerrit:** los revisores piden cambios en tu *change*. Describe los comandos para actualizarlo y qué ocurre en Gerrit.
   *Respuesta:* modificar el código, `git add`, `git commit --amend` (conserva el `Change-Id`), `git push origin HEAD:refs/for/main`; se crea un **nuevo patch set** y se vuelve a ejecutar el CI.
2. **Bisect:** hay 800 commits entre una versión buena y una mala. ¿Cuántos pasos de `bisect` esperas como máximo? *(⌈log₂ 800⌉ = 10)*.
3. **Backport:** una corrección se hizo en `main` y debe llegar a `release/B-Sample`. ¿Qué haces y qué verificas?
   *Respuesta:* `git switch release/B-Sample`, `git cherry-pick <hash>`, resolver conflictos, ejecutar la regresión de esa rama y registrar la trazabilidad del backport.
4. **Pipeline:** diseña las etapas para una regresión nocturna en 3 bancos HIL con una suite de 600 pruebas de 36 s. Calcula la duración ideal y menciona qué harías ante un banco que falla.
   *Respuesta:* 600 × 36 s = 21 600 s = 6 h; con 3 bancos ≈ 2 h ideales. Se reparte la suite en tres lotes, cada banco se bloquea y se prepara; si uno falla por infraestructura, se reintenta el lote en otro banco y se marca claramente el fallo como de infraestructura.
5. **Docker:** reordena este Dockerfile para aprovechar mejor la caché y explica por qué:
   ```dockerfile
   COPY . /workspace
   RUN pip install -r requirements.txt
   ```
   *Respuesta:* copiar primero solo `requirements.txt`, instalar dependencias y después copiar el resto del código; así un cambio en el código no invalida la capa de dependencias.
6. **Merge vs rebase:** una rama compartida por cinco personas necesita integrar `main`. ¿Usarías rebase? Justifica.
   *Respuesta:* no se debe reescribir historia compartida; se prefiere *merge* de `main` a la rama o acordar un flujo explícito.
7. **Diagnóstico:** el pipeline pasó ayer y hoy falla sin cambios de código. Enumera cuatro hipótesis y cómo las descartarías.

## Preguntas de teoría
1. Explica el modelo de Git (commit, rama, HEAD, *index*) y la diferencia entre `fetch` y `pull`.
2. ¿Cuándo usarías merge y cuándo rebase? ¿Cuál es la regla de oro del rebase?
3. Diferencia entre `git revert` y `git reset`, y cuándo usar cada uno en historia compartida.
4. ¿Cómo ayuda `git bisect` a investigar una regresión?
5. Compara trunk-based, GitFlow y ramas de liberación por hito en un programa automotriz.
6. ¿Cómo gestionas correcciones que deben estar en varias ramas de liberación? ¿Qué trazabilidad mantienes?
7. Describe el flujo de revisión en Gerrit (Change-Id, patch sets, votos Code-Review y Verified).
8. Compara una revisión en Gerrit con un pull request de GitHub/Bitbucket.
9. ¿Qué protecciones de rama configurarías y por qué son evidencia de proceso?
10. ¿Qué diferencia hay entre integración continua, entrega continua y despliegue continuo? ¿Cuál aplica a una ECU de producción?
11. ¿Qué etapas incluirías en un pipeline embebido y en qué orden? Justifica el criterio "rápido primero".
12. ¿Cómo integrarías un banco HIL en un pipeline (agente, bloqueo, flasheo, regresión, reportes)?
13. ¿Cómo distinguirías un fallo del software de uno de la infraestructura en una ejecución de regresión?
14. Describe la arquitectura de Jenkins (controlador, agentes, ejecutores, plugins).
15. ¿Qué ventajas tiene un `Jenkinsfile` frente a un trabajo freestyle? ¿Declarativo vs scripted?
16. ¿Para qué sirven los *Multibranch Pipelines* y las *Shared Libraries*?
17. ¿Cómo gestionarías credenciales en Jenkins?
18. Explica imagen, contenedor, Dockerfile, capa y registro.
19. Compara contenedores con máquinas virtuales.
20. ¿Por qué CANoe y el hardware Vector no suelen contenerizarse y para qué sí usarías Docker en tu flujo?
21. ¿Cómo garantizas que un build de hace un año pueda reproducirse?
22. ¿Qué haces con binarios grandes (modelos, logs, artefactos)?
23. ¿Qué métricas usarías para evaluar la salud de un pipeline?
24. ¿Cómo aportan Git y CI evidencia a un assessment de ASPICE?

## Errores comunes
- Describir Git como "copias de seguridad" en lugar de un grafo de commits con ramas como punteros.
- Reescribir historia compartida (rebase o `push --force`) y romper el trabajo de otros.
- Hacer commits enormes y mezclados, con mensajes vagos y sin referencia a tickets.
- Mantener ramas de larga vida sin integrar y sufrir conflictos masivos.
- Confundir `git reset --hard` (descarta cambios) con un deshacer seguro y compartido (`revert`).
- Tratar el pipeline como "un script que corre pruebas" sin pensar en rapidez, determinismo, trazabilidad y diagnóstico.
- Poner las etapas más lentas y costosas al principio, desperdiciando bancos y tiempo.
- No bloquear los bancos HIL y permitir ejecuciones simultáneas sobre el mismo hardware.
- Ignorar las pruebas intermitentes en lugar de aislarlas y corregirlas, lo que erosiona la confianza.
- Usar `latest` o dependencias sin versión en las imágenes, y no poder reproducir un build.
- Guardar credenciales en el repositorio, en la imagen o en logs.
- Querer contenerizar todo (incluido CANoe) o, al contrario, no aprovechar Docker donde sí es portable.
- Ejecutar trabajo pesado en el controlador de Jenkins y no mantener actualizados los plugins.
- Confundir entrega continua con despliegue continuo y proponer despliegue automático a producción en una ECU de seguridad.
- Versionar binarios grandes directamente en Git sin LFS ni repositorio de artefactos.
- No vincular builds, commits, pruebas y artefactos, perdiendo la trazabilidad que exige ASPICE.