# Guía de estudio — Módulo 1: Introduction to Cybersecurity

*Cybersecurity Engineering — Grado en Ingeniería Informática (UC3M)*

## 0. Resumen general y objetivos

Este módulo introductorio sienta las bases conceptuales de toda la asignatura: **qué es** la seguridad informática, **qué se protege** (assets), **de qué** se protege (threats, vulnerabilities) y **cómo** se decide diseñar un mecanismo de protección (los ocho principios de diseño de Saltzer y Schroeder). No hay todavía herramientas ni ataques concretos — es el vocabulario y el marco mental (*framework*) que se reutilizará en el resto de módulos.

El módulo se divide en dos partes:
- **Part I — Computer Security Concepts:** assets, el paradigma Vulnerability-Threat-Control, la tríada CIA (Confidentiality, Integrity, Availability), Access Control, Authentication, tipos de amenazas y de atacantes, y tipos de controles.
- **Part II — Design Principles:** los ocho principios clásicos de diseño seguro (Saltzer & Schroeder, 1975), que siguen siendo la referencia estándar en la industria y en el resto del temario.

Al terminar esta guía deberías ser capaz de:

- Definir seguridad informática en términos de *assets* y su valor, y explicar por qué ese valor puede ser asimétrico entre víctima y atacante.
- Aplicar el paradigma **Vulnerability-Threat-Control** para analizar cualquier incidente de seguridad.
- Distinguir con precisión los tres pilares de la **tríada CIA** (Confidentiality, Integrity, Availability) y detectar solapes entre ellos.
- Explicar la diferencia entre *authentication* y *access control*, y por qué la primera es precondición de la segunda.
- Clasificar amenazas según la naturaleza del daño (interception, interruption, modification, fabrication) y según el tipo de atacante.
- Clasificar controles por objetivo (prevent, deter, mitigate, detect, respond, recover) y por tipo (physical, procedural, technical).
- Enumerar y justificar los ocho principios de diseño seguro (P1–P8) y aplicarlos a un caso concreto.

---

## Part I — Computer Security Concepts

## 1. ¿Qué es la seguridad informática?

> **Anécdota inicial — Emergency Alert System hijacking (11 de febrero de 2013, EE.UU.):** atacantes comprometieron el sistema de alertas de emergencia (EAS) de varias cadenas de televisión en Montana, Michigan, Wisconsin y Nuevo México, e interrumpieron la programación con un falso aviso de "los muertos se están levantando de sus tumbas y atacando a los vivos". El incidente es trivial en contenido, pero real en mecanismo: un sistema **crítico** (alertas de emergencia reales) fue comprometido porque sus credenciales por defecto no se habían cambiado. Sirve para ilustrar la idea central del módulo: casi todos los sistemas modernos dependen de ordenadores (agua, banca, transporte, salud, tráfico, alimentación...), y quien controla el sistema informático controla el servicio real que hay detrás.

**Idea de partida:** la mayoría de estos sistemas funcionan como se espera, pero **ocasionalmente fallan**, ya sea por fallos benignos (bugs, errores humanos) o por ataques maliciosos. El objetivo final de la asignatura es aprender **algunas** formas en que un sistema informático puede fallar y **cómo protegerse** frente a esos fallos — siempre que sea posible, y si no, al menos **mitigar sus efectos**.

**Definición formal:**

> La seguridad informática es la **protección** de los **activos valiosos** (*valuable assets*) de un ordenador o sistema informático.

Esta definición tiene dos componentes que hay que desmenuzar por separado: **qué es un asset** y **qué significa protegerlo**.

**Resumen 1:** la seguridad informática no es un fin en sí mismo, sino un medio para proteger activos valiosos. Todo el resto del módulo (CIA, vulnerabilidades, controles) es una forma de operacionalizar esa protección.

---

## 2. Assets (activos)

Un *asset* es cualquier elemento de valor de un sistema informático. Las diapositivas los agrupan en tres categorías:

| Categoría | Ejemplos |
|---|---|
| **Hardware** | Ordenador, dispositivos, equipamiento de red |
| **Software** | Sistema operativo, utilidades, aplicaciones comerciales, aplicaciones propias |
| **Data (datos)** | Documentos, fotos, música/audio, vídeos, email, proyectos de clase |

### Valor de los assets

- **Costes:** coste monetario independiente (cuánto cuesta comprarlo), coste de reemplazo (cuánto cuesta sustituirlo si se pierde) y coste personal (valor sentimental o de reputación, no monetizable directamente).
- **El valor puede ser asimétrico para víctima y atacante:** un dato que para el dueño vale poco puede valer mucho para quien lo roba (p. ej. una contraseña reutilizada), y viceversa (un atacante puede destruir algo de gran valor para la víctima sin obtener ningún beneficio directo, solo por sabotaje).
- **El valor puede surgir de la agregación, no de los elementos individuales:** un solo registro de cliente vale poco; una base de datos completa de un millón de clientes vale mucho más que la suma de sus partes — habilita ataques (phishing dirigido, venta de datos) imposibles con un solo registro.

> **Por qué importa:** valorar un asset correctamente es el primer paso de cualquier análisis de riesgo. Si se subestima su valor (p. ej. "solo son logs, no importan"), se subinvertirá en protegerlo; si se ignora el efecto de agregación, se protegerán los datos individuales pero no la base de datos completa que los contiene.

**Resumen 2:** los assets son hardware, software y datos. Su valor no es solo monetario (coste de reemplazo, coste personal), puede ser asimétrico entre atacante y víctima, y puede surgir de la agregación de elementos individualmente poco valiosos.

---

## 3. El paradigma Vulnerability-Threat-Control

La pregunta "¿cómo pueden ir mal las cosas?" se descompone en dos preguntas más concretas: **¿qué cosas malas le pueden pasar a los assets?** y **¿quién o qué puede causar (o permitir) que ocurran?**. La respuesta estructurada a ambas preguntas es el **Vulnerability-Threat-Control Paradigm**: un *framework* que describe cómo los activos pueden sufrir daño y cómo contrarrestar o mitigar ese daño.

| Término | Definición |
|---|---|
| **Threat (amenaza)** | Conjunto de circunstancias que tiene el potencial de causar pérdida o daño |
| **Vulnerability (vulnerabilidad)** | Debilidad en el sistema que puede ser explotada para causar pérdida o daño |
| **Attacker (atacante)** | Alguien o algo que explota una vulnerabilidad para perpetrar un ataque sobre el sistema |
| **Control / countermeasure (control / contramedida)** | Técnica o mecanismo que elimina o reduce una vulnerabilidad |

**Frase clave para el examen:** *"Controls prevent threats from exercising vulnerabilities"* — los controles impiden que las amenazas exploten las vulnerabilidades. Es la relación causal central de todo el módulo: sin vulnerabilidad no hay ataque posible por mucha amenaza que exista, y un control actúa precisamente sobre esa vulnerabilidad (no directamente sobre el atacante).

**Resumen 3:** el paradigma V-T-C da el vocabulario base de toda la seguridad informática: una amenaza (potencial) solo se materializa en daño real si existe una vulnerabilidad (debilidad) que un atacante explota; un control es lo que rompe esa cadena, actuando sobre la vulnerabilidad.

---

## 4. La tríada CIA

La tríada **CIA** (Confidentiality, Integrity, Availability) es el modelo clásico para especificar **qué** propiedad de seguridad se quiere proteger en un asset.

### 4.1 Confidentiality (confidencialidad)

**Definición:** solo las personas o sistemas autorizados pueden acceder a los datos protegidos.

Preguntas que la definición deja abiertas (y que hay que resolver en cada sistema concreto):
- ¿Quién determina qué personas o sistemas están autorizadas?
- ¿Qué significa "acceder"? ¿Leer el fichero entero? ¿Solo una parte? ¿Modificar el contenido? ¿Añadir contenido? ¿Divulgarlo a terceros?

Formalmente, la confidencialidad se implementa mediante una **(access) policy**:

```
Subject + Access mode + Object = Yes/No
   (Who)     (How)      (What)   (Decision)
```

Es decir: dado un **sujeto** (quién), un **modo de acceso** (cómo — leer, escribir, ejecutar...) y un **objeto** (qué recurso), la política de acceso decide si la operación se permite o no.

### 4.2 Integrity (integridad)

**Más difícil de precisar que la confidencialidad.** Preservar la integridad de un elemento puede significar cosas distintas según el contexto: que esté **sin modificar**, que se haya **modificado de formas autorizadas**, que se haya **modificado solo por partes autorizadas**, que sea **consistente**, etc.

Se aplica (se hace cumplir) de la misma manera que la confidencialidad: mediante un **control riguroso de quién puede acceder a qué recursos y de qué formas** — es decir, la misma política Subject+Access mode+Object, pero enfocada en modificación en lugar de en lectura.

### 4.3 Availability (disponibilidad)

Igual que la integridad, tiene significados distintos según el contexto. Un objeto o servicio se considera **disponible** si:
- Está presente en forma usable.
- Tiene capacidad suficiente para satisfacer las necesidades del servicio.
- El servicio se completa en un periodo de tiempo aceptable.

La disponibilidad **solapa con otras propiedades no puramente de seguridad**: capacidad (capacity), rendimiento (performance), tolerancia a fallos (fault tolerance) y usabilidad (usability) — las cuatro convergen en el concepto de disponibilidad, que no es un compartimento estanco sino la intersección de todas ellas.

**Idea clave para el examen:** de las tres propiedades, **confidencialidad** es la más fácil de definir con precisión (una política binaria de acceso); **integridad** y **disponibilidad** son inherentemente más difusas porque su significado depende del contexto de la aplicación concreta.

**Resumen 4:** CIA = Confidentiality (solo acceden los autorizados, formalizable como Subject+Access mode+Object), Integrity (el dato es correcto/no modificado indebidamente, se aplica con el mismo control de acceso pero sobre modificación), Availability (el servicio está utilizable a tiempo, solapa con capacidad, rendimiento, tolerancia a fallos y usabilidad).

---

## 5. Access Control y Authentication

### 5.1 Access Control

Para implementar una política de seguridad, el sistema debe **controlar todos los accesos de todos los sujetos a todos los objetos protegidos en todos los modos de acceso posibles** — ninguna excepción vale, porque un solo acceso no controlado rompe la garantía completa.

Un control de acceso **pequeño y centralizado** es fundamental para preservar confidencialidad e integridad (menos puntos de fallo, más fácil de auditar). La pregunta que las diapositivas dejan abierta y que conecta con el punto anterior es: **¿y para la disponibilidad?** — un control de acceso mal diseñado puede en sí mismo convertirse en un problema de disponibilidad (si es demasiado restrictivo o lento, deniega accesos legítimos o ralentiza el servicio).

### 5.2 Authentication

**Definición:** el proceso de descubrir (*identification*) o confirmar una identidad.

**Relación con access control:** la autenticación es **precondición** del control de acceso. No tiene sentido preguntar "¿puede este sujeto acceder a este objeto?" si primero no se sabe con certeza **quién** es el sujeto. Primero se autentica (se establece la identidad), después se autoriza (se decide qué puede hacer esa identidad).

**Resumen 5:** el control de acceso exige mediar *todos* los accesos sin excepción; la autenticación (identificar o confirmar quién es el sujeto) es el paso previo obligatorio para que ese control de acceso tenga sentido.

---

## 6. Vulnerabilities (vulnerabilidades)

**Definición:** debilidad en el sistema (en la especificación, el diseño, la implementación, los procedimientos, etc.) que puede ser explotada para causar pérdida o daño.

Los sistemas informáticos presentan vulnerabilidades típicas en varias capas:
- Autenticación débil (*weak authentication*).
- Falta de control de acceso (*lack of access control*).
- Errores en los programas (*errors in programs*, es decir, bugs explotables).
- Recursos finitos o inadecuados (*finite or inadequate resources* — p. ej. buffers limitados, capacidad insuficiente).
- Protección física inadecuada (*inadequate physical protection*).
- Y más — la lista no es exhaustiva.

**Por qué importa la variedad de capas:** una vulnerabilidad no es solo "un bug de código". Puede estar en el **diseño** (una especificación mal pensada), en la **implementación** (un error de programación concreto), en los **procedimientos** (un proceso operativo mal definido) o en lo **físico** (una puerta sin cerrar). Un análisis de seguridad completo tiene que revisar las cuatro capas, no solo el código.

**Resumen 6:** las vulnerabilidades no se limitan a errores de programación — aparecen en especificación, diseño, implementación, procedimientos y protección física. Reconocer en qué capa está una vulnerabilidad concreta determina qué tipo de control hace falta para cerrarla.

---

## 7. Threats (amenazas)

**Definición:** conjunto de circunstancias que tiene el potencial de causar pérdida o daño. Un adversario que **explota** (*exploits*) una vulnerabilidad **perpetra** (*perpetrates*) un ataque sobre el sistema — nótese la cadena: amenaza (potencial) → vulnerabilidad (debilidad concreta) → explotación → ataque (materialización real).

### 7.1 El triángulo de la amenaza

Para que una amenaza sea real (no meramente teórica), el atacante necesita simultáneamente:

```
        Threat
       /      \
 Intention   Opportunity
       \      /
      Capability
```

- **Intention (intención):** querer causar el daño.
- **Capability (capacidad):** tener los medios técnicos/recursos para causarlo.
- **Opportunity (oportunidad):** tener acceso o circunstancias que permitan intentarlo.

**Por qué importa este triángulo:** faltando cualquiera de los tres vértices, la amenaza no se materializa. Es una herramienta de análisis de riesgo: un actor con intención y capacidad pero sin oportunidad (p. ej. sin acceso a la red interna) no es un riesgo inmediato; un control puede centrarse en eliminar cualquiera de los tres vértices, no necesariamente los tres a la vez.

### 7.2 Naturaleza del daño

Las propiedades básicas de seguridad se relacionan con la **naturaleza del daño** causado:

| Tipo de daño | Descripción |
|---|---|
| **Interception (intercepción)** | Un tercero no autorizado accede al dato → rompe **confidencialidad** |
| **Interruption (interrupción)** | El servicio/dato deja de estar disponible → rompe **availability** |
| **Modification (modificación)** | El dato se altera indebidamente → rompe **integrity** |
| **Fabrication (fabricación)** | Se crea información/tráfico falso, haciéndolo pasar por legítimo → rompe **integrity** (y a menudo confidencialidad indirectamente) |

Otros términos relacionados que aparecen en la literatura y que conviene poder mapear a los cuatro anteriores: **disclosure** (divulgación, ≈ interception), **deception** (engaño, ≈ fabrication), **disruption** (≈ interruption) y **usurpation** (toma de control no autorizada del sistema).

### 7.3 Tipos de atacantes

*¿Quiénes son? ¿Qué los motiva?*

| Tipo de atacante | Motivación típica |
|---|---|
| **Individuals (individuos)** | Diversión, reto personal, venganza |
| **Organized, worldwide groups (grupos organizados globales)** | Ganancia financiera común, agenda política común |
| **Organized crime (crimen organizado)** | Fraude, extorsión, blanqueo de capitales, tráfico |
| **Terrorists (terroristas)** | El ordenador como {objetivo, método, facilitador, potenciador} de ataques |

**Idea clave:** el perfil del atacante condiciona qué controles tienen sentido. Un individuo motivado por el reto personal se disuade con dificultad técnica (deter); el crimen organizado, motivado por beneficio económico, se disuade encareciendo el ataque más allá del beneficio esperado (economía del ataque); un terrorista que usa el ordenador como *enabler* de un ataque físico requiere controles distintos a uno que ataca el ordenador como objetivo en sí mismo.

**Resumen 7:** una amenaza necesita intención + capacidad + oportunidad para materializarse. El daño que causa se clasifica en interception (confidencialidad), interruption (disponibilidad), modification y fabrication (integridad). El tipo de atacante (individuo, grupo organizado, crimen organizado, terrorista) determina su motivación y, por tanto, qué controles son efectivos frente a él.

---

## 8. Controls (controles / contramedidas)

**Definición:** acción, dispositivo, procedimiento o técnica que **elimina o reduce** una vulnerabilidad. Recordando la frase clave de la sección 3: los controles impiden que las amenazas exploten las vulnerabilidades.

### 8.1 Controles por objetivo

| Objetivo | Cómo actúa |
|---|---|
| **Prevent (prevenir)** | Bloqueando el ataque o cerrando la vulnerabilidad |
| **Deter (disuadir)** | Haciendo el ataque más difícil (no imposible) |
| **Mitigate (mitigar)** | Haciendo su impacto menos severo si ocurre |
| **Detect (detectar)** | En el momento en que ocurre, o algún tiempo después |
| **Respond (responder)** | Gestionando el incidente activamente |
| **Recover (recuperar)** | Restaurando el sistema tras el impacto |

**Por qué importa esta secuencia:** no todos los controles actúan en el mismo momento del ataque. `Prevent`/`Deter` actúan **antes**, `Detect` actúa **durante**, `Respond`/`Recover` actúan **después**. Un programa de seguridad maduro no confía solo en la prevención (que puede fallar) — necesita capacidad de detección y respuesta, porque asume que tarde o temprano un control preventivo fallará.

### 8.2 Controles por tipo

| Tipo | Ejemplos |
|---|---|
| **Physical (físico)** | Cerraduras, guardias humanos, sprinklers (contra incendios) |
| **Procedural (procedural / administrativo)** | Leyes, políticas, procedimientos, contratos, copyright, patentes |
| **Technical (técnico)** | Contraseñas, cifrado, firewalls, IDS, controles del sistema operativo |

### 8.3 Defense in depth (defensa en profundidad)

**Concepto:** *overlapping controls* — más de un control, y más de una clase de controles, protegiendo el mismo asset simultáneamente.

> **Por qué importa:** ningún control individual es perfecto. Combinar controles de tipos distintos (físico + procedural + técnico) significa que el fallo de uno no deja el asset completamente desprotegido — es la misma lógica que "no pongas todos los huevos en la misma cesta", aplicada a mecanismos de protección en vez de a activos. Es el principio que conecta esta sección con el resto del módulo: ningún principio de diseño (Part II) sustituye a los demás, se combinan.

**Resumen 8:** los controles se clasifican por objetivo (prevent, deter, mitigate, detect, respond, recover — cubriendo antes, durante y después del ataque) y por tipo (physical, procedural, technical). La defensa en profundidad combina controles solapados de distintos tipos para que el fallo de uno no comprometa el asset completo.

---

## Part II — Design Principles

## 9. Los ocho principios de diseño seguro (Saltzer & Schroeder)

Estos ocho principios (**P1–P8**) son un clásico de la ingeniería de seguridad (formulados originalmente por Saltzer y Schroeder en 1975) y siguen siendo el marco de referencia para diseñar cualquier mecanismo de seguridad, desde un sistema operativo hasta una API. Se derivan de **dos reglas** subyacentes:

| Regla | Justificación |
|---|---|
| **Simplicity (simplicidad)** | Menos cosas pueden ir mal; menos inconsistencias posibles; más fácil de entender |
| **Restriction (restricción)** | Minimizar el acceso; inhibir la comunicación innecesaria |

Casi todos los ocho principios son una aplicación concreta de una de estas dos reglas — por eso conviene tenerlas presentes como "el porqué" detrás de cada P.

### P1 — Least Privilege (mínimo privilegio)

Un sujeto debe recibir **únicamente** los privilegios necesarios para completar su tarea.
- El control se basa en la **función**, no en la identidad ("¿qué necesita hacer este proceso/usuario?", no "¿quién es?").
- Los derechos se añaden cuando se necesitan y se descartan después de usarlos.
- Resultado: **protección mínima** por defecto — el daño potencial de una cuenta comprometida queda acotado a lo estrictamente necesario para su función.

### P2 — Fail-Safe Defaults (valores por defecto seguros ante fallo)

- La acción por defecto es **denegar** el acceso (no conceder).
- Si una acción falla, el sistema debe quedar **tan seguro como estaba antes** de que la acción comenzara — un fallo nunca debe dejar el sistema en un estado más permisivo del que tenía.

### P3 — Economy of Mechanism (economía de mecanismo)

- Mantenerlo **lo más simple posible** — el principio **KISS** (*Keep It Simple, Stupid*) aplicado a seguridad.
- Simple significa que hay menos cosas que puedan fallar, y cuando ocurren errores, son más fáciles de entender y de corregir.
- Especialmente crítico en **interfaces e interacciones** — es donde la complejidad suele esconder los fallos más peligrosos.

### P4 — Complete Mediation (mediación completa)

- **Comprobar cada acceso**, sin excepción.
- El error típico es comprobar el acceso **una sola vez**, en la primera acción (p. ej. algunos UNIX comprueban permisos al abrir el fichero, pero no vuelven a comprobarlos después).
- Riesgo: si los permisos cambian **después** de esa primera comprobación, un sujeto puede acabar con acceso no autorizado que ya no debería tener, porque el sistema nunca vuelve a verificar.

### P5 — Open Design (diseño abierto)

- La seguridad **no debe depender** del secreto del diseño o la implementación.
- Es un principio **popularmente malinterpretado** como "el código fuente debe ser público" — no es eso exactamente.
- El secreto **puede** aportar algo de seguridad adicional (*might enhance security*), pero **si el diseño queda expuesto, la seguridad del mecanismo no debe verse comprometida** por ello.
- Es la base del concepto de **"security through obscurity"** (seguridad por oscuridad) como antipatrón: un sistema que solo es seguro mientras nadie conoce su diseño no es realmente seguro.

### P6 — Separation of Privilege (separación de privilegios)

- *"With great power comes great... danger!"* — el privilegio/autoridad debe repartirse entre **más de una entidad** para prevenir fraude, sabotaje, robo y otros usos indebidos.
- **Ejemplo clásico:** quien firma los cheques no puede ser quien los imprime, porque si no, una sola persona podría robar dinero por sí sola. Con la separación, el defraudador tiene que **comprometer a dos personas**, no a una — el coste del ataque se multiplica.

### P7 — Least Common Mechanism (mínimo mecanismo común)

- Los mecanismos (es decir, los recursos) **no deberían compartirse** entre sujetos distintos si se puede evitar.
- Razón: la información puede fluir a través de canales compartidos — son los llamados **covert channels** (canales encubiertos), vías de fuga de información que no fueron diseñadas como tales pero que existen precisamente porque el recurso es compartido.
- Mecanismo práctico para aplicarlo: **isolation** (aislamiento) — máquinas virtuales, *sandboxing*.

### P8 — Acceptability (aceptabilidad)

- Los mecanismos de seguridad **no deben añadir dificultad** al acceso legítimo a un recurso.
- Hay que **ocultar la complejidad** que introduce el propio mecanismo de seguridad al usuario final.
- Importa la **facilidad de instalación, configuración y uso**.
- Los **factores humanos son críticos** aquí: factores psicológicos, creencias personales — si un control de seguridad es demasiado incómodo, los usuarios lo evitarán o lo desactivarán, anulando su efecto.

> **Por qué importa P8 en particular:** conecta con P3 (economía de mecanismo) pero desde el punto de vista del usuario, no del diseñador. Un mecanismo técnicamente perfecto pero inaceptable en la práctica (demasiado lento, demasiado confuso) termina siendo eludido — la seguridad "de papel" que nadie sigue es peor que ninguna, porque genera una falsa sensación de protección.

**Resumen 9:** los ocho principios se derivan de simplicidad y restricción. P1 (mínimo privilegio) y P6 (separación de privilegios) limitan cuánto poder tiene cada entidad; P2 (fail-safe) y P4 (mediación completa) garantizan que el control se aplica siempre y de forma segura por defecto; P3 (economía de mecanismo) y P7 (mínimo mecanismo común) reducen la superficie de fallo; P5 (diseño abierto) evita depender del secreto; P8 (aceptabilidad) reconoce que un control que el usuario evita no protege nada.

---

## 10. Epílogo — ideas clave del módulo

- **Simplicity** — apuntar siempre a la mínima complejidad posible.
- **Restrictiveness** — tensión permanente con la idea de que *"information wants to be free"* (la información tiende a fluir y compartirse; restringirla exige esfuerzo activo y continuo).
- **Understand** (entender) — antes de diseñar cualquier mecanismo hace falta comprender:
  - los **objetivos** de la seguridad propuesta (¿qué se quiere proteger y de qué?), y
  - el **entorno** en el que esos mecanismos se van a desarrollar y desplegar (no existe un control universalmente correcto, depende del contexto).

---

## 11. Preguntas de autoevaluación

**1. ¿Por qué la seguridad informática se define en términos de "assets" y no simplemente como "proteger el ordenador"?**
> Porque lo que realmente importa proteger no es el hardware en sí, sino el valor que representa: hardware, software y datos, cuyo valor puede ser monetario, de reemplazo o personal/sentimental, y no siempre coincide con lo que "parece" más importante a simple vista.

**2. Explica con un ejemplo por qué el valor de un asset puede ser asimétrico entre víctima y atacante.**
> Una contraseña reutilizada en varios servicios puede tener poco valor percibido para su dueño (es "solo una contraseña más"), pero para un atacante que la reutiliza contra otras cuentas de la misma persona (*credential stuffing*) su valor es mucho mayor, porque abre acceso a múltiples sistemas.

**3. Completa la frase clave del paradigma Vulnerability-Threat-Control: "Controls ___ ___ from exercising ___".**
> "Controls **prevent** **threats** from exercising **vulnerabilities**" — los controles actúan sobre la vulnerabilidad para impedir que la amenaza la explote; no actúan directamente "sobre" el atacante.

**4. De los tres pilares de la tríada CIA, ¿cuál es el más fácil de formalizar y por qué?**
> La confidencialidad, porque se puede expresar como una política binaria explícita: Subject + Access mode + Object = Yes/No. Integridad y disponibilidad son más difusas porque su significado ("sin modificar", "modificado de forma autorizada", "disponible en tiempo aceptable"...) depende del contexto concreto de cada sistema.

**5. ¿Por qué la disponibilidad "solapa" con otras propiedades como el rendimiento o la tolerancia a fallos?**
> Porque un servicio solo se considera disponible si está presente en forma usable, tiene capacidad suficiente y responde en un tiempo aceptable — condiciones que dependen directamente de capacity, performance, fault tolerance y usability. No es una propiedad aislada, sino la intersección de esas cuatro.

**6. ¿Por qué se dice que la autenticación es "precondición" del control de acceso?**
> Porque una política de acceso decide si un sujeto puede operar sobre un objeto, y esa decisión no tiene sentido si primero no se sabe con certeza qué identidad tiene ese sujeto. Primero se identifica/confirma la identidad (authentication), después se decide qué puede hacer esa identidad (access control).

**7. Nombra las cuatro capas típicas en las que puede residir una vulnerabilidad.**
> Especificación, diseño, implementación y procedimientos (además de la protección física como categoría adicional mencionada en las diapositivas). Una vulnerabilidad no es solo "un bug de código": puede venir de un diseño mal pensado o de un proceso operativo deficiente.

**8. Describe el "triángulo de la amenaza" y qué implica que falte uno de sus vértices.**
> Una amenaza necesita intención (querer causar el daño), capacidad (tener los medios técnicos) y oportunidad (tener acceso o circunstancias favorables) simultáneamente. Si falta cualquiera de los tres, la amenaza no se materializa en un ataque real — es una herramienta de análisis de riesgo para saber en qué vértice actuar con un control.

**9. Relaciona cada tipo de daño (interception, interruption, modification, fabrication) con la propiedad CIA que rompe.**
> Interception rompe confidencialidad (acceso no autorizado a datos); interruption rompe availability (el servicio deja de estar disponible); modification rompe integrity (alteración indebida del dato); fabrication rompe integrity, generando información falsa que se hace pasar por legítima (y puede comprometer confidencialidad indirectamente).

**10. ¿Por qué el crimen organizado y un individuo motivado por el reto personal requieren estrategias de disuasión distintas?**
> Porque sus motivaciones son distintas: el crimen organizado busca beneficio económico, así que se disuade encareciendo el ataque por encima del beneficio esperado; un individuo motivado por el reto o la diversión puede disuadirse simplemente aumentando la dificultad técnica, sin que el cálculo económico sea el factor determinante.

**11. Explica la diferencia entre un control que "previene" (prevent) y uno que "mitiga" (mitigate) una amenaza.**
> Prevenir actúa antes del ataque, bloqueándolo o cerrando la vulnerabilidad que lo haría posible, de forma que el ataque no llega a producirse. Mitigar asume que el ataque puede ocurrir (o ya ha ocurrido) y actúa para que su impacto sea menos severo, sin necesariamente impedirlo.

**12. ¿Qué es la "defensa en profundidad" y por qué ningún control individual basta por sí solo?**
> Es el uso de controles solapados —más de uno, y de más de un tipo (físico, procedural, técnico)— sobre el mismo asset. Ningún control es perfecto; si solo hay uno y falla, el asset queda completamente desprotegido. Combinando tipos distintos, el fallo de uno no compromete el conjunto.

**13. Un sistema UNIX comprueba los permisos de un fichero solo al abrirlo, no en cada operación posterior. ¿Qué principio de diseño viola y qué riesgo introduce?**
> Viola P4 — Complete Mediation (mediación completa), que exige comprobar cada acceso sin excepción. El riesgo es que si los permisos cambian después de la apertura inicial, el proceso puede seguir operando con un nivel de acceso que ya no le correspondería, porque el sistema nunca vuelve a verificar.

**14. ¿Por qué "security through obscurity" se considera un antipatrón según el principio de Open Design (P5)?**
> Porque P5 exige que la seguridad no dependa del secreto del diseño: el secreto puede aportar algo, pero si el diseño se expone, el mecanismo debe seguir siendo seguro. Un sistema cuya seguridad depende únicamente de que nadie conozca su funcionamiento deja de estar protegido en cuanto ese secreto se filtra — y los secretos de diseño, tarde o temprano, se filtran.

**15. En el ejemplo clásico de "quien firma los cheques no puede ser quien los imprime", ¿qué principio se aplica y qué consigue exactamente?**
> Se aplica P6 — Separation of Privilege (separación de privilegios). Consigue que un único individuo no pueda cometer el fraude por sí solo: el atacante tendría que comprometer a dos personas distintas en vez de una, lo que multiplica el coste y la dificultad del ataque.

---

## 12. Nota sobre el material original

Este módulo, a diferencia de otros de la asignatura, no cierra con una sección explícita de "Preguntas para reflexionar" en las diapositivas originales — termina con el **Epílogo** (sección 10 de esta guía). Si en clase se plantean preguntas abiertas sobre este módulo, es útil apoyarse en:
- El paradigma **Vulnerability-Threat-Control** (sección 3) para analizar cualquier incidente propuesto (identificar la vulnerabilidad, la amenaza y el control que habría roto la cadena).
- La tríada **CIA** (sección 4) para clasificar qué propiedad concreta se vio comprometida en un caso dado.
- Los **ocho principios de diseño** (sección 9) para justificar por qué un mecanismo de seguridad concreto fue bien o mal diseñado.
