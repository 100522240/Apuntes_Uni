# Guía de estudio — Módulo 2: Authentication

**Asignatura:** Cybersecurity Engineering — UC3M
**Grupo de investigación de referencia:** COSEC (Computer Security Lab)

---

## 1. Resumen general y objetivos

Este módulo cubre la **autenticación**: el proceso de verificar que alguien es quien dice ser antes de concederle acceso a un recurso. El hilo conductor va de lo conceptual a lo práctico: primero se define qué es autenticar (y en qué se diferencia de identificar y de autorizar), después se analiza en profundidad el mecanismo más extendido —las **contraseñas**— con todo su ciclo de vida (cómo se almacenan, cómo se rompen, cómo se eligen bien, cómo se gestionan con herramientas dedicadas), se amplía el catálogo a **otros autenticadores** (OTP, dispositivos criptográficos, biometría), se repasan las **amenazas** genéricas a cada tipo de factor, y se cierra con la autenticación en **sistemas distribuidos** (identidad federada, SSO, SAML, OIDC).

**Resultados de aprendizaje:**
- Distinguir identificación, autenticación y autorización, y situar ambas en el framework NIST 800-63-3.
- Conocer los tres factores de autenticación (saber, tener, ser) y los tipos concretos de autenticadores dentro de cada uno.
- Entender por qué las contraseñas en texto plano son inaceptables, cómo se atacan las contraseñas hasheadas (fuerza bruta, diccionario, rainbow tables) y cómo se defienden (salting, key stretching).
- Aplicar heurísticas correctas para elegir contraseñas fuertes y diseñar verificadores que no perjudiquen la seguridad real.
- Conocer los gestores de contraseñas, sus ventajas, y los ataques históricos que han sufrido.
- Comprender los autenticadores biométricos: propiedades deseables, proceso de enrolamiento/verificación y ejemplos.
- Explicar cómo funciona la autenticación federada (IdP, RP, assertions) y los protocolos SAML/OIDC.

La idea central del módulo: **no existe un autenticador perfecto**. Cada factor (saber, tener, ser) tiene su propio modelo de amenaza —una contraseña se puede adivinar u observar, un token se puede robar o clonar, un rasgo biométrico se puede replicar—, y la seguridad real de un sistema depende de elegir el factor adecuado para el contexto, combinarlo con otros cuando hace falta (MFA), e implementarlo siguiendo las mejores prácticas conocidas (hash con salt y coste adaptativo, verificadores que no penalicen al usuario, protocolos de federación que minimicen la exposición de datos).

---

## 2. ¿Qué es la autenticación?

### 2.1 La base de la seguridad informática: acceso controlado

Todo control de acceso se reduce a tres piezas: **Who** (alguien) realiza una **Action** (alguna acción) sobre un **Resource** (algo). La **Authentication (AuthN)** es el proceso de verificar o confirmar una identidad — comprobar el "quién" antes de conceder acceso. La **Authorization (AuthZ)** —permisos, capacidades, control de acceso— depende por completo de conocer primero esa identidad: **AuthZ sin AuthN no tiene sentido**, porque no se puede decidir qué puede hacer alguien sin saber antes quién es.

El mecanismo habitual es **usuario + secreto** (`username` + `secret`). El secreto intenta limitar el acceso no autorizado, y sus propiedades deseables son ser **unforgeable** (no falsificable), **unguessable** (no adivinable) y **revocable**.

### 2.2 Identificación vs. Autenticación

| Aspecto | Identificación | Autenticación |
|---|---|---|
| Qué hace | Afirmar quién es una persona (*asserting*) | Probar la identidad afirmada (*proving*) |
| Base | Propiedades de identidad digital: bien conocidas, predecibles, adivinables | Basada en ítem(s) **privados** |
| Objetivo | — | Confirmar que la persona es quien dice ser |
| Ejemplos | Email, número de teléfono, cuenta bancaria, nombre de usuario | Contraseñas, preguntas de seguridad, cara, huellas dactilares, tarjetas, tokens |

**Idea clave:** el identificador (p. ej. tu email) es público y no es un secreto — cualquiera puede saberlo. Lo que demuestra que *tú* eres el dueño de ese identificador es el autenticador (algo privado).

### 2.3 Framework: NIST 800-63-3

Es el marco de referencia para digital identity guidelines. Define los roles y flujos:

- **Applicant** → se convierte en **Subscriber** tras el proceso de **Enrollment and Identity Proofing** ante un **CSP** (*Credential Service Provider*), que le emite/registra un **Authenticator**.
- **Subscriber** → se convierte en **Claimant** cuando intenta **Authenticate** ante un **Verifier**.
- El **Verifier** valida el authenticator/credential binding contra el CSP y recibe atributos (*Attributes*).
- Si la autenticación es correcta, el Verifier emite una **Authentication Assertion** a la **Relying Party (RP)**, que establece una **Authenticated Session** con el subscriber.

Este flujo (Applicant → Subscriber → Claimant, con CSP y Verifier de por medio) es la base conceptual que luego se reutiliza en la sección de sistemas distribuidos (IdP/RP, sección 9).

### 2.4 Autenticadores (= tipos de "secretos")

Los tres factores clásicos de autenticación:

| Factor | Descripción | Ejemplos |
|---|---|---|
| **Something you know** | Información memorizada | Contraseñas, PINs, passphrases, preguntas de seguridad |
| **Something you have** | Objetos físicos | Smart cards, tokens hardware, key fobs, móviles |
| **Something you are** | Rasgos físicos inherentes | Huellas dactilares, reconocimiento facial, iris |

**One vs. multiple factors (MFA):** la autenticación multifactor combina **dos o más factores independientes**, lo que mejora drásticamente la seguridad porque comprometer un solo factor ya no basta para suplantar la identidad.

Dentro de estos tres factores, NIST clasifica 9 tipos concretos de authenticators (desarrollados en la sección 6 de esta guía): *memorized secrets*, *look-up secrets*, *out-of-band devices*, *single-factor OTP device*, *multi-factor OTP device*, *single-factor crypto software*, *multi-factor crypto software*, *single-factor crypto device*, *multi-factor crypto device*.

**Resumen T2.1:** AuthN verifica identidad; AuthZ decide permisos y depende de AuthN. Identificación = afirmar quién eres (público); autenticación = probarlo (privado). NIST 800-63-3 define el flujo Applicant→Subscriber→Claimant con CSP y Verifier. Tres factores: saber, tener, ser — MFA los combina.

---

## 3. Contraseñas

### 3.1 Memorized secrets — qué son

También conocidas como **passwords** (o PINs si son puramente numéricas). Es el factor "something you know" más ampliamente implementado.

**La paradoja de las contraseñas:**

| A favor | En contra |
|---|---|
| No requieren entrenamiento — todo el mundo sabe usarlas | La gente no puede recordar ítems poco usados o poco cambiados |
| Baratas de implementar — sin dispositivo físico | No podemos "olvidar bajo demanda" |
| Cómodas — no hay que llevar nada extra | **Recall** (recordar de cero) es más difícil que **recognition** (reconocer) |
| Fáciles de gestionar centralizadamente | Las palabras sin significado son más difíciles de recordar |
| Ampliamente soportadas por todo sistema/web/app | — |

**La regla de oro (the Strength Principle):** deben ser lo bastante "fuertes" para que adivinarlas sea impracticable. Pero se consideran erróneamente "sencillas de usar" — la sección 4 de esta guía muestra por qué ese supuesto es engañoso.

### 3.2 Almacenamiento de contraseñas: texto plano

Un sistema tiene que **validar** las contraseñas que el usuario introduce, por lo que necesita **almacenarlas** de alguna forma. El enfoque más básico —y **totalmente inaceptable**— es guardarlas tal cual (`plain text`).

**Escenario de ataque:**
1. **System Compromise:** el atacante viola el perímetro de red y compromete sistemas centrales.
2. **File Theft:** busca y roba el fichero de contraseñas del sistema.
3. **Immediate Exploitation:** si están en texto plano, ya están listas para usarse — el atacante puede iniciar sesión inmediatamente como cualquier usuario, **incluido root**.

> **Estándar de seguridad crítico:** las contraseñas **nunca** deben almacenarse en texto plano.

### 3.3 Almacenamiento de contraseñas: hashed passwords

**Idea central:** almacenar una versión criptográficamente hasheada de la contraseña en lugar de texto plano, usando una función de un solo sentido (*one-way function*).

**Propiedades criptográficas de un buen hash:**

1. **Determinista:** la misma contraseña siempre produce el mismo hash — `hash(x) = hash(x)`.
2. **Alta entropía (efecto avalancha):** un cambio mínimo en la entrada produce una salida completamente distinta. Ejemplo: `MD5('security') = e91e6348157868de...` frente a `MD5('Security') = 2fae32629d4ef4fc...`.
3. **Resistente a colisiones:** es computacionalmente inviable encontrar un `y` tal que `hash(x) = hash(y)`.

### 3.4 El problema: contraseñas débiles

**¿Son los hashes seguros frente al cracking? No.** Aunque la función hash es difícil de invertir matemáticamente, las contraseñas hasheadas siguen siendo vulnerables al **cracking offline**: el reto no es "deshacer" el hash, sino explotar el hecho de que las personas eligen contraseñas predecibles.

> **El elemento humano:** los informes desde finales de los años 70 sugieren repetidamente que la gente elige contraseñas **débiles**.

**Estudios y datos reales:**

| Estudio | Hallazgo |
|---|---|
| **Morris & Thomson (1979)** — estudio pionero sobre seguridad de contraseñas UNIX | El 86% de las contraseñas caían en categorías triviales: caracteres ASCII simples, palabras de diccionario, letras minúsculas, etc. |
| **Egg Online Bank (2002)** — análisis de PINs elegidos por clientes | El 50% usaba nombres de familiares; el 8% usaba nombres de famosos |
| **Imperva (2009)** — análisis de 34 millones de contraseñas de Facebook filtradas | El 30% tenía menos de 7 caracteres; el 50% eran triviales (nombres o palabras de diccionario) |

### 3.5 Cracking de contraseñas: fuerza bruta

La fuerza bruta prueba sistemáticamente todas las combinaciones posibles.

| Escenario | Combinaciones | Tiempo estimado |
|---|---|---|
| Contraseñas cortas (≤3 caracteres, no sensible a mayúsculas) | 18.278 (26¹+26²+26³) | ~5 horas a 1 ms/intento |
| Contraseñas de 6 caracteres, CPU x86 estándar | ~690.000 millones | 3,5 horas |
| Contraseñas de 6 caracteres, GPU moderna (hardware paralelo) | ~690.000 millones | **15 minutos** |

**Por qué la fuerza bruta escala mal para el defensor:** el crecimiento del espacio de búsqueda es **exponencial** al añadir longitud o símbolos, pero el factor crítico es que **la búsqueda se puede paralelizar** — cada intento es independiente, así que miles de GPUs pueden repartirse el trabajo.

> **Caso real — EFF DES Cracker (1998):** DES usa una clave relativamente pequeña de **56 bits**. La Electronic Frontier Foundation construyó una máquina especializada con **1.856 chips ASIC** por un coste de **250.000 $**, capaz de romper por fuerza bruta **todas** las claves DES en **56 horas**. Ataques prácticos similares se han demostrado contra **MD5** y **SHA1**, funciones hash antaño consideradas seguras.

### 3.6 Cracking de contraseñas: ataques de diccionario

**Premisa:** dado que la gente no elige contraseñas verdaderamente aleatorias (sección 3.4), ¿por qué no tomar una lista precompilada de candidatos (un **diccionario**) y probarlos sistemáticamente?

```
Dictionary.txt (word1, word2, ..., wordN)
        │
        ▼
   Hash Function
        │
        ▼
  ¿Coincide con hash del fichero robado? ──Sí──▶ ¡Contraseña encontrada!
        │
        No
        │
        ▼
  Probar siguiente candidato
```

**Resultado:** es habitual crackear el **60-70%** de las contraseñas hasheadas de un sistema en **menos de 24 horas**, precisamente porque los diccionarios explotan la predictibilidad humana en lugar de recorrer el espacio completo de posibilidades.

### 3.7 Rainbow Tables

Combinan tres ideas para optimizar el cracking offline:

1. **Hashes deterministas:** las funciones hash criptográficas son deterministas — la misma entrada siempre produce la misma salida. Esta consistencia es el fundamento matemático de los ataques de precomputación.
2. **Estrategia offline:** el adversario construye listas precomputadas de hashes **offline**, sin necesidad de acceso al sistema objetivo. La ventaja es que se cambia tiempo de preparación offline por un cracking online casi instantáneo.
3. **Rainbow Tables propiamente dichas:** contienen un espectro estructurado de contraseñas hasta longitud N, de forma altamente compacta y optimizada, permitiendo una reversión de hashes eficiente en tiempo-espacio mediante ataques **TMTO** (*Time-Memory Trade-Off*).

### 3.8 Hardening de hashes de contraseñas

**Solución 1 — Salting**

| Elemento | Descripción |
|---|---|
| **La solución** | Hacer cada hash de contraseña **únicamente identificable**, añadiendo un número aleatorio (*salt*) a cada contraseña antes de hashearla |
| **Fundamento** | `Hash(p + r1) != Hash(p + r2)`. Cada usuario recibe un salt completamente único y aleatorio, así que dos usuarios con la **misma** contraseña producen hashes **distintos** |
| **Almacenamiento** | El salt se guarda en **texto plano** junto al hash. **No es un secreto** — el objetivo es la unicidad de los hashes, no la ocultación del salt |

El salt inutiliza las rainbow tables precomputadas genéricas: un atacante tendría que precomputar una tabla distinta por cada salt, lo que anula la ventaja de la precomputación offline.

**Solución 2 — Adaptive hashes / Key stretching**

| Concepto | Descripción |
|---|---|
| **La amenaza** | Hardware especializado (GPUs) cada vez más rápido, con cientos/miles de núcleos, fácilmente **alquilable en la nube** para lanzar ataques de alta velocidad |
| **Key idea** | Hacer el hashing **deliberadamente lento**. Los hashes criptográficos estándar están diseñados para ser **rápidos**, lo cual es contraproducente para proteger contraseñas |
| **Key stretching** | Aplicar el hash múltiples veces de forma recursiva (*multiple rounds*). Ejemplo: **bcrypt** usa tradicionalmente 25 rondas de DES internamente |
| **Adaptive hashes** | Funciones hash especiales con un **factor de trabajo (work factor)** configurable que controla su lentitud. Estándares de la industria: **PBKDF2**, **scrypt** y **bcrypt** |

**Resumen T2.3 (Contraseñas):** nunca texto plano; hash con propiedades deterministas/alta entropía/resistencia a colisiones. Los hashes solos no bastan porque la gente elige contraseñas predecibles (fuerza bruta, diccionario, rainbow tables). La defensa correcta es **salt + hash adaptativo con work factor** (bcrypt/PBKDF2/scrypt), nunca solo un hash rápido (MD5/SHA1 son inadecuados para contraseñas).

---

## 4. Elegir contraseñas

### 4.1 ¿Qué es una contraseña fuerte? La entropía

La métrica clásica es la **entropía**:

```
S = log₂(N^L)
```

Donde `N` = número de símbolos posibles, `L` = longitud de la contraseña, `S` = "fuerza" en bits.

**Aplicación y límites:** esta fórmula calcula la longitud `L` necesaria para alcanzar una fortaleza objetivo `S`. Pero tiene una **asunción crucial**: solo es válida para una contraseña **generada verdaderamente al azar**. Dado que los humanos rara vez eligen contraseñas verdaderamente aleatorias, las métricas de entropía tradicionales fallan a menudo en escenarios reales — la contraseña puede tener el "aspecto" de ser fuerte según la fórmula y ser trivialmente adivinable si sigue un patrón predecible.

### 4.2 Algoritmos mentales — por qué fallan

Durante años se enseñaron (y se vendieron como "seguros") procedimientos mentales del tipo:

```
1. Elegir una palabra
2. Poner en mayúscula la primera o última letra
3. Añadir un número (y quizá un símbolo) al principio o final

1. Elegir una palabra
2. Sustituir algunas letras por símbolos (a→@, s→$, ...)
3. Quizá poner en mayúscula la primera o última letra (o ambas)
```

El resultado es algo como `Computer3@` o `Cl4ssr00m`. El problema: los adversarios modernos tienen herramientas de cracking sofisticadas que **no necesitan fuerza bruta** — usan diccionarios que incluyen contraseñas de filtraciones anteriores y **aplican precisamente estas reglas comunes** de los algoritmos mentales. Un algoritmo mental "popular" es, por definición, predecible para un atacante que conoce ese mismo algoritmo.

### 4.3 Políticas de contraseñas (estudio Shay et al.)

> **Shay et al., ACM TISSEC 2016** — comparó empíricamente distintas políticas de composición de contraseñas midiendo qué porcentaje se adivinaba en función del número de intentos.

| Política | Descripción |
|---|---|
| `comp n` / `basic n` | Requiere usar al menos `n` caracteres |
| `k word n` | Combinar al menos `k` palabras usando al menos `n` caracteres |
| `d class n` | Usar al menos `d` tipos de carácter (mayúsculas, minúsculas, dígitos, símbolos) con al menos `n` caracteres |

**Resultado del estudio:** políticas como `basic12` o `comp8` (longitud mínima sin requisitos de clase) resultaron ser las **más débiles** en la práctica (hasta el 50% adivinadas), mientras que políticas basadas en **múltiples clases de carácter combinadas con longitud** (`3class16`, `basic20`) resultaron sensiblemente más resistentes (por debajo del 15-20% adivinadas) frente al mismo número de intentos.

### 4.4 Buenas heurísticas para elegir contraseñas

**La fórmula de entropía revisitada:** `S = L × log₂ N`. En la práctica, la **longitud (L)** importa mucho más que el tamaño del alfabeto de símbolos (N) — añadir un carácter más aporta más seguridad que forzar símbolos especiales.

1. **Elegir contraseñas más largas:** apuntar a 16+ caracteres. No hace falta forzar mayúsculas, dígitos o símbolos complejos si la longitud ya es suficiente.
2. **Usar mnemotécnicas:** elegir una frase memorable y reducirla a la primera letra de cada palabra, creando un secreto memorizable de alta fortaleza.
3. **Insertar variaciones aleatorias:** para ampliar aún más el espacio de búsqueda, introducir algunas mayúsculas, dígitos y símbolos aleatorios en la frase, en puntos fáciles de recordar.

> **xkcd #936 — "Password Strength":** compara `Tr0ub4dor&3` (~28 bits de entropía, fácil de adivinar con las reglas típicas de los algoritmos mentales, pero difícil de recordar) frente a `correct horse battery staple` (cuatro palabras comunes al azar, ~44 bits de entropía, mucho más difícil de crackear a fuerza bruta y, paradójicamente, más fácil de recordar). Conclusión de la viñeta: "durante 20 años de esfuerzo, hemos entrenado con éxito a todo el mundo para usar contraseñas difíciles de recordar para los humanos, pero fáciles de adivinar para los ordenadores."

### 4.5 Buenos verificadores (*good verifiers*)

Recomendaciones para quien **diseña** el sistema que valida contraseñas (no solo para quien las elige):

1. **Sin pistas de contraseña:** no dar pistas que puedan ayudar a un atacante a adivinar el secreto.
2. **Rate limiting:** limitar o bloquear intentos fallidos repetidos para prevenir ataques de fuerza bruta.
3. **Permitir "pegar" (paste):** habilitar el uso de gestores de contraseñas modernos permitiendo pegar el valor en el campo.
4. **Opciones de desenmascarado visible:** permitir ver la contraseña mientras se escribe, para reducir errores de tecleo.
5. **Sin políticas de expiración forzada:** evitar exigir cambios periódicos de contraseña — la evidencia muestra que esto lleva a los usuarios a elegir secretos **más débiles** y predecibles (p. ej. incrementar un número al final).
6. **Hashing salado seguro:** almacenar las contraseñas saladas y hasheadas usando un estándar con factor de coste elevado (ver sección 3.8).

**Resumen T2.4 (Elegir contraseñas):** la entropía solo es válida para contraseñas verdaderamente aleatorias. Los algoritmos mentales son predecibles y ya están incorporados en los diccionarios de los atacantes. La longitud importa más que la complejidad de símbolos — usar frases largas (mnemotécnicas) es más efectivo que forzar reglas de composición. Un buen verificador no penaliza al usuario con expiraciones forzadas ni pistas reveladoras, y aplica salted hashing con coste adaptativo.

---

## 5. Gestores de contraseñas

### 5.1 Qué son y características comunes

Un **password manager (PM)** es una herramienta que permite a los usuarios gestionar y acceder a sus contraseñas. Almacena todas las contraseñas en una base de datos y la protege con una **contraseña maestra**. Para recuperar una contraseña (p. ej. para acceder a un servicio online), el usuario introduce la contraseña maestra para desbloquear la base de datos; en muchos casos, el propio gestor inicia sesión automáticamente por el usuario.

**Características comunes:**
- Almacenamiento cifrado.
- Base de datos de contraseñas: local vs. en la nube; sincronizada entre múltiples dispositivos.
- Standalone vs. integrado (p. ej. con el navegador).
- Generación de contraseñas y políticas de expiración.
- MFA para desbloquear el propio gestor.

### 5.2 Funcionalidades extra (diferenciación de producto)

- **"Vaults":** almacenamiento para información sensible distinta de contraseñas — identificaciones, tarjetas de crédito e información de pago, imágenes, documentos, etc.
- Servicio de alerta al visitar sitios comprometidos.
- Compartición segura de contraseñas con otra persona.
- **Travel mode:** elimina datos sensibles de los dispositivos al cruzar fronteras y los restaura al volver.

### 5.3 Ejemplos

**1Password**, **LastPass**, **Bitwarden**, **Dashlane**, **Keeper** son ejemplos conocidos en el mercado, cada uno con su propio equilibrio entre funcionalidades, modelo de negocio (open source vs. propietario) y arquitectura de almacenamiento (local vs. nube).

### 5.4 Pros y contras

En conjunto, ¿son buena idea?

| A favor (++) | En contra (--) |
|---|---|
| Pueden mejorar la seguridad y compensar las debilidades que surgen de tener demasiadas contraseñas | **Seguridad:** son un único punto de fallo (*single point of failure*) — ¿son realmente seguros? |
| Enormes beneficios de usabilidad en dispositivos con pantallas pequeñas y/o teclados multicapa (p. ej. smartphones) | **Usabilidad:** ¿se integran realmente sin fricción? |

### 5.5 Ataques a gestores de contraseñas

**El problema de la nube:** cuando el vault se sincroniza vía servicios en la nube (p. ej. Google Photos, Google Drive), errores del propio proveedor pueden filtrar datos entre cuentas no relacionadas. Un caso documentado (Google, febrero 2020): un fallo técnico exportó incorrectamente vídeos de "Descarga tus datos" de Google Photos a **archivos de usuarios no relacionados**. La lección conceptual: **"There is no cloud. It's just someone else's computer"** — sincronizar en la nube añade una superficie de confianza y de ataque adicional fuera del control directo del usuario.

> **Caso real — El ataque de autofill (2014):** Silver et al., *"Password Managers: Attacks and Defenses"*, USENIX Security Symposium 2014. Un atacante puede extraer muchas (o todas) las contraseñas guardadas en un PM si el usuario se conecta a una red WiFi maliciosa ("Evil Coffee Shop Attacker" — control temporal de un router o punto de acceso). Los 10 gestores más populares del momento resultaron vulnerables. Mecanismo: el atacante intercepta una petición HTTP (`GET papajohns.com`), redirige (`REDIRECT att.com`), inyecta JavaScript malicioso en la respuesta, y el **autofill automático** del gestor rellena las credenciales guardadas para `att.com` en el formulario controlado por el atacante — la contraseña queda robada. Consecuencia: **LastPass** dejó de autorrellenar contraseñas dentro de iFrames, y **1Password** cambió su política para páginas HTTPS servidas sobre conexiones HTTP.

> **Caso real — Bitwarden (marzo 2023):** un fallo permitía a atacantes robar contraseñas explotando **iframes**, un vector conceptualmente emparentado con el ataque de autofill de 2014 — la superficie de riesgo del autofill sigue siendo un problema recurrente casi una década después.

> **Caso real — La brecha de LastPass (2022):** un incidente inicial en agosto de 2022 resultó ser **mucho peor** de lo que la compañía pensaba en un principio. Los atacantes obtuvieron acceso extraordinario dirigiéndose a un **empleado concreto de DevOps** (spear-phishing/targeting), explotando una vulnerabilidad **ya parcheada hacía tiempo** (mayo 2020, "unas 75 versiones atrás") en un software de terceros (Plex Media Server) instalado en el **ordenador personal** de ese empleado, para implantar un keylogger. El keylogger capturó la **contraseña maestra** del empleado en el momento en que la introducía, tras autenticarse con MFA, dándoles acceso al vault corporativo de DevOps de LastPass. Con ello, los atacantes accedieron al almacenamiento en la nube de la compañía y exfiltraron copias cifradas de los vaults de contraseñas de los clientes junto con otra información personal. **Lección:** ni la MFA ni el cifrado del vault protegen si el endpoint del empleado con privilegios está comprometido — la cadena de confianza es tan fuerte como su eslabón más débil (un software de terceros sin parchear en un ordenador personal).

**Resumen T2.5 (Gestores de contraseñas):** resuelven el problema de recordar muchas contraseñas fuertes y únicas, con vaults cifrados y funcionalidades extra. Pero son un single point of failure: ataques de red (autofill 2014, iframes 2023) y ataques dirigidos a la cadena de confianza (LastPass 2022, vía un empleado y software de terceros sin parchear) muestran que ni el cifrado ni el MFA bastan si el endpoint o el proceso humano fallan.

---

## 6. Otros autenticadores (más allá de la contraseña)

### 6.1 Look-up secrets

Dispositivo físico o electrónico que almacena un secreto compartido — factor "something you have". Se usa habitualmente como **"recovery keys"** o para 2FA (p. ej. tarjetas de cuadrícula tipo Entrust, donde el usuario introduce el valor correspondiente a una coordenada indicada por el verificador).

**Requisitos:** el contenido debe generarse con un RNG adecuado y cada secreto debe tener al menos **20 bits de entropía**. Cada secreto individual del token (p. ej. cada celda de una tarjeta de cuadrícula) debe usarse **solo una vez**.

### 6.2 Out-of-band devices

Dispositivo poseído y controlado por el usuario que puede comunicarse de forma segura con el verificador a través de un **canal de comunicación secundario** (distinto del canal primario de login). Puede operar de dos formas:
- Transferir al 1er canal un secreto recibido por el 2º canal (p. ej. un código SMS que se introduce en la web).
- Transferir un ítem observado en el 1er canal al 2º canal (p. ej. escanear un código QR mostrado en pantalla).

**Limitación:** si el 2º canal es la **PSTN** (red telefónica conmutada tradicional, p. ej. SMS), las garantías de seguridad son más limitadas (interceptación, SIM swapping).

### 6.3 OTP devices (One Time Password)

| Tipo | Descripción |
|---|---|
| **Single-factor OTP device** | Hardware o software con un secreto compartido embebido. **No** requiere activación mediante otro secreto. Muestra el OTP y el usuario lo introduce manualmente en otro lugar. Prueba posesión y control del dispositivo. Similar a los look-up secrets, pero los secretos se generan criptográfica e independientemente por ambas partes. El OTP se calcula a partir de un *nonce* que puede ser **time-based** (basado en tiempo), **counter** (basado en contador) o **challenge** (basado en un reto) |
| **Multi-factor OTP device** | Dispositivo OTP **bloqueado** que necesita desbloquearse mediante un segundo factor adicional (algo que se sabe o algo que se es) antes de generar el código |

> **Ejemplo — Time-Based Token Authentication (RSA SecurID):** el `PASSCODE` que el usuario introduce es `PIN + TOKENCODE`. El *tokencode* cambia cada 60 segundos, generado a partir de una **semilla única** (unique seed) y un reloj sincronizado a UTC — de ahí que el servidor y el token puedan calcular de forma independiente el mismo valor sin necesidad de comunicación directa entre ambos.

### 6.4 Crypto software y crypto devices

| Tipo | Descripción |
|---|---|
| **Single-factor crypto software** | Clave criptográfica almacenada en disco o en algún medio "soft". La autenticación se logra demostrando la posesión de la clave, típicamente mediante algún tipo de mensaje firmado sobre un *nonce*. Se almacena en un almacenamiento adecuadamente seguro (p. ej. keychain o TPM/TEE si está disponible) |
| **Multi-factor crypto software** | Igual que el anterior, pero el acceso a la clave requiere activación mediante un segundo factor (saber o ser). Prueba tanto la posesión como el control de la clave |
| **Single-factor crypto device** | Dispositivo con claves criptográficas embebidas para realizar operaciones criptográficas. Proporciona el output del autenticador directamente (a menudo un mensaje firmado sobre un nonce proporcionado). **No** requiere un segundo factor para activarse. La clave **no debe ser exportable**. Debe requerir activación física (p. ej. pulsar un botón). Los dispositivos USB son habituales (p. ej. **YubiKey**) |
| **Multi-factor crypto device** | Igual que el anterior, pero el acceso a la clave requiere activación mediante un segundo factor. Prueba tanto la posesión del dispositivo como el control de la clave |

**Resumen T2.6 (Otros autenticadores):** más allá de las contraseñas, "something you have" se materializa en look-up secrets (tarjetas de recuperación), out-of-band devices (canal secundario), OTP devices (códigos time/counter/challenge-based) y crypto devices/software (claves embebidas, no exportables, con posible activación física o por segundo factor). Cada variante single-factor tiene su contraparte multi-factor, que añade un segundo factor de desbloqueo.

---

## 7. Biometría

### 7.1 Qué es y tipos

Rasgos biológicos basados en alguna característica física del cuerpo humano. Ejemplos: huella dactilar, geometría de la mano, retina e iris, voz, cara/cabeza, vasos sanguíneos de la mano, escritura/firma manuscrita, movimiento de la mano.

### 7.2 ¿Qué hace bueno a un rasgo biométrico?

1. **Universalidad:** toda persona debería poseerlo.
2. **Unicidad:** los individuos deben poder distinguirse por él.
3. **Permanencia:** debería ser invariante en el tiempo.
4. **Recolectabilidad / medibilidad:** la adquisición debe ser sencilla.
5. **Aceptabilidad:** los usuarios no deben tener objeciones a su uso.
6. **Rendimiento:** el nivel de precisión y la velocidad de reconocimiento deben ser adecuados.
7. **Resistencia a la circunvención:** debe ser difícil de falsificar.

### 7.3 Enrolamiento y verificación (*Enrolment and match*)

Durante el **enrolamiento**, el algoritmo extrae un **patrón** (template) que representa una medición del rasgo biométrico. Durante la **autenticación**, se toma un nuevo conjunto de mediciones y se compara (*match*) contra el template almacenado; la autenticación tiene éxito si ambos son **"suficientemente parecidos"** (no una coincidencia exacta bit a bit, a diferencia de una contraseña).

### 7.4 Ejemplos de sistemas biométricos

- **Huellas dactilares (fingerprints):** el reconocimiento se basa en minucias — *ridge ending* (fin de cresta), *enclosure* (encierro), *bifurcation* (bifurcación), *island* (isla) — y en patrones globales como Arch, Tented Arch, Whorl y Loop.
- **Mano e iris:** lectores de geometría de la mano, lectores de venas de la mano (hand vein reader), escáneres de iris.
- **Cara/cabeza (face/head):** reconocimiento facial, usado a gran escala en sistemas de videovigilancia (p. ej. sistemas de reconocimiento facial en vivo desplegados en espacios públicos para identificar personas buscadas entre una multitud).

**Resumen T2.7 (Biometría):** un buen rasgo biométrico debe ser universal, único, permanente, fácil de medir, aceptable, preciso y resistente a la falsificación. El proceso siempre es enrolamiento (crear template) + verificación (comparar y decidir "suficientemente parecido"), nunca una coincidencia exacta.

---

## 8. Amenazas a los autenticadores

Cada factor tiene su propio modelo de amenaza específico:

| Factor | Amenazas |
|---|---|
| **Something you know** | Puede ser **divulgado** (compartido voluntariamente), **adivinado**, **observado** o **interceptado** mientras se introduce; revelado por el propio usuario mediante *phishing* o ingeniería social |
| **Something you have** | Puede **perderse**, **dañarse**, **robarse** o **clonarse** |
| **Something you are** | Puede ser **replicado** (p. ej. huellas falsas, deepfakes para reconocimiento facial) |

**Idea clave:** ningún factor es inmune por sí solo — esta es la razón última por la que MFA combina factores de naturaleza distinta: comprometer un secreto memorizado (phishing) no da acceso al atacante si también hace falta un dispositivo físico robado o un rasgo biométrico replicado, y viceversa.

**Resumen T2.8:** cada factor tiene un vector de compromiso propio (divulgación/adivinación para "saber", pérdida/robo/clonación para "tener", replicación para "ser"). MFA reduce el riesgo combinando factores con amenazas independientes entre sí.

---

## 9. Autenticación en sistemas distribuidos

### 9.1 Identidad federada y Single Sign-On (SSO)

La **identidad federada** es la unión de sistemas de identificación y autenticación separados. **Objetivo:** reducir la fatiga del usuario asociada a gestionar múltiples autenticadores.

- **Federated identity system:** un único perfil con un único método de autenticación, compartido por una federación de apps y sistemas.
- **Single sign-on (SSO):** un único módulo de gestión de identidad reemplaza la identificación y autenticación en todos los sistemas — el usuario se autentica una vez ante un *SSO Shell*, que reparte contraseñas/tokens a cada aplicación sin que el usuario tenga que volver a introducir credenciales.

### 9.2 Federaciones: IdP y RP

En un esquema de federación intervienen tres actores:

- **Subscriber:** se comunica tanto con el **IdP** (Identity Provider) como con la **RP** (Relying Party), a menudo a través del navegador.
- **IdP y RP se comunican entre sí** a través del **front channel** (usando redirecciones del subscriber) o del **back channel** (directamente entre servidores).
- El IdP **asevera** (asserts) los resultados de autenticación y los atributos de identidad ante la RP.

### 9.3 Assertions

Una **assertion** es un conjunto empaquetado de atributos asociados a un subscriber autenticado, pasado del IdP a la RP. Su uso principal es la autenticación, aunque también puede usarse para otros fines (p. ej. personalización de un sitio web).

**Debe incluir (SHALL):** subject (sujeto), issuer (emisor), audience (audiencia), issuance (emisión), expiration (expiración), identifier (identificador), signature (firma) y authentication time (momento de autenticación). Opcionalmente puede incluir otros atributos (info del subscriber, dispositivo, ubicación, etc.). Se envía por **canales autenticados seguros** y va **siempre firmada** por el IdP (mediante criptografía de clave pública) o autenticada con MAC usando una clave compartida.

### 9.4 Presentación de assertions: back-channel vs. front-channel

**Back-channel presentation:**
1. El IdP envía una **referencia** a la assertion al subscriber por el front channel.
2. El subscriber reenvía esa referencia a la RP por el front channel.
3. La RP presenta la referencia al IdP por el **back channel**; el IdP valida las credenciales y devuelve la assertion completa directamente a la RP.

| Ventajas | Desventajas |
|---|---|
| Información limitada solo a las partes que la necesitan; la RP espera la assertion del IdP, por lo que la superficie de ataque se reduce | Más transacciones de red |

**Front-channel presentation:**
1. El IdP envía la **assertion completa** al subscriber por el front channel.
2. El subscriber reenvía la assertion completa a la RP, también por el front channel.

| Ventajas | Desventajas |
|---|---|
| Menos transacciones de red; más difícil para la RP consultar al IdP por atributos adicionales | La assertion queda bajo el control del subscriber (visible para él), lo que puede causar filtraciones (p. ej. *profiling* de usuario) si se redirige a partes no previstas, incluso con la audiencia restringida |

En **ambos** casos, la assertion debe validarse correctamente: verificación del emisor, verificación de la firma, validación temporal y restricción de audiencia.

### 9.5 Ejemplos: SAML y OIDC

**SAML (Security Assertion Markup Language):** framework basado en XML para crear e intercambiar información de autenticación y de atributos a través de Internet. Tres componentes principales:
- El **esquema XML de assertions**, que define la estructura de una assertion.
- Los **protocolos SAML**, usados para solicitar assertions y referencias a assertions.
- Los **bindings**: protocolos de transporte subyacentes (p. ej. HTTP o SOAP).

**OpenID Connect (OIDC):** capa de autenticación construida sobre **OAuth 2.0** y **JOSE** (JSON Object Signing and Encryption). El IdP emite:
- Un **ID Token** = una assertion firmada en formato **JWT** (JSON Web Token).
- Un **token de acceso OAuth** estándar, que se puede usar ante el IdP para obtener un JSON con información completa sobre el usuario.

La RP parsea el ID Token y usa el token de acceso OAuth cuando necesita más información del usuario.

### 9.6 Requisitos de privacidad en la federación

La federación implica la transferencia de atributos personales desde un tercero (el IdP) que, en principio, no participa directamente en la transacción entre subscriber y RP — esto le da al IdP **visibilidad sobre las actividades del subscriber**.

- Los CSP a menudo tienen propósitos de negocio distintos de la mera provisión de identidad.
- El **back channel** revela al IdP **dónde** está el subscriber realizando una transacción; si el mismo IdP sirve a **múltiples RPs**, esto habilita **profiling y tracking** del usuario a través de sitios distintos.
- Existe un trade-off entre privacidad y otros objetivos legítimos, como la mitigación de fraude o el cumplimiento de procesos legales.

**Resumen T2.9 (Sistemas distribuidos):** SSO/federación resuelven la fatiga de gestionar múltiples credenciales delegando la autenticación en un IdP. Las assertions (firmadas, con subject/issuer/audience/expiration) viajan del IdP a la RP por front-channel (menos tráfico, más expuestas al subscriber) o back-channel (más tráfico, menos superficie de exposición). SAML (XML) y OIDC (JWT sobre OAuth2) son las implementaciones de referencia. El coste de la federación es la visibilidad que el IdP gana sobre la actividad del subscriber en múltiples RPs.

---

## 10. Tendencias (lecturas adicionales señaladas en el módulo)

- **Passwordless authentication:** eliminar la contraseña como factor, apoyándose en **FIDO2/WebAuthn** y en **passkeys** (Google, Apple) — credenciales criptográficas resistentes a phishing que sustituyen al par usuario/contraseña por autenticadores de tipo crypto device (sección 6.4) ligados al dispositivo del usuario.
- **Continuous authentication:** en vez de autenticar una única vez al inicio de la sesión, verificar la identidad de forma continua a lo largo de toda la sesión (patrones de comportamiento, biometría pasiva), en línea con los principios de arquitecturas **Zero Trust**.

---

## 11. Preguntas de autoevaluación

**1. ¿Por qué la autorización (AuthZ) no tiene sentido sin autenticación (AuthN) previa?**
> Porque AuthZ decide qué puede hacer un sujeto (permisos, capacidades, control de acceso), y esa decisión depende por completo de saber primero quién es ese sujeto. Sin verificar la identidad no hay base sobre la que aplicar ningún permiso.

**2. ¿Cuál es la diferencia esencial entre identificación y autenticación?**
> La identificación es afirmar quién eres usando datos públicos y predecibles (email, teléfono, nombre de usuario). La autenticación es probar esa identidad afirmada mediante algo privado (contraseña, huella, token). El identificador no es secreto; el autenticador sí debe serlo.

**3. ¿Por qué las contraseñas hasheadas siguen siendo vulnerables al cracking, si la función hash es prácticamente imposible de invertir matemáticamente?**
> Porque el ataque no intenta invertir la función hash, sino aprovechar que las personas eligen contraseñas predecibles: se generan candidatos plausibles (fuerza bruta, diccionarios, rainbow tables), se hashean con la misma función, y se comparan los hashes resultantes contra los robados. El problema no es la función hash, es la baja entropía real de las contraseñas humanas.

**4. Explica por qué el salting por sí solo no es suficiente y hace falta también key stretching (adaptive hashing).**
> El salt evita que dos usuarios con la misma contraseña tengan el mismo hash y anula las rainbow tables precomputadas genéricas, pero no ralentiza el cálculo de cada hash individual. Un atacante con GPUs modernas puede seguir probando miles de millones de candidatos por segundo contra un hash salado si la función es rápida (como MD5 o SHA1 sin adaptar). El key stretching (bcrypt, PBKDF2, scrypt) introduce un factor de trabajo configurable que hace cada intento deliberadamente lento, reduciendo drásticamente el ritmo de prueba del atacante.

**5. Según la fórmula de entropía `S = L × log₂ N`, ¿por qué se recomienda priorizar la longitud sobre la complejidad de símbolos?**
> Porque el crecimiento de la entropía es lineal en `L` pero solo logarítmico en `N`: añadir un carácter más a la contraseña aporta más bits de entropía que ampliar el alfabeto de símbolos permitido (mayúsculas, dígitos, símbolos). Además, forzar símbolos complejos empuja a los usuarios hacia algoritmos mentales predecibles (sección 4.2), mientras que una frase larga (mnemotécnica) es más fácil de recordar y más difícil de crackear.

**6. ¿Por qué los "algoritmos mentales" tradicionales para elegir contraseñas (mayúscula + número al final, sustituir letras por símbolos) ya no se consideran seguros?**
> Porque, al ser procedimientos populares enseñados durante años en formación de seguridad, los atacantes ya los conocen y los han incorporado a sus diccionarios y reglas de generación de candidatos. Una contraseña generada siguiendo un patrón conocido (p. ej. `Cl4ssr00m`) no es aleatoria, aunque lo parezca a simple vista, y cae rápidamente ante un diccionario con reglas de mutación.

**7. Según el estudio de Shay et al. (ACM TISSEC 2016), ¿qué tipo de política de contraseñas resultó más débil frente al cracking?**
> Las políticas basadas únicamente en longitud mínima sin requisitos de diversidad de caracteres (`basicN`/`compN`, p. ej. basic12), que permitieron adivinar hasta el 50% de las contraseñas. Las políticas que combinan múltiples clases de carácter con mayor longitud (p. ej. `3class16`) resultaron sensiblemente más resistentes.

**8. Da tres recomendaciones para diseñar un buen verificador de contraseñas y explica por qué las políticas de expiración forzada son contraproducentes.**
> Por ejemplo: permitir pegar la contraseña (compatibilidad con gestores), aplicar rate limiting contra fuerza bruta, y almacenar con hash salado y coste adaptativo. Las políticas de expiración forzada son contraproducentes porque, en la práctica, empujan a los usuarios a elegir contraseñas más débiles y predecibles (p. ej. incrementar un número al final de la contraseña anterior) en lugar de generar secretos realmente nuevos y fuertes.

**9. ¿Qué papel jugó exactamente la MFA en la brecha de LastPass de 2022, y por qué no evitó el incidente?**
> El atacante capturó la contraseña maestra del empleado de DevOps con un keylogger instalado tras explotar una vulnerabilidad sin parchear en su ordenador personal, justo en el momento en que el empleado la introducía **después de** autenticarse con MFA. La MFA protegía el acceso inicial, pero una vez el endpoint estaba comprometido con un keylogger, capturar la contraseña maestra en texto claro bastó para acceder al vault corporativo — la MFA no protege contra la captura directa de credenciales en un dispositivo ya comprometido.

**10. ¿En qué consistió el ataque de autofill de 2014 contra los gestores de contraseñas, y qué cambio de diseño provocó en LastPass y 1Password?**
> Un atacante con control temporal de un punto de acceso WiFi (p. ej. una red pública) podía redirigir una petición HTTP e inyectar JavaScript malicioso; el autofill automático del gestor de contraseñas rellenaba entonces las credenciales del usuario en el formulario del atacante sin interacción explícita del usuario. Tras el estudio, LastPass dejó de autorrellenar contraseñas dentro de iFrames, y 1Password cambió su política para páginas HTTPS servidas sobre conexiones HTTP.

**11. Explica la diferencia entre un dispositivo OTP single-factor y uno multi-factor.**
> El single-factor OTP genera el código simplemente con la posesión y control del dispositivo, sin necesitar un segundo secreto para activarse. El multi-factor OTP está bloqueado y requiere desbloquearse primero con un segundo factor (algo que se sabe o algo que se es) antes de poder generar el código — añade una capa de protección si el dispositivo se pierde o es robado.

**12. ¿Por qué la clave de un "single-factor crypto device" (como una YubiKey) no debe ser exportable, y qué otra medida de seguridad suele exigirse?**
> Porque si la clave pudiera exportarse, un atacante que accediera al dispositivo (o a un backup) podría copiarla y usarla desde otro sitio, rompiendo la garantía de "algo que tienes". Además, suele exigirse **activación física** (p. ej. pulsar un botón en el dispositivo) para evitar que software malicioso en el ordenador anfitrión use la clave sin que el usuario lo note.

**13. Nombra tres propiedades que debería tener un buen rasgo biométrico y explica por qué "permanencia" es problemática para algunos rasgos.**
> Por ejemplo: universalidad, unicidad y resistencia a la circunvención. La permanencia es problemática porque algunos rasgos cambian con el tiempo o con circunstancias (una cicatriz que altera una huella dactilar, cambios en la voz por una enfermedad, envejecimiento facial), lo que puede degradar la precisión del sistema de verificación si el template de enrolamiento no se actualiza.

**14. ¿Cuál es la diferencia clave entre la presentación de assertions por front-channel y por back-channel, en términos de privacidad?**
> En front-channel, la assertion completa pasa a través del navegador del subscriber, quedando bajo su control y visible para él, lo que puede causar filtraciones si se redirige a partes no previstas (p. ej. profiling), incluso con audiencia restringida. En back-channel, solo se envía una referencia a la assertion por el front channel; la assertion completa se transmite directamente del IdP a la RP por un canal separado, limitando la información expuesta solo a quien realmente la necesita, aunque a costa de más transacciones de red.

**15. ¿Por qué el uso de un mismo IdP por parte de múltiples Relying Parties plantea un riesgo de privacidad, y qué lo hace posible técnicamente?**
> Porque cada vez que el subscriber se autentica en una RP distinta usando el mismo IdP, el IdP (especialmente si la comunicación pasa por el back channel) puede observar en qué RP y en qué momento se está autenticando el subscriber. Al repetirse esto a través de múltiples RPs, el IdP puede construir un perfil de la actividad del usuario a través de sitios distintos (*profiling & tracking*) — algo que un sistema de autenticación independiente por sitio no permitiría, porque ningún tercero vería todas esas transacciones juntas.

---

## 12. Para profundizar

El módulo remite explícitamente a estas lecturas/recursos para ampliar más allá del contenido de las diapositivas:

- **Passwordless authentication:** introducción general (Auth0), FIDO2 (fidoalliance.org), Google Passkeys y Apple Passkeys.
- **Continuous authentication:** su relación con las arquitecturas Zero Trust.

*(No hay una sección explícita de "preguntas para reflexionar" en el material original de este módulo — la sección 11 de autoevaluación cubre ese propósito con preguntas de razonamiento abierto sobre los mismos conceptos.)*
