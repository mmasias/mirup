# Protocolo de iteración — miRUP

> *Este protocolo se inspira y está derivado de los trabajos de Luis Fernández Muñoz.*

Secuencia concreta que sigue el orquestador (o el director humano) para recorrer
una iteración de principio a fin. No describe los artefactos RUP estándar, sino
el cómo y el orden específicos de este método.

Nace de la gestión y ejecución humana de proyectos en RUP. Se traza
aquí para que también un LLM pueda construir siguiendo el proceso: aportando su
potencia pero ciñéndose al protocolo. Por eso cada disciplina se escribe con criterio
de entrada, criterio de salida y contraindicaciones explícitos, en lugar de confiar
en el juicio tácito del director. El director humano pasa a ser orquestador; las
decisiones que antes tomaba por instinto se vuelven reglas o puntos de control.

---

## Estructura del protocolo: dos niveles

El protocolo opera en dos niveles:

- **Nivel macro** (este apartado): el bucle de iteración. Qué entra en una iteración,
  en qué orden recorre las disciplinas y qué ocurre entre ellas.
- **Nivel micro** (los pasos numerados que siguen): el contenido concreto de cada
  disciplina.

### Base y bucle

No todo se produce con la misma cadencia:

- **Base del proyecto** (se produce una vez, al inicio o al abrir un incremento mayor):
  el modelo del dominio y el modelo de casos de uso completo. Es el mapa del territorio.
  Corresponde a los pasos 1 y 2, e incluye la priorización (2.4), que parte ese mapa
  en ramilletes y los ordena.
- **Bucle de iteración** (se recorre una vez por ramillete, en orden de prioridad):
  toma un ramillete y lo lleva por el pipeline de disciplinas hasta dejarlo construido
  y verificado de punta a punta.

La **unidad de avance es el ramillete**: la rama completa del diagrama de contexto -
todos los casos de uso que comparten entidad y espacio de estados (definido en 2.4).
No se avanza caso de uso a caso de uso suelto, sino ramillete a ramillete.
Esta correspondencia es uno-a-uno bajo gestión humana; ver sección "Gestión humana
y orquestación CORRAL" para el caso CORRAL.

El pipeline de disciplinas que recorre cada ramillete:

```
requisitos (detalle + prototipo + estructuración) -> análisis -> diseño -> implementación -> pruebas
```

entre cada par de disciplinas se intercala una pausa arquitectónica.

### Tipos de artefacto

En todas las disciplinas conviven dos tipos de artefacto con ciclos de vida distintos:

**Artefacto de ramillete:** cardinalidad 1 por caso de uso o por ramillete. Se produce
iterando, vive en el slot de la iteración:
`corral-rup/{fase}/{iteracion}/{disciplina}/{artefacto}`. Es el candidato natural a
delegación en agentes subordinados.

**Artefacto transversal (de proyecto):** cardinalidad 1 por proyecto. Se consolida
reuniendo los artefactos de ramillete de todos los casos de uso trabajados. No
pertenece a una iteración concreta sino a todas. Vive en `rup/` y crece con cada
ramillete. Ejemplos: el modelo del dominio, el diagrama de clases de análisis
completo, el diagrama de clases de diseño completo, la Vista de Implementación, la
Vista de Despliegue.

**Mecánica de consolidación:** los artefactos transversales se consolidan por
convergencia (topología `converge` de CORRAL-RUP): fan-out para producir las clases
caso a caso en los ramilletes, fan-in para consolidarlas en el diagrama del sistema.
Cada milestone promueve las clases nuevas del ramillete al diagrama consolidado.
Los artefactos transversales se diferencian solo por cuándo se consolidan: el modelo
del dominio existe antes (prerequisito), los diagramas de clases se consolidan durante
(incremental por ramillete), las vistas de despliegue se cierran al final. Todos
comparten cardinalidad 1 por proyecto.

### La pausa arquitectónica

Punto de control entre disciplinas. Es un elemento de primer orden del protocolo, no
un trámite.

**Qué es.** Su naturaleza varía según la incertidumbre y el momento del proceso. Puede
ser una de estas, o ambas a la vez:

- **Verificación:** ¿lo que produjo la disciplina anterior se sostiene? Go / no-go
  antes de abrir la siguiente.
- **Decisión:** ¿qué decisiones transversales (capas, patrones, tecnología) hay que
  tomar o confirmar, porque condicionan al resto del ramillete y a los ramilletes
  siguientes?

**Quién la ejecuta.** El orquestador, y solo el orquestador: el humano o el LLM de alto
nivel (Claude). **Nunca se delega a agentes subordinados.** El reparto entre humano y
LLM se rige por dos factores que miden cosas distintas y se componen (no se suman) según
detalla la sección de gobierno: la madurez del proyecto en esa disciplina y el riesgo
arquitectónico del caso. La tabla siguiente da la intuición de cada factor por separado.

| Gradiente | Tira hacia el humano | Tira hacia el LLM de alto nivel |
|---|---|---|
| Madurez del proyecto | Iniciático, sin arquitectura asentada | Encaminado, con arquitectura clara y patrones establecidos |
| Posición en el pipeline | Cerca de requisitos | Disciplinas tardías |

La lógica del segundo gradiente: cerca de requisitos las decisiones comprometen más el
resto del proyecto y tienen menos patrón previo, por eso las retiene el humano; en
disciplinas tardías la decisión ya está acotada por lo decidido aguas arriba y hay
patrón que seguir, terreno donde el LLM opera con seguridad.

**Función de aprendizaje.** En un proyecto iniciático el humano ejecuta la pausa con el
LLM de alto nivel observando. La pausa es el mecanismo por el que el LLM aprende la
arquitectura del proyecto, hasta poder asumirla en iteraciones posteriores: a medida
que el proyecto madura, el primer gradiente se desplaza solo hacia el LLM.

**Salida.** Una decisión explícita: go/no-go más, si las hubo, las decisiones
arquitectónicas tomadas. Esa salida se convierte en el criterio de entrada de la
disciplina siguiente.

**Registro.** La pausa produce un fichero mínimo co-ubicado con la iteración:
`corral-rup/{fase}/{iteracion}/pausa-{disciplina}.md`. Contiene solo lo
no derivable de los artefactos: las decisiones arquitectónicas tomadas y, si el
resultado fue no-go, las condiciones para reabrir. Si la pausa fue limpia y no hubo
decisiones nuevas, el fichero no se crea: su ausencia es evidencia de paso sin
observaciones. El registro lo escribe el orquestador; nunca se delega.

Este es también el tercer punto donde humano y CORRAL divergen en su comportamiento
— ver sección siguiente.

### Validación entre disciplinas

Cada disciplina consume el producto de la anterior, y la facilidad con que lo consume
es el criterio de calidad de la anterior. Si una disciplina encuentra fricción (no
puede derivar limpiamente lo que necesita del artefacto previo), la causa no suele
estar en la disciplina en curso sino aguas arriba. La fricción es un detector de
defectos, no un problema del trabajo actual.

El rigor del detector crece a lo largo del pipeline: en análisis la detecta el juicio
(un caso de uso que cuesta analizar revela requisitos incompletos); en implementación
la detecta el compilador sin opinión (componentes que no enlazan revelan diseño mal
integrado); en pruebas la detecta la ejecución. Cuanto más tarde, más objetivo el
detector.

Consecuencia operativa: la fricción siempre se atribuye a su origen aguas arriba,
pero dónde se corrige depende del modo. El director humano vuelve a la disciplina
donde está el defecto y lo corrige allí (un defecto que aflora en diseño se resuelve
en análisis o en el modelo del dominio). CORRAL no retrocede de fase: registra la
fricción como observación del milestone y la corrige hacia delante, en una iteración
de corrección dentro de la fase actual. Lo común a ambos modos es la atribución al
origen; lo que difiere es si se vuelve allí o se compensa hacia delante. Esta es la
misma divergencia humano/CORRAL que describe la sección "Gestión humana y
orquestación CORRAL" (manifestación 1, retroceso).

### Patrón de delegabilidad

El reparto entre orquestador y agentes subordinados sigue un patrón constante en
todas las disciplinas del bucle:

- **Retiene el orquestador:** la actividad de arquitectura (establecer el esqueleto:
  A1, D1, I1) y la de integración o estructura global (A4, D4, I2). Son decisiones
  que comprometen al conjunto.
- **Se delega:** la derivación caso a caso (A2, D2, I3). Una vez fijado el esqueleto
  es mecánica.

El esqueleto y la integración no se delegan; el relleno sí. Las secciones de cada
disciplina refieren este patrón y marcan cada actividad como delegable o no según él.

### Gestión humana y orquestación CORRAL: dónde divergen

El cuerpo de este protocolo describe el método tal como lo aplica un director humano.
CORRAL lo ejecuta siguiendo el mismo método, pero con un comportamiento diferente en
tres puntos concretos. La tesis que los unifica:

**CORRAL convierte el juicio continuo y reversible del director en una serie de puertas
discretas e irreversibles hacia delante.** Bajo orquestación CORRAL, el avance de un
ramillete es monótono: cada fase se cierra validada antes de abrir la siguiente. No se
elimina el retroceso por suerte — se previene por diseño con una puerta entre cada fase.

Las tres manifestaciones concretas:

**1. Retroceso.** El director humano puede volver a una fase anterior si descubre un
problema aguas abajo. CORRAL lo sustituye por rechazo en el milestone: si una fase no
pasa la validación, se abre una iteración de corrección dentro de esa misma fase sin
retroceder. El avance nunca da marcha atrás; los errores se corrigen hacia delante.

**2. Correspondencia ramillete-iteración.** Bajo gestión humana la relación es uno a
uno: un ramillete se lleva en una pasada continua, con vueltas internas si hacen falta.
Bajo CORRAL es uno a varios: el ramillete se reparte en una iteración por fase, cada
tramo cerrado por su milestone antes de abrir el siguiente. El ramillete avanza en
tramos discretos, no en una pasada continua.

**3. Quién ejecuta la pausa arquitectónica.** Descrito en la sección anterior con los
dos gradientes (madurez del proyecto y posición en el pipeline). Es la manifestación
más fina del mismo principio: a medida que el proyecto madura y las fases avanzan, la
pausa se desplaza del humano al orquestador LLM, siguiendo la misma lógica de puertas
discretas con responsabilidad progresivamente delegable. La mecánica completa de
ese desplazamiento - las tres regiones, el riesgo y la banda - se describe en la
sección de gobierno siguiente.

---

## Espacio de gobierno: el reparto humano/LLM

El cuerpo del protocolo (pasos 1-8) describe las disciplinas en registro agnóstico de
orquestador: qué produce cada una, sus criterios de entrada y salida, su delegabilidad
y su topología. Esta sección describe lo que es propio de la ejecución bajo CORRAL y no
pertenece a ninguna disciplina: cómo se reparte entre humano y LLM senior lo que el
orquestador retiene. Va aparte por la misma razón por la que va aparte en el filesystem.

### Producto y gobierno: dos espacios

Conviven dos espacios con responsabilidad distinta:

- **Espacio de producto**, gobernado por el Contrato 1 de CORRAL-RUP
  (`{fase}/{iteracion}/{disciplina}/{artefacto}`): lo que las disciplinas producen y se
  delega a becarios. Diagramas, código, casos de prueba.
- **Espacio de gobierno**, con estructura estable propia: el milestone colgando de la
  fase (`{fase}/milestone`) y la pausa colgando de la iteración
  (`{iteracion}/pausa-{disciplina}.md`). Lo que regula transiciones, no delegable,
  escrito siempre por el orquestador.

El Contrato 1 nunca fue para los objetos de gobierno, solo para los artefactos de
producto. El milestone ya vivía fuera de los slots, y eso no era una excepción al
contrato: era un objeto de otra categoría. La pausa es su hermana - gobierna la
transición entre disciplinas como el milestone gobierna el cierre de fase - y vive en el
mismo espacio. No se mete la pausa en los slots ni se declara excepción: se reconoce una
segunda categoría que ya operaba sin nombre.

La separación física refleja la separación de responsabilidad. El espacio de gobierno es
el del juicio (pausas y milestones, los puntos no delegables); el de producto es el del
relleno delegable. Becarios escriben en producto, orquestador en gobierno. Que estén en
sitios distintos del filesystem es que los escriben agentes distintos.

### Las tres regiones de la pausa

Los dos gradientes de la pausa arquitectónica (madurez del proyecto y posición en el
pipeline) se concretan en tres regiones de comportamiento. La región clasifica quién
ejecuta la pausa intermedia; el go/no-go del milestone que cierra fase es humano
invariante en las tres, y nunca se delega. Como el humano siempre ve el milestone, la
clasificación puede inclinarse hacia la delegación sin riesgo: el milestone es el
backstop humano que atrapa cualquier pausa mal delegada.

| Región | Comportamiento | Quién decide |
|---|---|---|
| **Delegada** | El LLM senior ejecuta la pausa; CORRAL sigue sin parar. El humano se entera por el registro. | LLM, autónomo |
| **Retenida** | CORRAL para y espera decisión humana. El LLM no decide. | Humano, asistido |
| **Banda** | El LLM propone; CORRAL marca la decisión provisional y la presenta al humano para confirmación rápida. No para del todo, pero no promueve sin visto bueno. | Compartido, colaborativo |

La banda implementa el "compartido" como protocolo de propuesta-confirmación,
ejecutable, sin convertirlo en frontera nítida. Es donde ocurre el aprendizaje: cada
confirmación o corrección desplaza el sesgo de esa disciplina.

La frontera orquestador/becario (qué se delega al subordinado) es nítida y fija, escrita
por actividad. La frontera humano/LLM (dentro de lo que retiene el orquestador, quién lo
ejecuta) no lo es: es una banda ancha, área y no línea. Su anchura no es imprecisión; es
donde viven el aprendizaje y la libertad del orquestador para situarse en el borde
cuando el caso lo pide. Colapsar la banda en una línea elimina la transferencia gradual
de confianza.

### Riesgo arquitectónico y residencia en banda

El tiempo que una disciplina reside en banda antes de poder delegarse lo fija el riesgo
arquitectónico de lo que tiene delante, no la disciplina en abstracto. Requisitos no es
exigente por ser requisitos: lo es cuando requisita algo no trivial. Un CRUD sale casi
de la heurística 2.1 y reside poco; un caso de uso que define estados operacionales
nuevos reside más. La variable real es cuánto del juicio es derivable.

**Tres niveles, identidad con la residencia.** `riesgo_introducido` toma valores
`bajo | medio | alto`, y `residencia_min = {bajo:1, medio:2, alto:3}` milestones
limpios. El nivel es el número de milestones; no hay multiplicador intermedio. Tres
niveles porque es un juicio que humano o LLM emiten con consistencia; más sería falsa
precisión. No se deriva del orden de priorización 2.4: ese orden es ordinal y relativo
al proyecto (secuencia ramilletes), mientras la residencia necesita escala absoluta
pequeña; y 2.4 solo conoce el riesgo de requisitos, no el que cada disciplina posterior
introduce.

**Herencia monótona.** El riesgo de una disciplina es el máximo entre el heredado de la
anterior y el que ella misma introduce:

```
riesgo(D) = max( riesgo(D-1), riesgo_introducido(D) )
```

No decreciente al descender el pipeline. La complejidad se hereda; la simplicidad no. Un
caso difícil en requisitos llega a diseño con riesgo heredado alto que diseño no rebaja
aunque tecnológicamente sea simple. Un caso trivial en requisitos llega con riesgo bajo,
pero si diseño introduce 3D su riesgo sube y su banda se ensancha. Cada disciplina aporta
su dimensión: requisitos la del comportamiento, diseño la de la tecnología,
implementación la del algoritmo.

**Estimación en cadena, sin recursión.** `riesgo_introducido(D)` se estima en la pausa
que abre D, que es cola del cierre de D-1. Detectar que un caso trivial necesitará 3D es
juicio de pausa, y encaja con que la pausa pre-diseño sea la de máxima retención humana:
elegir stack y detectar que un caso pide 3D son la misma decisión en el mismo punto.
Cuando D abre, su riesgo le viene dado desde atrás; D no calcula su peaje.

**Dónde se registra.** En el frontmatter de la pausa que abre D:

```yaml
---
abre: diseno
riesgo-introducido: alto
---
```

Con la convención que preserva "ausencia = paso limpio": sin fichero de pausa,
`riesgo_introducido = bajo` (residencia 1). El fichero aparece solo cuando hay algo que
registrar - riesgo medio o alto, o decisiones arquitectónicas - y la ausencia sigue
significando "nada destacable". `riesgo_introducido(requisitos)` no tiene pausa de
apertura (requisitos es base); su nivel se registra en la priorización 2.4 del registry,
junto al orden.

### Los dos pliegues

Todo el reparto humano/LLM se reduce a dos pliegues sobre datos que ya existen, ambos
derivables, ninguno persistido (principio de derivados de CORRAL-RUP):

- **región(D) = pliegue sobre el historial ordenado de milestones**, con la regla de
  transición de abajo.
- **riesgo(D) = pliegue `max` sobre la cadena de pausas del ramillete** (semilla de 2.4,
  luego pausa-analisis, pausa-diseno... hasta D).

El segundo parametriza la condición (b) del primero. Misma disciplina formal en los dos.
Ninguno se cachea: persistir el acumulado crearía dos fuentes de verdad y quedaría
obsoleto al corregir un riesgo aguas arriba. El coste de replay es nulo (a lo sumo cinco
valores por ramillete).

**Regla de transición de la región** (replay sobre los milestones, por disciplina):

```
estado inicial: lo fija la posición.
  disciplinas tempranas (requisitos) arrancan en retenida
  disciplinas tardías (pruebas) arrancan en banda
al replicar cada milestone limpio que contiene D (aprobado, sin ningún rechazo):
  retenida -> banda : al primer milestone limpio
  banda -> delegada : si (a) la madurez supera el coste de avance
                         (alto en disciplinas tempranas, bajo en tardías: el eje posición)
                      y (b) se han acumulado al menos residencia_min(D)
                         milestones limpios estando en banda
un rechazo de D: retrocede un escalón (confianza perdida)
```

Madurez de D = número de pares (fase, iteración) donde D aparece y cuyo milestone es
aprobado-limpio (exactamente una creación y una evaluación aprobada, cero rechazos). Se
cuenta del historial, no de ficheros de pausa: una pausa limpia no deja fichero, y
contar ficheros subcontaría justo los pasos limpios.

**Anti-colapso.** `residencia_min(D)` tiene suelo absoluto en uno aunque el riesgo sea
cero: ninguna disciplina salta de retenida a delegada sin pasar por banda. La banda no
se cruza, se atraviesa, y atravesarla cuesta siempre al menos un milestone limpio. Lo
que encoge con la madurez es el coste de avance, que cae hacia su suelo de uno pero
nunca a cero; la banda se adelgaza, no desaparece. Cada disciplina nueva y cada
iteración nueva vuelven a entrar por ella.

### Madurez contra riesgo: por qué dos ejes

Madurez y riesgo no son redundantes porque miden cosas distintas. La madurez es
competencia general del proyecto en una disciplina (de proyecto, historial de milestones
limpios): dice "ya sabemos diseñar". El riesgo es exigencia intrínseca de un caso
concreto (de ramillete): dice "pero este caso pide más". A igual madurez de proyecto, el
diseño de un CRUD se delega pronto (suelo 1) y el del caso 3D sigue en banda (suelo 3).
El conteo de milestones limpios en banda es de proyecto; el umbral que debe superar es
del ramillete. Experiencia acumulada general contra exigencia de este caso.

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

### Autoridad sobre las entidades

El modelo del dominio es la autoridad sobre las entidades y sus campos. Todas las
disciplinas derivan sus representaciones de entidad del dominio; ninguna inventa
campos ni los renombra por su cuenta. Cuando dos disciplinas discrepan sobre los
campos de una entidad, el defecto se resuelve en el dominio, que fija el vocabulario
canónico, y las disciplinas se realinean a él. La deriva de vocabulario entre
disciplinas es un caso particular de fricción (ver sección macro) y se corrige hacia
arriba, en el dominio, no lateralmente.

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

### 2.3.1 Captura de requisitos no funcionales

Al cerrar el diagrama de contexto de cada actor se hace una primera revisión de
requisitos no funcionales que son visibles a ese nivel: rendimiento, seguridad,
disponibilidad, usabilidad como restricción. Se acumulan en un documento dedicado:
**Requisitos no funcionales / Especificación suplementaria**.

Al cerrar el detalle de cada caso de uso se hace una segunda revisión: los requisitos
no funcionales que emergen de la interacción concreta se añaden al mismo documento.

Las reglas de negocio no tienen artefacto propio: viven en el detalle del caso de
uso que las hace cumplir.

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

### Criterios de cierre de requisitos

Requisitos opera en dos cadencias y su delegabilidad es un gradiente, no un patrón
binario. Los cierres son dos:

**Cierre de base** (antes del primer ramillete): modelo del dominio completo, casos
de uso identificados y depurados (heurística CRUD más conversación con actores),
diagramas de contexto cerrados por actor, no funcionales de primer nivel capturados,
ramilletes priorizados por riesgo arquitectónico.

**Cierre de bucle** (antes de la pausa que abre análisis, por ramillete): casos de
uso del ramillete con detalle, prototipo y estructuración completados, no funcionales
de segundo nivel capturados. Coincide exactamente con el criterio de entrada que
análisis declara.

---

## Paso 5: Análisis

**Disciplina:** análisis

Es un modelo en sí mismo, no un
borrador del diseño.

**Criterio de entrada:** casos de uso del ramillete priorizados y especificados
(detalle + prototipo + estructuración completados), modelo del dominio, requisitos
no funcionales acumulados, y pausa arquitectónica post-requisitos resuelta.

**Propósito:** refinar los requisitos hacia el lenguaje de los desarrolladores,
produciendo una especificación más precisa y estructurada que sirve de entrada
esencial al diseño.

**Continuidad de artefactos:** el modelo del dominio no se transforma en clases de
análisis. Las clases de Modelo se derivan de él manteniendo trazabilidad. Dominio
y clases de análisis son artefactos distintos enlazados por trazabilidad, no
versiones sucesivas del mismo artefacto.

### A1. Analizar la arquitectura

Establece el esqueleto de clases de análisis antes de entrar caso por caso, bajo
el estilo MV*:

- **Clases de Vista:** una por actor humano (ventana principal de su interacción),
  una primitiva por cada clase de modelo, una central por cada actor que sea sistema
  externo.
- **Clases Controladoras:** una por caso de uso, o agrupaciones semánticamente
  cohesivas (~15 métodos por controlador compartido).
- **Clases de Modelo:** derivadas del modelo del dominio y de la descripción de los
  casos de uso.

**Dimensionado sistemático:** el estilo MV* permite estimar el volumen de software.
Un Modelo por entidad de negocio, una Vista por pantalla y componente reutilizable,
un Controlador por caso de uso (o agrupados), un Presentador por pantalla como
máximo. Más clases abstractas al jerarquizar (~5-10%). Paquetes a 12-20 clases.
Da una estimación temprana del tamaño y su reparto.

**Identificar paquetes de análisis:** agrupar casos de uso por actor o proceso de
negocio; extraer lo común a un paquete general del que dependan los demás; definir
dependencias siguiendo la dirección de las relaciones entre contenidos.

**Identificar requisitos especiales comunes:** persistencia, distribución/concurrencia,
seguridad, tolerancia a fallos, gestión de transacciones. Se capturan aquí para
gestionarlos en diseño, en diagramas de clases/paquetes estereotipados.

A1 no es delegable: es una decisión de estructura global que vive en el orquestador
y se confirma en la pausa arquitectónica.

### A2. Analizar casos de uso

**Principio rector:** analizar un caso de uso, cuando requisitos está bien hecho, es
derivación casi mecánica. Si cuesta, es que requisitos no estaba terminado. La
facilidad del análisis es el criterio de calidad de la disciplina anterior.

La derivación es directa en tres direcciones, sin información nueva:

- Un caso de uso -> un **controlador** que lo realiza.
- Su prototipo de interfaz -> una **vista** (deriva del prototipo de requisitos, no
  se inventa aquí).
- Las entidades del dominio que el caso de uso toca -> los **modelos** (solo las que
  participan, no todo el dominio).

Cada clase traza hacia atrás a un artefacto de requisitos: controlador <-> caso de
uso, vista <-> prototipo, modelos <-> entidades del dominio.

**Par entidad/repositorio:** una entidad del dominio que requiere persistencia se
representa en análisis como dos clases estereotipadas: la entidad y su repositorio
(`Entidad` y `EntidadRepository`), ambas agnósticas de tecnología. El repositorio en
análisis no es una decisión técnica: es el reconocimiento de que esa entidad se
persiste y se recupera, y es la forma concreta de capturar la persistencia como
requisito especial (lo que A1 manda identificar). En diseño ese par se concreta al
stack: el repositorio gana tecnología (queries, ORM) y, si la relación lo exige,
aparecen las tablas intermedias del modelo relacional.

**La dificultad como señal:** si analizar un caso de uso cuesta (no se derivan
limpiamente controlador, vista y modelos), la causa no suele estar en el análisis
sino aguas arriba: el caso de uso está mal requisitado o mal identificado. La
fricción en análisis es un detector de defectos de requisitos. No se retrocede de
fase; la dificultad se reporta como observación para corrección dentro de la fase
o para el milestone.

**Delegabilidad:** A2 es delegable a agentes subordinados (topología fan-out: un
agente por caso de uso, son independientes entre sí). A1 y A4 no son delegables.

Pasos formales de A2, para cada caso de uso del ramillete:

1. **Identificar clases de análisis:** las clases de Control, Entidad y Vista
   necesarias para realizarlo, con nombres, responsabilidades y relaciones, en un
   diagrama de clases.
2. **Describir las interacciones entre objetos de análisis:** mediante diagramas de
   colaboración/comunicación. El caso de uso lo invoca un mensaje del actor a un
   objeto vista. Cada clase identificada participa con al menos un objeto, por
   trazabilidad. El foco son las relaciones (enlaces) y los mensajes
   (responsabilidades), no la secuencia temporal.
3. **Capturar requisitos especiales:** los no funcionales que emergen del caso de
   uso concreto, identificados aquí y gestionados en diseño e implementación.

### A3. Analizar clases

Para cada clase de análisis, completar su definición reuniendo todos los roles que
juega en la realización de los distintos casos de uso:

- **Responsabilidades:** combinación de todos sus roles a lo largo de los casos de
  uso del ramillete.
- **Atributos:** nombre sustantivo, tipos conceptuales (no restringidos por
  implementación). Los de Entidad son obvios por trazabilidad con el dominio; los
  de Vista representan información que manipula el actor; los de Controladora son
  raros por su corto tiempo de vida. Si una clase se complica por sus atributos,
  se separan en clase propia.
- **Relaciones:** asociaciones y agregaciones (con multiplicidades y nombres de rol),
  y generalizaciones para extraer comportamiento común. Las generalizaciones se
  mantienen conceptuales y de alto nivel; pueden desaparecer en diseño.

### A4. Analizar paquetes

Mantener los paquetes cohesivos y poco acoplados: definir y mantener dependencias
sobre paquetes cuyas clases están asociadas, asegurar que cada paquete contiene solo
clases de funcionalidad relacionada, limitar dependencias externas (reubicar clases
dependientes de varios paquetes).

No delegable: es una decisión de estructura global.

### Artefactos producidos

Todos en PlantUML. Forman la Vista Lógica de las 4+1:

| Actividad | Artefacto | Cardinalidad | Tipo |
|---|---|---|---|
| A1 | Diagrama de paquetes de análisis | 1 por proyecto | transversal |
| A2 | Diagrama de clases + diagrama de comunicación | 1 por caso de uso | ramillete |
| A2 | Diagrama de clases de análisis consolidado | 1 por proyecto | transversal |
| A3 | Diagrama de clases con clases relacionadas (aferentes y eferentes) | 1 por clase | ramillete |
| A4 | Diagrama de paquetes con paquetes relacionados (aferentes y eferentes) | 1 por paquete | transversal |

El diagrama de clases de análisis consolidado reúne por convergencia todos los
controladores, vistas y modelos producidos caso a caso. Crece con cada ramillete
promovido; no se produce de una vez.

### Criterio de salida

Casos de uso del ramillete analizados (clases identificadas con responsabilidades,
atributos y relaciones completados), arquitectura de paquetes definida y revisada,
requisitos especiales capturados para diseño, y trazabilidad completa: cada clase
de análisis traza a los casos de uso que la requieren, cada objeto en colaboración
traza a su clase. Listo para la pausa arquitectónica que abre diseño.

---

## Paso 6: Diseño

**Disciplina:** diseño

A diferencia de análisis (tecnológicamente
agnóstico), diseño incorpora la tecnología concreta: lenguaje, framework, persistencia,
concurrencia, despliegue.

**Criterio de entrada:** modelo de análisis del ramillete, casos de uso, requisitos no
funcionales, requisitos especiales capturados en análisis, y pausa arquitectónica
post-análisis resuelta.

**Propósito:** desarrollar el modelo enfocado sobre los requisitos no funcionales y el
dominio de la solución, preparando implementación y pruebas. El diseño es un
refinamiento sin fisuras: la implementación debe rellenar la carne sin cambiar la
estructura.

**Continuidad de artefactos (derivación uno-a-muchos por stack):** las clases de diseño
se derivan de las de análisis añadiendo la dimensión tecnológica. Una clase de análisis
se proyecta en varias clases de diseño según el stack elegido. Ejemplo: una tríada MVC
de análisis se despliega en front Angular (XComponent/XService/XValidator/XTemplate)
más backend Spring (XController/XResource/XRepository/XEntity/XDto), todas con traza
`<<trace>>` a la clase de análisis. Derivación, no evolución.

### D1. Diseñar la arquitectura

Decisiones de estructura global. No delegable. **Es la pausa arquitectónica de máxima
retención humana de todo el pipeline:** aquí se elige el stack tecnológico (lenguaje,
framework, persistencia), decisión que compromete todo el ramillete y los siguientes.
En proyecto iniciático tira fuertemente hacia el humano; el LLM senior puede proponer,
idealmente alineado con los requisitos no funcionales, pero la decisión la retiene el
humano. Una vez fijado el stack, D2 y D3 se vuelven derivación mecánica delegable.

Actividades:
- **Identificar nodos y configuraciones de red:** diagramas de despliegue (nodos,
  conexiones, protocolos, anchos de banda, disponibilidad). La configuración física
  impacta la arquitectura del software.
- **Identificar subsistemas e interfaces:** partir de los paquetes de análisis,
  refinarlos por asuntos de diseño/implementación/despliegue. Identificar subsistemas
  middleware y de sistema (SO, gestor BD, comunicaciones, frameworks UI). Se refina
  respecto al paquete de análisis cuando varios paquetes caen en un subsistema, cuando
  no reflejan división de trabajo, cuando se usan productos reutilizados o sistemas
  heredados, o cuando no se despliegan bien en nodos.
- **Identificar clases de diseño arquitectónicamente significativas, clases activas**
  (por concurrencia) **y mecanismos de diseño genéricos** para los requisitos especiales
  que análisis marcó.

### D2. Diseñar casos de uso

Para cada caso de uso del ramillete. Delegable una vez fijado el stack:

- **Identificar clases de diseño participantes:** las que trazan a las clases de análisis
  del caso de uso, más las que realizan sus requisitos especiales vía mecanismos
  genéricos. Si falta alguna, se comunica a arquitectura.
- **Describir interacciones entre objetos de diseño:** mediante diagramas de secuencia
  (no colaboración como en análisis). El orden cronológico de mensajes es el foco porque
  la realización-diseño es la entrada directa a implementación. Cada clase de diseño
  tiene al menos un objeto. Los mensajes se convertirán en operaciones.
- **Descubrir caminos alternativos y excepciones nuevas** no vistas en requisitos ni
  análisis: timeouts por caída de nodos/conexiones, entradas erróneas de actores.
- **Capturar requisitos de implementación:** los no funcionales que emergen en diseño
  pero se tratan en implementación.

### D3. Diseñar clases

Completar cada clase de diseño con tecnología:

- **Clases de Vista:** dependen de la tecnología de interfaz elegida.
- **Clases de Entidad persistentes:** implican tecnología de BD; problema central: la
  proyección objeto-relacional.
- **Clases de Control:** delicadas (encapsulan secuencia, coordinación, lógica de
  negocio). Considerar distribución (separar por nodos), rendimiento (a veces fundir
  control en vista/entidad), transacciones (incorporar tecnología transaccional).
- **Operaciones, atributos, relaciones:** derivados de las clases de análisis que traza,
  restringidos por el lenguaje. Minimizar relaciones. La navegabilidad la determina la
  dirección de mensajes en los diagramas de interacción.
- **Estados:** diagrama de estados para objetos controlados por estado.
- **Métodos:** normalmente no se especifican en diseño.

### D4. Diseñar subsistemas

Cohesión y bajo acoplamiento: mejor depender de una interfaz que de un subsistema
(permite sustituir implementación); interfaces refinables según evoluciona el diseño;
el subsistema cumple las operaciones de las interfaces que presta. No delegable.

### Artefactos producidos

| Actividad | Artefacto | Cardinalidad | Tipo |
|---|---|---|---|
| D1 | Diagrama de paquetes de diseño | 1 por proyecto | transversal |
| D2 | Diagrama de clases + diagrama de secuencia | 1 por caso de uso | ramillete |
| D3 | Diagrama de clases con clases relacionadas | 1 por clase | ramillete |
| D3 | Diagrama de clases de diseño consolidado | 1 por proyecto | transversal |
| D4 | Diagrama de paquetes con subsistemas | 1 por subsistema | transversal |
| D1 | Vista de Implementación | 1 por proyecto | transversal |
| D1 | Vista de Despliegue | 1 por proyecto | transversal |

El diagrama de clases de diseño consolidado reúne por convergencia todas las clases
de diseño producidas caso a caso. Crece con cada ramillete promovido; no se produce
de una vez.

### Criterio de salida

Casos de uso del ramillete diseñados (clases completas con tecnología, secuencias
descritas), arquitectura de subsistemas y despliegue definida, requisitos especiales
resueltos o pospuestos explícitamente a implementación, trazabilidad completa
análisis -> diseño. Listo para la pausa arquitectónica que abre implementación.

**Nota sobre la delegabilidad creciente:** a medida que el stack se asienta y los
patrones se establecen, más parte de diseño se vuelve derivación mecánica (D2/D3) y
menos requiere decisión humana (D1). El primer gradiente de la pausa arquitectónica
(madurez del proyecto) se desplaza hacia el LLM precisamente por esto.

---

## Paso 7: Implementación

**Disciplina:** implementación

Implementa el sistema en términos de
componentes: código fuente, binarios, ejecutables.

**Criterio de entrada:** modelo de diseño del ramillete (clases de diseño completas
con tecnología, secuencias), modelo de despliegue, requisitos de implementación
capturados en diseño, y pausa arquitectónica post-diseño resuelta.

**Propósito:** organizar el código en subsistemas de implementación por capas,
implementar las clases como componentes, probarlos como unidades e integrarlos en
un sistema ejecutable. El diseño fue un refinamiento sin fisuras: implementar es
rellenar la carne sin cambiar la estructura.

**La construcción (build):** unidad interna de la implementación de un ramillete. Un
ramillete se implementa en una o varias construcciones, cada una un incremento
integrable que se basa en la anterior. La construcción vive dentro del tramo de
implementación del ramillete; no cruza ramilletes.

### I1. Implementar la arquitectura

No delegable. Identificar los componentes de importancia arquitectónica y los
componentes ejecutables y su correspondencia con los nodos del despliegue. Asignar
las clases activas (encontradas en diseño) a componentes ejecutables. Precaución:
no identificar demasiados componentes ni ahondar en detalle prematuro. Comparada
con la pausa de diseño es la menor del pipeline: el stack ya está fijado; aquí solo
se confirma el mapa componente-nodo.

### I2. Integrar sistemas

No delegable. Es el núcleo no trivial de la disciplina: planifica y ejecuta las
construcciones del ramillete. Tres reglas:

- **Orden por capas:** la construcción inicial empieza por las capas inferiores; cada
  construcción se basa en la anterior y expande hacia arriba y por los lados de la
  jerarquía de subsistemas. Lo que sostiene a otros se construye primero.
- **Saturación por construcción:** una construcción no debe incluir demasiados
  componentes nuevos o refinados, porque integrar y probar la acumulación se vuelve
  inmanejable. Si hace falta, se usan stubs/dobles para minimizar componentes nuevos.
- **Integración por convergencia:** integrar una construcción es recopilar las
  versiones correctas de los componentes, compilarlas y enlazarlas. Es el fan-in del
  código: los agentes produjeron componentes en paralelo (fan-out en I3), la
  integración los reúne. La compilación conjunta es el test más duro de que la
  orquestación funcionó: si los componentes no encajan, falla aquí.

La construcción que pasa integración se somete a pruebas del sistema (Paso 8) si es
la última de la iteración.

### I3. Implementar clase

La actividad más delegable de todo el pipeline. Dada la clase de diseño con su
tecnología, generar el código es derivación mecánica sin juicio arquitectónico.
Topología fan-out: un agente por clase o componente, independientes entre sí hasta
la integración.

Para cada clase:
- Vincular el código fuente a un fichero de componente (es común implementar varias
  clases de diseño en un fichero; las convenciones del lenguaje restringen cómo se
  delinean).
- Generar el código de operaciones, atributos y relaciones de la clase.
- Implementar cada operación (salvo virtuales/abstractas, que implementan los
  descendientes): elegir algoritmo y estructuras de datos, codificar. Los estados
  descritos en diseño impactan cómo se implementan las operaciones.
- El componente resultante debe proveer las mismas interfaces que las clases de diseño
  que implementa.

### I4. Realizar prueba de unidad

El implementador prueba cada componente como unidad, pegado a la clase que implementa.
Dos tipos:

- **Prueba de especificación (caja negra):** verifica el comportamiento externo
  observable, sin mirar la implementación interna.
- **Prueba de estructura (caja blanca):** verifica la implementación interna,
  cubriendo todo el código.

La prueba de unidad vive en implementación porque es del implementador y va pegada al
componente. Las pruebas de integración y de sistema pertenecen a la disciplina de
pruebas (Paso 8).

### Artefactos producidos

| Actividad | Artefacto | Cardinalidad | Tipo |
|---|---|---|---|
| I1 | Modelo de implementación (componentes y su traza a subsistemas) | 1 por proyecto | transversal |
| I2 | Plan de integración de construcciones | 1 por ramillete | ramillete |
| I3 | Componentes (código fuente, binarios, ejecutables) | 1 por clase/componente | ramillete |
| I4 | Resultados de prueba unitaria | 1 por componente | ramillete |

### Criterio de salida

Componentes del ramillete implementados y probados como unidades, construcciones
integradas en orden de capas, sistema ejecutable que compila y pasa integración,
trazabilidad completa diseño -> código. Listo para la pausa que abre pruebas.

---

## Paso 8: Pruebas

**Disciplina:** pruebas

Verifica por ejecución que el ramillete construido realiza sus casos de uso y cumple sus
requisitos no funcionales. Cubre integración y sistema; la prueba de unidad ya se hizo
en implementación (I4), pegada al componente. Los artefactos de especificación de prueba
se expresan en el mismo ecosistema que el resto (PlantUML donde aplique); el código de
prueba, en el stack de la solución.

**Criterio de entrada:** sistema ejecutable del ramillete que compila y pasa integración
de construcciones (criterio de salida de implementación), modelo de implementación,
casos de uso con su detalle y prototipo, requisitos no funcionales acumulados, y pausa
arquitectónica post-implementación resuelta.

**Propósito:** ejercitar el ramillete integrado contra su especificación para confirmar
que lo construido es lo requerido, y atribuir cualquier defecto a su origen. Es el
detector de fricción más objetivo del pipeline: donde en análisis la detecta el juicio y
en implementación el compilador, en pruebas la detecta la ejecución, sin opinión.

**Continuidad de artefactos:** los casos de prueba se derivan del detalle del caso de uso
(flujos e interacción) y de su prototipo (contrato de interacción), con trazabilidad
directa. No añaden información: transcriben la especificación a verificación ejecutable.

### T1. Planificar la prueba

Establece la estrategia y el alcance antes de entrar caso por caso: qué se prueba
(integración entre componentes del ramillete, sistema contra casos de uso y contra no
funcionales), criterios de aceptación derivados de la especificación, y orden de prueba
según la integración por capas que dejó implementación. No delegable: es decisión de
estructura, vive en el orquestador y se confirma en la pausa. Transversal: el plan de
pruebas del proyecto crece con cada ramillete.

### T2. Diseñar los casos de prueba

Para cada caso de uso del ramillete, derivar los casos de prueba. Delegable, topología
fan-out: un agente por caso de uso, independientes entre sí. Ramillete.

- **Prueba de especificación (caja negra):** del detalle del caso de uso salen los flujos
  a verificar; del prototipo, las entradas, las salidas observables y los códigos de
  resultado. Cada caso de prueba traza a un caso de uso.
- Los caminos alternativos y las excepciones que diseño descubrió (timeouts, entradas
  erróneas) son casos de prueba obligatorios, no opcionales.

La cobertura estructural (caja blanca) ya se cubrió a nivel de unidad en I4; aquí la
prueba es de comportamiento contra la especificación.

### T3. Implementar la prueba

Automatizar los casos de prueba diseñados en el stack de prueba que corresponde al stack
de la solución (fijado en diseño). Una vez fijado el caso de prueba y el stack, es
derivación mecánica. Delegable, topología fan-out: un agente por caso de uso o
componente. Ramillete.

### T4. Ejecutar y evaluar

Ejecutar la prueba sobre el sistema integrado y evaluar el resultado. Topología cycle,
con moderador:

```
ejecutar -> fallo -> atribuir el defecto a su origen aguas arriba -> corregir -> reejecutar
```

El moderador (papel no delegable) decide en cada vuelta una de dos cosas: si el defecto
cae dentro del ramillete y se corrige hacia delante hasta que la prueba pasa, o si
pertenece a una fase anterior, y entonces no se corrige aquí: se registra como
observación para el milestone, que no retrocede de fase (regla de no-retroceso). La
atribución sigue la regla de validación entre disciplinas: la fricción se imputa a su
origen, no a la disciplina en curso. Las vueltas del ciclo (ejecutar y corregir) son
delegables; la moderación no.

### Artefactos producidos

| Actividad | Artefacto | Cardinalidad | Tipo |
|---|---|---|---|
| T1 | Plan de pruebas | 1 por proyecto | transversal |
| T2 | Casos de prueba (especificación) | 1 por caso de uso | ramillete |
| T3 | Pruebas automatizadas (código de prueba) | 1 por caso de uso | ramillete |
| T4 | Resultados de ejecución y defectos atribuidos | 1 por ramillete | ramillete |

### Criterio de salida

Ramillete verificado en integración y sistema, casos de prueba trazados a casos de uso,
requisitos no funcionales comprobados, defectos residuales atribuidos a su origen
(corregidos hacia delante dentro de la fase, o registrados como observación del
milestone). El ramillete pasa en verde lo que es suyo. Es la última disciplina antes del
milestone que cierra la fase: su criterio de salida es la evidencia que el milestone
evalúa.

---

<!-- gestion-config y gestion-proyecto: disciplinas de soporte, fuera del pipeline por ramillete; tratamiento pendiente de decidir -->
