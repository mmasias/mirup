# Protocolo de iteración — miRUP

Secuencia concreta que sigue el orquestador (o el director humano) para recorrer
una iteración de principio a fin. No describe los artefactos RUP estándar, sino
el cómo y el orden específicos de este método.

---

## Paso 1: Modelo del dominio

**Disciplina:** requisitos

Primer artefacto producido en cualquier proyecto nuevo, antes que ningún otro.
Se construye a partir del conocimiento del dominio que tiene el director.
Todos los diagramas se expresan en PlantUML.

### Diagrama de clases

Se construye en cuatro pasos acumulativos:

| Paso | Qué añade |
|---|---|
| Básico | Entidades + relaciones sin dirección ni semántica |
| Base | Dirección y tipo de relación (composición, agregación, asociación) |
| Detallado | Atributos y enums |
| Extendido | Multiplicidades |

### Diagramas de estados

Para las entidades con comportamiento temporal relevante.

### Diagramas de objetos

Solo cuando el modelo estático deja ambigüedad estructural sin resolver,
o cuando hay estados significativos del dominio que merece la pena instanciar.
Las notas en estos diagramas pueden capturar reglas del dominio.

### Glosario

Solo si el vocabulario del dominio es ambiguo o específico del negocio.

### Criterio de completitud

Todos los artefactos se producen en su forma más completa.
Las disciplinas posteriores actúan como integración tests del dominio:
si falta algo, requisitos o análisis-diseño lo exigirán.

---

## Paso 2: Casos de uso

**Disciplina:** requisitos

### 2.1 Heurística CRUD (previo a la disciplina formal)

Antes de cualquier interacción con actores, se genera un conjunto de casos de uso
candidatos a partir del modelo del dominio. La heurística es: cuatro casos de uso
por entidad, correspondientes a las operaciones Create, Read, Update y Delete.

Los nombres no son los del CRUD genérico sino los del dominio. Para cada operación
se busca primero el verbo que el dominio ya tiene para esa acción; solo si no existe
se cae al patrón `mostrar`+entidad o `editar`+entidad.

Ejemplo: para la entidad `Matricula` en un sistema universitario:
- C -> `matricular`
- R -> `mostrarMatricula`
- U -> `editarMatricula`
- D -> `desmatricular`

Sobre el conjunto resultante se itera para descartarlo o reclasificarlo:
- Si la operación no ocurre en el dominio, el caso de uso se descarta.
- Si ocurre de forma diferente a lo esperado, se reclasifica y renombra con el verbo
  del dominio. Ejemplo: en un sistema universitario no se elimina una matrícula, se
  anula; eso convierte el D en un U y lo renombra `desmatricular`.

El resultado es un conjunto inicial de casos de uso candidatos, depurado y con
nombres del dominio, que entra como input a la actividad formal de requisitos.

### 2.2 Identificación de actores y casos de uso

Se identifican actores mediante conversación. Para cada actor se identifican sus
casos de uso. Los casos de uso candidatos del paso anterior alimentan esta actividad;
los casos de uso identificados con los actores tienen mayor peso.

El cruce entre ambos conjuntos es un checklist sin artefacto explícito: un candidato
sin actor es sospechoso; un caso de uso de actor sin correspondencia en el CRUD merece
atención.

Los diagramas de casos de uso se expresan en PlantUML, organizados por actor y área
funcional. Las relaciones `<<include>>` y `<<extend>>` no se determinan aquí sino en
el paso siguiente.

### 2.3 Diagrama de contexto

**Nombre formal del artefacto:** diagrama de contexto. Se produce uno por actor.

**Qué modela:** estados del sistema desde la perspectiva del actor — no estados de la
interfaz, no estados del dominio, sino contexto operacional. Un estado como
`CURSO_ABIERTO` no representa una pantalla: representa que hay una entidad activa y
que las operaciones disponibles están acotadas a ese contexto. El diagrama es
tecnológicamente agnóstico: la misma especificación describe el comportamiento del
sistema con independencia de si la interacción se realiza mediante GUI, CLI, API REST
o interfaz conversacional. Todos los diagramas se expresan en PlantUML.

**Cómo se construye:** los estados son los contextos operacionales del actor; las
transiciones son casos de uso. Solo se puede transitar entre estados mediante casos de
uso identificados. Si para ir de un estado a otro no existe ningún caso de uso, o bien
falta un caso de uso o bien la transición no debería existir.

#### Convenciones de nomenclatura de estados

| Estado | Convención | Ejemplo |
|---|---|---|
| Estado inicial | `SESION_CERRADA` | Punto de entrada al sistema |
| Hub de navegación | `SISTEMA_DISPONIBLE` | Menú principal |
| Lista de entidades | `Xs_ABIERTO` (plural) | `PROGRAMAS_ABIERTO` |
| Entidad en foco | `X_ABIERTO` (singular) | `PROGRAMA_ABIERTO` |

#### Patrón CRUD en el diagrama

Las cuatro operaciones CRUD se mapean a transiciones con semántica específica:

| Operación | Transición | Efecto |
|---|---|---|
| Create | `Xs_ABIERTO -> X_ABIERTO` via `crearX()` | Pasa a estado de edición |
| Read/List | `SISTEMA_DISPONIBLE -> Xs_ABIERTO` via `abrirXs()` | Abre la lista |
| Update | `Xs_ABIERTO -> X_ABIERTO` via `editarX()` o auto-transición en `X_ABIERTO` | Pasa a o permanece en edición |
| Delete | Auto-transición en `Xs_ABIERTO` via `eliminarX()` | Operación in situ, sin cambio de estado |

Create y Update comparten estado destino (`X_ABIERTO`). Esto no es una simplificación
del diagrama sino la materialización de la filosofía C→U: el caso de uso Create es
deliberadamente delgado — solicita solo los datos mínimos indispensables, crea la
entidad con información básica y transfiere inmediatamente el control al caso de uso
Update para la edición completa. La topología del diagrama hace visible esta decisión:
si Create y Update llevan al mismo estado, el sistema no distingue entre "recién
creado" y "en edición".

#### Patrón de retorno al hub

Todo estado que no sea `SESION_CERRADA` ni `SISTEMA_DISPONIBLE` tiene una transición
de retorno al hub mediante el caso de uso `completarGestion()`.

#### Propiedades

El diagrama actúa como validador bidireccional del modelo de casos de uso:
- Transición sin caso de uso -> el caso de uso falta o la transición es incorrecta.
- Caso de uso sin ninguna transición que lo use -> su existencia es cuestionable.

Las relaciones `<<include>>` son consecuencias topológicas del diagrama, no decisiones
de diseño. Si la única ruta al estado donde ocurre `eliminarX` pasa por `abrirXs`,
entonces `eliminarX` incluye `abrirXs` necesariamente.

El diagrama es una especificación generativa: a partir de él se puede inferir
sistemáticamente navegación, endpoints, permisos, tests y documentación, porque
contiene la lógica operacional completa del sistema para ese actor.

El diagrama hace explícitas visualmente las precondiciones de navegación, eliminando
la necesidad de especificación textual adicional.

---

### 2.4 Priorización de casos de uso

La unidad de trabajo es la rama completa del diagrama de contexto: todos los casos de
uso que comparten entidad y espacio de estados se abordan juntos. Esto garantiza
cohesión, contexto acotado y resultado verificable de punta a punta al cierre de la
iteración.

El orden entre ramas lo determina el riesgo arquitectónico: la rama que fuerza más
decisiones de diseño va primero. Las ramas de CRUD puro sobre entidades simples se
dejan para iteraciones posteriores.

---

## Paso 3: Detalle y prototipo de casos de uso

**Disciplina:** requisitos

### 3.1 Detalle del caso de uso

Cada caso de uso se detalla mediante un diagrama de estados en PlantUML. El caso de
uso es un estado compuesto; dentro, estados anónimos (sin nombre) conectados por
transiciones cuyas notas describen la interacción. Las transiciones de salida del
compuesto enlazan con los estados del diagrama de contexto, preservando su topología.
El detalle es un zoom-in del diagrama de contexto sobre un único caso de uso.

#### Vocabulario de interacción

La descripción de cada transición usa vocabulario controlado según el agente:

| Actor únicamente | Sistema únicamente |
|---|---|
| introduce | permite introducir |
| solicita introducir | permite solicitar |
| solicita | presenta / muestra / visualiza |

#### Filosofía C→U en el detalle de Create

El detalle de cualquier caso de uso Create sigue el principio del delgado: recoge
únicamente los datos mínimos indispensables para que la entidad exista, crea la
entidad y cierra transfiriendo el control a Update. No recoge datos opcionales,
no valida más allá de lo estrictamente necesario para crear. Todo lo demás pertenece
al detalle de Update.

#### Contraindicaciones

Se excluyen tres categorías de contenido:

- **Pensamiento o intención del actor** (`el usuario decide...`, `el jugador mira...`):
  no atañe al sistema porque no se programa.
- **Ejecución del software** (`se graba en la base de datos...`): pertenece a
  análisis/diseño; incluirlo aquí es una decisión prematura.
- **Diseño de interfaz** (`se pulsa un botón...`, `ventana de...`, `arrastra el ratón...`):
  pertenece a la actividad de prototipo, no al detalle.

---

### 3.2 Prototipo

El prototipo especifica el contrato de interacción del caso de uso: qué información
se intercambia y qué operaciones están disponibles. Es el único punto del proceso
donde el lenguaje de interfaz está permitido. No especifica implementación.

La trazabilidad es directa en ambos sentidos: los campos que el prototipo presenta son
exactamente los que el detalle especificó mediante "presenta / muestra"; las acciones
disponibles son exactamente las que el detalle especificó mediante "permite solicitar".
El prototipo no añade información nueva: transcribe el detalle a contrato de interacción.

El criterio de corte entre lo que pertenece al prototipo y lo que no: todo lo que
requiere conocer la tecnología de persistencia, el framework o la infraestructura
pertenece a disciplinas posteriores.

#### Prototipo de interfaz gráfica

Se expresa en PlantUML Salt, dentro del mismo ecosistema que el resto de artefactos.
Especifica estructura de interacción (campos, acciones, organización), no diseño visual.

#### Prototipo de API REST

La interfaz técnica también se prototipa. El prototipo de API especifica:
- Endpoints y su propósito
- Estructura de campos del request/response
- Códigos HTTP como semántica de resultado (200, 400, 401, 403...)
- Traza directa desde el detalle del caso de uso

Quedan fuera del prototipo: consideraciones de implementación, rendimiento,
escalabilidad y detalles de seguridad técnica. Pertenecen a análisis/diseño.

#### Prototipo CLI

Qué comandos existen, qué parámetros aceptan y qué output producen. Derivable
directamente del diagrama de contexto: cada transición es un comando posible.

#### Prototipo de interfaz conversacional

Qué utterances disparan qué transiciones y qué responde el sistema. Mismo modelo
de estados, superficie de voz.

#### Prototipo de eventos

Qué eventos emite el sistema ante cada transición y cuál es la estructura de su
payload. Aplica cuando el sistema tiene comportamiento asíncrono o notificaciones.
Solo se especifica la estructura del mensaje, no el broker ni el mecanismo de entrega.

#### Prototipo batch/fichero

Formato de entrada y salida para operaciones que procesan datos en bloque en lugar
de interacción estado a estado.

---

## Paso 4: Estructuración de casos de uso

**Disciplina:** requisitos

Al detallar múltiples casos de uso aparecen sub-flujos que se repiten en varios de
ellos. Esos sub-flujos se factorizan en casos de uso nuevos, lo que produce las
relaciones `<<include>>` y `<<extend>>` del modelo.

Este paso tiene dos fuentes de `<<include>>`:

- **Topológica** (ya presente desde el diagrama de contexto): consecuencia de la
  navegación. Si la única ruta a un estado pasa por otro caso de uso, la inclusión
  es necesaria.
- **Por factorización** (emerge aquí): sub-flujos comunes detectados al comparar
  los detalles de varios casos de uso.

El resultado actualiza tres artefactos:
- Los diagramas de casos de uso (paso 2.2): se añaden las relaciones `<<include>>`
  y `<<extend>>` identificadas.
- Los detalles de los casos de uso involucrados (paso 3.1): se sustituyen los
  sub-flujos factorizados por la referencia al nuevo caso de uso.
- Se crean nuevos casos de uso con su propio detalle y prototipo correspondientes.

---

<!-- paso 5 y siguientes: en construcción -->
