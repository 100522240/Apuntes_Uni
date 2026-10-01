# Soluciones PR3.4 y PR3.5

> Arquitectura de Datos · Tema 3: Redis · Curso 2026/2027
>
> Todo el código Python de este documento se ha ejecutado contra `redis:7-alpine` (Redis 7.4.11) con `redis-py`. Las salidas que aparecen son las reales.

---

# PR3.4 — Contador distribuido y PK distribuida

La **Parte A** es guiada: solo hay que ejecutar los pasos 1–6 del PDF y comprobar que la salida coincide. Las tareas que hay que resolver están en la **Parte B** (Tareas 1–5) y en las preguntas de reflexión de ambas partes.

**Caso práctico (Parte B):** una tienda online con 5 vendedores. Cada tienda tiene su propia MariaDB y los pedidos compartidos necesitan un ID único global.

## Tarea 1 — Esquema de claves

### Diseño

| Clave | Tipo | Uso |
|---|---|---|
| `pk:orders` | String (entero) | IDs globales de pedidos compartidos entre las 5 tiendas |
| `pk:users` | String (entero) | IDs globales de usuarios |
| `pk:products` | String (entero) | IDs globales de productos |
| `pk:store:{storeId}:orders` | String (entero) | IDs locales de cada tienda, si alguna entidad no necesita ser global |

Convención: `pk:<ámbito>:<entidad>`, en minúsculas y con `:` como separador.

### Justificación

- **Prefijo `pk:`**: separa los generadores de claves primarias del resto de claves de la aplicación (`session:`, `cache:`…). Así se pueden listar con `SCAN 0 MATCH pk:*` y se evitan colisiones de nombres. Es la convención `objeto:id:campo` de T3.2.4.
- **Separador `:`**: es la convención estándar de Redis. DataGrip y RedisInsight lo usan para mostrar las claves en forma de árbol.
- **Una clave por entidad, no una global**: si `users` y `orders` compartieran contador, los IDs de cada tabla tendrían huecos enormes. Además, todas las tiendas competirían por la misma clave. Con claves separadas, cada secuencia es independiente y densa.
- **String y no Hash**: `INCR` trabaja sobre Strings con codificación `int`, que es O(1) y ocupa muy poca memoria. Un Hash con `HINCRBY pk orders 1` también sería atómico, pero tiene dos inconvenientes:
  1. Todo el Hash vive en un único *hash slot*. En Redis Cluster todas las secuencias acabarían en el mismo nodo (punto caliente).
  2. No se puede poner TTL ni hacer backup por campo.

  Con una clave String por entidad, en un Cluster las secuencias se reparten entre nodos.
- **Escalabilidad**: para añadir una entidad nueva basta con usar una clave nueva, sin migraciones. El ámbito `store:{id}` permite añadir tiendas sin tocar las secuencias globales.

## Tarea 2 — Implementación del generador

```python
import re
import redis

r = redis.Redis(host="localhost", port=6379, decode_responses=True)

ENTITY_RE = re.compile(r"^[a-z][a-z0-9_]{0,31}$")


def generate_id(entity: str) -> int:
    if not ENTITY_RE.match(entity):
        raise ValueError(f"Nombre de entidad no válido: {entity!r}")
    return r.incr(f"pk:{entity}")
```

Justificación:

- **`INCR` cumple los tres requisitos funcionales**:
  - Hay un contador independiente por entidad porque cada entidad tiene su propia clave.
  - Es atómico porque Redis ejecuta los comandos en un único hilo.
  - Es secuencial (1, 2, 3…). Además, si la clave no existe la crea con valor 1, así que no hace falta inicializarla.
- **Validación del nombre**: evita que una entrada como `"orders:hack"` cree claves fuera del esquema (inyección de claves). También evita que una errata (`"Orders"`) cree un contador nuevo en silencio.

Ejecución con la interfaz del enunciado:

```python
[generate_id("orders"), generate_id("orders"), generate_id("users"), generate_id("orders")]
# [1, 2, 1, 3]
```

## Tarea 3 — Análisis de concurrencia

**Escenario:** dos usuarios pulsan «Comprar» a la vez.

Si el generador no fuera atómico (por ejemplo, `GET` + `SET`), podría ocurrir este entrelazado:

| t | Cliente A | Cliente B | Valor en Redis |
|---|---|---|---|
| 1 | `GET pk:orders` → 41 | | 41 |
| 2 | | `GET pk:orders` → 41 | 41 |
| 3 | `SET pk:orders 42` → usa ID 42 | | 42 |
| 4 | | `SET pk:orders 42` → usa ID 42 | 42 |

Resultado: **los dos pedidos reciben el ID 42** y el contador solo ha avanzado una vez (*lost update*).

**Prueba real:** 20 hilos generando 200 IDs cada uno, es decir, 4000 peticiones.

```
INCR     -> generados: 4000 únicos: 4000 valor final: 4003
GET+SET  -> generados: 4000 únicos: 416  valor final: 416
```

(El contador de `INCR` termina en 4003 porque ya iba por 3 tras la Tarea 2.) Con `GET` + `SET`, el **90 %** de los IDs estaban duplicados.

### Respuestas a las preguntas

- **¿Qué problema surge con GET + SET?** Una condición de carrera de tipo *read-modify-write*. La lectura y la escritura son dos comandos distintos y otro cliente puede ejecutar los suyos en medio.
- **¿Cómo afecta a la integridad referencial en MariaDB?**
  - Si el pedido va a una sola tabla con `PRIMARY KEY`, el segundo `INSERT` falla con `ERROR 1062 Duplicate entry`.
  - En este caso es peor: **cada tienda tiene su propia MariaDB**. Dos tiendas pueden insertar el pedido 42 en bases de datos distintas sin ningún error. Al consolidar los datos o cruzarlos, las líneas de pedido, los pagos o los envíos (FK → `order_id`) apuntarían al pedido equivocado. Esa corrupción es silenciosa.
- **¿Qué errores aparecen en la aplicación?** Excepciones de clave duplicada (HTTP 500 en el checkout), pedidos que se pierden o se sobrescriben, cobros asociados al pedido de otro cliente y fallos intermitentes que no se reproducen en local.
- **¿Cómo se manifiesta con mucho tráfico?** La probabilidad de colisión crece con la concurrencia. En pruebas funcionales con un solo usuario no aparece nunca. En un pico (Black Friday) aparece de forma masiva, que es justo cuando más daño hace. Por eso se necesita una operación atómica (`INCR`) y no un *lock* aplicativo.

## Tarea 4 — Estrategia de fallback

**Escenario:** Redis cae durante 5 segundos y 3 usuarios hacen pedidos en ese intervalo.

### Comparativa de estrategias

| Estrategia | ¿Unicidad global? | Latencia | Complejidad | Reconciliación al volver Redis |
|---|---|---|---|---|
| **Rangos preasignados (hi/lo)**: cada instancia reserva un bloque con `INCRBY pk:orders 100` y lo consume en memoria | **Sí**. Los bloques son disjuntos porque los reparte `INCRBY`, que es atómico | Mínima: 1 llamada a Redis cada 100 IDs; el resto se sirve desde memoria | Baja (un objeto con dos enteros y un lock) | No hace falta: los IDs ya eran únicos. Solo quedan huecos (los IDs no usados de un bloque si la instancia se reinicia) |
| **UUID temporal** | Sí, en la práctica (UUIDv4: 122 bits aleatorios) | Nula (se genera localmente) | Media: la columna `id` tiene que admitir dos formatos (BIGINT y UUID), o hay que usar una columna `public_id` aparte | Hay que asignar un ID numérico después y actualizar todas las FK que apuntan al UUID. Es costoso |
| **Cola de reintentos**: el pedido se encola y se procesa cuando vuelva Redis | Sí (el ID lo acaba dando `INCR`) | Alta: el usuario no tiene número de pedido hasta que Redis vuelve (≥5 s) | Alta: broker (RabbitMQ/Kafka), consumidores, idempotencia, aviso al usuario | Automática: el consumidor vacía la cola en orden |
| **Reconciliación posterior**: cada tienda usa su `AUTO_INCREMENT` y se corrige después | **No**. Las 5 MariaDB generan 1, 2, 3… y colisionan entre sí | Nula | Muy alta: detectar duplicados, renumerar y propagar a las FK | Manual o con un *job*. Hay que hacer `SET pk:orders <max>` para que Redis no reutilice esos IDs |

### Estrategia recomendada

Usar **rangos preasignados como funcionamiento normal**, no solo como plan B. Cada instancia de cada tienda tiene siempre un bloque reservado en memoria. Si Redis cae 5 segundos, los 3 pedidos se sirven del bloque sin ni siquiera notar la caída. Solo habría problema si una instancia agotara su bloque durante la caída. Para ese caso se añade un fallback secundario a UUID con reintento.

```python
import threading


class BlockAllocator:
    """Reserva bloques de IDs con INCRBY (patrón hi/lo)."""

    def __init__(self, entity: str, size: int = 100):
        self.key, self.size = f"pk:{entity}", size
        self.next, self.end = 1, 0  # fuerza la reserva en la primera llamada
        self.lock = threading.Lock()

    def next_id(self) -> int:
        with self.lock:
            if self.next > self.end:
                self.end = r.incrby(self.key, self.size)
                self.next = self.end - self.size + 1
            self.next += 1
            return self.next - 1
```

Prueba con 3 instancias que piden 150 IDs cada una (bloques de 100):

```
Bloques  -> generados: 450 únicos: 450 contador: 600
```

Se usaron 6 bloques y no hubo ningún duplicado.

> **Ojo con el fallback del Paso 6 del PDF.** Usa `SELECT IFNULL(MAX(id),0)+1 FROM users`, que tiene dos problemas:
> 1. `MAX+1` es otra vez un *read-modify-write*: tiene la misma condición de carrera que `GET` + `SET`.
> 2. En este escenario hay **5 MariaDB independientes**, así que el `MAX(id)` de una tienda no conoce los IDs de las otras y los IDs colisionarían entre tiendas.
>
> Si se quiere usar MariaDB como fallback, hay que darle a cada tienda un espacio disjunto. Por ejemplo, con `auto_increment_increment=5` y `auto_increment_offset=<n.º tienda>`, la tienda 1 genera 1, 6, 11… y la tienda 2 genera 2, 7, 12…; además, esos IDs deben quedar fuera del rango que reparte Redis.

## Tarea 5 — Recuperación tras pérdida total

**Escenario:** el volumen de Docker se corrompe y Redis arranca vacío.

El peligro es que, si el contador arranca en 0 (o en un valor antiguo), `INCR` devolverá IDs que **ya se asignaron**. Esos IDs se duplicarían en las MariaDB de las tiendas.

### Qué guardar en MariaDB (información mínima)

```sql
CREATE TABLE id_sequences (
  entity      VARCHAR(32) PRIMARY KEY,   -- 'orders', 'users'...
  high_water  BIGINT      NOT NULL,      -- fin del último bloque reservado
  updated_at  TIMESTAMP   NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

Basta con guardar la **marca de agua alta**: el mayor ID que Redis puede haber entregado. Con el patrón hi/lo coincide con el final del último bloque reservado.

### Con qué frecuencia sincronizar

**Cada vez que se reserva un bloque.** Así la marca guardada en MariaDB nunca va por detrás de los IDs entregados, y el coste es 1 escritura cada 100 IDs.

Sincronizar «cada X minutos» no es seguro: todo lo que se asignó desde la última sincronización quedaría fuera de la marca.

### Cómo reconstruir sin perder ni repetir IDs

1. Calcular `base = max(high_water, MAX(id) de cada una de las 5 MariaDB)`.
2. Sumar un **margen de seguridad** (por ejemplo, +10 000) para saltarse los IDs que se entregaron pero cuyo pedido aún no se había escrito.
3. Escribir el contador con un script Lua que **solo lo sube, nunca lo baja**. Así, si varias instancias reconstruyen a la vez (lo que advierte el PDF), ninguna puede sobrescribir el contador con un valor más antiguo:

```python
SYNC_MAX = r.register_script("""
local actual = tonumber(redis.call('GET', KEYS[1]) or '0')
local nuevo = tonumber(ARGV[1])
if nuevo > actual then
  redis.call('SET', KEYS[1], nuevo)
  return nuevo
end
return actual
""")
```

```
SYNC 42   -> 50     # el contador valía 50: no retrocede
SYNC 1042 -> 1042   # sí avanza
```

### Si el contador reconstruido es menor que el último ID asignado

Es exactamente el fallo que hay que evitar, y se previene con lo anterior: se toma el máximo de todas las fuentes, se suma un margen y nunca se decrementa. Si aun así se detecta, por ejemplo con un `INSERT` que falla por clave duplicada, la aplicación debe:

- Reintentar con un nuevo `INCR`, nunca reutilizar el ID.
- Disparar una alerta y volver a ejecutar `SYNC_MAX` con un margen mayor.

Los huecos en la numeración son aceptables; los duplicados no.

### Persistencia en Redis (T3.4.1)

Hay que activar **AOF con `appendfsync everysec`**, de modo que se pierde como mucho 1 s. Con RDB se podrían perder minutos y, al reiniciar, el contador **retrocedería**, lo que provocaría duplicados.

Aun así, ningún mecanismo de persistencia cubre un volumen corrupto. Por eso la fuente de verdad para reconstruir es `id_sequences` en MariaDB.

---

## Preguntas de reflexión (Partes A y B)

**1. ¿Qué propiedad de INCR garantiza la unicidad? / ¿Por qué INCR y no GET + SET + 1?**
La atomicidad. Redis procesa los comandos de forma secuencial en un solo hilo, así que leer, sumar y escribir ocurre como una única operación indivisible. Ningún otro cliente puede intercalarse ni leer un valor intermedio. Además, se resuelve en el servidor en un solo viaje de red, sin locks distribuidos.

**2. ¿Qué pasaría con GET + SET?**
Dos clientes leen el mismo valor *n*, ambos escriben *n+1* y ambos usan el ID *n+1*. Se pierde un incremento y aparece un ID duplicado. En la prueba de la Tarea 3, 4000 peticiones produjeron solo 416 IDs distintos.

**3. ¿Qué pasa si Redis cae después del INCR y antes de usar el ID? (Parte A)**
Ese ID se pierde y queda un **hueco** en la secuencia. No se viola la integridad: un hueco no es un duplicado, y las PK no necesitan ser consecutivas. Hay un caso peligroso: Redis cae, se pierde ese último incremento porque no se llegó a persistir, y la aplicación sí usó el ID. Al reiniciar, Redis volvería a entregar ese mismo ID. Por eso se aplica el margen de seguridad de la Tarea 5.

**4. ¿Cómo manejarías varios vendedores con contadores separados?**
Con claves con ámbito: `pk:store:{storeId}:orders`. Cada tienda tiene su propio espacio de IDs y el aislamiento lo da el propio nombre de la clave.

Si además hace falta que el ID sea único globalmente, hay dos opciones:
- Usar el contador global `pk:orders`.
- Componer el ID como `(storeId, localId)`, o como un único entero `localId * 10 + storeId` (con hasta 10 tiendas).

En Redis Cluster, escribir `pk:store:{3}:orders` con *hash tag* fuerza a que todas las claves de la tienda 3 vivan en el mismo slot.

**5. ¿Cuándo INCR y cuándo SETNX?**
- `INCR`: cuando hace falta un **valor nuevo y distinto** para cada llamada (secuencias, contadores, rate limiting).
- `SET key val NX` (el antiguo `SETNX`): cuando el objetivo es **crear solo si no existe**. Ejemplos:
  - Locks distribuidos: `SET lock:x 1 NX EX 10`.
  - Idempotencia: marcar una petición como procesada.
  - Inicializar un contador sin pisar uno existente: `SET pk:orders 42 NX`.

`SETNX` no genera valores; solo decide quién llega primero.

**6. Alternativas: UUID, Snowflake, ULID**

| | Tamaño | ¿Ordenable? | ¿Necesita coordinación? | Inconvenientes |
|---|---|---|---|---|
| Redis `INCR` | 64 bits | Sí, secuencial perfecto | Sí, depende de Redis | Punto único de fallo y latencia de red; expone el volumen de negocio (el pedido 1042 indica cuántos se han hecho) |
| UUIDv4 | 128 bits | No | No | Índices B-tree fragmentados porque las inserciones caen en posiciones aleatorias; ocupa 16 B; poco legible |
| UUIDv7 (RFC 9562) | 128 bits | Sí, por tiempo | No | Mismo tamaño que UUIDv4 |
| Snowflake (Twitter) | 64 bits: 41 de timestamp en ms + 10 de máquina + 12 de secuencia | Sí, aproximadamente | Solo para asignar el ID de máquina | Depende del reloj (si el reloj retrocede, puede duplicar); límite de 4096 IDs/ms por máquina |
| ULID | 128 bits: 48 de timestamp + 80 aleatorios; 26 caracteres en base32 | Sí, lexicográficamente | No | 128 bits; el orden dentro del mismo milisegundo no está garantizado entre máquinas |

Un UUID tiene sentido cuando el ID se genera **sin conexión** (en el cliente, offline o entre sistemas que no comparten nada) o cuando no se quiere que el ID sea predecible ni que revele el volumen. Un ID secuencial tiene sentido cuando importa la compacidad, la localidad del índice y la legibilidad (número de pedido visible para el cliente).

**7. ¿Cómo escalarías a varios data centers?**
No conviene tener un único Redis compartido entre continentes: añade latencia y un punto de fallo. Hay varias alternativas:
- **Intercalado**: cada data center tiene su propio Redis y genera `id = INCR * N + dcId`. Con 3 data centers, DC0 genera 0, 3, 6…; DC1 genera 1, 4, 7… Los conjuntos nunca se solapan.
- **Rangos por data center**: DC0 usa [0, 10¹²), DC1 usa [10¹², 2·10¹²)…
- **Snowflake** con bits de data center (5 bits de DC + 5 de máquina).

En todos los casos la unicidad se consigue por construcción y no hace falta coordinación entre data centers.

**8. ¿Qué pasa si el contador se queda sin espacio?**
`INCR` trabaja con enteros con signo de 64 bits, cuyo máximo es 2⁶³−1 ≈ 9,2·10¹⁸. Al superarlo, Redis devuelve un error:

```
Overflow -> increment or decrement would overflow
```

(comprobado con `SET pk:max 9223372036854775807` + `INCR`).

A 1 millón de IDs por segundo se tardaría unos 292 000 años en llegar, así que en la práctica no es un problema. Lo importante es que la columna en MariaDB sea `BIGINT` (no `INT`, que se agota en unos 2 100 millones) y monitorizar el valor con una alerta, por ejemplo al 80 % del rango de la columna.

**9. ¿Cómo sincronizar los contadores entre el primario y las réplicas?**
La replicación de Redis es **asíncrona**. El primario confirma el `INCR` al cliente antes de enviarlo a la réplica. Si el primario cae justo después, la réplica promocionada puede tener el contador **por detrás** y volvería a entregar IDs ya usados.

Mitigaciones:
- `WAIT 1 100` después del `INCR`: espera a que al menos 1 réplica confirme. Reduce la ventana, pero no da consistencia fuerte.
- `min-replicas-to-write 1`: el primario rechaza escrituras si no tiene réplicas sincronizadas.
- Después de cada *failover*, hacer `INCRBY pk:orders <margen>` (o `SYNC_MAX` con la marca de MariaDB) antes de volver a servir IDs.
- Con hi/lo el riesgo cae mucho: solo hay una escritura cada 100 IDs y la marca queda persistida en MariaDB.

**10. ¿Cómo recuperar el contador sin persistencia? (Parte A, pregunta 3)**
Es la Tarea 5: reconstruir a partir de `max(high_water, MAX(id) de cada tienda) + margen` usando `SYNC_MAX`.

---

# PR3.5 — Sesiones de usuario

**Escenario:** plataforma SaaS con 10 000 usuarios concurrentes. Se pide login/logout, expiración automática, invalidación por seguridad y escalabilidad horizontal.

## Tarea 1 — Esquema de sesiones

### Claves

| Clave | Tipo | Contenido | TTL |
|---|---|---|---|
| `session:{sid}` | **Hash** | `user_id`, `created_at`, `last_activity`, `device` | 1800 s + *jitter* (sliding) |
| `user:{userId}:sessions` | **Set** | Los `sid` activos del usuario | 8 h (tope absoluto), se renueva en cada login |

Ejemplo real:

```
HGETALL session:Xq3...   →  user_id "42", created_at "1790879756",
                             last_activity "1790879756", device "PC"
TTL session:Xq3...       →  1817
```

### Justificación de cada decisión

- **Prefijo `session:`**: sigue la convención `objeto:id` de T3.2.4. Permite:
  - Identificar el tipo de dato.
  - Hacer `SCAN MATCH session:*`.
  - Aplicar políticas propias (por ejemplo, un Redis o una base lógica dedicados a sesiones).
- **Separador `:`**: es el estándar de Redis y lo entienden las herramientas (vista en árbol en DataGrip/RedisInsight).
- **Session ID aleatorio (no el userId)**:
  - La clave no revela quién es el usuario.
  - Es imposible de adivinar.
  - Un mismo usuario puede tener varias sesiones (Tarea 4).
- **Hash en lugar de JSON serializado**:

| | Hash (elegido) | String con JSON |
|---|---|---|
| Actualizar `last_activity` | `HSET` de un campo, atómico | Leer, deserializar, modificar y reescribir: tiene condición de carrera si llegan dos peticiones a la vez |
| Leer un solo campo | `HGET session:x user_id` | Hay que traer y parsear el objeto entero |
| Memoria | Codificación *listpack* compacta para Hash pequeños | Similar |
| Datos anidados | No (estructura plana) | Sí |
| TTL | Uno para todo el Hash | Uno para todo el String |

  Una sesión es un registro plano con un campo que cambia en cada petición, así que el Hash encaja mejor. JSON solo compensa si la sesión guarda estructuras anidadas (un carrito, preferencias).
- **No usar varias claves sueltas** (`session:x:user_id`, `session:x:last_activity`…): habría que poner y renovar el TTL de cada una por separado y podrían caducar en momentos distintos, dejando sesiones a medias.
- **Set `user:{userId}:sessions`**: es el índice inverso usuario → sesiones. Hace falta para detectar sesiones duplicadas (Tarea 4) y para invalidar todas las sesiones de un usuario (Tarea 6).

## Tarea 2 — Login / logout

```python
import random
import secrets
import time

SESSION_TTL = 1800       # 30 min de inactividad (sliding)
JITTER = 120             # hasta 2 min aleatorios (anti thundering herd, Tarea 5)
ABSOLUTE_MAX = 8 * 3600  # tope absoluto de vida de la sesión


def ttl_with_jitter() -> int:
    return SESSION_TTL + random.randint(0, JITTER)


def login(user_id: str, password: str, device: str = "") -> str | None:
    if not check_password(user_id, password):  # scrypt + compare_digest contra MariaDB
        return None
    sid = secrets.token_urlsafe(32)  # 256 bits de entropía (más que UUIDv4: 122)
    now = int(time.time())
    ttl = ttl_with_jitter()
    pipe = r.pipeline(transaction=True)  # MULTI/EXEC
    pipe.hset(f"session:{sid}", mapping={
        "user_id": user_id, "created_at": now, "last_activity": now, "device": device,
    })
    pipe.expire(f"session:{sid}", ttl)
    pipe.sadd(f"user:{user_id}:sessions", sid)
    pipe.expire(f"user:{user_id}:sessions", ABSOLUTE_MAX)
    pipe.execute()
    return sid


def logout(sid: str) -> bool:
    user_id = r.hget(f"session:{sid}", "user_id")
    pipe = r.pipeline(transaction=True)
    pipe.delete(f"session:{sid}")
    if user_id:
        pipe.srem(f"user:{user_id}:sessions", sid)
    return pipe.execute()[0] == 1
```

(La función `check_password` completa está en el script de pruebas. Usa `hashlib.scrypt` con sal y `hmac.compare_digest` para comparar en tiempo constante.)

### Justificación

- **Identificador de sesión**: `secrets.token_urlsafe(32)` usa el generador criptográfico del sistema operativo. UUIDv4 también sería válido (`uuid.uuid4()` usa `os.urandom`), pero tiene menos entropía. **Nunca** se debe usar `random`, que es predecible. Se genera un sid **nuevo en cada login** para evitar ataques de *session fixation*.
- **¿SETEX o SET + EXPIRE?** `SETEX` (o `SET k v EX 1800`) es **atómico**. Con `SET` seguido de `EXPIRE` son dos comandos: si la aplicación cae entre ambos, queda una sesión **inmortal** sin TTL, que es tanto un riesgo de seguridad como una fuga de memoria.

  Como aquí la sesión es un Hash y `HSET` no admite TTL, se consigue la misma atomicidad envolviendo `HSET` + `EXPIRE` + `SADD` + `EXPIRE` en **`MULTI/EXEC`** (`pipeline(transaction=True)`). Se ejecutan todos los comandos o ninguno. Si la sesión fuera un String JSON, bastaría con `SET session:{sid} <json> EX 1800`.
- **TTL del Set de sesiones = tope absoluto, renovado en cada login.** Cada sesión vive como mucho `ABSOLUTE_MAX` desde que se creó, y el Set caduca `ABSOLUTE_MAX` después del último login. Por tanto, el Set siempre dura más que todas sus sesiones, y si el usuario deja de entrar, Redis lo borra solo.
- **Cookie**: `Set-Cookie: session_id=<sid>; HttpOnly; Secure; SameSite=Lax`. Así JavaScript no puede leerla (protege frente a XSS) y solo viaja por HTTPS.

Ejecución:

```
login malo   -> None
sesión PC    -> {'user_id': '42', 'created_at': '1790879756', 'last_activity': '1790879756', 'device': 'PC'} TTL 1817
logout       -> True | validate: None
```

## Tarea 3 — Expiración (sliding vs fixed)

**Escenario:** la sesión tiene un TTL de 30 minutos y el usuario hace una petición en el minuto 29.

**¿Qué debe pasar?** Con *sliding expiration*, el TTL vuelve a 30 minutos y la sesión dura hasta el minuto 59. El usuario sigue activo, así que no tiene sentido expulsarlo.

### Implementación

En cada petición se valida y se renueva **en un único paso atómico** con un script Lua:

```python
TOUCH = r.register_script("""
local uid = redis.call('HGET', KEYS[1], 'user_id')
if not uid then return nil end
local created = tonumber(redis.call('HGET', KEYS[1], 'created_at'))
local now = tonumber(ARGV[1])
if now - created > tonumber(ARGV[3]) then
  redis.call('DEL', KEYS[1])
  return nil
end
redis.call('HSET', KEYS[1], 'last_activity', now)
redis.call('EXPIRE', KEYS[1], ARGV[2])
return uid
""")


def validate(sid: str) -> str | None:
    return TOUCH(keys=[f"session:{sid}"],
                 args=[int(time.time()), ttl_with_jitter(), ABSOLUTE_MAX])
```

**¿Por qué Lua y no `HGET` + `HSET` + `EXPIRE` desde la aplicación?** La sesión podría caducar justo entre la comprobación y el `HSET`. En ese caso, el `HSET` **volvería a crear** un Hash «zombi» que solo tendría `last_activity`, y esa petición se daría por válida. Con Lua, la comprobación y la renovación son atómicas.

Ejecución (se fuerza un TTL de 60 s para simular el minuto 29):

```
TTL antes -> 60 | validate: 42 | TTL después: 1840
```

### Comparativa

| | Sliding expiration | Fixed (absolute) expiration |
|---|---|---|
| Funcionamiento | Cada petición renueva el TTL (`EXPIRE`) | El TTL se fija en el login y no se toca |
| Experiencia de usuario | Buena: un usuario activo nunca es expulsado | Mala: puede ser expulsado en mitad de una tarea |
| Seguridad | Peor: una cookie robada se mantiene viva indefinidamente si el atacante hace peticiones | Mejor: la ventana de uso de una cookie robada está acotada |
| Coste | 1 escritura (`EXPIRE`) por petición | Ninguno extra |
| Uso típico | Redes sociales, SaaS, correo | Banca, administración pública, paneles de administración |

**Solución elegida: híbrida.** Sliding de 30 minutos por inactividad más un **tope absoluto de 8 horas** (el `ABSOLUTE_MAX` del script). Así se obtiene la comodidad del sliding y la ventana acotada del fixed. Es la recomendación de OWASP (*idle timeout* + *absolute timeout*). Comprobado:

```
tope abs. -> None | existe: 0
```

## Tarea 4 — Sesiones duplicadas

**Escenario:** el usuario inicia sesión desde el ordenador y, 5 minutos después, desde el móvil.

- **¿Permitir ambas?** Depende de la política de la aplicación. En un SaaS sí: es lo esperado, porque el usuario trabaja desde varios dispositivos. Cada login crea su propio `sid`.
- **¿Cómo detectarlas?** Con el Set `user:{userId}:sessions`. `SCARD` da el número de sesiones y `SMEMBERS` las lista. `SADD` y `SISMEMBER` son O(1).
- **Limpieza de sesiones caducadas dentro del Set.** Cuando `session:{sid}` caduca por TTL, Redis **no** borra el `sid` del Set. Hay que limpiarlo de forma perezosa al consultarlo:

```python
def active_sessions(user_id: str) -> list[str]:
    """Lista sesiones vivas y limpia las caducadas del Set (limpieza perezosa)."""
    key = f"user:{user_id}:sessions"
    sids = list(r.smembers(key))
    pipe = r.pipeline()
    for sid in sids:
        pipe.exists(f"session:{sid}")
    alive = pipe.execute()
    dead = [s for s, a in zip(sids, alive) if not a]
    if dead:
        r.srem(key, *dead)
    return [s for s, a in zip(sids, alive) if a]
```

```
Set (bruto) -> 3 | activas tras limpieza: 2 | Set: 2
```

  Además, el TTL del propio Set garantiza que no crece indefinidamente.

  *Alternativa:* un **Sorted Set** con `score = instante de expiración`. Permite limpiar con `ZREMRANGEBYSCORE key -inf <ahora>` y quitar la sesión más antigua con `ZPOPMIN` (útil para limitar a N dispositivos). A cambio, hay que actualizar el score en cada petición.
- **¿Qué pasa si cambia la contraseña desde el móvil?** **Sí**, se deben cerrar las sesiones del ordenador y del resto de dispositivos, **excepto la del móvil**, que es desde donde se hizo el cambio. El motivo es que un cambio de contraseña suele indicar que la anterior pudo verse comprometida. Se implementa en la Tarea 6.
- **Política de sesiones múltiples:**

| | Aplicación bancaria | Red social |
|---|---|---|
| Sesiones simultáneas | **1**: un login nuevo cierra la anterior (con `invalidate_all(user, keep=nuevo_sid)`) | Ilimitadas o un máximo alto (~10) |
| TTL | 5–10 min de inactividad, tope absoluto corto | Días o semanas («recordarme») |
| Al detectar un dispositivo nuevo | Avisar al usuario y pedir 2FA | Notificación informativa |
| Gestión por el usuario | — | Pantalla «Dispositivos conectados» con la opción «cerrar sesión en los demás» |

## Tarea 5 — Thundering herd

**Escenario:** 10 000 sesiones expiran exactamente a las 12:00:00 y todos los usuarios intentan volver a iniciar sesión a la vez.

**¿Cómo se llama y por qué es perjudicial?** Es el **thundering herd** (manada estruendosa): muchos clientes reaccionan a la vez a un mismo evento. Es especialmente grave con el login porque:

- El hashing de contraseñas (bcrypt/scrypt/argon2) es **lento a propósito**, para dificultar los ataques de fuerza bruta. 10 000 hashes simultáneos saturan la CPU de los servidores de aplicación.
- Cada login consulta MariaDB, lo que provoca un pico de conexiones y puede agotar el *pool*.
- Aumentan la latencia y los *timeouts*. Los clientes reintentan y la carga se multiplica, lo que puede desembocar en una caída en cascada.

Para Redis, el pico de `HSET` es trivial. El cuello de botella está en la aplicación y en la base de datos.

### Estrategias de mitigación

1. **Jitter en el TTL** (ya incluido en `ttl_with_jitter()`): el TTL es `1800 + random(0, 120)`. Las expiraciones se reparten en una ventana de 2 minutos en lugar de caer en un único segundo:

   ```
   jitter -> min 1800 max 1920 | máx. expiraciones en un mismo segundo: 104
   ```

   Con 10 000 sesiones, el peor segundo pasa de 10 000 expiraciones a unas 100. Es una reducción de unas 100 veces solo por añadir aleatoriedad.
2. **Sliding expiration**: los usuarios activos renuevan su TTL en momentos distintos, así que sus expiraciones se desincronizan solas. El problema solo aparece cuando muchas sesiones se crean a la vez y no se usan (por ejemplo, tras una migración o un login masivo a las 8:00).
3. **Renovación anticipada (*refresh-ahead*)**: si al validar quedan menos de 5 minutos de TTL, se renueva o se emite un *refresh token*. El usuario nunca llega a ver la expiración.
4. **Rate limiting del login**: `INCR ratelimit:login:{minuto}` + `EXPIRE` (PR3). Si se supera el umbral, se responde 429 con `Retry-After`.
5. **Backoff exponencial con jitter en el cliente**: los reintentos se espacian 1 s, 2 s, 4 s… cada uno con un componente aleatorio.

## Tarea 6 — Invalidación por seguridad

**Escenario:** el usuario cambia la contraseña desde otro dispositivo y hay que cerrar todas sus sesiones.

- **Relación usuario → sesiones**: el Set `user:{userId}:sessions` de la Tarea 1.
- **Comandos y atomicidad**: un script Lua que recorre el Set y borra cada sesión, **salvo la actual**:

```python
INVALIDATE_ALL = r.register_script("""
local sids = redis.call('SMEMBERS', KEYS[1])
local borradas = 0
for _, sid in ipairs(sids) do
  if sid ~= ARGV[1] then
    borradas = borradas + redis.call('DEL', 'session:' .. sid)
    redis.call('SREM', KEYS[1], sid)
  end
end
return borradas
""")


def invalidate_all(user_id: str, keep_sid: str = "") -> int:
    return INVALIDATE_ALL(keys=[f"user:{user_id}:sessions"], args=[keep_sid])
```

Ejecución: PC, móvil y tablet tenían sesión (la tablet ya había caducado). Se cambia la contraseña desde el móvil:

```
invalidadas -> 1
PC válida?  -> None | móvil válida? -> 42
```

### Respuestas a las preguntas

- **¿Por qué Lua?** El script se ejecuta de forma atómica: ningún comando se intercala, así que no puede colarse una sesión nueva a medias ni validarse una sesión que ya debería estar muerta. Además, se hace en **1 viaje de red** en lugar de 2 + N.
- **¿Cómo se excluye la sesión que cambió la contraseña?** Se pasa su `sid` como `ARGV[1]` y el script la salta. Otra opción más estricta: invalidarlas todas y emitir un `sid` nuevo para el dispositivo actual, lo que además rota el identificador tras un cambio de credenciales.
- **¿Es eficiente con 50 sesiones?** Sí. Son 50 `DEL` de O(1) dentro de una sola llamada al servidor, del orden de microsegundos. Hacerlo desde la aplicación con 50 comandos sueltos costaría 50 viajes de red y no sería atómico. Para no bloquear Redis con usuarios que tengan miles de sesiones, se puede usar `UNLINK` (borrado en segundo plano) en lugar de `DEL`.
- **Limitación en Redis Cluster**: el script construye claves (`'session:'..sid`) que **no declara en `KEYS`**, y además esas claves están repartidas por distintos slots. En un Cluster, eso daría error `CROSSSLOT`. Una solución que funciona en Cluster sin operaciones multiclave es el **versionado de sesiones** (*auth epoch*):

  ```
  user:42:auth_epoch          → INCR al cambiar la contraseña
  session:{sid}  campo epoch  → copia del epoch en el momento del login
  ```

  Al validar, si el `epoch` de la sesión es distinto de `GET user:42:auth_epoch`, la sesión deja de ser válida. Invalidar todas las sesiones es **un único `INCR`, O(1)**, sin importar cuántas sesiones haya ni en qué nodo estén. Las claves antiguas desaparecen solas por TTL. La sesión desde la que se hizo el cambio se actualiza al epoch nuevo.

## Tarea 7 — Escalabilidad

**Escenario:** 100 000 usuarios concurrentes y un único nodo Redis que no da abasto.

Antes de proponer la arquitectura, conviene dimensionar. Una sesión de este esquema ocupa unos 200–300 B (Hash pequeño en *listpack* más la clave), así que 100 000 sesiones son unos **30 MB**, que caben en un solo nodo. Si la memoria se agota, lo primero es revisar si las sesiones guardan datos que no deberían (carritos o perfiles enteros). Aun así, a esa escala el límite suele ser de **operaciones por segundo** y de **alta disponibilidad**, y ambos justifican la arquitectura distribuida.

### Arquitectura propuesta

**Redis Cluster** con 3 primarios y 1 réplica por primario (6 nodos como mínimo), repartidos en zonas de disponibilidad distintas.

- **Particionado (*sharding*)**: el que hace Redis Cluster mediante **hash slots**. Hay 16 384 slots, cada clave va al slot `CRC16(clave) mod 16384` y cada primario tiene asignado un rango de slots.

  Se descarta el sharding manual en el cliente (por ejemplo, `hash(sid) % N`), porque añadir un nodo obliga a mover casi todas las claves. Con Cluster, en cambio, se pueden mover slots de un nodo a otro sin downtime (`redis-cli --cluster reshard`) y los clientes se redirigen solos con `MOVED`/`ASK`.
- **Clave de reparto**: el `sid` es aleatorio, así que el reparto entre nodos es **uniforme** y no hay puntos calientes.
- **Operaciones multiclave y *cross-slot***:
  - Un `MULTI`, un `DEL` con varias claves o un script Lua solo funcionan si **todas las claves están en el mismo slot**; si no, Redis devuelve `CROSSSLOT`.
  - El `login` de la Tarea 2 (con `session:{sid}` y `user:{uid}:sessions` en la misma transacción) y el `INVALIDATE_ALL` de la Tarea 6 dejarían de funcionar tal cual.
  - Hay dos soluciones posibles:
    1. **Hash tags**: claves `session:{42}:<token>` y `user:{42}:sessions`. Solo se usa lo que va entre llaves para calcular el slot, así que todas las claves del usuario 42 caen en el mismo nodo y el Lua sigue siendo válido. El inconveniente es que la cookie tiene que llevar el `userId` para poder construir la clave.
    2. **Versionado (*auth epoch*)** de la Tarea 6: las sesiones se reparten por `sid` y la invalidación es un único `INCR`. Ninguna operación necesita varias claves en la misma transacción; el login hace un `SADD` aparte, que es idempotente.

  **Elijo la opción 2.** Mantiene el reparto uniforme y la invalidación O(1), y no expone el `userId`.
- **Alta disponibilidad**: si un primario cae, sus réplicas detectan el fallo y una se promociona sola (*failover* automático en unos segundos, configurable con `cluster-node-timeout`). La replicación es asíncrona, así que se pueden perder las últimas escrituras. Para sesiones es aceptable: en el peor caso, unos pocos usuarios tienen que volver a hacer login.
- **Política de memoria**: `maxmemory-policy volatile-ttl` (expulsa primero las claves a las que menos les queda de vida) o `noeviction` con alerta. Nunca `allkeys-lru` en un Redis compartido, porque podría expulsar sesiones activas y desconectar a usuarios que están trabajando.

### Métricas a monitorizar

| Métrica (fuente) | Qué indica | Umbral de alerta orientativo |
|---|---|---|
| `used_memory` / `maxmemory` (`INFO memory`) | Saturación de RAM | > 80 % |
| `evicted_keys` (`INFO stats`) | Se están expulsando sesiones, es decir, usuarios desconectados sin motivo | > 0 |
| `mem_fragmentation_ratio` | Memoria desperdiciada | > 1,5 |
| Latencia p99 (`redis-cli --latency`, `LATENCY DOCTOR`) | Saturación de CPU o comandos lentos | > 5 ms |
| `instantaneous_ops_per_sec` | Carga por nodo y desequilibrio entre nodos | > 70 % de la capacidad medida |
| `keys` / `expires` (`INFO keyspace`) | Número de sesiones; deberían coincidir porque todas tienen TTL | `keys` ≠ `expires`: hay sesiones sin TTL |
| `expired_keys` por segundo | Picos de expiración (thundering herd) | Picos muy por encima de la media |
| `connected_clients`, `rejected_connections` | Agotamiento de conexiones | `rejected_connections` > 0 |
| `master_link_status`, `cluster_state` | Replicación o clúster degradados | ≠ `up` / ≠ `ok` |

---

## Preguntas de reflexión

**1. ¿Por qué usar TTL y no dejar las sesiones indefinidas?**
Por tres motivos:
- **Seguridad**: si alguien olvida cerrar sesión en un ordenador público, el siguiente usuario entra en su cuenta. Con TTL, a los 30 minutos de inactividad ya no puede.
- **Memoria**: cada sesión abandonada ocupa RAM para siempre, y Redis vive en memoria.
- **Higiene**: no hace falta ningún proceso de limpieza.

**2. ¿Qué diferencia hay entre expirar por TTL e invalidar por evento?**
El TTL es **pasivo y diferido**: elimina la sesión por inactividad. La invalidación por evento es **activa e inmediata**: cambio de contraseña, logout remoto, detección de intrusión o baja de la cuenta. Se necesitan las dos. El TTL cubre los olvidos y los abandonos, y la invalidación cubre las emergencias. Un TTL de 30 minutos no sirve de nada si un atacante tiene la cookie y el usuario acaba de cambiar la contraseña.

**3. ¿Cómo afecta la pérdida de Redis a los usuarios con sesión activa?**
Sin persistencia, todos tienen que volver a hacer login. Es molesto, pero no se pierden datos de negocio. Por eso, para sesiones basta con **RDB** (instantáneas cada pocos minutos) o **AOF con `everysec`** (se pierde como mucho 1 s). Un nivel de pérdida aceptable son las sesiones creadas en los últimos segundos o minutos.

Hay un detalle importante: al restaurar desde RDB podrían **resucitar sesiones invalidadas** después de la instantánea. Por eso, la invalidación por versión (*auth epoch*) debería persistirse también en MariaDB.

**4. ¿Qué estrategia de caché usarías para sesiones?**
En realidad, ninguna de las tres en sentido estricto. Las sesiones son datos efímeros y **Redis es su almacén principal**, no una caché delante de MariaDB. Cache-aside no tiene sentido porque no hay ninguna base de datos de la que «recargar» una sesión. Si se necesita auditoría (historial de inicios de sesión), lo adecuado es **write-behind**: la sesión se escribe en Redis y se encola un registro hacia MariaDB de forma asíncrona, sin penalizar la latencia del login. *Write-through* solo se justificaría si la auditoría tuviera que ser síncrona (por normativa, como en banca).

**5. ¿Cómo equilibrar seguridad y usabilidad?**
Con el esquema híbrido de inactividad + tope absoluto, más reautenticación para las operaciones sensibles (pedir de nuevo la contraseña o 2FA para transferir, cambiar el email…).

| Tipo de aplicación | Inactividad (sliding) | Tope absoluto |
|---|---|---|
| Banca | 5–10 min | 30 min – 1 h |
| Correo electrónico | 30 min – 2 h | 1–2 semanas con «recordarme» |
| Red social | Días | 30–90 días con *refresh token* |
| SaaS (este caso) | 30 min | 8 h (una jornada) |

**6. ¿Qué pasaría si usaras SET en lugar de SETEX?**
Las sesiones no caducarían nunca. La memoria crecería sin límite (sesiones abandonadas, bots) hasta llegar a `maxmemory`; a partir de ahí, Redis empezaría a expulsar claves o a rechazar escrituras, según la política configurada. Haría falta un *job* periódico que recorriera las claves con `SCAN`, comparara `last_activity` con la hora actual y borrara las viejas. Ese job añade carga, es complejo y deja ventanas en las que sesiones caducadas siguen siendo válidas. Además, las sesiones olvidadas en ordenadores públicos quedarían abiertas para siempre.

**7. ¿Cómo monitorizarías el número de sesiones activas?**
Un contador con `INCR` en el login y `DECR` en el logout **no funciona**: las sesiones que caducan por TTL nunca llaman a `DECR` y el contador solo crece. Hay varias alternativas:
- **Sorted Set global** `sessions:active`, con `score = última actividad`. Se actualiza con `ZADD` en cada validación y se consulta con `ZCOUNT sessions:active <ahora-1800> +inf`. La limpieza se hace con `ZREMRANGEBYSCORE`. Es exacto, pero añade una escritura por petición.
- **`INFO keyspace`**, si las sesiones viven en una base lógica o una instancia dedicada: el número de `keys` es el número de sesiones. Es gratis y aproximado.
- **HyperLogLog** `PFADD active:{minuto} <userId>`: cuenta usuarios únicos activos con unos 12 KB de memoria y un error del 0,81 %.
- No conviene basarse en las *keyspace notifications* (`notify-keyspace-events Ex`): usan Pub/Sub, que no garantiza la entrega (*fire-and-forget*), así que se pueden perder eventos.

**8. ¿Qué métricas de Redis indicarían problemas?**
Las de la tabla de la Tarea 7. Las más específicas de sesiones son:
- `evicted_keys > 0`: se están expulsando sesiones.
- `keys ≠ expires` en `INFO keyspace`: hay sesiones sin TTL, probablemente por un bug en el que se usó `SET` sin `EX`.
- Picos de `expired_keys`: aviso de un posible thundering herd.
- Latencia p99 por encima de 5 ms: cada petición HTTP valida la sesión, así que la latencia de Redis se suma a todas las peticiones.
