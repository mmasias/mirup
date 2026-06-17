# Registro de decisiones descartadas — miRUP / CORRAL-RUP

**Heurística de estimación 2-20h por actividad (de Luis).** Descartada como criterio. Era gestión humana de tiempo y reparto de carga entre personas; CORRAL no estima esfuerzo para planificar sprints. Se conservó solo su mitad cualitativa (la dificultad como señal de defecto aguas arriba), que es atemporal; la cuantitativa se fue.

**Tercer rol explícito por actividad (humano/LLM/becario).** Propuesto por Claude Code al diseñar pruebas. Descartado. El reparto humano/LLM no se cablea por actividad porque está diseñado para moverse con la madurez (es el mecanismo de aprendizaje). Fijarlo por actividad lo congela. El tercer rol ya existe como posición móvil en el área, no como etiqueta fija. Con él se descartó también la retro-pasada sobre las disciplinas 1-7 que habría requerido.

**Reparto humano/LLM como línea (diagonal).** Descartado a favor de área. Una línea asigna reparto por punto y mata la banda donde viven el aprendizaje (coexistencia humano/LLM sobre la misma tarea) y la libertad del orquestador (situarse en el borde). El área es difusa a propósito; la frontera nítida sería falsa precisión sobre la confianza, que es un degradado, no un umbral.

**Región de la pausa como función del contador instantáneo.** Descartado. Cualquier función de un escalar a tres regiones es una escalera: existe un valor donde salta de banda a delegada de golpe (la línea por la puerta del contador). Se sustituyó por un pliegue sobre el historial ordenado de milestones: la path-dependence es lo que rompe el escalón. La banda es peaje obligatorio de residencia, no intervalo que se vacía.

**Persistir el riesgo acumulado (cachear el max).** Descartado. Persistir un derivado crea dos fuentes de verdad y queda obsoleto si se corrige un riesgo aguas arriba. Se usa replay (pliegue max sobre la cadena de pausas), coste nulo (cinco valores por ramillete). Es el principio de no-persistir-derivados, el mismo por el que la región tampoco se persiste.

**Derivar el nivel de riesgo del orden de priorización 2.4.** Descartado. El orden de 2.4 es ordinal y relativo al proyecto (secuencia ramilletes); la residencia necesita escala absoluta pequeña. Y 2.4 solo conoce el riesgo de requisitos, no el que cada disciplina introduce después (el 3D que aparece en diseño). Se usa nivel discreto de tres valores (bajo/medio/alto = residencia 1/2/3), juicio de pausa, sin multiplicador intermedio.

**Multiplicador nivel→residencia.** Descartado. Un factor entre nivel y residencia sería el parámetro mágico. Identidad directa: el nivel es el número de milestones de residencia.

**Milestone como decisión binaria (promote/discard).** Descartado (hoy, con el auditor). El milestone es ternario: aprobar, corregir local hacia delante, o resetear. El binario se saltaba la corrección local que el protocolo ya tenía. El selector entre corregir y resetear es el alcance del defecto vía la atribución existente.

**Meter la pausa en el esquema de slots del Contrato 1.** Descartado. La pausa es objeto de gobierno, hermana del milestone, no artefacto de producto. El Contrato 1 nunca fue para los objetos de gobierno. No es excepción al contrato: es reconocer una segunda categoría (espacio de gobierno) que ya operaba sin nombre (el milestone ya colgaba fuera de los slots).

**Separar la sección de gobierno a un documento aparte (CORRAL-RUP puro).** Considerado y descartado por decisión tuya. miRUP es miRUP+CORRAL-RUP a propósito; el "mi" abarca método y encarnación. La frontera se mantiene entre secciones, no entre ficheros. La ortogonalidad sigue disponible si algún día hace falta desacoplar, pero no se ejerce ahora.

**Engram (búsqueda semántica entre proyectos).** No instalar. Roles 1-2 cubiertos por `~/.claude/projects/` y el registry; solo el rol 3 sería ortogonal, pero la factorización RUP hace navegacionales casi todas las búsquedas. Reevaluar solo si aparece necesidad recurrente de búsqueda por contenido en diagonal a los ejes RUP.

**De las 6 Delegation Trigger Rules de gentle-ai:** solo 2 fueron a CLAUDE.md global (incident rule; review adversarial con contexto fresco). Multi-file write ya es topología chain; 4-file y long-session son el mismo mecanismo de saturación (van al protocolo de iteración). Documentado con sección de exclusiones para no rediscutirlas.

**Corregir editarAula / crearEdificio para usarlo de ejemplo canónico.** Descartado a favor de elegir un ramillete que cierre limpio. Aprendizaje colateral: ningún caso de pySigHor está limpio de fábrica (deriva real entre disciplinas). El ejemplo canónico será un ramillete depurado aplicando el método, no uno hallado perfecto. Criterio de depuración: recorrer la columna preguntando de qué deriva cada cosa, que no aparezca nada huérfano.

**Elevar el principio de validación entre disciplinas en cuanto apareció (con una instancia).** Se esperó a tener evidencia en varias disciplinas antes de subirlo a la sección macro. Se elevó cuando hubo tres instancias reales (recursos de editarAula, código de crearEdificio, vocabulario de Edificio). Criterio general: no elevar a principio con una sola instancia.
