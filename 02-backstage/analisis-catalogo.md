
## Nombre: Luis Alejandro Sarmiento 

# Análisis del Catálogo de APIs en Backstage

## 1. Información que muestra Backstage sobre las APIs sincronizadas desde Microcks

Al conectar el provider `MicrocksApiEntityProvider`, Backstage descubrió automáticamente 2 ApiEntities desde la instancia de Microcks (`Discovered ApiEntity Petstore - 1.0.0` y `Discovered ApiEntity TaskManager - 1.0.0` en el log del backend) y las agregó al catálogo junto con la entidad de ejemplo preexistente `example-grpc-api`.

En la vista "API Explorer" (`/api-docs`), cada API sincronizada muestra:

- **Name**: nombre + versión del contrato (`Petstore_1.0.0`, `TaskManager_1.0.0`).
- **System**: `microcks` para ambas — este valor viene de la configuración `systemLabel: domain` en `app-config.yaml`; como nuestros contratos OpenAPI no definen una etiqueta `domain` propia, el provider usa un valor por defecto.
- **Owner**: `microcks` — mismo mecanismo, mapeado desde `ownerLabel: team`.
- **Type**: `openapi` — detectado automáticamente a partir del formato del contrato (sería `asyncapi`, `graphql` o `grpc` para otros protocolos, como se ve en `example-grpc-api`).
- **Lifecycle**: `dev` — corresponde a la clave del entorno que configuramos en `catalog.providers.microcksApiEntity.dev` del `app-config.yaml` (si tuviéramos varios entornos de Microcks —dev, staging, prod— cada uno aparecería con su propio lifecycle).
- **Description**: extraída directamente del campo `info.description` del contrato OpenAPI.

Al hacer clic en una API individual, Backstage abre una ficha con pestañas (Overview, Definition, Dependencies) y un enlace "View Documentation" que redirige de vuelta a la ficha nativa de Microcks — ahí sí se ve el detalle operativo completo que Backstage no replica: dispatcher por operación, samples/mocks disponibles, índice de conformancia (Conformance index/score), invocaciones registradas y el servidor MCP expuesto por Microcks. Es decir, Backstage actúa como capa de descubrimiento e indexación (qué APIs existen, quién es dueño, en qué contrato se basan), mientras que Microcks sigue siendo la fuente de verdad operativa (mocks, tests de conformidad, métricas de uso).

## 2. Descubrimiento automático vs. registro manual en un equipo de 50+ ingenieros

Con el registro manual, cada equipo tendría que crear y mantener a mano un `catalog-info.yaml` por API, abrir un PR, y acordarse de actualizarlo cada vez que cambia el owner, el lifecycle o la versión del contrato. En una organización de 50+ ingenieros repartidos en varios equipos esto falla de forma predecible:

- **Desactualización**: nadie actualiza el catálogo cuando una API cambia de versión o se deprecía; el catálogo termina mostrando información que ya no es cierta.
- **APIs huérfanas o duplicadas**: equipos que no saben que una API ya fue registrada por otro, o que olvidan registrar la suya.
- **Cuello de botella de PRs**: agregar o corregir una entrada depende de que alguien revise y apruebe el cambio en el repositorio del catálogo.
- **Conocimiento tribal**: "¿quién es el dueño de esta API?" termina respondiéndose preguntando en Slack, no consultando el catálogo.

El descubrimiento automático (el provider de Microcks corriendo cada 2 minutos, como se configuró en `schedule: frequency: { minutes: 2 }`) resuelve esto de raíz: en cuanto un equipo publica un contrato nuevo en Microcks, aparece en Backstage sin que nadie tenga que escribir ni revisar un YAML de catálogo. Además, al ser una sincronización completa (`Applying the mutation with N entities`), el catálogo se autocorrige: si una API se elimina de Microcks, desaparece también del catálogo en el siguiente ciclo, evitando entradas fantasma. Esto es exactamente lo que se necesita a partir de cierta escala de equipo: una fuente única y siempre actualizada de qué APIs existen, sin depender de disciplina manual.

## 3. Diferencia entre `Component` y `API` en el modelo de datos de Backstage

Un **`Component`** representa una pieza de software que un equipo construye y despliega — un servicio, un sitio web, una librería. Es la unidad de "esto es código que corremos".

Una **`API`** representa un contrato o interfaz expuesta — independiente de quién la implementa. En este ejercicio, Microcks solo conoce el contrato OpenAPI (`Petstore`, `TaskManager`), no el servicio real que lo implementaría, por lo que estas entidades quedaron en el catálogo como `API` "sueltas", sin ningún `Component` asociado.

En un catálogo completo, la relación se modela explícitamente: un `Component` declara `providesApis: [TaskManager]` si es el servicio que implementa esa API, o `consumesApis: [...]` si depende de una API externa. Esta separación importa porque desacopla "qué interfaz existe" de "quién la implementa": una misma API puede tener múltiples implementaciones (por ejemplo, durante una migración de un monolito a microservicios), y un mismo `Component` puede proveer o consumir varias APIs distintas. Es lo que permite construir el grafo de dependencias que se ve en "Catalog Graph" — quién depende de quién a nivel de contrato, no de código.
