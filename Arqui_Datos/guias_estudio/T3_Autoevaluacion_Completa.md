# Autoevaluación completa — Tema 3 Redis

*Arquitectura de Datos — UC3M | Todas las preguntas de T3.1 a T3.5 con respuesta*

---

## T3.1 — Introducción a Redis (15 preguntas)

**1. ¿Por qué Redis se considera un "servidor de estructuras de datos" y no simplemente una caché?**
Porque, además de strings, ofrece hashes, listas, sets, sorted sets, bitmaps, HyperLogLog, etc., todos con operaciones atómicas de alto nivel (incrementar, push/pop, añadir a un ranking) que la aplicación no tiene que reimplementar. Memcached solo almacena cadenas.

**2. ¿`MULTI`/`EXEC` en Redis ofrece las mismas garantías que una transacción SQL con aislamiento?**
No. Garantiza que los comandos del lote se ejecutan de forma serial sin intercalar comandos de otros clientes (atomicidad de lote), pero no ofrece aislamiento real: otro cliente puede ver estados intermedios antes del `EXEC`. Para aislamiento real se usa `WATCH` + reintento.

**3. ¿Por qué el diseño monohilo de Redis no es un cuello de botella en la mayoría de los casos?**
Porque cada operación tarda nanosegundos (todo en RAM) y la E/S de red es no bloqueante (epoll/kqueue/IOCP). Mientras un cliente espera su respuesta, el hilo sigue procesando comandos de otros clientes, alcanzando cientos de miles de comandos por segundo sin locks.

**4. Un contador de "likes" con `INCR` se pierde si Redis cae sin persistencia. ¿Por qué empresas como Instagram lo usan igualmente?**
Porque `INCR` es atómico y O(1), y el patrón Write-Behind persiste el dato de forma asíncrona en una BD duradera (Cassandra). El riesgo de perder unos segundos de likes es aceptable frente al beneficio de latencia.

**5. Diferencia entre RDB y AOF como mecanismos de persistencia.**
RDB hace snapshots periódicos (rápido, compacto, puede perder cambios entre snapshots). AOF registra cada operación de escritura (más seguro, como máximo ~1s de pérdida con `appendfsync everysec`, ficheros más grandes). El modo híbrido combina ambos.

**6. ¿Por qué no se pueden hacer JOINs en Redis?**
Porque cada clave es independiente; no existe mecanismo interno de relación entre claves. Para "unir" datos habría que leer múltiples claves y combinarlas en la lógica de la aplicación.

**7. Explica la diferencia entre `KEYS` y `SCAN`, y por qué importa en producción.**
`KEYS` es O(N) y bloquea el hilo único de Redis durante toda la búsqueda. `SCAN` es iterativo (usa un cursor) y no bloquea el servidor. En producción siempre se usa `SCAN`.

**8. ¿Qué es el optimistic locking con `WATCH` y cuándo se prefiere sobre un lock pesimista?**
`WATCH` monitoriza una clave; si cambia entre `WATCH` y `EXEC`, la transacción se aborta (devuelve `nil`) y hay que reintentar. Se prefiere cuando los conflictos son poco frecuentes, ya que evita el coste de bloquear recursos innecesariamente.

**9. ¿Qué diferencia hay entre `pipeline` y `MULTI`/`EXEC`?**
Pipeline agrupa comandos en un único roundtrip TCP para reducir latencia de red (no garantiza atomicidad). `MULTI`/`EXEC` garantiza que los comandos del bloque se ejecutan sin interrupciones de otros clientes (no reduce necesariamente roundtrips). Pueden combinarse.

**10. ¿Qué llevó a la creación de Valkey y qué implica para un desarrollador que ya usa Redis?**
En marzo de 2024, Redis Ltd. cambió la licencia de BSD a RSALv2/SSPL. Esto motivó Valkey, fork de la Linux Foundation bajo BSD-3, totalmente compatible con el protocolo RESP. Migrar de Redis a Valkey suele ser un cambio de configuración, no de código.

**11. Da un ejemplo de cuándo Redis NO sería la herramienta adecuada.**
Como BD principal de un sistema de facturación que necesita transacciones ACID completas, relaciones entre tablas y consultas complejas con JOINs — ahí una BD relacional es lo correcto; Redis podría usarse como caché complementaria.

**12. ¿Qué significa que Redis tenga "consistencia eventual" con réplicas, y en qué casos no es suficiente?**
Las réplicas convergerán al mismo valor que el primary, pero sin garantía exacta de cuándo. Es suficiente para cachés y sesiones, pero no para pagos o inventario crítico donde se necesita consistencia fuerte.

**13. ¿Por qué se recomienda `HGET` en lugar de `HGETALL` cuando solo se necesita un campo?**
`HGET` es O(1). `HGETALL` es O(N) y puede bloquear el hilo único si el hash tiene millones de campos, afectando a todos los clientes conectados.

**14. Explica el patrón de rate limiting con `INCR` y `EXPIRE`/`EX`.**
Se crea una clave por usuario con contador a 0 y TTL igual a la ventana (p. ej. `EX 3600`). En cada petición se ejecuta `INCR`; si el valor supera el límite, se rechaza (HTTP 429). Al expirar el TTL, el contador se reinicia automáticamente.

**15. ¿Qué lección deja el caso de Twitch Plays Pokémon sobre la escalabilidad de Redis?**
Que al ser single-threaded, una sola instancia tiene un límite de CPU claro (un núcleo). Cuando ese límite se alcanza, la solución es particionar los datos entre múltiples instancias (sharding por canal de chat), no optimizar el código — que es exactamente lo que resuelve Redis Cluster.

---

## T3.2 — Estructuras de datos (15 preguntas)

**1. ¿Qué estructura usarías para guardar el perfil de un usuario con nombre, email y ciudad, que siempre se leen juntos?**
Un **Hash** (`HSET user:1000 name ... email ... ciudad ...`), porque los campos se leen siempre juntos y se ahorra memoria permitiendo `HGETALL` en una sola llamada.

**2. Si ese mismo usuario tiene un campo con TTL de 30 minutos (sesión) y otro sin expiración (perfil), ¿qué harías?**
Separar en dos claves distintas: una para la sesión con `SETEX`/`EX` de 30 min, y otra (Hash) para el perfil sin TTL. Un Hash único no permite TTL por campo, solo por clave completa.

**3. ¿Por qué Twitter usa Sorted Sets en lugar de Lists para las timelines?**
Porque necesita mantener orden cronológico y poder consultar por rango de tiempo (`ZREVRANGEBYSCORE`) usando el timestamp como `score`. Una List no tiene noción de puntuación ni permite búsquedas eficientes por rango de valores.

**4. Si un post de Instagram recibe 10 millones de "me gusta", ¿qué estructura usarías para el contador y por qué?**
Un **String numérico con `INCR`**: operación atómica O(1), sin condiciones de carrera bajo alta concurrencia. Un Set con un miembro por usuario sería mucho más costoso en memoria y no es necesario si solo se quiere el conteo.

**5. ¿Cuál es la diferencia entre `SINTER` y `SUNION`?**
`SINTER` devuelve los elementos presentes en **todos** los conjuntos (intersección). `SUNION` devuelve los elementos presentes en **al menos uno** de los conjuntos (unión).

**6. ¿Qué ventaja tiene `BRPOP` frente a hacer `RPOP` en un bucle (polling)?**
`BRPOP` bloquea la conexión del cliente a nivel de red sin consumir CPU, y Redis lo despierta en cuanto hay datos. El polling con `RPOP` desperdicia CPU y añade latencia media hasta el intervalo del sondeo.

**7. ¿Qué es un `intset` y cuándo lo usa Redis?**
Es la codificación interna de un Set cuando **todos** sus elementos son enteros y el conjunto es pequeño (~128 elementos). Es un array ordenado de enteros con búsqueda binaria O(log N), más eficiente en memoria que una hash table porque no usa punteros por elemento.

**8. ¿Qué estructura interna usa un Sorted Set grande y qué complejidad ofrece?**
Una combinación de **skip list** (recorridos ordenados y por rango, O(log N)) y **hash table** (acceso directo O(1) por miembro, como en `ZSCORE`).

**9. Explica la diferencia entre fan-out en escritura y fan-out en lectura para un feed social.**
Fan-out en escritura: al publicar, se copia el contenido a los timelines de todos los seguidores (lectura rápida O(1), escritura cara). Fan-out en lectura: al consultar el feed, se buscan los posts de las cuentas seguidas (escritura barata, lectura más lenta). Instagram usa un híbrido según el número de seguidores.

**10. ¿Por qué `HSET` puede ser más eficiente en memoria que guardar el mismo objeto como JSON en un String?**
Porque un Hash usa codificación compacta (listpack) sin el overhead de sintaxis JSON y permite lectura/escritura de campos individuales sin deserializar el objeto completo. En la práctica supone un ahorro de ~30% de memoria.

**11. ¿Qué comando usarías para saber cuántos usuarios están online a la vez, sin duplicados?**
`SADD usuarios:online <user_id>` en cada conexión, y `SCARD usuarios:online` para obtener el total sin duplicados (O(1)).

**12. ¿Cuándo usarías HyperLogLog en lugar de un Set para contar elementos únicos?**
Cuando la cardinalidad puede ser muy alta (millones) y no es necesario saber *cuáles* son los elementos, solo *cuántos* únicos hay, y se puede tolerar ~0,81% de error. Un Set almacenaría cada elemento (mucha memoria); HyperLogLog usa solo ~12 KB.

**13. ¿Qué problema resuelve `MULTI`/`EXEC` que `HSET` y `HGET` por sí solos no resuelven?**
La atomicidad entre varias operaciones sobre distintos campos o claves. `HSET`/`HGET` son atómicos individualmente, pero si se necesita que varias operaciones se ejecuten como una unidad indivisible, hace falta `MULTI`/`EXEC` (o un script Lua).

**14. Da un ejemplo de cuándo usarías RedisJSON en lugar de un Hash.**
Cuando el objeto tiene estructura **anidada** (p. ej. `address` con subcampos `city` y `zip`) y se necesita consultar o actualizar rutas anidadas directamente (`$.address.city`), algo que un Hash plano no soporta de forma nativa.

**15. Si tuvieras que justificar en un examen por qué elegiste Sorted Set y no List para un ranking de videojuego, ¿qué dirías?**
Un ranking necesita mantener a los jugadores **ordenados por puntuación** y consultar la posición (`ZRANK`) o el top N (`ZREVRANGE`) eficientemente (O(log N)). Una List solo mantiene orden de inserción y no tiene noción de puntuación, por lo que habría que reordenarla manualmente en cada actualización (O(N log N) por actualización).

---

## T3.3 — Caché y patrones de uso (15 preguntas)

**1. ¿Por qué Redis en memoria es tan efectivo como caché delante de una BD en disco?**
Porque la diferencia de latencia es de varios órdenes de magnitud: RAM (~100 ns) es ~1.000× más rápida que SSD (~100 µs) y ~100 millones de veces más rápida que una petición cross-region. Esa brecha, no una simple mejora incremental, justifica la caché.

**2. ¿Qué es el *cache miss penalty* en Cache-Aside y por qué ocurre?**
Es el coste de un fallo de caché: hacen falta **tres accesos** en lugar de uno — `GET` a Redis (falla), `SELECT` a la BD, y `SET` de vuelta a Redis — antes de poder devolver el dato al cliente.

**3. ¿Cuál es la diferencia fundamental entre Write-Through y Write-Behind?**
Write-Through escribe de forma **síncrona** en la BD antes de confirmar al cliente (consistencia fuerte, escritura más lenta). Write-Behind confirma inmediatamente tras escribir en Redis y persiste en la BD de forma **asíncrona** vía una cola (escritura muy rápida, con riesgo de pérdida de datos si la cola falla).

**4. ¿Por qué Instagram usa Write-Behind para los "me gusta" y no Write-Through?**
Porque necesita máxima velocidad de escritura (millones de likes/día) y puede tolerar la pérdida temporal de unos pocos likes si el sistema falla antes de persistir en Cassandra — el impacto es mínimo comparado con el coste de duplicar cada escritura de forma síncrona.

**5. ¿En qué se diferencia Read-Through de Cache-Aside?**
En quién carga el dato desde la BD tras un fallo de caché. En Cache-Aside lo hace la aplicación (lógica explícita). En Read-Through lo hace la propia caché de forma transparente — la aplicación solo habla con Redis, y este consulta la BD internamente.

**6. Explica el problema del *thundering herd* y dos formas de mitigarlo.**
Ocurre cuando expira una clave muy consultada y cientos de clientes consultan simultáneamente la BD para regenerarla, saturándola. Se mitiga con: (1) un **lock** con `SETNX ... NX EX` para que solo un cliente cargue el dato, o (2) **jitter**: aleatorizar el TTL de claves similares para que no expiren todas a la vez.

**7. ¿Por qué `SET lock:clave "loading" NX EX 10` resuelve el thundering herd y sin `NX` no lo haría?**
`NX` garantiza que la operación solo tiene éxito si la clave no existía — solo un cliente obtiene `OK`, el resto recibe `nil`. Sin `NX`, todos los clientes sobrescribirían la clave y todos "creerían" tener el lock, sin ninguna exclusión real.

**8. ¿Cuál es la diferencia entre invalidación y evicción de caché?**
La invalidación resuelve un problema de **consistencia**: mantener la caché sincronizada con los datos reales cuando cambian (TTL, `DEL` por evento). La evicción resuelve un problema de **memoria**: qué claves eliminar cuando la caché se llena (políticas LRU/LFU).

**9. ¿Qué diferencia hay entre las políticas `allkeys-lru` y `volatile-lru`?**
`allkeys-lru` puede expulsar cualquier clave, tenga o no TTL. `volatile-lru` solo expulsa claves con TTL asignado; las claves sin TTL (configuración permanente) nunca se eliminan con esta política, aunque la memoria esté llena.

**10. Si no configuras `maxmemory-policy`, ¿qué ocurre cuando Redis se queda sin memoria?**
Se aplica `noeviction` por defecto: Redis **devuelve error** en las operaciones de escritura en lugar de expulsar claves. Por eso en producción se recomienda configurar explícitamente `allkeys-lru` u otra política adecuada.

**11. ¿Por qué GitHub usa Sorted Sets (`ZADD`) en lugar de `INCR` + `EXPIRE` para su rate limiting?**
Porque `INCR` y `EXPIRE` son dos comandos separados y no son atómicos entre sí: si el cliente falla entre ambos, la clave queda sin TTL (fuga de memoria). `ZADD` registra cada petición como una operación atómica única, sin depender de una segunda llamada para fijar la expiración.

**12. Describe el patrón de rate limiting con ventana deslizante usando Sorted Sets.**
Cada petición se añade con `ZADD` usando el timestamp como score. Antes de contar, se eliminan con `ZREMRANGEBYSCORE` las entradas anteriores al inicio de la ventana. Después se cuenta con `ZCARD`; si supera el límite, se rechaza (HTTP 429). A diferencia de `INCR`+`EXPIRE`, la ventana es continua y no se reinicia de golpe.

**13. ¿Qué significa que la caché de Netflix (EVCache) tenga *fallback* automático a Cassandra?**
Que si la consulta a la caché falla o no encuentra el dato, el sistema recurre automáticamente a Cassandra como fuente de verdad, sin que la aplicación gestione ese caso manualmente. Es el patrón Lookaside Cache: Aplicación → EVCache → Cassandra.

**14. ¿Por qué se recomienda usar `SETEX` en lugar de `SET` seguido de `EXPIRE` al crear una sesión?**
Porque `SETEX clave TTL valor` combina `SET` y `EXPIRE` en una única operación atómica. Si se hicieran por separado, un fallo entre ambas llamadas dejaría una clave de sesión sin TTL — riesgo de fuga de memoria y sesiones que nunca expiran.

**15. Un ranking de videojuego necesita mostrar el top 10 en tiempo real y servir de caché ante caídas de la BD. ¿Qué combinación aplicarías?**
Un **Sorted Set** para el ranking, estrategia **Cache-Aside** para poblarlo desde la BD si Redis se reinicia, **invalidación por evento** cuando cambian las puntuaciones, y política de expulsión `allkeys-lru`/`volatile-lru` si convive con otros datos en la misma instancia.

---

## T3.4 — Persistencia y distribución (12 preguntas)

**1. ¿Cuál es la diferencia fundamental entre RDB y AOF?**
RDB crea snapshots periódicos del dataset completo en un fichero binario comprimido — rápido para backups pero puede perder datos entre snapshots. AOF registra cada operación de escritura en un log — pierde como máximo 1 segundo de datos (con `appendfsync everysec`) pero genera ficheros más grandes y recuperación más lenta.

**2. ¿Qué es Copy-on-Write (COW) y por qué permite que BGSAVE no bloquee el servidor?**
COW es un mecanismo del SO por el que el proceso hijo comparte las mismas páginas de memoria con el padre. Solo cuando uno escribe en una página, el SO crea una copia. Esto permite que el snapshot se tome sin copiar toda la memoria de golpe, y el padre sigue procesando comandos mientras el hijo escribe a disco.

**3. ¿Cuándo se recomienda el modo híbrido RDB+AOF y por qué?**
En producción, siempre. RDB permite backups y restauraciones rápidas; AOF garantiza una pérdida máxima de 1 segundo de datos. La combinación ofrece velocidad de backup de RDB y durabilidad de AOF. `aof-use-rdb-preamble yes` hace que el AOF empiece con un snapshot RDB para una recuperación más eficiente.

**4. ¿Qué diferencia hay entre SDOWN y ODOWN en Redis Sentinel?**
SDOWN (Subjectively Down) es la opinión de un único Sentinel: detecta que el primary no responde, pero puede ser un problema de red del propio Sentinel. ODOWN (Objectively Down) es el consenso: múltiples Sentinels han confirmado que el primary está caído, alcanzando el quórum. Solo con ODOWN se inicia el failover.

**5. ¿Por qué se necesitan mínimo 3 Sentinels (y no 2)?**
Para el quórum. Con 2 Sentinels y quórum=2, si uno pierde conectividad con el primary (pero este está bien), no puede alcanzar quórum solo y el failover no se ejecuta. Con 3 Sentinels y quórum=2, dos deben estar de acuerdo, evitando que un Sentinel aislado ejecute un failover fantasma.

**6. ¿Cómo sabe Redis Cluster a qué nodo enviar una clave?**
Calculando el hash slot: `Slot = CRC16(key) % 16384`. El cluster tiene 16.384 slots distribuidos entre los nodos. El cliente calcula el slot y envía la clave al nodo responsable. Si se equivoca de nodo, este responde con `-MOVED <slot> <ip:port>` y el cliente redirige automáticamente.

**7. ¿Qué es el Gossip Protocol en Redis Cluster y para qué sirve?**
Es el protocolo de comunicación peer-to-peer entre nodos. Cada segundo, cada nodo envía un PING a un nodo aleatorio con su visión del estado del cluster; el receptor responde con PONG con su propia visión. Tras pocas rondas, todos los nodos convergen sin necesidad de un coordinador central.

**8. ¿Qué significa `mem_fragmentation_ratio` y cuándo es un problema?**
Es el ratio entre la RAM que el SO asigna a Redis (`used_memory_rss`) y la que Redis cree que usa (`used_memory`). Un ratio > 1.5 indica mucha fragmentación. Un ratio < 1 indica que Redis usa swap (órdenes de magnitud más lento que RAM) — situación muy grave.

**9. ¿Qué hace `evicted_keys` y cuándo deberías preocuparte?**
Muestra el número de claves eliminadas por falta de memoria. Si este valor crece constantemente, Redis está activamente expulsando datos — la caché está llena. La solución es aumentar `maxmemory`, añadir más RAM, reducir TTLs o revisar la política de evicción.

**10. ¿Cuándo elegirías Sentinel y cuándo Redis Cluster?**
Sentinel es para **alta disponibilidad** con un único shard (los datos caben en un nodo): detecta caídas y hace failover en 10-30 segundos. Redis Cluster es para **escalabilidad horizontal**: cuando los datos no caben en un nodo o la carga de escritura supera la capacidad de un único primary.

**11. Si Redis usa `fork()` en un dataset de 50 GB, ¿qué problema de latencia podrías experimentar?**
En datasets grandes, el `fork()` puede causar un pico de latencia porque copia la tabla de páginas completa. Si el proceso padre escribe mucho durante el snapshot (COW activo), la memoria puede duplicarse temporalmente. Solución: usar instancias más pequeñas con Redis Cluster o planificar BGSAVE en horas de baja carga.

**12. ¿Qué diferencia hay entre Sentinel (alta disponibilidad) y Cluster (escalabilidad)?**
Sentinel resuelve **disponibilidad** con un único shard: promueve automáticamente una réplica si el primary cae, con downtime de 10-30 segundos. Cluster resuelve **escala**: distribuye datos en múltiples shards, cada uno responsable de un rango de hash slots. Cluster incluye failover automático por shard, combinando disponibilidad y escalabilidad.

---

## T3.5 — Ejemplo integrador: tienda en línea (10 preguntas)

**1. ¿Por qué se usa un Hash y no un String para el carrito de compra?**
Porque el carrito tiene múltiples campos (producto → cantidad) y el Hash permite operar sobre campos individuales de forma atómica (`HINCRBY cart:user:1000 "product:456" 1`) sin serializar/deserializar el objeto completo. Con un String habría que hacer GET → deserializar → modificar → serializar → SET, con riesgo de condición de carrera si dos peticiones modifican el carrito simultáneamente.

**2. El catálogo usa Cache-Aside con TTL de 5 minutos. ¿Por qué no Write-Through?**
Porque el catálogo se lee con mucha más frecuencia de lo que se escribe. Cache-Aside es ideal para alta ratio lectura/escritura: solo se carga en caché lo que realmente se consulta. Write-Through doblaría el coste de cada actualización de producto sin beneficio real cuando las actualizaciones son infrecuentes.

**3. ¿Por qué se usa `INCR` para los contadores de vistas y compras, en lugar de leer, incrementar y escribir?**
Porque `INCR` es atómico: Redis lo ejecuta como una sola instrucción. Si dos peticiones simultáneas hicieran GET → +1 → SET por separado, podrían leer el mismo valor y la segunda escritura sobreescribiría el incremento de la primera (pérdida de 1 vista). Con `INCR` es imposible: cada llamada incrementa exactamente en 1, sin importar la concurrencia.

**4. ¿Por qué se vacía el carrito con `DEL cart:user:1000` después de completar la compra, en lugar de esperar al TTL?**
Por coherencia del estado: una vez procesada la compra, el carrito no debe seguir accesible. Esperar al TTL de 24h significaría que el usuario podría ver un carrito con productos ya comprados. La invalidación explícita con `DEL` es correcta para datos que cambian de estado por un evento, no por el paso del tiempo.

**5. En el flujo de compra, ¿por qué se usa `ZINCRBY top:sales 1 "product:456"` y no `ZADD` con el nuevo score?**
Porque `ZINCRBY` incrementa el score de forma atómica, sin necesitar el valor actual. Con `ZADD` habría que hacer `ZSCORE` → leer → sumar 1 → `ZADD` con el nuevo valor, introduciendo una condición de carrera si dos compras del mismo producto ocurren simultáneamente.

**6. ¿Qué es lo que Redis NO debería ser en la arquitectura de la tienda?**
La fuente de verdad para datos transaccionales críticos (órdenes, pagos, inventario exacto). Las órdenes van a MariaDB porque necesitan garantías ACID que Redis no ofrece. Redis complementa la BD, no la sustituye.

**7. Si la tienda crece de 1.000 a 1.000.000 de usuarios diarios, ¿qué cambios de arquitectura necesitarías en Redis?**
(1) **Replicación** con 1 primary + 2-3 réplicas para escalar lecturas; (2) **Sentinel** para failover automático; (3) Evaluar **Redis Cluster** si el dataset supera la RAM de un nodo o la tasa de escritura supera la capacidad del primary; (4) Revisar la política de evicción (`allkeys-lru`) y `maxmemory`.

**8. ¿Qué patrón de caché usarías para 10.000 productos con 500.000 usuarios diarios? Justifica.**
**Cache-Aside** con TTL de 5-15 minutos y jitter (±10%). Razones: (1) alta ratio lectura/escritura; (2) simple de implementar; (3) dataset manejable en RAM (~10 MB si cada producto es ~1 KB); (4) los productos más vendidos concentran la mayor parte del tráfico (Pareto), por lo que el hit rate natural será alto. El jitter evita el *thundering herd* cuando muchos productos populares expiran a la vez.

**9. ¿Por qué se usa `ZREVRANGE top:sales 0 9 WITHSCORES` y no `ZRANGE` para el top 10?**
Porque `ZREVRANGE` devuelve los elementos de mayor a menor score (descendente), que es lo que necesita un ranking: el producto más vendido primero. `ZRANGE` devuelve de menor a mayor, mostrando el producto menos vendido primero.

**10. ¿Cuál es la diferencia entre invalidación por TTL y por evento, y cuándo usar cada una en la tienda?**
La invalidación por **TTL** borra la clave automáticamente al expirar (p. ej. catálogo a los 5 min) — adecuada cuando el dato cambia con poca frecuencia y una consistencia eventual es aceptable. La invalidación por **evento** (`DEL`) borra la clave cuando algo ocurre (p. ej. vaciar el carrito tras la compra) — adecuada cuando el dato cambia de estado por una acción concreta y no se puede tolerar que el obsoleto persista hasta el TTL.

---

## Tabla resumen de estructuras

| Estructura | Cuándo usarla | Complejidad | Interno |
|---|---|---|---|
| String | Valor único, contador, TTL simple | O(1) | int / embstr / raw |
| Hash | Objeto con varios campos independientes | O(1) por campo | listpack → hash table |
| List | Cola FIFO (`BRPOP`), pila LIFO, historial | O(1) extremos | listpack → quicklist |
| Set | Conjunto sin duplicados, operaciones conjuntos | O(1) inserción/membresía | intset → listpack → hash table |
| Sorted Set | Ranking, timeline ordenada por score | O(log N) | listpack → skip list + hash table |

## Tabla resumen de estrategias de caché

| Estrategia | Quién carga el dato | Consistencia | Cuándo usarla |
|---|---|---|---|
| Cache-Aside | La aplicación | Eventual (TTL) | Alta ratio lectura/escritura |
| Write-Through | La aplicación (escribe en BD y caché) | Fuerte | Escrituras frecuentes, consistencia crítica |
| Write-Behind | La aplicación (escribe en caché; BD en cola) | Eventual | Escrituras masivas tolerantes a pérdida leve |
| Read-Through | La propia caché (transparente) | Eventual | Simplificar lógica de aplicación |
