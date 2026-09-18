# Guía de estudio — Tema 2: Bases de Datos NoSQL

*Asignatura: Arquitectura de Datos (4º Grado Ingeniería Informática, UC3M)*

## 0. Resumen general y objetivos

Este tema presenta las **bases de datos NoSQL** como respuesta a las limitaciones del modelo relacional cuando se trabaja con grandes volúmenes de datos, alta velocidad de crecimiento y necesidad de escalar horizontalmente. La idea central que atraviesa todo el tema es:

> **No existe una base de datos universal.** La elección de la tecnología depende del problema: qué datos hay, cómo se consultan, cuánto volumen se maneja y qué garantías de consistencia/disponibilidad se necesitan.

Los objetivos de aprendizaje declarados en las diapositivas son:

1. **Describir** la arquitectura general de una plataforma de datos moderna y los principios de las BBDD NoSQL.
2. **Reconocer y diferenciar** los distintos modelos de datos (relacional, clave-valor, documentos, columnas y grafos), analizando sus características y aplicaciones prácticas.

El tema se organiza en tres bloques:

- **2.1. Introducción a las BBDD NoSQL** — contexto histórico, normalización, escalabilidad, consistencia, ventajas/inconvenientes, comparación con relacional.
- **2.2. Modelos de datos / Modelos de agregación** — clave-valor, documentos, columnas, grafos.
- **2.3. Selección de una BD NoSQL** — criterios de decisión y ejemplos prácticos.

---

## 1. Introducción a las bases de datos NoSQL

### 1.1 Contexto: el "orden antiguo" vs el "nuevo orden"

Durante décadas, el modelo relacional fue el estándar universal para almacenar datos. Este **"orden antiguo"** se basaba en:

- Un **modelo universal**: la estructura de los datos es **estática y fija**, mientras que el procesamiento (las consultas) es lo dinámico.
- **Integración funcional** en torno a un **modelo de datos único** para toda la organización.
- **Estandarización**: una arquitectura común y un lenguaje de acceso único (SQL).
- El concepto de **transacción** para controlar el acceso concurrente y garantizar la consistencia.

Con la llegada de internet, las redes sociales y el Big Data, apareció un **"nuevo orden"** con premisas distintas:

- El foco pasa a estar en el **volumen** y la **velocidad** de los datos, no solo en su estructura.
- **No siempre es imprescindible** la integridad y coherencia totales (a veces basta con que el sistema "se ponga al día" más tarde).
- **No existe una solución universal, única y perfecta** para todos los problemas.
- Filosofía de **pragmatismo, flexibilidad y minimalismo**.
- Contraposición entre modelos **BASE** y **ACID** (se explica en detalle más abajo).

**Idea clave a recordar:** *"No empieces por la herramienta. Empieza por el problema: qué datos, qué consultas, qué volumen y qué garantías necesitas."* Este mensaje se repite de formas distintas en casi todas las diapositivas del tema y es, probablemente, la idea más importante para el examen: **NoSQL no es "mejor" que SQL, es una alternativa que se ajusta a otro tipo de problemas.**

**Ejemplo para fijar la idea:** Imagina que gestionas los datos de un hospital (historiales clínicos con relaciones estrictas entre pacientes, médicos, tratamientos y facturación) frente a los datos de una red social (millones de publicaciones, likes y comentarios que crecen sin parar y que no necesitan una consistencia perfecta al instante). El primer caso pide un modelo relacional con transacciones ACID; el segundo, un modelo NoSQL optimizado para escritura masiva y disponibilidad.

### 1.2 Normalización

La **normalización** es un proceso característico del modelo relacional que persigue:

- **Reducir la redundancia** de los datos.
- **Aumentar la coherencia**, protegiendo frente a errores de actualización (si un dato solo existe "en un sitio", actualizarlo es seguro y consistente).

**Mecanismo:** cada pieza de información se ubica en un único lugar, y el resto de elementos que la necesitan acceden a ella **por referencia** (claves foráneas). Esto genera una **red de vínculos** que hay que recorrer (mediante `JOIN`) para reconstruir la información tal como la solemos usar.

**El problema de la normalización no es que sea mala idea — siempre es deseable en abstracto — sino su coste:**

- El mecanismo que garantiza la coherencia (la normalización) tiene un **impacto directo en el rendimiento**, especialmente en lectura, porque cada consulta que junta información relacionada necesita `JOIN`s.
- Existe un compromiso (trade-off) entre:
  - **Normalización**: máxima coherencia, pero rendimiento de lectura penalizado.
  - **Desnormalización**: mejor rendimiento de lectura, pero riesgo de inconsistencias y redundancia.
  - **Enfoques mixtos**: un término medio.

**Pregunta que debes hacerte (y que aparece literalmente en las diapositivas):** *si la consulta más frecuente exige muchos `JOIN`, ¿conviene almacenar el dato de otra forma para ese caso de uso?* Esta pregunta es el germen de la idea de **agregado**, que se explica más adelante.

**Ejemplo concreto:** En un e-commerce relacional, para mostrar un pedido completo con sus líneas y la dirección de envío, hay que hacer `JOIN` entre `Pedido`, `LineaPedido`, `Producto` y `Direccion`. Si esta consulta se ejecuta miles de veces por segundo, el coste de los `JOIN`s puede ser el cuello de botella del sistema.

### 1.3 Escalabilidad

Existen dos estrategias para que un sistema soporte más carga:

| Estrategia | También llamada | Qué significa | Asociada tradicionalmente a |
|---|---|---|---|
| **Escalado vertical** | *Scale up* | Usar una máquina más potente (más CPU, más RAM) | BBDD relacionales |
| **Escalado horizontal** | *Scale out* | Añadir más nodos que trabajan conjuntamente | BBDD NoSQL |

En las BBDD NoSQL, el procesamiento y el almacenamiento se distribuyen a través de una **red de nodos**. La escalabilidad horizontal permite crecer según las necesidades **añadiendo nodos "en caliente"** (sin afectar a la disponibilidad del sistema, es decir, sin parar el servicio).

**Arquitectura distribuida — Fragmentación (sharding):**

En arquitecturas distribuidas, los datos se **fragmentan** para repartirse entre nodos. Hay dos formas:

- **Fragmentación horizontal (sharding):** se divide por **filas**. Se usa habitualmente en BBDD clave-valor (mediante técnicas de dispersión basadas en *hash*) y en BBDD orientadas a documentos/columnas (en función del valor de ciertos atributos).
- **Fragmentación vertical:** se divide por **columnas** (típico de BBDD orientadas a columnas).

Además, estas arquitecturas se caracterizan por la **replicación masiva** de los datos en diferentes servidores, lo que aporta:

- Mayor **paralelismo**.
- Mejor **eficiencia de consultas**.
- Tolerancia a fallos (si un nodo cae, otro tiene una copia).

**Ejemplo:** Cassandra reparte las filas de una tabla entre distintos nodos del clúster usando una función *hash* sobre la clave de partición. Si tienes 100 millones de filas y 10 nodos, cada nodo almacena aproximadamente una fracción de esas filas (más las réplicas que le correspondan), y se pueden añadir más nodos para repartir aún más la carga sin detener el sistema.

### 1.4 Modelos de consistencia: ACID vs BASE

Este es uno de los conceptos **más importantes** del tema.

**Modelo relacional → ACID:**

| Propiedad | Significado |
|---|---|
| **A**tomicidad | Todo o nada: una transacción se ejecuta completa o no se ejecuta |
| **C**onsistencia | Los datos pasan de un estado válido a otro estado válido (coherencia) |
| **I**solation (Aislamiento) | Las transacciones concurrentes se comportan como si se ejecutaran en serie (serialización) |
| **D**urabilidad | Una vez confirmados, los cambios son permanentes |

**Modelo NoSQL → BASE:**

| Propiedad | Significado |
|---|---|
| **B**asically Available (Disponibilidad básica) | El sistema garantiza disponibilidad de los datos, aunque no siempre perfecta |
| **S**oft state (Estado flexible/transitorio) | El estado del sistema puede cambiar con el tiempo, incluso sin nueva entrada de datos (por la propagación de la consistencia) |
| **E**ventual consistency (Consistencia eventual) | Con el tiempo, si no llegan nuevas actualizaciones, todos los nodos acabarán convergiendo al mismo valor |

**Aviso importante que dan las diapositivas explícitamente:** *"NoSQL no significa 'sin transacciones' ni 'sin consistencia': cada tecnología ofrece garantías concretas."* No hay que pensar que "NoSQL = sin garantías". Cada motor (MongoDB, Cassandra, Redis...) implementa su propio nivel de consistencia configurable.

**Ejemplo de consistencia eventual:** Publicas una foto en una red social distribuida en varios centros de datos. Un amigo que está conectado al mismo centro de datos que tú la ve al instante, pero otro amigo conectado a un centro de datos en otro continente puede tardar unos segundos en verla, porque el dato todavía se está replicando. Pasado ese breve intervalo, ambos verán el mismo estado: eso es consistencia eventual, frente a la consistencia fuerte de un banco, donde una transferencia debe reflejarse instantáneamente e igual en todas partes.

### 1.5 Ventajas de las BBDD NoSQL

- Enfoque hacia **sistemas abiertos**.
- Orientación al **clúster masivo** y muy poco acoplado (nodos independientes entre sí).
- Orientación hacia **datos no estructurados**:
  - **Ausencia de esquema** (schemaless).
  - **Relajación** de la integridad y consistencia.
  - **Flexibilidad** para adaptarse a nuevas situaciones.
- Son sistemas **políglotas**: permiten usar cada herramienta/producto donde es más efectivo (concepto de *persistencia políglota*, que se retoma en el apartado 3).
- **No hay lenguaje estándar** de acceso: no existe un "SQL universal" para NoSQL. Cada tecnología tiene su propio lenguaje de manipulación o su API, y se ofrecen *drivers* para distintos lenguajes de programación.

### 1.6 Inconvenientes de las BBDD NoSQL

- **No soportan modelos de fiabilidad fuerte** como ACID en general: son inadecuados para sistemas operacionales que exigen máxima integridad (p. ej. sistema transaccional bancario).
- **Trasladan complejidad al código de las aplicaciones**: si la BD no impone estructura, es la aplicación la que debe encargarse de mantenerla correctamente.
- **No hay independencia lógica** (los cambios en el modelo de datos suelen impactar directamente en el código).
- **No hay estándar**: baja portabilidad entre tecnologías distintas.
- **Esquema de datos complicado** de gestionar sin disciplina.
- La **administración, seguridad y observabilidad** dependen mucho de la tecnología concreta y su ecosistema.
- **Madurez desigual** según tecnología, proveedor y ecosistema.
- **Curva de aprendizaje** significativa.

### 1.7 Comparación BBDD Relacionales vs NoSQL

| Aspecto | BBDD Relacionales | BBDD NoSQL |
|---|---|---|
| Modelo de datos | Modelo de datos único (tablas relacionales) | Distintos tipos de modelos de datos (clave-valor, documentos, columnas, grafos) |
| Esquema | Esquema fijo | Esquema flexible |
| Tipo de datos | Datos estructurados | Datos estructurados, semiestructurados y no estructurados |
| Distribución | Sin distribuir / distribuidas con modelos sencillos | Altamente distribuidas |
| Garantías | Propiedades ACID | Propiedades BASE |

### 1.8 Conclusiones del bloque introductorio

Se necesita una BD NoSQL si se cumple **al menos una** de estas condiciones:

1. El entorno de la aplicación requiere **esquemas de datos flexibles** (en comparación con lo que ofrecen los relacionales).
2. Es un sistema **altamente distribuido** que necesita gestionar **grandes volúmenes de datos**, los cuales deben estar **siempre disponibles**.

**Criterios de elección del tipo de BD NoSQL** (se retoman ampliados en el apartado 3):

- Volumen de datos.
- Concurrencia estimada.
- Número de operaciones por unidad de tiempo.
- Tipos de operaciones más frecuentes.
- Escalabilidad deseada.
- Grado de integridad y consistencia deseado.
- Naturaleza de los datos.

---

## 2. Modelos de datos y modelos de agregación

### 2.1 El concepto de "agregado"

Este es **el concepto central** de todo el bloque de modelos de datos NoSQL, así que conviene entenderlo muy bien.

> Un **agregado** es un **conjunto de datos relacionados** que se gestiona **como una sola unidad**.

Características del agregado:

- Se identifica por una **clave única**.
- Es la **unidad mínima** de almacenamiento y de consulta.
- Los **cambios se aplican al agregado completo** (no a "partes sueltas" repartidas en distintas tablas).

**Comparación con el modelo relacional (ejemplo "Pedido"):**

- **Relacional:** el Pedido vive repartido en varias tablas — `Pedido` (cabecera) + `LineaPedido` (detalle en tabla aparte) — relacionadas por claves foráneas. Para recuperarlo completo hace falta un `JOIN`.
- **Agregado (NoSQL):** el Pedido es **un único documento JSON** que contiene, incrustadas dentro de sí, sus líneas de pedido y su dirección. Recuperar el pedido completo es una sola lectura, sin `JOIN`.

| Aspecto | Relacional (SQL) | Agregado (NoSQL) |
|---|---|---|
| Acceso | Varias tablas (`JOIN`s) | Documento, familia de columnas, etc. en una sola unidad |
| Flexibilidad | Alta (consultas complejas arbitrarias) | Media (limitada al patrón de acceso para el que se diseñó) |
| Rendimiento | Penalización por `JOIN`s | Optimizado para las consultas típicas de ese agregado |
| Diseño | Normalización formal (reglas fijas: 1FN, 2FN, 3FN...) | Ad-hoc, guiado por el uso (por las consultas reales de la aplicación) |

**Diseño del agregado:**

- Se guía por los **patrones de casos de uso** de la aplicación: ¿cuáles son las consultas más frecuentes?
- El **perímetro del agregado** (qué se incluye dentro y qué se deja fuera) es una **decisión de diseño**: define qué datos "viajan siempre juntos".
- Bien diseñados, los agregados **reducen accesos** y **evitan `JOIN`s costosos**.
- El proceso de diseño es **menos formal** que en el modelo relacional y **más orientado a la práctica** (ad hoc), frente a las reglas estrictas de normalización (1FN, 2FN, 3FN).

**Idea clave (frase textual de las diapositivas):** *"El agregado se diseña desde las consultas: qué información debe viajar junta y qué coste aceptas al actualizarla."*

**Ejemplo del e-commerce:** en lugar de tener `Pedido`, `LineaPedido` y `Direccion` en tres tablas distintas, se guarda **Pedido + Items del pedido en un único documento JSON**, de forma que leer un pedido completo es una sola lectura (una sola unidad), a cambio de que actualizar, por ejemplo, el precio de un producto que aparece en miles de pedidos ya guardados sea más costoso (habría que tocar cada documento donde aparece, porque el dato está duplicado/incrustado).

**Esquema del agregado (schemaless):**

- En una BD relacional el esquema es **fijo**: todos los registros de una tabla siguen la misma estructura.
- En NoSQL, la **ausencia de esquema** otorga **flexibilidad**: permite almacenar **datos heterogéneos** (p. ej., en MongoDB, un documento `Usuario` puede tener campos `{nombre, email}` y otro documento `Usuario` distinto puede tener `{dirección, teléfono}` sin que la base de datos lo impida).
- **Matiz importante:** no es "ausencia total de esquema". **El esquema se desplaza a la lógica de la aplicación**, lo que implica **riesgos** si no se controla bien (p. ej., inconsistencias entre documentos que deberían tener campos parecidos, errores de tipeo en nombres de campos, etc.).

**Frase clave:** *"'SCHEMALESS' / Esquema flexible no significa datos sin diseño: parte del control pasa a la aplicación y a las prácticas de modelado."*

Los modelos de agregación son útiles cuando:

- La **funcionalidad está definida de antemano** y no se esperan cambios frecuentes.
- **No existen relaciones complejas** entre entidades.
- Los datos sufren **pocas actualizaciones**.

**Escenarios típicos de uso:**

- Almacenamiento de **logs de usuario** (p. ej. clics en una web).
- **Perfiles de usuario** en apps.
- **Pedidos en e-commerce** (pedido + ítems en un solo documento).
- **Gestión de contenidos** (blogs, prensa digital).
- **Analítica web** en tiempo real.
- **Series de datos de IoT** (p. ej. datos de sensores cada segundo).

### 2.2 Los cuatro modelos de agregación NoSQL (y el modelo de grafos)

Las diapositivas ordenan los modelos según su **expresividad semántica** creciente:

```
Clave-valor → Orientado a columnas → Orientado a documentos  |  Orientado a grafos
        (- expresividad) ────────────────────────► (+ expresividad)
```

Los tres primeros (clave-valor, columnas, documentos) son "modelos de agregación" en sentido estricto (guardan unidades autocontenidas). El modelo de **grafos** es distinto: su valor está en las **relaciones explícitas** entre elementos, no en agrupar datos en una unidad aislada.

#### 2.2.1 Modelo clave-valor

- Es el **modelo más sencillo** de todos.
- **Menor expresividad semántica.**
- Cada elemento se identifica de forma única por una **clave**: `clave → valor`.
- La clave puede pertenecer al dominio del problema (DNI, NSS, e-mail...) o ser un identificador arbitrario.
- **Atomicidad a nivel de clave**: las operaciones son atómicas sobre un par clave-valor concreto.

**Características:**

- **Alto rendimiento** de lectura/escritura.
- **Velocidad** en las consultas (acceso directo por clave, sin necesidad de recorrer índices complejos).
- **Fáciles de escalar** y de **implementar**.

**Punto conceptual importante:** la base de datos **no conoce la estructura interna del agregado** (lo trata como una "**caja negra**" u **objeto opaco**). Si el valor tiene una estructura interna (p. ej. es un JSON), esa estructura **solo la conocen los programas de aplicación** que acceden a la BD — la propia base de datos no la interpreta ni la indexa.

**Ejemplo (gestión de pedidos):** `clave = pedidoId`, `valor = todo el contenido del pedido (fecha, líneas, dirección...)` almacenado como un blob opaco. La BD sabe recuperar el valor asociado a `pedidoId`, pero no "sabe" qué hay dentro de ese valor.

Otro ejemplo típico: `User:2:friends → {23, 76, 233, 11}` y `User:2:settings → "Theme: dark, cookies: false"`.

**SGBD representativos:** Redis, Amazon DynamoDB, etcd. (Redis se estudia en detalle en el **Tema 3**).

**Pregunta guía para saber si usar clave-valor:** *"Necesitas recuperar un dato constantemente y ya conoces su clave. ¿Realmente necesitas hacer una consulta compleja?"* → Si conozco la clave y necesito mucha velocidad → **clave-valor**.

**Caso de uso real:** cachés de sesión web (guardar el `sessionId` del usuario como clave, y sus datos de sesión como valor, con expiración TTL), contadores, carritos de compra temporales.

#### 2.2.2 Modelo orientado a documentos

- Es una **extensión** del modelo clave-valor.
- Los agregados tienen una **estructura interna** que recibe el nombre de **documento**, almacenada en formato **JSON, XML**, etc.
- A diferencia del clave-valor puro, **la BD sí conoce e interpreta esa estructura interna**.
- Se puede acceder a los agregados de dos formas:
  - A través de la **clave**.
  - Al **contenido**, a través de los **atributos del documento** (esto es la gran diferencia respecto a clave-valor: se puede consultar por campos internos, crear índices sobre esos atributos, etc.).
- Los documentos se pueden **agregar en colecciones**.
- Sobre un documento se puede: **recuperar** todo su contenido, **modificar** una parte, **crear índices** sobre sus atributos...
- **Atomicidad a nivel de documento**.

**Ejemplo (gestión de pedidos):**

```json
{
  "pedidoId": 500,
  "fecha": "15/06/2018",
  "clienteId": 50000,
  "pagoId": 10,
  "linea_pedido": [
    { "productoId": 25, "productoNombre": "Moa", "numUnidades": 1, "precio": 19.99 }
  ],
  "direccion_pedido": [
    { "calle": "Alcala, 140", "codigoPostal": 28028 }
  ]
}
```

Aquí `pedidoId` actúa como **clave**, `clienteId`/`pagoId` son **referencias** a otros agregados, y `linea_pedido`/`direccion_pedido` son **documentos incrustados** dentro del documento principal.

**SGBD representativos:** MongoDB (se estudia en detalle en el **Tema 4**), CouchDB, Azure Cosmos DB, Couchbase.

**Pregunta guía:** *"Un portátil tiene CPU y RAM; una camiseta, talla y color. ¿Tiene sentido obligar a todos los productos a tener exactamente la misma estructura?"* → Si los objetos tienen estructuras variables y se consultan como unidades → **documentos**.

**Caso de uso real:** catálogo de productos de un e-commerce donde cada categoría de producto tiene atributos distintos (una camiseta tiene talla/color, un portátil tiene CPU/RAM), perfiles de usuario con campos opcionales, gestión de contenido (CMS de un blog).

#### 2.2.3 Modelo orientado a columnas

- Ve los datos como una **matriz bidimensional**:
  - **Filas**: agregaciones de datos, accedidas por **clave**.
  - **Columnas**: los atributos de las agregaciones se representan mediante la **tripleta**: `<nombre, valor, timestamp>` (el *timestamp* permite, entre otras cosas, resolver conflictos de escritura y "expirar" datos).
- **Atomicidad a nivel de fila.**
- Las columnas pueden agruparse en **familias** (una familia representa un concepto dentro de la agregación). Ejemplo: un agregado "Profesor" con dos familias: *Información personal* e *Información académica*.

**Características de rendimiento:**

- Adecuados para aplicaciones con **archivos distribuidos**.
- **Menos escalables que clave-valor**, y **más lentas en escritura**.
- **Más velocidad en lectura.**
- Son más **eficientes** cuando:
  - Se insertan **múltiples registros al mismo tiempo** (se actualizan bloques de columnas de forma eficiente).
  - Se quiere **acceder solo a algunas columnas** (no hace falta leer la fila entera, a diferencia de una BD orientada a filas).

**Ejemplo (gestión de pedidos):** para el `PedidoID 10.097` se guardan agrupadas por familias: *Información general* (fecha, clienteID, pagoID), *direcciones_pedidos* (calle, ciudad, codPostal, país) y *líneas_pedidos* (productos y cantidades). Esto permite dos tipos de consultas muy eficientes:
- Obtener **toda la información general de todos los pedidos** (leer solo la familia "Inf. general" de todas las filas → muy rápido, no hace falta tocar el resto de familias).
- Obtener **toda la información de un pedido concreto** (leer todas las familias de una fila → también eficiente porque están indexadas por la misma clave).

**SGBD representativos:** Cassandra (se estudia en detalle en el **Tema 5**), ScyllaDB, Google Bigtable, Apache HBase.

**Pregunta guía:** *"Millones de eventos llegan continuamente y ya se sabe qué consultas deberá soportar el sistema. ¿Diseñarías primero las entidades o las consultas?"* → En Cassandra, **el modelo se diseña pensando en cómo se va a consultar** (modelo *query-first*, al contrario que el diseño relacional clásico que parte de las entidades).

**Caso de uso real:** histórico de eventos de sensores IoT, donde cada segundo llegan millones de lecturas y hay que poder consultar rápidamente "todas las lecturas de un sensor concreto en un rango de fechas" — encaja perfectamente en el patrón de particionar por sensor y ordenar por tiempo dentro de la partición (ver ejemplo de Cassandra en el apartado 3).

#### 2.2.4 Modelo orientado a grafos

- Utiliza **estructuras de grafo** para representar y almacenar los datos.
- Elementos básicos:
  - **Nodos**: representan **conceptos** y **objetos** del mundo real.
  - **Aristas**: representan de forma **explícita las relaciones** entre nodos.
- Tipos de grafos:
  - **Dirigidos** y **no dirigidos**.
  - **Etiquetados** (se aporta semántica a nodos y aristas, p. ej. una arista llamada `FRIEND_OF`) y **de propiedad etiquetados** (*property graphs*: se asignan propiedades/atributos tanto a los nodos como a las aristas, p. ej. la arista `FRIEND_OF` puede tener la propiedad `since: 01/09/2013`).

**Características:**

- Adecuado para **datos altamente relacionados**: útil cuando la **importancia de los datos está en sus interrelaciones** (típicamente: pocos objetos pero muchas relaciones entre ellos).
- Las **relaciones explícitas** mejoran mucho el **tiempo de respuesta** en consultas que navegan entre relaciones (recorrer relaciones es una operación nativa y muy barata, al contrario que en relacional donde cada "salto" es un `JOIN`).
- **Difícilmente escalables** (es el modelo con más dificultad para el escalado horizontal, precisamente porque las relaciones cruzan fronteras de partición fácilmente).
- El **esquema está implícito** en la propia estructura del grafo.
- Proporcionan **lenguajes de consulta de alto nivel** (p. ej. Cypher en Neo4j) pensados para expresar "caminos" y patrones de relación.
- Útiles cuando la información se puede representar como una **red**.
- **Ámbitos típicos:** redes sociales (RRSS), logística, mapas/rutas, aplicaciones semánticas.

**Ejemplo (gestión de pedidos):** en vez de tablas o documentos, se modela como nodos `Pedido`, `Cliente`, `Pago`, `LineaPedido`, `DireccionPedido`, unidos por aristas etiquetadas como `cliente`, `infPago`, `lineaPedido`, `direccionPedido`. La ventaja aparece cuando las consultas son del tipo "recorrer relaciones", no cuando se accede a un pedido aislado.

**SGBD representativos:** Neo4j, Amazon Neptune, JanusGraph.

**Pregunta guía:** *"¿Qué amigos de mis amigos han visto esta película? ¿Qué resulta más importante aquí: los datos de cada objeto o las relaciones entre ellos?"* → Cuando el valor está en **recorrer relaciones** → **grafos**.

**Caso de uso real:** un sistema de recomendación de una red social ("amigos de amigos que les gusta X"), un sistema de detección de fraude bancario (encontrar cadenas de transacciones sospechosas entre cuentas), o un planificador de rutas logísticas.

### 2.3 Tabla resumen de los cuatro modelos

| Modelo | Unidad de acceso | Expresividad | Escalabilidad | Ejemplo SGBD | Cuándo usarlo |
|---|---|---|---|---|---|
| **Clave-valor** | Par clave-valor (opaco) | Muy baja | Muy alta | Redis, DynamoDB | Acceso directo y rapidísimo por clave conocida |
| **Documentos** | Documento (JSON/XML) | Media | Alta | MongoDB, CouchDB | Datos semiestructurados, con estructura variable, consultables por atributos |
| **Columnas** | Fila / familia de columnas | Media | Alta (algo menos que clave-valor) | Cassandra, HBase | Grandes volúmenes de eventos, consultas conocidas de antemano (query-first) |
| **Grafos** | Nodo + aristas | Alta | Baja | Neo4j, Amazon Neptune | Datos muy interrelacionados, consultas de tipo "recorrer relaciones" |

---

## 3. Selección de una base de datos NoSQL

### 3.1 Criterios de selección

Idea de partida: **no existe una base de datos óptima para todos los problemas**. La elección depende de varios criterios:

1. **Requisitos funcionales y no funcionales**
   - *Funcionales*: qué **operaciones del negocio** debe ejecutar la BD (búsquedas complejas, transacciones...).
   - *No funcionales*: atributos de calidad como **latencia** o **rendimiento**.
   - Evaluar ambos asegura que la tecnología cumple las reglas del sistema sin comprometer los niveles de servicio.

2. **Estructura de los datos**
   - La **naturaleza** de la información determina la BD NoSQL más adecuada.
   - Elegir según la **variabilidad** o **jerarquía** del esquema evita almacenar la información en un formato ineficiente.

3. **Patrones de lectura/escritura**
   - Analiza si la aplicación tiene **carga intensiva de escrituras masivas**, **lecturas frecuentes de baja latencia**, o un equilibrio entre ambas.
   - Ayuda a elegir motores optimizados para el tipo de tráfico más frecuente.

4. **Volumen y crecimiento**
   - Contempla el **tamaño actual** de los datos y la **velocidad estimada** de crecimiento.
   - Permite elegir modelos que soporten **escalado horizontal (sharding)** fluido sin degradar el rendimiento a gran escala.

5. **Necesidades de consistencia**
   - Establece si el sistema exige que **todos los nodos devuelvan exactamente el mismo dato al instante** (consistencia fuerte) o si se tolera un **desfase temporal** (consistencia eventual).
   - Es un punto clave del **Teorema CAP** (ver abajo).

6. **Necesidades de disponibilidad**
   - Mide el grado de **tolerancia a fallos** en los nodos para garantizar que el sistema siga operando sin interrupciones.
   - En aplicaciones críticas se priorizan motores distribuidos con mayor capacidad de **replicación** y **redundancia**.

7. **Persistencia políglota**
   - Contempla la **combinación de múltiples tecnologías** de almacenamiento dentro de una misma arquitectura para resolver necesidades distintas en cada módulo.
   - Permite aprovechar la **fortaleza de cada motor** en lugar de depender de una única solución.

**Frase clave:** *"La tecnología es consecuencia de los requisitos, no el punto de partida."*

### 3.2 El Teorema CAP

Es un punto clave dentro de "Necesidades de consistencia". El teorema CAP afirma que un sistema distribuido **solo puede garantizar simultáneamente 2 de las 3 propiedades siguientes**:

- **C — Consistencia**: todos los clientes ven **exactamente los mismos datos** al mismo tiempo, sin importar a qué nodo se conecten.
- **A — Disponibilidad (Availability)**: cada petición recibe una respuesta (éxito o fallo), garantizando que el sistema no se bloquee.
- **P — Tolerancia a particiones**: el sistema sigue funcionando aunque la red falle y se rompa la comunicación entre nodos.

Como en un sistema distribuido real **las particiones de red pueden ocurrir** (P es casi obligatorio asumirlo), en la práctica la elección real está entre:

| Categoría | Propiedades garantizadas | Qué se sacrifica | Riesgo |
|---|---|---|---|
| **CP** | Consistencia + Tolerancia a particiones | Disponibilidad | Hay riesgo de que algunos datos no estén disponibles |
| **AP** | Disponibilidad + Tolerancia a particiones | Consistencia | Los clientes pueden leer datos inconsistentes |
| **CA** | Consistencia + Disponibilidad | Tolerancia a particiones | Un problema en la red puede parar el sistema (en la práctica, casi inviable en sistemas realmente distribuidos) |

**Decisión de diseño (frase clave):** *"En sistemas distribuidos, la red puede fallar; a menudo la decisión real se sitúa entre la consistencia fuerte y la máxima disponibilidad."*

**Ejemplo aplicado:**
- Un sistema bancario de transferencias necesita **CP**: prefiere rechazar una operación (indisponibilidad temporal) antes que mostrar un saldo incorrecto.
- Un sistema de "me gusta" en redes sociales necesita **AP**: prefiere seguir funcionando (aunque el contador de "me gusta" tarde unos segundos en actualizarse en todos los nodos) antes que bloquear al usuario.

### 3.3 Ejemplos prácticos de selección

#### Ejemplo 1 — Plataforma de streaming: construir una playlist completa

**Modelo relacional:**

| Tablas | Campos | Claves |
|---|---|---|
| Usuarios | id, nombre, email | |
| Playlists | id, usuario_id, nombre | FK → Usuarios.id |
| Canciones | id, título, artista, álbum | |
| Playlist_Canciones | playlist_id, cancion_id, orden | FK → Playlists, Canciones |

Para reconstruir una playlist completa se necesita un `JOIN` de **4 tablas**.

**Diseño como agregado orientado a documento (MongoDB):**

```json
{
  "playlist_id": 321,
  "usuario": { "id": 88, "nombre": "Luis" },
  "nombre": "Favoritas verano",
  "canciones": [
    { "id": 1, "titulo": "Song A", "artista": "X" },
    { "id": 2, "titulo": "Song B", "artista": "Y" }
  ],
  "fecha_creacion": "2025-09-09"
}
```

| | Pros | Contras |
|---|---|---|
| Documento (MongoDB) | ✔ Lectura rápida de playlist completa · ✔ Ideal para APIs y microservicios | ✘ Redundancia (duplicación de datos de canciones/artistas) · ✘ Actualizaciones masivas más costosas (si cambia el nombre de un artista, hay que tocar todas las playlists donde aparece) |

#### Ejemplo 2 — Geolocalización: posiciones GPS de un repartidor

**Modelo relacional:**

- `Repartidor (id, nombre, telefono)`
- `EventoGPS (id, repartidor_id, ts, lat, lon, precision_m, origen_app)`, con `FK: EventoGPS.repartidor_id → Repartidor.id`

```sql
SELECT ts, lat, lon
FROM EventoGPS
WHERE repartidor_id = 981
    AND ts BETWEEN '2025-09-09 10:00'
    AND '2025-09-09 11:00'
ORDER BY ts DESC;
```

| | Pros | Contras |
|---|---|---|
| Relacional | ✔ Integridad referencial · ✔ Fácil unir con otras tablas | ✘ Tablas enormes, índices grandes · ✘ Escalado más complejo |

**Diseño orientado a columnas (Cassandra):**

```sql
CREATE TABLE gps_by_rider_day (
  rider_id     bigint,
  day          date,
  ts           timestamp,
  lat          double,
  lon          double,
  precision_m  float,
  origen_app   text,
  PRIMARY KEY ((rider_id, day), ts)
) WITH CLUSTERING ORDER BY (ts DESC);
```

| | Pros | Contras |
|---|---|---|
| Columnas (Cassandra) | ✔ Muy eficiente para series temporales · ✔ Lecturas más rápidas por clave + rango · ✔ Escala horizontal fácil | ✘ Modelo *query-first* (hay que conocer de antemano los casos de uso) · ✘ Duplicación de datos · ✘ Hay que controlar el tamaño de las particiones |

**Ejemplo de partición resultante en Cassandra:**

```
Partición: (rider_id=981, day='2025-09-09')

ts                    lat        lon        precision_m  origen_app
2025-09-09 10:59:58   40.4201    -3.7049    6.2          android
2025-09-09 10:59:48   40.4200    -3.7051    7.1          android
2025-09-09 10:59:38   40.4197    -3.7053    5.8          android
...
```

El "**agregado**" aquí es la **partición** (`rider_id + day`), optimizada exactamente para la consulta objetivo: recuperar los eventos de un repartidor en un día concreto, ya ordenados por tiempo.

### 3.4 Conclusión final: ¿SQL o NoSQL?

**La pregunta correcta no es "¿SQL o NoSQL?", sino "¿qué necesita este caso de uso?"**. Preguntas guía:

- ¿Qué datos tengo y cómo cambian?
- ¿Cómo se consultan y cómo se escriben?
- ¿Qué volumen, latencia y concurrencia debo soportar?
- ¿Qué consistencia, disponibilidad y tolerancia a fallos necesito?
- ¿Hace falta una única base de datos o una **arquitectura políglota**?

**No hay una respuesta universal: hay una decisión razonada para un contexto concreto.**

**Ejemplo integrador (plataforma tipo Netflix):**

| Necesidad | Modelo recomendado | Tecnología |
|---|---|---|
| Sesiones de usuario, caché y TTL | Clave-valor | Redis |
| Catálogo con atributos variables | Documentos | MongoDB |
| Histórico masivo de eventos (reproducciones, clics) | Orientado a columnas (wide-column) | Cassandra |
| Pagos y facturación | Relacional | SQL |
| Relaciones complejas entre usuarios, actores, títulos y géneros | Grafos | Neo4j, etc. |

**Conclusión textual de las diapositivas:** *"Una plataforma tipo Netflix combina varias tecnologías: sesiones/caché, catálogo, eventos, pagos y analítica no tienen los mismos requisitos."* Esto ejemplifica perfectamente la **persistencia políglota**.

### 3.5 Otros tipos de bases de datos (mención breve)

Además de los cuatro modelos NoSQL clásicos, existen otras categorías especializadas que las diapositivas mencionan brevemente:

| Tipo | Para qué sirve | Ejemplos |
|---|---|---|
| **Series temporales** | Datos ordenados en el tiempo: métricas, sensores, monitorización | InfluxDB, TimescaleDB |
| **Búsqueda** | Índices, texto completo, relevancia y recuperación rápida de información | Elasticsearch, OpenSearch |
| **Vectoriales** | Similitud entre vectores/*embeddings*, búsqueda semántica, aplicaciones de IA | Milvus, Weaviate |
| **Multimodelo** | Un mismo sistema soporta varios modelos o formas de acceso | ArangoDB, Cosmos DB |

---

## 4. Resumen / puntos clave para repasar

- **No hay una BD universal**: la elección se hace en función del problema (datos, consultas, volumen, garantías).
- **Orden antiguo vs nuevo orden**: SQL prioriza estructura fija, integración funcional y transacciones (ACID); NoSQL prioriza volumen, velocidad, flexibilidad y pragmatismo (BASE).
- **Normalización**: reduce redundancia pero exige `JOIN`s → impacta el rendimiento de lectura. Trade-off normalización/desnormalización.
- **Escalado vertical (scale up)** = máquina más potente; **escalado horizontal (scale out)** = más nodos. NoSQL favorece el escalado horizontal mediante **sharding** (horizontal por filas, vertical por columnas) y **replicación**.
- **ACID** (Atomicidad, Consistencia, Aislamiento, Durabilidad) es el modelo del mundo relacional; **BASE** (Basically Available, Soft state, Eventual consistency) es el modelo típico NoSQL. NoSQL no implica ausencia total de garantías.
- **Ventajas NoSQL**: sistemas abiertos, clústeres desacoplados, datos no estructurados, esquema flexible, sistemas políglotas, sin lenguaje estándar (cada uno el suyo).
- **Inconvenientes NoSQL**: sin ACID fuerte, complejidad trasladada a la aplicación, sin independencia lógica, sin estándar, madurez desigual, curva de aprendizaje.
- **Agregado** = unidad de datos relacionados gestionada como un todo, identificada por clave única. Es la pieza central de los modelos clave-valor, documentos y columnas. Se diseña **desde las consultas**, no desde reglas de normalización.
- **Esquema flexible (schemaless)** no significa "sin diseño": el control del esquema pasa a la aplicación.
- **Cuatro modelos de agregación/estructura**, ordenados de menor a mayor expresividad semántica (excepto grafos, que van aparte por su enfoque relacional explícito):
  1. **Clave-valor** (Redis, DynamoDB): caja negra, clave conocida, máxima velocidad y escalabilidad.
  2. **Documentos** (MongoDB, CouchDB): la BD interpreta el JSON/XML, se puede consultar por atributos internos.
  3. **Columnas** (Cassandra, HBase): matriz bidimensional, tripletas `<nombre, valor, timestamp>`, familias de columnas, diseño *query-first*.
  4. **Grafos** (Neo4j, Neptune): nodos + aristas, el valor está en recorrer relaciones, difícil de escalar.
- **Teorema CAP**: en un sistema distribuido solo se pueden garantizar 2 de 3 propiedades (Consistencia, Disponibilidad, Tolerancia a particiones). Como P casi siempre es necesaria, la decisión real suele ser **CP vs AP**.
- **Criterios de selección de una BD NoSQL**: requisitos funcionales/no funcionales, estructura de los datos, patrones de lectura/escritura, volumen y crecimiento, necesidades de consistencia, necesidades de disponibilidad, persistencia políglota.
- **Persistencia políglota**: usar varias tecnologías distintas dentro de la misma arquitectura, cada una para lo que mejor resuelve (ejemplo Netflix: Redis + MongoDB + Cassandra + SQL + Neo4j).
- La pregunta final no es "¿SQL o NoSQL?" sino "¿qué necesita este caso de uso concreto?".

---

## 5. Autoevaluación

**1. ¿Cuál es la diferencia fundamental entre el "orden antiguo" y el "nuevo orden" de gestión de datos?**
> El orden antiguo se basa en un modelo de datos único, estructura estática, integración funcional y estandarización mediante SQL, priorizando la integridad total (ACID). El nuevo orden prioriza el volumen y la velocidad, acepta cierta incoherencia temporal, es pragmático y flexible, y admite sistemas políglotas (BASE).

**2. ¿Por qué la normalización, siendo "siempre deseable" en teoría, puede ser un problema en la práctica?**
> Porque el mecanismo que garantiza la coherencia (evitar redundancia mediante referencias) obliga a usar `JOIN`s para reconstruir la información, lo cual tiene un coste en el rendimiento de lectura. El problema no es la normalización en sí, sino lo que cuesta conseguirla.

**3. Diferencia entre escalado vertical y escalado horizontal. ¿Cuál favorecen típicamente las BBDD NoSQL?**
> El escalado vertical (*scale up*) consiste en usar una máquina más potente; el escalado horizontal (*scale out*) consiste en añadir más nodos que trabajan conjuntamente. Las BBDD NoSQL fueron diseñadas para facilitar el escalado horizontal, distribuyendo procesamiento y almacenamiento entre una red de nodos.

**4. Explica las siglas ACID y BASE y a qué modelo de base de datos corresponde cada una.**
> ACID (Atomicidad, Consistencia, Aislamiento, Durabilidad) corresponde al modelo relacional. BASE (Basically Available, Soft state, Eventual consistency) corresponde al modelo NoSQL. Importante: NoSQL no implica ausencia de transacciones o consistencia, cada tecnología ofrece garantías concretas.

**5. ¿Qué es un "agregado" en el contexto de las bases de datos NoSQL?**
> Es un conjunto de datos relacionados que se gestiona como una sola unidad, identificado por una clave única, que constituye la unidad mínima de almacenamiento y consulta; los cambios se aplican al agregado completo. Es el concepto central que sustituye a la normalización/`JOIN` del modelo relacional.

**6. ¿Cómo se diseña un agregado, y en qué se diferencia ese proceso del diseño relacional normalizado?**
> El agregado se diseña a partir de los patrones de consulta más frecuentes de la aplicación (qué información debe "viajar junta"), de forma ad-hoc y práctica. El diseño relacional, en cambio, sigue reglas formales de normalización (1FN, 2FN, 3FN) orientadas a minimizar redundancia, independientemente de cómo se vaya a consultar después.

**7. ¿Qué significa que un modelo clave-valor trate el valor como una "caja negra"?**
> Que la base de datos no conoce ni interpreta la estructura interna del valor asociado a una clave; solo sabe almacenar y devolver ese valor. Si el valor tiene estructura (p. ej. JSON), esa estructura solo la interpretan los programas de aplicación que acceden a la BD.

**8. ¿En qué se diferencia el modelo orientado a documentos del modelo clave-valor puro?**
> El modelo de documentos es una extensión del clave-valor en la que la base de datos sí conoce e interpreta la estructura interna del agregado (el documento, en JSON/XML). Esto permite consultar y crear índices sobre atributos internos del documento, no solo acceder por clave, y los documentos se pueden agrupar en colecciones.

**9. En el modelo orientado a columnas, ¿qué representa la tripleta `<nombre, valor, timestamp>` y para qué sirve agrupar columnas en familias?**
> Representa un atributo de la fila/agregación: su nombre, su valor y una marca de tiempo (útil para resolver conflictos de escritura o expirar datos). Agrupar columnas en familias permite representar distintos conceptos dentro de una misma agregación (p. ej. "información personal" e "información académica" de un profesor) y optimizar el acceso a solo un subconjunto de columnas.

**10. ¿Por qué se dice que en Cassandra "el modelo se diseña pensando en cómo se va a consultar" (query-first)?**
> Porque, a diferencia del diseño relacional que parte de las entidades y sus relaciones, en un modelo orientado a columnas como Cassandra hay que conocer de antemano las consultas que el sistema deberá soportar (p. ej. "eventos GPS de un repartidor en un rango de fechas") y diseñar la tabla y la clave de partición específicamente para que esa consulta sea eficiente.

**11. ¿Cuándo tiene sentido usar un modelo orientado a grafos en lugar de documentos o columnas?**
> Cuando el valor de la información está en las relaciones entre los datos (pocos objetos, muchas relaciones), y las consultas típicas consisten en "recorrer relaciones" (p. ej., "amigos de mis amigos que vieron esta película"). Los grafos ofrecen relaciones explícitas que mejoran mucho el tiempo de respuesta de este tipo de consultas, a costa de ser difícilmente escalables.

**12. Enuncia el Teorema CAP y explica por qué, en la práctica, la elección suele reducirse a "CP vs AP".**
> El Teorema CAP dice que un sistema distribuido solo puede garantizar simultáneamente dos de estas tres propiedades: Consistencia, Disponibilidad y Tolerancia a particiones. Como en un sistema realmente distribuido las particiones de red pueden ocurrir (hay que asumir P), en la práctica la decisión de diseño real está entre priorizar la Consistencia (CP) o la Disponibilidad (AP).

**13. Da un ejemplo de sistema que debería priorizar CP y otro que debería priorizar AP, justificando la elección.**
> Un sistema bancario de transferencias debería priorizar CP: prefiere rechazar temporalmente una operación (indisponibilidad) antes que mostrar un saldo incorrecto a un cliente. Un sistema de contador de "me gusta" en redes sociales debería priorizar AP: prefiere seguir respondiendo siempre (aunque el número tarde unos segundos en converger en todos los nodos) antes que bloquear al usuario.

**14. ¿Qué es la "persistencia políglota" y por qué es relevante para una plataforma compleja tipo Netflix?**
> Es la combinación de múltiples tecnologías de almacenamiento dentro de una misma arquitectura, usando cada motor donde mejor encaja, en lugar de depender de una única solución. Es relevante porque los distintos módulos de una plataforma compleja tienen necesidades muy distintas: sesiones/caché (Redis), catálogo (MongoDB), histórico de eventos (Cassandra), pagos (SQL) y relaciones entre entidades (grafos) no se resuelven igual de bien con la misma tecnología.

**15. Menciona al menos cuatro criterios que se deben evaluar antes de elegir una base de datos NoSQL concreta.**
> Cualquier combinación de: requisitos funcionales y no funcionales, estructura de los datos, patrones de lectura/escritura, volumen y crecimiento esperado, necesidades de consistencia, necesidades de disponibilidad, y la posibilidad de adoptar una arquitectura de persistencia políglota.
