# Guía de estudio — Lab 1: Authentication in Linux

**Asignatura:** Cybersecurity Engineering — UC3M (Cybersecurity Engineering Labs, Student's Guide v1.0, septiembre 2026)
**Bloque:** Prácticas — Lab 1 de 6

---

## 1. Resumen general y objetivos

Este primer laboratorio es una introducción práctica a la **autenticación en sistemas Linux**: cómo se gestionan las cuentas de usuario, cómo se almacenan realmente las contraseñas, cuán fácil (o difícil) es "romperlas" con herramientas de cracking, y cómo el sistema decide, mediante **PAM** (Pluggable Authentication Modules), qué combinación de mecanismos de autenticación se exige en cada situación.

**Objetivos del lab (tal como los define la guía oficial):**
1. Adquirir nociones básicas de gestión de usuarios en Linux.
2. Aprender cómo se crean, almacenan y usan las contraseñas de los usuarios.
3. Tener un primer contacto práctico con la fortaleza de contraseñas usando herramientas de cracking.
4. Estudiar métodos de autenticación alternativos (PAM).

**Lo que necesitas:**
- Una distribución Linux donde tengas privilegios de `root` (Kali o Parrot OS ya traen las herramientas preinstaladas).
- El comando `mkpasswd` y `john` (John the Ripper) instalados: `sudo apt-get install john whois` (`mkpasswd` viene en el paquete `whois` en Debian/Ubuntu).

El hilo conductor del lab tiene tres bloques claramente diferenciados: **(1)** cómo Linux modela un usuario y dónde vive su contraseña, **(2)** cómo un atacante (o un auditor) intenta recuperar esa contraseña a partir de su hash con John the Ripper, y **(3)** cómo PAM permite construir políticas de autenticación flexibles (bloqueos por intentos fallidos, restricción de `su` a un grupo, scripts de alerta, módulos personalizados) sin tocar el código de las aplicaciones.

---

## 2. Gestión básica de usuarios en Linux

### Lecturas preparatorias

Antes del lab: Patrick Louis, *A Compendium of Access Control on Unix-Like OSes*, secciones **"Proving Who We Are"** y **"The Password And Group Files"**. Es el documento de referencia que se reutilizará durante toda la asignatura.

### Conceptos clave

Crear un usuario con `useradd` no solo genera una entrada en un fichero: reserva un UID, opcionalmente crea un directorio `home`, asigna una shell de login y (según las opciones) fuerza el cambio de contraseña en el primer inicio de sesión.

```bash
# Sintaxis básica
sudo useradd <opciones> <nombre_usuario>

# Flags más relevantes
-m, --create-home       # crea el directorio /home/<usuario>
-s, --shell             # shell de login (p. ej. /bin/bash)
-u, --uid               # fuerza un UID concreto
-g, --gid               # grupo primario
-G, --groups            # grupos secundarios (separados por coma)
-c, --comment           # campo GECOS (nombre completo, info de contacto)
-e, --expiredate        # fecha de expiración de la cuenta
-r, --system            # crea una cuenta de sistema (sin login interactivo)
```

**Idea clave para el examen:** `useradd` por sí solo **no establece contraseña** — la cuenta queda bloqueada (`!` en el campo de hash) hasta que se ejecuta `passwd <usuario>`. Es un error común olvidar este paso y luego no entender por qué la cuenta "no funciona".

### El fichero `/etc/passwd`

Cada línea describe un usuario con 7 campos separados por `:`:

```
usuario:x:UID:GID:comentario (GECOS):directorio_home:shell
```

| Campo | Significado | Ejemplo |
|---|---|---|
| `usuario` | Nombre de login | `juan` |
| `x` | Marcador — la contraseña real **no** está aquí, está en `/etc/shadow` | `x` |
| `UID` | User ID numérico | `1001` |
| `GID` | Group ID del grupo primario | `1001` |
| `GECOS` | Información opcional (nombre completo, teléfono...) | `Juan García,,,` |
| `directorio_home` | Ruta del home del usuario | `/home/juan` |
| `shell` | Shell que se lanza al iniciar sesión | `/bin/bash` |

**Por qué importa el campo `x`:** en sistemas UNIX muy antiguos, el hash de la contraseña se guardaba directamente en `/etc/passwd`, que es un fichero **legible por cualquier usuario** (necesario para que comandos como `ls -l` puedan traducir UID → nombre). Eso permitía a cualquier usuario local copiar todos los hashes y atacarlos offline. La solución (*shadow passwords*) fue mover el hash a `/etc/shadow`, un fichero legible solo por `root` (y el grupo `shadow`), dejando en `/etc/passwd` un simple marcador `x`.

### El fichero `/etc/shadow`

```
usuario:hash:último_cambio:mín:máx:aviso:inactividad:expiración:reservado
```

| Campo | Significado |
|---|---|
| `usuario` | Nombre de login (debe coincidir con `/etc/passwd`) |
| `hash` | Hash de la contraseña, con formato `$id$salt$hash` (ver sección 3) |
| `último_cambio` | Días desde 1970-01-01 hasta el último cambio de contraseña |
| `mín` | Días mínimos entre cambios de contraseña |
| `máx` | Días máximos antes de forzar un cambio |
| `aviso` | Días de aviso antes de la expiración |
| `inactividad` | Días de gracia tras expirar antes de deshabilitar la cuenta |
| `expiración` | Fecha (en días desde 1970) en que la cuenta expira |
| `reservado` | Campo reservado, sin uso actualmente |

**Valores especiales del campo `hash`:** `!` o `!!` significa cuenta **bloqueada** (sin contraseña válida, típico tras `useradd` sin `passwd`); `*` significa que el login con contraseña está **deshabilitado** (cuentas de servicio); una cadena vacía significa que **no se requiere contraseña** (peligroso).

**Comando `unshadow`:** combina `/etc/passwd` y `/etc/shadow` en un único fichero con el formato que espera John the Ripper (`usuario:hash:...`), reconstruyendo lo que sería un `/etc/passwd` "clásico" con los hashes reales incluidos:

```bash
sudo unshadow /etc/passwd /etc/shadow > hashes.txt
```

**Resumen sección 2:** un usuario en Linux es una entrada en `/etc/passwd` (identidad, metadatos, shell) más una entrada en `/etc/shadow` (hash de la contraseña y política de expiración). La separación entre ambos ficheros —y sus permisos distintos— es la razón histórica por la que hoy es mucho más difícil robar hashes de contraseñas en un sistema Linux bien configurado.

---

## 3. Cracking de contraseñas con John the Ripper (JtR)

### Lecturas preparatorias

- Cracking modes: openwall.com/john/doc/MODES.shtml
- Examples: openwall.com/john/doc/EXAMPLES.shtml
- Rules: openwall.com/john/doc/RULES.shtml
- Configuration file: openwall.com/john/doc/CONFIG.shtml
- Wordlists: github.com/kkrypt0nn/wordlists

### Prerrequisitos: wordlists

Las distribuciones orientadas a seguridad (Kali, Parrot) ya traen diccionarios instalados. En otras distribuciones:

```bash
sudo apt update && sudo apt install wordlists
cd /usr/share/wordlists/
sudo gunzip rockyou.txt.gz
```

> **`rockyou.txt`** es el diccionario de contraseñas más usado en cracking educativo: proviene de la filtración real de 2009 de la empresa RockYou (~32 millones de contraseñas en texto plano). Su popularidad como wordlist ilustra un punto importante: las contraseñas reales de usuarios reales siguen patrones muy predecibles, y una filtración histórica sigue siendo útil para crackear contraseñas actuales porque la gente reutiliza variaciones de las mismas ideas.

### Formatos de hash y detección automática

John the Ripper detecta automáticamente el formato de hash en la mayoría de casos. Si aparece el mensaje `No password hashes loaded`, hay que indicar el formato explícitamente con `--format`:

```bash
john --format=crypt --mask='?d?d?d?d' hashes.txt
```

### Modos de ataque de JtR

| Modo | Qué hace | Cuándo usarlo |
|---|---|---|
| **Single crack** | Usa variaciones del propio nombre de usuario/GECOS | Contraseñas basadas en datos del propio usuario |
| **Wordlist (diccionario)** | Prueba cada palabra de un fichero (+ reglas opcionales) | Contraseñas basadas en palabras reales |
| **Incremental (fuerza bruta)** | Prueba todas las combinaciones posibles de un alfabeto | Contraseñas cortas o cuando falla el diccionario |
| **Mask attack** | Fuerza bruta dirigida con una plantilla de posiciones | Cuando se conoce parcialmente la estructura (p. ej. "4 dígitos") |
| **External** | Lógica de generación de candidatos en C compilado en tiempo de ejecución | Casos muy específicos, generación de candidatos a medida |

```bash
# Wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Incremental
john --incremental hashes.txt

# Mask attack (4 dígitos numéricos)
john --mask='?d?d?d?d' hashes.txt

# Consultar el estado del proceso de cracking en cualquier momento
john --show hashes.txt
```

### Cheatsheet — Máscaras (mask attack)

| Máscara | Significado | Rango de caracteres |
|---|---|---|
| `?l` | Minúsculas | `abcdefghijklmnopqrstuvwxyz` |
| `?u` | Mayúsculas | `ABCDEFGHIJKLMNOPQRSTUVWXYZ` |
| `?d` | Dígitos | `0123456789` |
| `?s` | Caracteres especiales | `<>!"£$%^&*()_+{}[]#~\/|?` |
| `?a` | Todo lo anterior | (ASCII imprimible) |
| `?A` | ASCII completo | (todos los caracteres ASCII) |
| `?h` | Dígito hexadecimal | `0123456789abcdef` |
| `?H` | Dígito hexadecimal en mayúsculas | `0123456789ABCDEF` |
| `?w` | Híbrido | (elemento actual de una wordlist) |
| `?1`…`?9` | Placeholder personalizado | definido con el flag `--1='...'`, etc. |

**Ejemplo:** una máscara `?u?l?l?l?d?d?d` genera candidatos con la forma "Mayúscula + 3 minúsculas + 3 dígitos" (p. ej. `Abcd123`) — mucho más eficiente que un ataque incremental genérico si se sospecha esa estructura.

### Cheatsheet — Reglas (rules)

Las **reglas** son directivas de transformación que se aplican sobre cada palabra de una wordlist para generar variantes probables **sobre la marcha**, sin tener que almacenar físicamente todas las variantes en el diccionario:

| Categoría | Ejemplo de transformación |
|---|---|
| **Case toggling** | `password` → `Password` / `PASSWORD` |
| **Append/Prepend** | `admin` → `admin123!` / `@admin` |
| **Character substitution (leet speak)** | `password` → `p@ssw0rd` |
| **Structural edits** | `pass` → `ssap` (reverse) / `passpass` (duplicate) / `pas` (truncate) |

```ini
# Ejemplos de sintaxis de reglas en john.conf

# Invertir el orden ("hello" -> "olleh")
[List.Rules:Reverse]
r

# Anteponer un carácter ("hi" -> "@hi")
[List.Rules:Prepend]
A0"@"

# Duplicar la palabra completa ("hi" -> "hihi")
[List.Rules:Duplicate]
:
d

# Todo a minúsculas ("HeLLo" -> "hello")
[List.Rules:Lower]
l

# Capitalizar ("hELLo" -> "Hello")
[List.Rules:Capitalize]
c

# Mayúsculas + añadir "123" ("foo" -> "FOO123")
[List.Rules:Append]
u Az"123"

# Alternar (toggle) mayúscula/minúscula en distintas posiciones
[List.Rules:Toggle]
T0
T1
T2
T0T1
T0T2
T1T2
T0T1T2
```

**Por qué importa:** combinar una wordlist modesta con un buen conjunto de reglas suele ser **mucho más efectivo** que un diccionario enorme sin reglas, porque cubre las variaciones que los humanos aplican intuitivamente a palabras conocidas (mayúscula inicial, año al final, sustitución de letras por símbolos) sin necesitar almacenar ni recorrer todas esas variantes explícitamente.

### El fichero de configuración `john.conf`

`john.conf` es el fichero central de configuración de JtR, organizado en secciones con cabeceras `[Section.Type:Name]`:

| Sección | Qué controla |
|---|---|
| `[Options]` / `[System]` | Parámetros globales: rutas de wordlists por defecto, hilos de CPU, prioridad, checkpoints |
| `[List.Rules:*]` | Secuencias de mutación integradas o personalizadas (`Wordlist`, `Jumbo`...) |
| `[Incremental:*]` | Configuración de los modos de fuerza bruta (límites de longitud, alfabetos, ficheros `.chr`) |
| `[List.External:*]` | Lógica personalizada en C compilada en tiempo de ejecución (generación/filtrado de candidatos a medida) |
| `[Markov:*]` | Ataques estadísticos tipo cadena de Markov, ponderando secuencias de caracteres realistas |
| `[Disabled:*]` | Desactiva formatos de hash o métodos de ataque concretos para acelerar el proceso |

**Buena práctica:** no modificar `john.conf` directamente. Las reglas y configuraciones personalizadas deben añadirse en `/usr/share/john/john-local.conf` (la ruta exacta puede variar según la distribución), que JtR carga además del fichero principal.

**Resumen sección 3:** John the Ripper combina un **formato de hash**, un **modo de ataque** (wordlist, incremental, mask, external) y opcionalmente **reglas** de mutación. `--show` consulta el progreso sin relanzar el ataque. El fichero `john.conf` (o su variante `-local`) centraliza toda la configuración avanzada.

---

## 4. Experimento de cracking a gran escala (ejercicio de síntesis)

El lab plantea un escenario ficticio de dos filtraciones de datos (`ForoMotos.com`, con más de 180.000 pares usuario/hash, y `Meneate.net`, con más de 220.000) para practicar crackear el máximo número de contraseñas posible combinando **todo lo anterior**.

**Pistas que da el propio enunciado y cómo traducirlas en estrategia:**

- *"ForoMotos.com generaba por defecto contraseñas alfanuméricas en minúsculas de 5 caracteres (luego 6); más tarde, auto-generadas con mayúsculas y un símbolo obligatorio."* → sugiere **mask attacks** específicos por rango de fechas de registro, no un único ataque para todo el dataset.
- *"La longitud mínima de ForoMotos.com es 4 caracteres."* → acota el espacio de búsqueda de cualquier ataque de fuerza bruta o máscara.
- *"Ambos sitios están poblados por usuarios de habla hispana."* → un diccionario genérico en inglés (o incluso `rockyou.txt`, mayoritariamente anglosajón) rendirá peor que uno enriquecido con palabras y patrones en español (nombres propios, equipos de fútbol, "contraseña", "12345", fechas en formato DD-MM...).
- *"Hay un subforo de hacking con jerga de 'script kiddie'."* → sugiere añadir al diccionario jerga específica de ese colectivo (`h4ck3r`, `pwned`, `1337`...).
- *"Reutilización de contraseñas entre sitios."* → las contraseñas ya crackeadas de un dataset son, en sí mismas, un **wordlist de alta calidad** para atacar el otro dataset (y viceversa) — es una técnica real llamada *credential stuffing* aplicada al propio proceso de cracking.

**Metodología recomendada:** empezar siempre por el ataque más barato computacionalmente (diccionario + reglas con `rockyou.txt` y wordlists en español) antes de pasar a mask attacks dirigidos y, solo como último recurso, a incremental puro — que es el más costoso en tiempo.

---

## 5. Profundizando en fortaleza de contraseñas (ejercicio opcional, metodología experimental)

Este ejercicio propone un diseño experimental real para medir empíricamente la fortaleza de contraseñas:

1. **Generar 25 datasets de 100 contraseñas** cada uno, agrupados por complejidad: solo minúsculas, solo mayúsculas, solo dígitos, alfanumérico+símbolos, y basadas en diccionario — todos variando la longitud entre 3 y 7 caracteres.
2. **Repetir con un algoritmo de hash de coste computacional muy distinto** (p. ej. `md5crypt` frente a `SHA-512`), para poder comparar el efecto del propio algoritmo de hash, no solo el de la contraseña.
3. **Diseñar al menos 5 estrategias de cracking distintas** con JtR (modo + parámetros/reglas/máscara/configuración).
4. **Ejecutar cada estrategia contra cada dataset con un límite de tiempo**, registrando **porcentaje de contraseñas crackeadas** y **media/mediana del tiempo** por contraseña.
5. **Tabular y analizar** los resultados.

**Por qué importa este ejercicio:** traduce en datos concretos una intuición que se repite durante todo el módulo teórico de autenticación: la fortaleza de una contraseña no es solo "cuántos caracteres tiene", sino una combinación de **espacio de búsqueda** (longitud × alfabeto), **algoritmo de hash** (más lento de calcular = más caro de crackear, ver *key stretching* en la guía teórica del Módulo 2) y **previsibilidad humana** (una contraseña "aleatoria" de diccionario cae casi instantáneamente frente a reglas bien diseñadas).

---

## 6. Pluggable Authentication Modules (PAM)

### Lecturas preparatorias

- Patrick Louis, *A Compendium of Access Control on Unix-Like OSes*, sección **"Pluggable Authentication Module (PAM)"**.
- `man PAM` (página de manual `PAM(8)`).

### Prerrequisitos

```bash
sudo apt-get install gcc libpam0g-dev
# pamtester puede requerir compilación o un paquete específico según distro
```

### ¿Qué es PAM y por qué existe?

**PAM** es una capa de abstracción entre las aplicaciones que necesitan autenticar usuarios (`login`, `sshd`, `su`, `sudo`...) y los mecanismos concretos de autenticación disponibles en el sistema (contraseña Unix clásica, huella dactilar, tarjeta inteligente, LDAP, 2FA...). En lugar de que cada aplicación reimplemente su propia lógica de autenticación, delega la decisión en PAM mediante ficheros de configuración declarativos en `/etc/pam.d/`.

**Idea clave:** gracias a PAM, cambiar **cómo** se autentica un usuario en `sshd` (añadir un segundo factor, bloquear tras N intentos fallidos, restringir por grupo) es cuestión de **editar un fichero de texto**, no de recompilar ni parchear el propio servidor SSH.

### La estructura de una línea PAM

```
<tipo>    <control_flag>    <módulo.so>    [argumentos]
```

| Tipo | Qué gestiona |
|---|---|
| `auth` | Verifica la identidad (¿quién eres?) |
| `account` | Comprueba si la cuenta tiene permitido el acceso (expirada, bloqueada, restringida por horario...) |
| `password` | Gestiona la actualización de credenciales |
| `session` | Configura el entorno antes/después de la sesión (montar directorios, registrar logs...) |

### Control flags — cómo se evalúa una pila de módulos

| Flag | Comportamiento |
|---|---|
| `required` | Si falla, la autenticación **fallará al final**, pero PAM sigue evaluando el resto de módulos de la pila antes de devolver el error (no corta inmediatamente) |
| `requisite` | Si falla, la autenticación **falla inmediatamente**, sin evaluar el resto de módulos restantes |
| `sufficient` | Si tiene éxito **y** ningún `required` anterior ha fallado, es suficiente para autenticar — no hace falta evaluar el resto |
| `optional` | Su resultado normalmente no determina el éxito/fracaso global, salvo que sea el único módulo de la pila |

**Por qué importa el orden:** con `sufficient` colocado *antes* que `required`, basta con que el módulo `sufficient` tenga éxito para autenticar, sin llegar siquiera a evaluar el `required` posterior. Invertir el orden (poner primero el `required`) cambia completamente el comportamiento resultante — este es exactamente el tipo de razonamiento que pide el `Ex.17` del lab (comparar una pila `sufficient` + `required` con su versión invertida usando `pam_permit.so` y `pam_deny.so`, que son módulos de prueba que siempre conceden o siempre deniegan el acceso).

### Módulos relevantes en este lab

| Módulo | Función |
|---|---|
| `pam_permit.so` | Siempre concede el acceso (módulo de prueba/depuración) |
| `pam_deny.so` | Siempre deniega el acceso (módulo de prueba/depuración) |
| `pam_unix.so` | Autenticación clásica contra `/etc/shadow` |
| `pam_wheel.so` | Restringe una operación (típicamente `su`) a los miembros de un grupo concreto (por defecto, `wheel`) |
| `pam_faillock.so` | Cuenta intentos fallidos consecutivos y bloquea la cuenta temporalmente al superar un umbral |
| `pam_exec.so` | Ejecuta un script o programa externo como parte del flujo de autenticación |

### Herramienta de prueba: `pamtester`

Permite simular el flujo de autenticación de una aplicación PAM sin tener que instalar/configurar la aplicación real, apuntando a un fichero de configuración de prueba (p. ej. `/etc/pam.d/pamtest`):

```bash
pamtester pamtest <usuario> authenticate
```

### Guía de los ejercicios de PAM

**Ex.17 — Orden de evaluación de la pila.** Comparar `sufficient` + `required` frente a su orden invertido con `pam_permit.so`/`pam_deny.so`. Para razonar sobre el resultado, aplica estrictamente la tabla de control flags de arriba: ¿se llega a evaluar el segundo módulo? ¿Importa si el primero tuvo éxito o fracasó? La segunda pregunta (sustituir `sufficient` por `requisite` en un módulo que falla al principio de la pila) pone a prueba si entiendes la diferencia exacta entre `required` y `requisite` — ambos "fallan", pero solo uno corta la evaluación inmediatamente.

**Ex.18 — Restricción de `su` y bloqueo por fuerza bruta.** Combina `pam_wheel.so` (para que solo el grupo `wheel` pueda usar `su` hacia root) con `pam_faillock.so` (bloqueo temporal tras 3 intentos fallidos). La pregunta sobre el flag `optional` en `pam_wheel.so` frente a `pam_unix.so` como `required` pone a prueba de nuevo la tabla de control flags: un módulo `optional` normalmente no puede por sí solo autenticar ni denegar si hay otros módulos determinantes en la pila.

**Ex.19 — Encadenar un script externo con `pam_exec.so`.** Un script que registra accesos root en un log, invocado como módulo `auth optional`. La pregunta clave — qué pasa si el script devuelve código de error 1 y el módulo está marcado como `required` en vez de `optional` — es exactamente la tabla de control flags aplicada a un caso real: con `required`, el fallo del script contaría como un fallo de ese paso de la pila (aunque la contraseña del usuario sea correcta), mientras que `optional` normalmente no bloquea el resto del flujo.

**Ex.20 — Módulo PAM personalizado en C.** Escribir `pam_passcode.c`, que solicita un PIN fijo antes del login estándar. La pregunta sobre inyectar variables de entorno en la sesión del usuario apunta a la función de la API de PAM `pam_putenv()` (dentro de las funciones de tipo `session`), que permite a un módulo exportar variables al entorno de la sesión que se está abriendo.

**Resumen sección 6:** PAM desacopla las aplicaciones de los mecanismos de autenticación mediante pilas de módulos declaradas en `/etc/pam.d/`. El comportamiento de cada pila depende tanto de **qué módulos** se usan como del **orden** y el **control flag** (`required`, `requisite`, `sufficient`, `optional`) de cada uno — el mismo conjunto de módulos, reordenado, puede producir un sistema de autenticación completamente distinto.

---

## 7. Tabla resumen de comandos del lab

| Comando | Para qué sirve |
|---|---|
| `useradd <opciones> <usuario>` | Crea una cuenta de usuario (sin contraseña hasta ejecutar `passwd`) |
| `passwd <usuario>` | Establece/cambia la contraseña de un usuario |
| `unshadow /etc/passwd /etc/shadow > hashes.txt` | Combina ambos ficheros en el formato que espera JtR |
| `mkpasswd` | Genera hashes de contraseña con un algoritmo y salt dados, sin cambiar la contraseña real de ningún usuario |
| `john [--format=] [--wordlist=] [--mask=] [--incremental] hashes.txt` | Lanza un ataque de cracking con JtR |
| `john --show hashes.txt` | Consulta el progreso/resultados sin relanzar el ataque |
| `ls /usr/share/john/*.chr` | Lista los ficheros estadísticos disponibles para modo incremental |
| `pamtester <servicio> <usuario> authenticate` | Simula el flujo PAM de un servicio sin la aplicación real |

---

## 8. Preguntas de autoevaluación

**1. ¿Por qué `useradd` por sí solo no permite iniciar sesión con el nuevo usuario?**
> Porque no establece ninguna contraseña válida: el campo de hash en `/etc/shadow` queda con un valor como `!` o `!!` (cuenta bloqueada). Hace falta ejecutar `passwd <usuario>` para fijar una contraseña utilizable.

**2. ¿Por qué se separó el hash de la contraseña de `/etc/passwd` a `/etc/shadow`?**
> Porque `/etc/passwd` debe ser legible por cualquier usuario del sistema (lo necesitan comandos como `ls -l` para traducir UID a nombre), lo que en sistemas antiguos exponía todos los hashes de contraseña a cualquier usuario local para un ataque offline. `/etc/shadow` es legible solo por `root` (y el grupo `shadow`), reduciendo drásticamente esa superficie de exposición.

**3. ¿Qué hace exactamente el comando `unshadow` y por qué es necesario antes de usar JtR sobre un sistema real?**
> Combina las líneas de `/etc/passwd` y `/etc/shadow` en un único fichero con el formato `usuario:hash:...` que John the Ripper espera como entrada, ya que JtR necesita tanto el nombre de usuario como el hash real (que vive en `/etc/shadow`, no en `/etc/passwd`).

**4. Diferencia entre un ataque de diccionario (wordlist) y un ataque incremental (fuerza bruta) en JtR.**
> El ataque de diccionario prueba palabras de una lista predefinida (opcionalmente mutadas con reglas), aprovechando que las contraseñas humanas suelen basarse en palabras reales. El incremental prueba sistemáticamente todas las combinaciones posibles de un alfabeto dado, sin asumir ninguna estructura — es exhaustivo pero mucho más costoso computacionalmente, especialmente a partir de cierta longitud.

**5. ¿Qué ventaja tienen las reglas (`rules`) de JtR frente a simplemente usar una wordlist enorme que ya incluya todas las variantes?**
> Las reglas generan las variantes (mayúsculas, sufijos numéricos, leet speak...) **sobre la marcha** a partir de una wordlist base, sin necesidad de almacenar ni recorrer físicamente todas esas variantes como entradas separadas — mucho más eficiente en espacio y a menudo en tiempo, y más fácil de mantener/extender.

**6. ¿Qué es un *mask attack* y cuándo tiene sentido usarlo en vez de fuerza bruta pura?**
> Es una fuerza bruta dirigida, donde se define una plantilla de posiciones (p. ej. `?u?l?l?l?d?d?d`) en vez de probar cualquier combinación en cualquier posición. Tiene sentido cuando se conoce (o sospecha) parcialmente la estructura de la contraseña — por ejemplo, un PIN de 4 dígitos, o una política conocida de "mayúscula inicial + 3 dígitos" — reduciendo drásticamente el espacio de búsqueda frente a un incremental genérico.

**7. En el escenario de ForoMotos.com/Meneate.net, ¿por qué las contraseñas ya crackeadas de un dataset son útiles para atacar el otro?**
> Porque los usuarios reutilizan contraseñas (o variaciones de la misma) entre distintos sitios web. Las contraseñas recuperadas de un dataset funcionan como una wordlist de alta calidad, muy adaptada al perfil real de esos usuarios, para atacar el segundo dataset — el mismo fenómeno que en un contexto de ataque real se llama *credential stuffing*.

**8. ¿Por qué el algoritmo de hash usado (p. ej. `md5crypt` frente a `SHA-512` con múltiples rondas) afecta tanto al tiempo de cracking como el propio texto de la contraseña?**
> Porque el coste computacional de calcular **un solo intento** de hash multiplica el tiempo total del ataque, que consiste en probar millones o miles de millones de candidatos. Un algoritmo de hash deliberadamente lento (más rondas, más memoria) hace que cada intento cueste más tiempo/recursos, ralentizando el cracking aunque la contraseña en sí no cambie — es el principio de *key stretching* visto en la teoría de autenticación (Módulo 2).

**9. ¿Cuál es la diferencia entre los control flags `required` y `requisite` en una pila PAM?**
> Ambos hacen que la autenticación termine en fallo si el módulo falla, pero `required` permite que PAM **siga evaluando** el resto de módulos de la pila antes de devolver el fallo final, mientras que `requisite` **corta inmediatamente** la evaluación en cuanto falla, sin llegar a ejecutar los módulos posteriores.

**10. Si una pila PAM tiene, en este orden, `auth sufficient pam_permit.so` seguido de `auth required pam_deny.so`, ¿puede un usuario autenticarse con éxito? ¿Y si se invierte el orden?**
> En el orden original, `pam_permit.so` (que siempre tiene éxito) es `sufficient`, así que basta con su éxito para autenticar — nunca se llega a evaluar `pam_deny.so`. Si se invierte el orden (`required pam_deny.so` primero, `sufficient pam_permit.so` después), `pam_deny.so` fallará, y aunque PAM siga evaluando el resto de la pila por ser `required`, el resultado final será fallo — un `sufficient` posterior no puede "rescatar" una autenticación que ya tiene un `required` fallido en la pila.

**11. ¿Para qué sirve `pam_wheel.so` y qué problema de seguridad concreto mitiga?**
> Restringe el uso de una operación privilegiada (típicamente `su` hacia root) a los miembros de un grupo determinado (por defecto `wheel`). Mitiga que cualquier usuario del sistema que conozca (o adivine/crackee) la contraseña de root pueda escalar privilegios con `su`, limitando esa vía a un conjunto reducido y controlado de cuentas de confianza.

**12. ¿Qué hace `pam_faillock.so` y por qué es una defensa distinta de restringir por grupo?**
> Cuenta los intentos de autenticación fallidos consecutivos de una cuenta y la bloquea temporalmente al superar un umbral (p. ej. 3 intentos → bloqueo de 15 minutos). Es una defensa complementaria: mientras `pam_wheel.so` restringe **quién puede intentar** una operación, `pam_faillock.so` limita **cuántas veces se puede fallar**, mitigando específicamente ataques de fuerza bruta online contra la propia autenticación del sistema.

**13. ¿Qué implica marcar `pam_exec.so` como `required` en vez de `optional` dentro de una pila `auth`, si el script que ejecuta puede fallar?**
> Que un fallo del script externo (por ejemplo, no poder escribir en su fichero de log por falta de permisos) se contabilizaría como un fallo de ese paso de la pila de autenticación, pudiendo bloquear el login de un usuario con credenciales correctas por un motivo ajeno a la propia contraseña. Con `optional`, ese mismo fallo normalmente no impide el resto del flujo de autenticación.

**14. ¿Por qué comandos como `ping` no necesitan que el usuario tenga privilegios de root, mientras que otros binarios de red (como `tcpdump`) sí los exigen por defecto?**
> Porque operaciones como abrir un *raw socket* o poner una interfaz en modo promiscuo (necesario para capturar todo el tráfico, no solo el dirigido a la propia máquina) requieren privilegios elevados en el kernel. Herramientas como `ping` suelen resolver esto mediante el bit `setuid` o, en sistemas modernos, mediante **capabilities** de Linux (p. ej. `CAP_NET_RAW`) asignadas específicamente al binario, en vez de exigir usar `sudo` o root para toda la herramienta — permitiendo otorgar solo el privilegio mínimo necesario en lugar de root completo.

**15. Un módulo PAM personalizado necesita exportar una variable de entorno a la sesión del usuario tras un login exitoso. ¿Qué función de la API de PAM se usaría y en qué tipo de módulo (`auth`, `account`, `password`, `session`) tendría sentido?**
> La función `pam_putenv()`, típicamente invocada desde un módulo de tipo `session` (aunque también puede llamarse desde `auth`), ya que es en la fase de apertura de sesión donde tiene sentido configurar el entorno que heredará el proceso de login del usuario.

---

## 9. Cuándo aplicar todo esto en la práctica

- Antes de crackear cualquier hash, identifica primero el **formato** (`--format`) — perder tiempo con un ataque mal configurado por un formato incorrecto es el error más común de principiante.
- Empieza siempre por el ataque más barato (diccionario + reglas) antes de escalar a mask attacks dirigidos o incremental puro.
- Al diseñar una pila PAM, dibuja primero el flujo de evaluación en papel (qué módulos, en qué orden, con qué flag) antes de tocar `/etc/pam.d/` — un error de configuración en `sshd` o `su` puede dejarte fuera del sistema.
- Usa siempre `pamtester` contra un servicio de prueba (`/etc/pam.d/pamtest`) antes de modificar servicios reales como `sshd` o `su`, para no arriesgarte a un bloqueo total de acceso.
- Haz una copia de seguridad (o snapshot de VM) antes de experimentar con `pam_faillock.so` o restricciones de `su` — es fácil bloquearte a ti mismo por accidente.
