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

### 2.3 Diagrama de estados del sistema

Para cada actor se construye un diagrama de estados del sistema. Este es el artefacto
central de la disciplina de requisitos.

**Qué modela:** estados del sistema desde la perspectiva del actor — no estados de la
interfaz, no estados del dominio, sino contexto operacional. Un estado como
`CURSO_ABIERTO` no representa una pantalla: representa que hay una entidad activa y
que las operaciones disponibles están acotadas a ese contexto. El diagrama es
tecnológicamente agnóstico: la misma especificación describe el comportamiento del
sistema con independencia de si la interacción se realiza mediante GUI, CLI, API REST
o interfaz conversacional.

**Cómo se construye:** los estados son los contextos operacionales del actor; las
transiciones son casos de uso. Solo se puede transitar entre estados mediante casos de
uso identificados. Si para ir de un estado a otro no existe ningún caso de uso, o bien
falta un caso de uso o bien la transición no debería existir.

**Propiedades:**

El diagrama actúa como validador bidireccional del modelo de casos de uso:
- Transición sin caso de uso -> el caso de uso falta o la transición es incorrecta.
- Caso de uso sin ninguna transición que lo use -> su existencia es cuestionable.

Las relaciones `<<include>>` son consecuencias topológicas del diagrama, no decisiones
de diseño. Si la única ruta al estado donde ocurre `eliminarX` pasa por `abrirXs`,
entonces `eliminarX` incluye `abrirXs` necesariamente.

El diagrama es una especificación generativa: a partir de él se puede inferir
sistemáticamente navegación, endpoints, permisos, tests y documentación, porque
contiene la lógica operacional completa del sistema para ese actor.

---

<!-- paso 3 y siguientes: en construcción -->
