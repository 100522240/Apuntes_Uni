# Cheatsheet de repaso — Redis (T3.1 + T3.2)

*Arquitectura de Datos — 4.º curso UC3M · Para repasar justo antes del examen*

> Esta hoja no sustituye a las guías completas (`T3.1 Introducción a Redis.md`, `T3.2 Estructuras de datos.md`), es un **resumen operativo**: por cada bloque tienes el concepto en 2-3 líneas + la tabla/comando que necesitas + un ejemplo resuelto. Los apartados marcados ⚠️ son trampas típicas de examen.

---

## 1. Los 5 conceptos que tienen que estar automatizados

| # | Concepto | En una frase |
|---|---|---|
| 1 | Redis = **servidor de estructuras de datos**, no una caché de strings | Memcached solo guarda strings; Redis ofrece hashes, listas, sets, zsets con operaciones atómicas |
| 2 | **Monohilo + E/S no bloqueante** (epoll/Reactor) | Un solo hilo ejecuta comandos → sin locks/deadlocks; miles de conexiones concurrentes sin bloquear |
| 3 | "Atómico" en Redis **≠** ACID/aislamiento SQL | `MULTI`/`EXEC` no permite que se intercalen comandos de otros, pero no aísla lecturas intermedias |
| 4 | Cada estructura tiene un **patrón de acceso óptimo** | La pregunta de examen es "¿cómo se lee/escribe esto?", no "¿qué comando existe?" |
| 5 | Redis es **complemento**, no sustituto de una BD relacional | Sin JOINs, 512 MB máx. por valor, coste RAM alto, persistencia no-ACID |

---

## 2. Mapa de decisión: ¿qué estructura uso?

```
¿Es un valor único (o número) con TTL?  ──────────────► String
¿Tiene varios campos que se leen juntos?  ────────────► Hash
¿Necesito cola FIFO / pila LIFO?  ────────────────────► List
¿Necesito unicidad + intersección/unión de conjuntos? ─► Set
¿Necesito orden por puntuación (ranking, timeline)?  ──► Sorted Set (ZSET)
¿Necesito contar únicos a gran escala con poco error? ─► HyperLogLog
¿Necesito coordenadas + radio de búsqueda?  ───────────► Geo
```

**Ejemplo resuelto:** *"Modela el sistema de 'me gusta' de un post de Instagram con 10M de likes."*
- Si solo necesito el **número total** → `String` + `INCR likes:post:123` (O(1), atómico, sin guardar cada usuario).
- Si necesito saber **quién** dio like (para mostrar avatares) → `Set` con `SADD likes:post:123 user:1000`.
- Si solo necesito el **conteo único aproximado** a escala masiva (p. ej. "vistos por" de una Story) → `HyperLogLog` (12 KB fijos, error ~0,81%).

Tres estructuras distintas para tres preguntas distintas sobre el *mismo* dato — esta es la clase de razonamiento que se pide en el examen.

---

## 3. Strings

**Qué son:** el tipo base; texto, número o bytes bajo una única clave. Se usa cuando el objeto **tiene un solo valor relevante**.

```bash
SET user:1000:name "Juan García"
GET user:1000:name                     # "Juan García"

# Contador atómico
SET views:456 0
INCR views:456        # 1  — O(1), atómico incluso con miles de clientes concurrentes
INCRBY views:456 10   # 11
DECR views:456        # 10

# TTL en una sola operación (clave para caché/sesión)
SET cache:product:456 "iPhone" EX 300   # expira en 5 min
SETEX session:abc123 1800 "userId:1000"

# Multi-operación (1 roundtrip en vez de N)
MSET user:1000:name "Juan" user:1000:email "juan@x.com"
MGET user:1000:name user:1000:email
```

**Ejemplo explicado — rate limiting (GitHub, 5.000 peticiones/hora):**
```bash
INCR ratelimit:user:1000:2026091815     # clave = usuario + hora actual
EXPIRE ratelimit:user:1000:2026091815 3600   # solo la primera vez (o SET ... EX de inicio)
# Si el valor devuelto por INCR > 5000  →  rechazar con HTTP 429
```
*Por qué funciona:* `INCR` es atómico (nadie más puede intercalarse), y el TTL hace que el contador se "autolimpie" sin necesidad de un cron — al cabo de una hora la clave desaparece sola y el próximo `INCR` la vuelve a crear desde 0 (`INCR` sobre clave inexistente empieza en 1).

⚠️ **Trampa:** `SET` sirve tanto para crear como para actualizar (no hay distinción INSERT/UPDATE como en SQL).

---

## 4. Hashes

**Qué son:** mapa `campo → valor` bajo una clave. Se usa para objetos con **varios campos que se actualizan por separado**.

```bash
HSET user:1000 name "Juan García" email "juan@x.com" age 28
HGET user:1000 name                # "Juan García" — O(1), no lee el resto
HMGET user:1000 name email
HGETALL user:1000                  # todo el objeto — O(N)
HINCRBY user:1000 age 1            # 29
HDEL user:1000 age
HEXISTS user:1000 email            # 1
```

**Ejemplo explicado — perfil de Instagram (embeber):**
```bash
HSET user:5000:profile nombre "Ana" bio "Fotógrafa" avatar_url "cdn.../a.jpg"
# Cambiar solo la bio, sin tocar el resto:
HSET user:5000:profile bio "Nueva bio"
```
*Por qué Hash y no un `SET` con JSON serializado:* si guardases `SET user:5000:profile '{"nombre":"Ana","bio":"...","avatar_url":"..."}'`, para cambiar solo la bio tendrías que leer todo el JSON, deserializar, modificar el campo, volver a serializar y escribir el string entero. Con Hash, `HSET user:5000:profile bio "..."` es una operación directa. Además, un Hash ocupa ~30% menos memoria que el mismo objeto como JSON (sin comillas ni llaves repetidas).

⚠️ **Trampa:** `HGETALL` es O(N) — con un hash de millones de campos puede bloquear el hilo único de Redis. Si solo necesitas un campo, usa siempre `HGET`.

---

## 5. Lists

**Qué son:** listas enlazadas bidireccionales. Inserción/extracción O(1) **solo por los extremos**. Base de colas (FIFO) y pilas (LIFO).

```bash
LPUSH cola:tareas "tarea1"     # inserta por la izquierda
RPUSH cola:tareas "tarea2"     # inserta por la derecha
LPOP cola:tareas               # extrae por la izquierda
RPOP cola:tareas               # extrae por la derecha
LRANGE cola:tareas 0 -1        # ver todos los elementos
LLEN cola:tareas
```

**Ejemplo explicado — cola productor/consumidor (patrón usado por Twitter para notificaciones):**
```bash
# Productor: encola un tweet para procesar
LPUSH cola:notificaciones '{"tweet":"abc123","tipo":"mencion"}'

# Consumidor (worker): espera de forma bloqueante, sin polling
BRPOP cola:notificaciones 0
# → si hay un elemento, lo devuelve inmediatamente
# → si no hay nada, el cliente queda "dormido" en la red (sin gastar CPU)
#   hasta que otro proceso haga LPUSH, o hasta que pase el timeout (0 = infinito)
```
*Por qué `BRPOP` y no un bucle con `RPOP` cada 100ms:* el *polling* desperdicia CPU constantemente y añade hasta 100ms de latencia media. `BRPOP` bloquea la conexión a nivel de red — Redis despierta al cliente en el instante exacto en que hay un dato, coste cero mientras espera.

⚠️ **Trampa:** `LPUSH` + `LPOP` (mismo extremo) = pila LIFO. `LPUSH` + `RPOP` (extremos opuestos) = cola FIFO. Confundir el extremo es un error clásico de examen.

---

## 6. Sets

**Qué son:** conjuntos desordenados **sin duplicados**, con operaciones de teoría de conjuntos.

```bash
SADD online "u1" "u2" "u3"
SISMEMBER online "u1"          # 1
SCARD online                   # 3 (tamaño)
SREM online "u1"

SADD likes:post1 "u1" "u2" "u3"
SADD likes:post2 "u2" "u3" "u4"
SINTER likes:post1 likes:post2   # {u2, u3} — dieron like a AMBOS
SUNION likes:post1 likes:post2   # {u1,u2,u3,u4} — dieron like a CUALQUIERA
SDIFF  likes:post1 likes:post2   # {u1} — solo en post1
```

**Ejemplo explicado — recomendación por intereses comunes (GitHub stars):**
```bash
SADD repo:456:stars "user:1" "user:2" "user:3"
SADD repo:789:stars "user:2" "user:3" "user:4"

SINTER repo:456:stars repo:789:stars
# → {user:2, user:3}  → a estos usuarios se les podría recomendar
#   "otros usuarios que han dado star a X también dieron star a Y"
```
*Por qué Set y no List:* necesitamos (a) que no haya duplicados si un usuario da doble-click y (b) las operaciones `SINTER`/`SUNION`/`SDIFF`, que no existen para listas. `SISMEMBER` es O(1) frente a recorrer una lista entera para comprobar pertenencia.

---

## 7. Sorted Sets (ZSET)

**Qué son:** como un Set, pero cada miembro tiene un **score** numérico y se mantiene siempre ordenado por él. Estructura interna: **skip list** (rango O(log N)) + hash table (acceso directo O(1)).

```bash
ZADD ranking 1500 "p1"
ZADD ranking 2300 "p2"
ZADD ranking 1800 "p3"

ZRANGE ranking 0 -1 WITHSCORES        # orden ascendente
ZREVRANGE ranking 0 2 WITHSCORES      # top 3 descendente
ZSCORE ranking "p1"                   # 1500
ZRANK ranking "p1"                    # posición (0-based, ascendente)
ZINCRBY ranking 100 "p1"              # 1600
ZRANGEBYSCORE ranking 1000 2000       # todos entre 1000 y 2000
```

**Ejemplo explicado — timeline de Twitter (score = timestamp):**
```bash
ZADD user:1000:timeline 1726500000 "tweet:abc123"
ZADD user:1000:timeline 1726500300 "tweet:def456"

# Los 50 tweets más recientes:
ZREVRANGE user:1000:timeline 0 49 WITHSCORES

# Solo los de los últimos 30 minutos:
ZREVRANGEBYSCORE user:1000:timeline +inf 1726498500
```
*Por qué ZSET y no List:* usar el timestamp como *score* da el orden cronológico "gratis" (no hay que reordenar nada manualmente al insertar) y permite `ZREVRANGEBYSCORE` para filtrar por ventana de tiempo — algo que una List no puede hacer (solo tiene orden de inserción y acceso por posición, no por valor).

⚠️ **Trampa clásica de examen:** *"¿Por qué no usar una List ordenada manualmente para un ranking?"* → porque insertar manteniendo orden en una List es O(N) (hay que desplazar elementos), mientras que `ZADD` es O(log N) gracias a la skip list.

---

## 8. Tabla resumen de las 5 estructuras (memorizar)

| Estructura | Comandos clave | Complejidad típica | Caso de uso canónico |
|---|---|---|---|
| **String** | `SET` `GET` `INCR` | O(1) | Caché, contadores, sesiones simples |
| **Hash** | `HSET` `HGET` `HGETALL` | O(1) por campo, O(N) `HGETALL` | Perfiles/objetos con varios campos |
| **List** | `LPUSH` `RPUSH` `LPOP` `BRPOP` | O(1) extremos, O(N) rango | Colas FIFO/LIFO, historiales |
| **Set** | `SADD` `SISMEMBER` `SINTER` | O(1) inserción/membresía | Unicidad, "quién dio like a ambos" |
| **Sorted Set** | `ZADD` `ZRANGE` `ZRANK` | O(log N) | Rankings, leaderboards, timelines |

---

## 9. Embeber vs referenciar (decisión de modelado)

```
¿Los campos se leen SIEMPRE juntos?           → SÍ → Hash (embeber)
¿Algún campo necesita TTL o escala distinta?  → SÍ → claves separadas (referenciar)
¿Necesito atomicidad entre varios campos?     → SÍ → MULTI/EXEC o script Lua (en cualquiera de las dos opciones)
```

| Criterio | Embeber (Hash) | Referenciar (claves separadas) |
|---|---|---|
| Memoria | ~30% menos | Más (prefijo repetido) |
| TTL por campo | ❌ (TTL de toda la clave) | ✅ (cada clave, el suyo) |
| Lectura completa | ✅ `HGETALL` (1 llamada) | ❌ N llamadas `GET` |
| Escalado en clúster | Todo en un nodo | Se puede repartir |

**Ejemplo resuelto (típico de examen):** *"Un usuario tiene perfil (nombre, ciudad — no expira) y sesión (userId — TTL 30 min). ¿Cómo lo modelas?"*
```bash
# Perfil → Hash, sin TTL (se lee siempre entero)
HSET user:1000:profile nombre "Juan" ciudad "Madrid"

# Sesión → clave separada, CON TTL (vida útil distinta al perfil)
SET session:abc123:userId "1000" EX 1800
```
No se puede meter todo en un único Hash porque el TTL en Redis es **por clave completa**, no por campo — si el Hash tuviera un TTL de 30 min, se perdería también el nombre y la ciudad al expirar la sesión.

---

## 10. Transacciones, pipeline y WATCH — el bloque que más se confunde

| Mecanismo | Qué resuelve | Qué NO resuelve |
|---|---|---|
| `MULTI` / `EXEC` | Que los comandos del lote no se entrelacen con los de OTRO cliente | Aislamiento tipo SQL (no hay rollback parcial ni "vista consistente" entre comandos individuales) |
| `pipeline` | Reduce roundtrips de red (rendimiento) | Atomicidad — es ortogonal a MULTI/EXEC, se pueden combinar |
| `WATCH` | Aislamiento real leer-modificar-escribir (optimistic locking) | Rendimiento — puede requerir reintentos si hay conflicto |

**Ejemplo explicado — transferencia bancaria con `WATCH` (optimistic locking):**
```bash
WATCH cuenta:saldo          # empieza a vigilar la clave
GET cuenta:saldo            # → 1000 (lo leo para decidir si hay saldo suficiente)

# --- si en este instante OTRO cliente hace SET cuenta:saldo 800 ---

MULTI
DECRBY cuenta:saldo 200
EXEC
# → (nil)  la transacción se ABORTA porque la clave cambió entre WATCH y EXEC

# Hay que reintentar desde el WATCH:
WATCH cuenta:saldo
GET cuenta:saldo            # → 800 (valor actualizado)
MULTI
DECRBY cuenta:saldo 200
EXEC
# → (integer) 600  ahora sí se aplica, porque nadie más tocó la clave en medio
```
*Por qué no basta con `MULTI`/`EXEC` a secas:* `MULTI`/`EXEC` solo encola los comandos y los ejecuta de forma seguida (sin que se intercalen los de otro cliente), pero **no comprueba si el dato que leíste antes de decidir la operación sigue siendo válido**. Si dos clientes leen el mismo saldo 1000 y ambos restan 200 con `MULTI`/`EXEC`, el resultado final sería 600 en vez de 800 (se "pierde" una resta) — porque ninguno de los dos sabe que el otro también escribió. `WATCH` evita justo eso: aborta si detecta que el dato cambió entre la lectura y la escritura.

⚠️ **Pregunta típica:** *"Diferencia entre lock pesimista y optimista."* → Pesimista (`SELECT ... FOR UPDATE` en SQL) bloquea el recurso ANTES de leer, todos los demás esperan. Optimista (`WATCH`) no bloquea nada, comprueba al escribir y reintenta si hubo conflicto — mejor cuando los conflictos son poco frecuentes.

---

## 11. Persistencia (RDB vs AOF)

| | RDB | AOF | Híbrido (recomendado) |
|---|---|---|---|
| Qué guarda | Snapshot completo periódico | Log de cada escritura | RDB + AOF regenerado desde el último snapshot |
| Riesgo de pérdida | Todo lo escrito desde el último snapshot | ~1s con `appendfsync everysec` | Mínimo |
| Tamaño/velocidad | Compacto y rápido de cargar | Más grande, algo más lento | Balance |

```conf
save 900 1              # snapshot si ≥1 cambio en 900s
appendonly yes           # activa AOF
appendfsync everysec     # fsync cada segundo (compromiso rendimiento/seguridad)
aof-use-rdb-preamble yes # modo híbrido
```

**Ejemplo explicado:** si Redis cae justo después de 500 `INCR` sobre un contador de vistas y el último snapshot RDB fue hace 10 minutos, con **solo RDB** se pierden esos 500 incrementos. Con **AOF** (`everysec`), como máximo se pierde 1 segundo de escrituras — mucho menos.

---

## 12. Redis vs Memcached vs SQL (tabla de examen)

| | Redis | Memcached | SQL (MariaDB) |
|---|---|---|---|
| Estructuras | String, Hash, List, Set, ZSet... | Solo strings | Tablas relacionales |
| Persistencia | RDB/AOF opcional | No | Sí (disco) |
| Transacciones | MULTI/EXEC (sin aislamiento SQL) | No | ACID completo |
| JOINs | No | No | Sí |
| Latencia lectura | ~0,1 ms (RAM) | ~0,1 ms (RAM) | ~1-5 ms (disco) |
| Uso típico | Caché + estructuras + colas | Caché pura | Fuente de verdad |

**Regla de oro para el examen:** si la pregunta pide **estructuras con semántica** (contador atómico, cola, ranking) → Redis. Si solo pide **cachear strings simples** → Memcached también vale, pero Redis es superset. Si pide **transacciones ACID / relaciones / consultas complejas** → SQL, Redis NO es sustituto.

---

## 13. Trampas típicas de examen (repaso final de 2 minutos)

1. ⚠️ **`KEYS *` en producción** — O(N), bloquea el hilo único de Redis entero. Usa siempre `SCAN` (iterativo, no bloqueante).
2. ⚠️ **`HGETALL` en hashes grandes** — O(N), puede bloquear. Usa `HGET` si solo necesitas un campo.
3. ⚠️ **`MULTI`/`EXEC` no es aislamiento SQL** — dos transacciones pueden "perderse" escrituras si no usas `WATCH`.
4. ⚠️ **`LPUSH`+`LPOP`** = pila (LIFO). **`LPUSH`+`RPOP`** = cola (FIFO). No confundir extremos.
5. ⚠️ **Redis no soporta JOINs** porque las claves son independientes entre sí — no hay motor de relaciones.
6. ⚠️ **El TTL es por clave, no por campo** — si necesitas TTLs distintos para partes de un mismo objeto, tienes que referenciar (claves separadas), no embeber en un solo Hash.
7. ⚠️ **`ZADD` es O(log N)**, no O(1) — es la excepción a "todo es O(1) en Redis" que casi siempre entra en examen.
8. ⚠️ **Consistencia eventual con réplicas** — válido para caché/sesiones, **no** para pagos o inventario crítico.
9. ⚠️ **Fan-out on write vs on read** — escritura rápida+lectura cara (pull) vs escritura cara+lectura rápida (push). Instagram combina ambos según el nº de seguidores.
10. ⚠️ **Pipeline ≠ atomicidad** — pipeline es solo optimización de red (agrupa roundtrips); si necesitas que no se intercalen comandos de otro cliente, eso lo da `MULTI`/`EXEC`, no pipeline.

---

## 14. Mini-simulacro (autoevaluación exprés)

1. Diseña la clave y el comando para cachear el precio de un producto 456 durante 5 minutos. → `SET product:456:price "29.99" EX 300`
2. ¿Qué comando usarías para saber cuántos usuarios están conectados a la vez sin duplicados? → `SADD online:users <id>` + `SCARD online:users`
3. Un ranking de videojuego necesita el top 10 de puntuaciones. ¿Estructura y comando? → `ZSET` + `ZREVRANGE ranking 0 9 WITHSCORES`
4. Quieres transferir saldo entre dos cuentas garantizando que nadie más lo modifique mientras tanto. → `WATCH` sobre la cuenta origen, leer saldo, `MULTI` + `DECRBY`/`INCRBY`, `EXEC`; si devuelve `nil`, reintentar.
5. ¿Por qué `KEYS user:*` es peligroso en un Redis con 10 millones de claves en producción? → Es O(N) y bloquea el único hilo de Redis durante todo el recorrido, congelando el servidor para todos los clientes; usar `SCAN` en su lugar.
6. Un objeto tiene 5 campos que se leen siempre juntos, sin necesidad de TTL distinto. ¿Hash o claves separadas? → Hash (embeber): menos memoria y una sola llamada `HGETALL`.
7. ¿Qué diferencia hay entre `SINTER` y `SDIFF`? → `SINTER` = elementos en todos los conjuntos; `SDIFF` = elementos que están en el primero pero no en los demás.
8. Explica en una frase por qué Redis monohilo puede superar 100.000 comandos/s. → Porque cada operación en RAM tarda ~100ns y la E/S de red es no bloqueante (epoll/Reactor), así que mientras un cliente espera respuesta el hilo sigue atendiendo a otros.
