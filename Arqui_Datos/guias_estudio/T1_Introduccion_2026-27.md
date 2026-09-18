# Guía de estudio — Tema 1: Introducción a las Arquitecturas de Datos

**Asignatura:** Arquitectura de Datos (4º Grado Ingeniería Informática, UC3M) — Curso 2026/2027

---

## 1. Resumen general y objetivos

Este tema sienta las bases conceptuales de toda la asignatura. Antes de estudiar bases de datos relacionales, NoSQL, Data Warehouses o Data Lakes en profundidad, hay que entender **el ecosistema completo en el que viven los datos**: qué tipos de datos existen, qué metadatos los acompañan, por qué fases pasan durante su vida, quién es responsable de gobernarlos, dónde se almacenan y cómo se integran entre sistemas.

La idea central que atraviesa todo el tema es esta:

> **La arquitectura de datos no es una lista de tecnologías.** Es el conjunto de decisiones que conectan las necesidades del negocio con los datos, las plataformas de almacenamiento, los procesos de integración y el gobierno de esos datos, de forma que la información fluya de manera fiable, segura y útil desde su origen hasta quien la necesita.

**Objetivos de aprendizaje** (tal como los define la asignatura):

1. Identificar los diferentes **tipos de datos** (estructurados, semiestructurados, no estructurados) y su utilidad según el contexto.
2. Distinguir entre los principales **repositorios de datos** (BBDD relacionales, NoSQL, Data Warehouses, Data Lakes) y sus casos de uso.
3. Explicar las etapas del **ciclo de vida de los datos**, desde la adquisición hasta la eliminación responsable.
4. Analizar la importancia de la **gobernanza de datos**: cadena de valor, calidad y roles clave.
5. Comparar estrategias de **integración de datos** (ETL vs. ELT) y conocer herramientas del mercado.

**Contenidos (estructura del tema):**
1.1. Ecosistema de los datos
1.2. Ciclo de vida de los datos
1.3. Gobernanza de los datos
1.4. Repositorios de los datos
1.5. Integración de datos
1.6. Arquitectura de plataformas de datos

---

## 2. Introducción a las arquitecturas de datos

La **arquitectura de datos** define cómo se **recopilan, procesan, almacenan, integran y aseguran** los datos dentro de una organización. Funciona como el **puente** entre los objetivos estratégicos del negocio y la infraestructura tecnológica que gestiona su información.

Su objetivo principal es garantizar que la información **fluya sin problemas, con calidad y de forma segura**, desde las fuentes de origen hasta los usuarios finales.

Se compone de **cuatro elementos esenciales**:

| Elemento | Qué cubre |
|---|---|
| **Modelos de datos** | El ecosistema: qué tipos de datos y metadatos existen |
| **Sistemas de almacenamiento** | Los repositorios: dónde viven los datos (BD relacional, NoSQL, DW, Data Lake...) |
| **Integración** | Los pipelines: cómo los datos viajan y se transforman entre sistemas |
| **Gobernanza y seguridad** | Quién es responsable, qué políticas se aplican, cómo se garantiza la calidad |

**Ejemplo aclaratorio:** Imagina una empresa de streaming como Netflix. No basta con "tener una base de datos". Hace falta decidir *qué tipo de dato es cada cosa* (el catálogo de películas es estructurado, los logs de reproducción son semiestructurados), *dónde se guarda cada cosa* (BBDD relacional para perfiles, Data Lake para eventos de visionado, BBDD NoSQL para caché de sesiones), *cómo viaja la información* entre esos sistemas (pipelines ETL/ELT) y *quién es responsable* de que esos datos sean correctos y estén protegidos (gobernanza). Eso, en conjunto, es la arquitectura de datos.

---

## 3. Ecosistema de los datos (1.1)

### 3.1. Tipos de datos

Existen tres grandes categorías de datos, y **el tipo de dato condiciona (pero no determina en solitario) el tipo de repositorio** donde se puede almacenar y las herramientas que se pueden usar para consultarlo.

#### a) Datos estructurados
- Siguen un **formato rígido, esquema o estándar**.
- Se organizan ordenadamente en **filas y columnas**.
- **Ejemplos:** bases de datos SQL, hojas de cálculo, formularios web, gestión de inventarios, transacciones bancarias.

```
ID | Nombre    | Departamento | Salario
1  | Ana Pérez | Ventas       | 2.500 €
2  | Luis Ruiz | IT           | 2.800 €
3  | Marta Gil | Finanzas     | 3.000 €
```

#### b) Datos semiestructurados
- Tienen **características consistentes** y datos, pero **no se ajustan** a una estructura rígida ni a un esquema fijo (el esquema puede variar de un registro a otro).
- Contienen **etiquetas y elementos (metadatos)** que sirven para agrupar los datos y organizarlos en una jerarquía.
- **Ejemplos:** JSON devuelto por una API, XML de facturas electrónicas, correos electrónicos (cabeceras + cuerpo), registros de eventos (logs).

```json
{
  "user": "Ana",
  "tweet": "Hola mundo",
  "fecha": "2025-09-07",
  "likes": 125
}
```

#### c) Datos no estructurados
- **No siguen ningún formato, secuencia, semántica ni regla** en particular.
- Son datos **complejos**, en su mayoría **información cualitativa** que no se puede organizar en filas y columnas.
- **Ejemplos:** publicaciones en redes sociales, documentos de texto libre, audios, imágenes, vídeos.

> **Idea clave:** el tipo de dato condiciona la elección del almacenamiento, pero **no la determina**: también importan los patrones de acceso, el volumen, la latencia, la escalabilidad, la consistencia, el coste y los requisitos de gobierno.

**Tabla resumen — de los datos a los repositorios:**

| Tipo de dato | Repositorios típicos |
|---|---|
| Estructurados | BBDD Relacionales, Data Warehouse, Lakehouse |
| Semiestructurados | BBDD NoSQL documental, Data Lake, Lakehouse |
| No estructurados | Object Storage, Data Lake, Lakehouse |

**Ejemplos combinados (muy importantes para el examen):**

- **Banca online:** las transacciones van a una BD relacional (estructurados); los eventos y logs van a un Data Lake / plataforma analítica.
- **Netflix (streaming):** catálogo y perfiles en distintas BBDD; eventos de visionado en almacenamiento distribuido/Data Lake; caché y sesiones en BBDD NoSQL clave-valor.
- **Instagram (redes sociales):** imágenes y vídeos en Object Storage; metadatos, perfiles y relaciones en BBDD especializadas.

### 3.2. Metadatos

Los **metadatos** son "datos sobre otros datos": añaden **contexto**, facilitan la **búsqueda**, su **gestión** y mejoran su **calidad**.

**Ejemplo sencillo:** una foto `foto.jpg` en sí misma es solo un archivo binario. Sus metadatos (resolución, tamaño, fecha) le dan **contexto, trazabilidad y gobierno**: sin ellos, no sabríamos cuándo se tomó ni cómo se generó.

> **Idea clave:** los metadatos no son solo "datos sobre datos". Hacen posible **encontrar, entender, trazar, proteger y gobernar** los datos. Los sistemas de IA, en particular, conectan datasets, modelos, inferencias y evidencias a través de metadatos.

#### Familias de metadatos

Tradicionalmente se distinguían metadatos técnicos, de procesos, de negocio y de gestión; en arquitecturas actuales se amplían con linaje, calidad, seguridad, cumplimiento e IA/ML. Cada familia aporta una perspectiva distinta y se complementan entre sí.

| Familia | Qué describe | Ejemplo (dataset de ventas e-commerce) |
|---|---|---|
| **Técnicos** | Esquema, formato, tipos, particiones, APIs — *cómo* están estructurados, almacenados y cómo se accede técnicamente | Estructura de la tabla (id, fecha, importe); formato (JSON, CSV, Parquet); claves e índices |
| **De procesos / operacionales** | Ejecución y operación de los procesos que producen, transforman o mueven datos (monitorización, auditoría) | Proceso ETL cada 24h; fecha/hora de inicio-fin; nº de registros procesados y errores |
| **De negocio / semánticos** | Significado y contexto desde la perspectiva del dominio: definiciones, unidades, KPIs, glosarios | "importe" = total de la venta con IVA |
| **De gestión y gobernanza** | Responsabilidades, políticas y condiciones de uso; identifican propietarios y permiten controlar acceso | Responsable: Data Steward Comercial; política: acceso solo a marketing y dirección |
| **De procedencia y linaje** | Fuente/origen, transformaciones realizadas, versiones, dependencias entre datasets/procesos | De dónde viene el dato, qué transformaciones ha sufrido |
| **De calidad** | Completitud, validez, originalidad, consistencia | ¿Cuántos valores nulos tiene el dataset? |
| **De seguridad / cumplimiento / IA-ML** | Acceso, finalidad, licencia, dataset y modelo usado en IA, métricas de evaluación | Qué modelo de IA usó este dataset, con qué evaluación |

**Ejemplo integrado — ETL:** un sistema de ETL genera metadatos operacionales que registran los tiempos de inicio y fin de cada fase (Extracción → 5 min → Transformación → 10 min → Carga), el origen y destino de los datos, y los posibles errores. Estos metadatos permiten **auditar y optimizar** la ejecución del pipeline.

**Buenas prácticas de metadatos** (ejemplo real: portales de datos abiertos como el del Ayuntamiento de Madrid):
- Nombres de datasets cortos, claros y sin ambigüedades.
- Descripción del contenido, fuente y propósito.
- Clasificación en sectores mediante taxonomía estándar.
- Palabras clave (keywords) para mejorar la búsqueda.
- Fechas de inicio y fin del periodo cubierto.
- Frecuencia de actualización.
- Responsable del dataset claramente definido.
- Documentación de estructura (diccionario de códigos, glosarios).
- Licencia estandarizada.

**Conclusión de esta sección:** los metadatos aportan **contexto, trazabilidad, interoperabilidad, gobierno y confianza**. Conectan lo técnico con lo estratégico (desde estructuras de BD hasta KPIs de negocio) y son una pieza esencial de la gobernanza: **almacenar no basta, hay que poder encontrar, entender y confiar en la información**.

---

## 4. Ciclo de vida de los datos (1.2)

Los datos pasan por **5 etapas** a lo largo de su ciclo de vida. Es importante mantener un **registro auditable** en todas ellas, preguntándose en cada etapa: *¿qué metadatos y evidencias necesito conservar para poder explicar después qué ocurrió con el dato?*

```
Adquisición → Procesamiento → Almacenamiento → Intercambio → Conservación/Eliminación
```

| # | Etapa | Qué se establece | Ejemplos |
|---|---|---|---|
| 1 | **Adquisición de datos** | Qué datos deben recopilarse y su base legal; el uso previsto y su política de privacidad; qué datos se necesitan para cumplir con los propósitos | Formulario de registro en e-commerce (nombre, email, teléfono); cookies de navegación; sensores IoT de un coche (velocidad, geolocalización) |
| 2 | **Procesamiento de datos** | Cómo se van a procesar los datos y la base legal (contrato o consentimiento) | Normalizar direcciones postales; detectar fraudes en tiempo real en pagos con tarjeta; anonimizar datos personales de pacientes |
| 3 | **Almacenamiento de datos** | Dónde se van a almacenar y las medidas para evitar amenazas de seguridad internas/externas | Guardar facturas digitales en BD relacional (SQL Server); logs de clics en un Data Lake en la nube (Amazon S3); backup de imágenes médicas con control de acceso |
| 4 | **Intercambio de datos** | Qué proveedores externos pueden tener acceso, y la responsabilidad contractual sobre esos accesos | Compartir inventario con proveedores vía API; exportar métricas de Google Analytics a Power BI; enviar registros clínicos entre hospitales bajo estándares HL7/FHIR |
| 5 | **Conservación o eliminación de datos** | Políticas y procesos para conservar o eliminar datos personales tras un tiempo designado | Mantener historiales académicos 5 años (ley educativa); eliminar cuentas inactivas tras 2 años; archivar correos corporativos antiguos antes de su eliminación |

### Ciclo de vida extendido (con IA)

En sistemas modernos, especialmente los que incorporan IA, el ciclo se amplía así:

```
Dato → Dataset → Entrenamiento → Modelo → Sistema de IA → Inferencia/Decisión → Monitorización → (Mejora continua) → nuevos datos
```

- Dato → dataset → producto de datos.
- Dataset → entrenamiento → modelo → sistema de IA.
- Sistema → inferencia / decisión / contenido generado.
- Monitorización → realimentación → nueva versión.

> **Idea clave:** el ciclo **no termina** al entrenar un modelo. La inferencia, la monitorización, la retroalimentación y las nuevas versiones también deben ser trazables. Cada transición debe dejar **metadatos y evidencias** (linaje, calidad, gobernanza, trazabilidad).

**Ejemplo:** un banco entrena un modelo de detección de fraude (dataset de transacciones históricas). El modelo se despliega (sistema de IA) y genera predicciones en tiempo real (inferencia). Se monitoriza su precisión; si empeora (deriva del modelo), se reentrena con datos nuevos → nueva versión. Todo este recorrido debe documentarse: qué datos entrenaron qué versión del modelo y qué resultados dio.

---

## 5. Gobernanza de los datos (1.3)

### 5.1. Qué es y por qué es importante

La **gobernanza de datos** es el conjunto de **políticas, procesos y roles** que aseguran que los datos se gestionen de forma **eficiente, segura y con valor para la organización**, alineando tecnología, negocio y cumplimiento legal.

Se apoya en tres pilares:

| Pilar | Contenido |
|---|---|
| **Políticas** | Reglas, normativas, estándares |
| **Procesos** | Ciclo de vida del dato, calidad, seguridad |
| **Roles** | CDO, Data Owners, Data Stewards, Data Manager, Data Users |

> La gobernanza convierte los datos en un **activo estratégico** para la toma de decisiones y la innovación.

**La gobernanza introduce requisitos a la arquitectura.** Toda arquitectura de datos bien gobernada debe poder responder a estas preguntas:
- **Catálogo:** ¿qué datos existen y qué significan?
- **Responsables:** ¿quién puede decidir y quién mantiene cada dato?
- **Linaje:** ¿de dónde viene el dato y qué transformaciones ha sufrido?
- **Calidad:** ¿es apto para la finalidad prevista?
- **Acceso y seguridad:** ¿quién puede usarlo y bajo qué condiciones?
- **Cumplimiento:** ¿hay evidencias, registros y documentación?

**Contexto normativo europeo actual** (importante para el examen, aunque sea de memoria breve):

| Norma | Qué regula |
|---|---|
| RGPD / LOPDGDD | Finalidad, minimización, exactitud, seguridad, responsabilidad proactiva |
| Esquema Nacional de Seguridad (ENS) | Acceso, integridad, trazabilidad, disponibilidad, conservación |
| Data Governance Act / Data Act (UE) | Intercambio, reutilización, interoperabilidad, metadatos |
| AI Act (UE) | Gobierno de datasets, procedencia, calidad, documentación, registros, transparencia |

### 5.2. Cadena de valor de los datos

La gobernanza no solo controla los datos: garantiza que sean un **activo valioso** que aporte valor al negocio → **los datos son el núcleo del negocio**.

El valor surge al combinar tres elementos:

```
Datos → Herramientas/Plataformas → Personas/Experiencia → Valor competitivo e innovación
```

- **Datos** en sí (volumen, diversidad, calidad).
- **Herramientas y plataformas** (DBMS, analítica, IA).
- **Experiencia humana** (análisis, privacidad, visión de negocio).

La Comisión Europea define este proceso como el "centro de la futura economía del conocimiento".

**Ejemplo:** una tienda online tiene millones de registros de compra (datos). Sin un sistema analítico (herramientas) o sin un analista que sepa interpretar patrones de compra (personas), esos datos no generan ninguna ventaja competitiva. La cadena de valor solo se completa cuando los tres elementos actúan juntos.

### 5.3. Calidad de los datos. "Datos buenos"

La gobernanza asegura que los datos sean **útiles y confiables**. Existen 6 criterios clásicos de calidad:

| # | Criterio | Definición | Cómo se verifica | Ejemplo |
|---|---|---|---|---|
| 1 | **Completitud** | Todos los datos requeridos están presentes | Recopilar todos los datos, capturar todas las propiedades, considerar todos los valores posibles | Registrar periódicamente los signos vitales de un paciente (presión, pulso, temperatura) |
| 2 | **Unicidad** | Sin duplicados | Cada evento individual se captura solo una vez | Dos registros no deben compartir el mismo identificador |
| 3 | **Precisión** | Refleja la realidad con exactitud | Comprobar que un número es correcto y una cadena está bien escrita | Una calificación debe estar entre 0 y 10 |
| 4 | **Conformidad / Validez** | Se ajusta a especificaciones establecidas (sintaxis, codificación) | Formatos correctos, códigos correspondientes, convenciones de nomenclatura respetadas | Códigos de provincia con dos letras; convenciones de dosis en recetas médicas |
| 5 | **Oportunidad** | Disponible en el momento necesario | Datos capturados y disponibles lo bastante pronto para ser útiles | Informes diarios (logs); toma de decisiones en bolsa |
| 6 | **Procedencia** | Se conocen el origen y la trazabilidad | Grado de visibilidad sobre los orígenes; relacionado con la confianza | Obtención de datos de clientes vía llamada telefónica o web |

> **Idea clave:** la calidad **no es una propiedad única**: se evalúa mediante múltiples dimensiones y siempre respecto a una **finalidad de uso**. Un dato puede ser válido y preciso y, aun así, **no ser adecuado** para un uso concreto. En IA, además, importan la **representatividad, los sesgos, la cobertura, las carencias y la deriva** del dato. La calidad debe documentarse y evaluarse **durante todo el ciclo de vida**, no solo al cargar el dato.

**Ejemplo aclarador:** un dataset de edades de clientes puede estar completo, sin duplicados y con valores válidos (entre 18 y 99), pero si se usa para entrenar un modelo de recomendación de productos para adolescentes, **no es adecuado** porque no cubre ese rango de edad (problema de representatividad, no de "corrección" del dato).

### 5.4. Roles y responsabilidades

La gobernanza asigna responsables claros para cada etapa del ciclo de vida del dato.

```
CDO → define estrategia para → Data Owners
Data Owners → delegan definición de reglas en → Data Stewards
Data Stewards → hacen cumplir las reglas a → Data Users
Data Stewards → las reglas las implementan → Data Managers
```

| Rol | Perspectiva | Responsabilidad principal |
|---|---|---|
| **Chief Data Officer (CDO)** | Dirección | Supervisa el equipo; define qué información debe capturarse, retenerse y explotarse, y con qué propósito; comunica procedimientos; colabora con otros líderes ejecutivos |
| **Data Owner (propietario)** | Negocio/dominio | Responsable de un conjunto de datos y su calidad dentro de un dominio específico; supervisa actividades de calidad; aprueba glosarios y definiciones; garantiza la precisión |
| **Data Steward (administrador)** | Operativa diaria | Gestiona los datos día a día; comprende y comunica significado y uso; propone políticas de calidad; documenta en glosarios; establece medidas de protección/privacidad/seguridad; define políticas de acceso |
| **Data Manager (gestor)** | Técnica | Diseña bases de datos, administra sistemas y desarrolla aplicaciones. Incluye: diseñadores de datos, administradores de BBDD, desarrolladores de BBDD, desarrolladores de aplicaciones |
| **Data User (usuario)** | Empresarial | **Productores**: crean, actualizan, eliminan, archivan y gestionan datos (empleados, aplicaciones, sensores IoT, datos externos). **Consumidores**: usan los datos para su trabajo (analistas de negocio, científicos de datos, desarrolladores, dirección) |

**Ejemplo integrado — Dinámica de trabajo:** un banco define un estándar único para el campo "número de cliente".
1. El **Data Steward** valida su uso correcto en todos los sistemas.
2. El **Data Owner** aprueba los cambios sobre ese estándar.
3. Los **Data Users** del negocio acceden con reglas claras de seguridad.

**Dinámica de trabajo general de la gobernanza (4 pasos):**
1. Determinar y mantener estándares (definiciones comunes, formatos, acceso, cumplimiento).
2. Establecer responsabilidades de los datos (roles claros).
3. Gestionar el proceso completo de desarrollo de datos y comunicar cambios.
4. Proporcionar información acerca del entorno de datos (control de metadatos, seguimiento de procedencia, evaluación de calidad).

### 5.5. Principios de la gobernanza de datos

1. Reconocer los datos como un **activo con valor real y medible**.
2. Definir los **propietarios y responsables** de forma clara.
3. Seguir **normas y regulaciones estandarizadas**.
4. Gestionar la **calidad** de forma periódica (auditoría de datos).
5. Gestionar los **cambios** con seguimiento a lo largo del tiempo.

> *"Sin gobernanza, los datos son un riesgo; con gobernanza, son un activo."*

> **Idea clave final:** gobernar no significa bloquear el uso del dato: significa hacerlo **utilizable** con responsabilidades, reglas y evidencias claras.

---

## 6. Repositorios de datos (1.4)

Un **repositorio de datos** es el lugar donde se ha recopilado, organizado y aislado datos para que puedan ser **utilizados, analizados o consultados**. Puede ser infraestructura pequeña o grande, con una o más bases de datos.

**Tipos de repositorios:** BBDD relacionales, BBDD NoSQL, Data Warehouses, Data Lakes, Almacenes Big Data.

### 6.1. Bases de datos

Una **base de datos** es una colección de datos diseñada para la entrada, almacenamiento, búsqueda, recuperación y modificación de estos. Un **DBMS** (Database Management System) es el conjunto de programas que crean y mantienen la base de datos.

#### BBDD Relacionales

| Característica | Descripción |
|---|---|
| Estructura y esquema bien definidos | Tablas, columnas, relaciones y restricciones claramente descritas |
| Datos homogéneos | Formato consistente, mejora la eficiencia |
| Formato tabular | Filas y columnas |
| Lenguaje de consulta estándar | SQL |
| Normalización | Reduce redundancia, mejora integridad y eficiencia |
| Procesamiento confiable de transacciones | Propiedades **ACID**: Atomicidad, Consistencia, Aislamiento, Durabilidad |

Van desde pequeños sistemas de escritorio hasta sistemas masivos en la nube. Se clasifican según el **modelo de licencia y soporte**:

| Modelo | Descripción | Ejemplo |
|---|---|---|
| Open-source con soporte interno | Código abierto, mantenido por el propio equipo de TI | PostgreSQL |
| Open-source con soporte comercial | Código abierto, con soporte de terceros | MySQL con soporte de Oracle |
| Commercial closed-source | Código cerrado, soporte del proveedor | Microsoft SQL Server, Oracle Database |
| **Database-as-a-Service (nube)** | Acceso a capacidad ilimitada de cómputo/almacenamiento en la nube | Amazon RDS, IBM DB2 on Cloud, Azure SQL, Google Cloud SQL, Oracle Cloud |

#### BBDD NoSQL

**Not Only SQL.** Surgieron en respuesta al **volumen, la diversidad y la velocidad** con la que se generan los datos actualmente, principalmente por los avances en computación en la nube, el IoT y la proliferación de redes sociales. Se usan ampliamente para procesar **grandes datos** (Big Data).

Se rigen por las propiedades **BASE** (en contraste con ACID):

| Propiedad | Significado | Ejemplo |
|---|---|---|
| **B**asically Available | Básicamente disponible para todos los usuarios de forma simultánea | Momentos de aumento de tráfico |
| **S**oft state | Datos flexibles con estados transitorios | Edición de un post en redes sociales |
| **E**ventual consistency | La consistencia se alcanza tras completarse las actualizaciones simultáneas | Edición simultánea en Google Docs |

> Este tema es introductorio: los modelos concretos de NoSQL (clave-valor, documental, columnar, grafos) se estudian en profundidad en temas posteriores (p. ej. Redis en el Tema 3).

### 6.2. Data Warehouses (DW)

Un **Data Warehouse** es un repositorio central que **fusiona información** proveniente de diferentes fuentes a través de un proceso **ETL** (Extracción, Transformación y Carga). Está orientado al **análisis** y al **Business Intelligence**.

Históricamente ha sido relacional, pero con la aparición de tecnologías NoSQL y nuevas fuentes, hoy también permite almacenar datos semi y no estructurados.

**Arquitectura de tres niveles:**

| Nivel | Función |
|---|---|
| **Inferior** | Servidores de bases de datos (relacionales o no relacionales) |
| **Intermedio** | Servidor **OLAP** (procesamiento analítico online); procesa y analiza información de múltiples servidores de BBDD |
| **Superior** | Front-end del cliente: herramientas para consultar, generar informes y hacer analítica |

**Tendencia actual:** migración de DW locales a la nube, motivada por el rápido crecimiento de datos y las herramientas de análisis sofisticadas. **Beneficios en la nube:** costes más bajos, almacenamiento ilimitado, capacidades de cómputo elásticas, pago por uso, recuperación rápida ante desastres.

**Ejemplos de DW más utilizados:** Teradata, Oracle Exadata, IBM DB2, Amazon Redshift, Google BigQuery, Snowflake.

### 6.3. Data Lakes

Un **Data Lake** es un repositorio que permite almacenar **grandes volúmenes de datos en su formato nativo**: estructurados (de BD relacionales), semiestructurados (JSON, XML, CSV) y no estructurados (documentos, correos, PDFs).

**Diferencia clave frente al Data Warehouse:**

| | Data Warehouse | Data Lake |
|---|---|---|
| Los datos se cargan... | ya limpiados, procesados y transformados | tal como llegan, sin estructura previa |
| Proceso | **ETL** (transformar antes de cargar) | **ELT** (cargar primero, transformar después según el caso de uso) |
| Estructura requerida al cargar | Sí, esquema definido de antemano | No, formato nativo |

**Ejemplos:** Amazon S3, Azure Data Lake Storage.

### 6.4. Almacenes Big Data

Infraestructura computacional y de almacenamiento **distribuida**, para almacenar, escalar y procesar conjuntos de datos **masivos**.

**Ejemplo:** **Apache Hadoop.** Proporciona almacenamiento y procesamiento distribuido de grandes conjuntos de datos en clústeres de PCs con hardware básico. Uno de sus componentes principales es el **HDFS (Hadoop Distributed File System)**, un sistema de almacenamiento diseñado específicamente para Big Data.

### 6.5. Factores para elegir un repositorio

- Volumen de datos.
- Uso previsto de los datos.
- Número de transacciones.
- Frecuencia de las actualizaciones.
- Tipos de operaciones realizadas.
- Consideraciones de almacenamiento.
- Criterios de confidencialidad, integridad y disponibilidad.
- Rendimiento y latencia.
- Necesidades de privacidad, seguridad y gobernanza.

> **Idea clave:** no empieces eligiendo el producto tecnológico. Empieza por el **patrón de acceso**, la **carga**, las **garantías necesarias** y las **restricciones del sistema**.

### 6.6. Elección según el sistema: OLTP vs OLAP

| | **OLTP** (Online Transaction Processing) | **OLAP** (Online Analytical Processing) |
|---|---|---|
| Diseñado para | Gran volumen de datos operativos diarios | Análisis de datos complejos |
| Ejemplos de uso | Transacciones bancarias online, cajeros automáticos, reservas de aerolíneas | Informes, dashboards, minería de datos |
| Tecnologías típicas | Suelen ser BBDD relacionales, aunque también pueden ser NoSQL | Relacionales, pero principalmente no relacionales, Data Warehouses y almacenes Big Data |

**Ejemplo integrador (banca):** cada transacción de un cajero automático se procesa con un sistema **OLTP** (rápido, consistente, transaccional). Al final del día, esos datos se cargan en un **Data Warehouse** y se analizan con un sistema **OLAP** para generar informes de gestión de riesgo o detección de patrones de fraude.

---

## 7. Integración de datos (1.5)

### 7.1. Introducción

La **integración de datos** abarca las técnicas y herramientas que permiten a las organizaciones **acceder, transformar y almacenar** datos de diferentes tipos. Las plataformas de integración combinan múltiples fuentes de datos (físicas o lógicas) para ofrecer una **vista unificada** y facilitar el análisis.

**Escenarios de uso:**
- Consistencia de datos entre aplicaciones.
- Gestión de datos maestros o críticos (**Master Data**).
- Compartición de datos entre empresas.
- Migración de datos.

**Ejemplo:** una empresa de e-commerce necesita unificar datos de pedidos procedentes de la web, la app móvil y el ERP logístico, para tener una única visión del estado de cada pedido.

### 7.2. Proceso de integración

Una vez recopilados los datos de diferentes fuentes, es necesario **procesarlos, limpiarlos e integrarlos** para que los usuarios puedan acceder a ellos a través de una única interfaz.

```
Recolección → Procesamiento → Limpieza → Integración → Usuarios
```

| Etapa | Ejemplo |
|---|---|
| Recolección | Importar logs de un servidor |
| Procesamiento | Transformar formatos de fechas |
| Limpieza | Eliminar duplicados de clientes |
| Integración | Cruzar ventas con campañas |
| Usuarios | Analistas de marketing consultando en Power BI |

### 7.3. Pipelines de datos: ETL vs ELT

Un **pipeline de datos** es el conjunto de herramientas y procesos que cubren todo el ciclo de vida de los datos, desde los sistemas de origen hasta su destino final. Los datos se integran mediante dos procesos principales:

| | **ETL** (Extract – Transform – Load) | **ELT** (Extract – Load – Transform) |
|---|---|---|
| Orden | Se extraen, se **transforman en un servidor intermedio** y luego se cargan en el destino (Data Warehouse) | Se extraen y se **cargan primero** los datos nativos en un Data Lake/DW, y luego se transforman dentro del propio sistema |
| Ventajas | Control de calidad antes de cargar; cumple mejor con normativas (banca, salud); datos limpios desde el inicio | Escalabilidad en cloud (BigQuery, Snowflake); más rápido para carga masiva (IoT, logs web); flexibilidad: distintos equipos transforman según necesidad |
| Limitaciones | Proceso más lento (requiere servidores ETL dedicados); poco flexible si los analistas necesitan datos crudos | Riesgo de sobrecarga en el Data Warehouse; posibilidad de datos redundantes o inconsistentes si no se gestiona bien |
| Encaja mejor con | Data Warehouses tradicionales | Data Lakes / plataformas cloud modernas |

**Ejemplo aclaratorio:** un hospital que necesita cumplir estrictamente con normativa de protección de datos de pacientes probablemente prefiera **ETL** (limpiar y anonimizar antes de cargar). Una empresa de IoT que recibe millones de eventos de sensores por segundo probablemente prefiera **ELT** (cargar todo rápido en un Data Lake y transformar bajo demanda).

### 7.4. Herramientas de integración

| Categoría | Ejemplos |
|---|---|
| Comerciales | IBM, Talend, SAP, Oracle, Microsoft, Qlik, TIBCO |
| Open source | Boomi, Jitterbit, SnapLogic |
| Basadas en cloud | Adeptia, Google, IBM, Informatica |

---

## 8. Resumen / Puntos clave

- La **arquitectura de datos** conecta negocio, datos, plataformas, integración y gobierno; no es solo un catálogo de tecnologías.
- Los datos se clasifican en **estructurados, semiestructurados y no estructurados**; el tipo de dato condiciona (no determina) el repositorio.
- Los **metadatos** son "datos sobre datos" y existen varias familias (técnicos, de procesos, de negocio, de gestión/gobernanza, de procedencia/linaje, de calidad, de seguridad/IA). Permiten encontrar, entender, trazar, proteger y gobernar los datos.
- El **ciclo de vida** de los datos tiene 5 etapas: adquisición, procesamiento, almacenamiento, intercambio, conservación/eliminación. En IA se extiende a: dato → dataset → entrenamiento → modelo → inferencia → monitorización → mejora continua.
- La **gobernanza de datos** son políticas + procesos + roles que convierten los datos en un activo estratégico. Se apoya en la **cadena de valor** (datos + herramientas + personas) y en la **calidad de los datos** (6 criterios: completitud, unicidad, precisión, conformidad, oportunidad, procedencia).
- Los **roles de gobernanza** forman una cadena: CDO → Data Owners → Data Stewards → Data Managers / Data Users.
- Los **repositorios de datos** principales son: BBDD relacionales (ACID, SQL), BBDD NoSQL (BASE, para Big Data), Data Warehouses (ETL, OLAP, tres niveles), Data Lakes (ELT, formato nativo), y almacenes Big Data (Hadoop/HDFS).
- **OLTP** procesa transacciones operativas; **OLAP** procesa análisis complejo.
- La **integración de datos** unifica fuentes distintas mediante pipelines; el debate central es **ETL** (transformar antes de cargar, más control) vs **ELT** (cargar antes de transformar, más escalable en la nube).

---

## 9. Preguntas de autoevaluación

**1. ¿Por qué se dice que "la arquitectura no es una lista de tecnologías"?**
> Porque la arquitectura de datos conecta las necesidades del negocio, los datos, las plataformas de almacenamiento, los procesos de integración y el gobierno de esos datos; elegir tecnologías sueltas sin esa visión de conjunto no constituye una arquitectura.

**2. Da un ejemplo de dato semiestructurado y explica por qué no es ni estructurado ni no estructurado.**
> Un JSON devuelto por una API (p. ej. `{"user":"Ana","tweet":"Hola"}`). No es estructurado porque no sigue un esquema rígido de tablas con filas/columnas fijas; no es no estructurado porque sí tiene etiquetas y una jerarquía consistente (claves-valor) que aportan contexto.

**3. Cita las tres familias "clásicas" de metadatos y añade dos familias que se han incorporado en arquitecturas actuales.**
> Clásicas: técnicos, de procesos, de negocio (y de gestión). Añadidas en arquitecturas actuales: linaje/procedencia, calidad, seguridad/cumplimiento, IA/ML.

**4. ¿Qué diferencia hay entre un metadato técnico y un metadato de negocio, usando el ejemplo del campo "importe" en un dataset de ventas?**
> El metadato técnico describe la estructura (tipo de dato, si es un número decimal, en qué tabla y columna está). El metadato de negocio describe el significado (p. ej. "importe = total de la venta con IVA").

**5. Enumera las 5 etapas del ciclo de vida de los datos y pon un ejemplo de cada una.**
> Adquisición (formulario de registro), Procesamiento (detección de fraude en tiempo real), Almacenamiento (backup de imágenes médicas), Intercambio (compartir inventario vía API con proveedores), Conservación/Eliminación (eliminar cuentas inactivas tras 2 años).

**6. ¿Qué significa que "el ciclo de vida no termina al entrenar un modelo de IA"?**
> Porque después del entrenamiento vienen la inferencia/decisión, la monitorización del modelo en producción, la retroalimentación y las nuevas versiones, y todas esas transiciones también deben generar metadatos y evidencias trazables.

**7. ¿Cuáles son los tres pilares de la gobernanza de datos?**
> Políticas (reglas, normativas, estándares), Procesos (ciclo de vida, calidad, seguridad) y Roles (CDO, Data Owners, Data Stewards, Data Manager, Data Users).

**8. Explica la cadena de valor de los datos y por qué "los datos por sí solos no generan valor".**
> El valor surge de combinar tres elementos: los datos en sí (volumen, diversidad, calidad), las herramientas/plataformas (DBMS, analítica, IA) y la experiencia humana (análisis, visión de negocio). Sin herramientas para procesarlos ni personas que sepan interpretarlos, los datos por sí solos no aportan ninguna ventaja competitiva.

**9. Nombra los 6 criterios clásicos de calidad de datos y explica brevemente la "Oportunidad".**
> Completitud, Unicidad, Precisión, Conformidad/Validez, Oportunidad, Procedencia. La Oportunidad se refiere a que los datos estén capturados y disponibles lo suficientemente pronto para ser útiles (p. ej. informes diarios de logs).

**10. Un dato puede ser preciso, válido y completo, y aun así "no ser adecuado". ¿Por qué? Pon un ejemplo.**
> Porque la calidad depende también de la finalidad y el contexto de uso, no solo de propiedades intrínsecas del dato. Ejemplo: un dataset de edades correctamente capturado puede no cubrir el rango de edad necesario para entrenar un modelo dirigido a adolescentes (problema de representatividad).

**11. Describe la cadena de responsabilidad entre los roles de gobernanza (CDO, Data Owner, Data Steward, Data User).**
> El CDO define la estrategia y se la transmite a los Data Owners; los Data Owners delegan la definición de reglas concretas en los Data Stewards; los Data Stewards hacen cumplir esas reglas a los Data Users y a los Data Managers, que las implementan técnicamente.

**12. ¿Qué diferencia hay entre las propiedades ACID (relacionales) y BASE (NoSQL)?**
> ACID (Atomicidad, Consistencia, Aislamiento, Durabilidad) garantiza transacciones estrictamente consistentes, típico de BBDD relacionales. BASE (Basically Available, Soft state, Eventual consistency) prioriza la disponibilidad y flexibilidad sobre la consistencia inmediata, típico de BBDD NoSQL para Big Data.

**13. Explica la arquitectura de tres niveles de un Data Warehouse.**
> Nivel inferior: servidores de bases de datos (relacionales o no). Nivel intermedio: servidor OLAP, que procesa y analiza información de múltiples servidores de BBDD. Nivel superior: front-end del cliente con herramientas para consultar, generar informes y hacer analítica.

**14. ¿Cuál es la diferencia fundamental entre un Data Warehouse y un Data Lake respecto al momento de transformación de los datos?**
> En un Data Warehouse los datos se transforman **antes** de cargarlos (proceso ETL, requiere esquema definido de antemano). En un Data Lake los datos se cargan tal como llegan, en su formato nativo, y se transforman **después**, según el caso de uso (proceso ELT).

**15. ¿Cuándo conviene usar ETL frente a ELT? Pon un ejemplo de cada caso.**
> ETL conviene cuando se necesita control de calidad y cumplimiento normativo estricto antes de cargar los datos (ej. datos sanitarios en un hospital). ELT conviene cuando se necesita escalabilidad y velocidad de carga masiva en plataformas cloud modernas (ej. millones de eventos de sensores IoT cargados en un Data Lake y transformados bajo demanda).
