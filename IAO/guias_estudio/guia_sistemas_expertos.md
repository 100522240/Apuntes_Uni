# Sistemas Expertos

## 1. Definición y contexto histórico

- Primeros sistemas expertos: década de **1970**; proliferan en los **años 80**.
- **Feigenbaum** (junto a otros autores) — considerado el "padre de los sistemas expertos" — los define como: programas de IA que consiguen una capacidad a nivel de experto en la resolución de problemas mediante la reproducción de un cuerpo de conocimiento (también llamados **sistemas basados en conocimiento**).
- Un sistema experto **no sustituye** a los expertos, pero permite que su conocimiento y experiencia estén disponibles para que los no expertos trabajen mejor.
- El dominio de un sistema experto debe estar **limitado** (no puede responder a cualquier pregunta — igual que un experto humano).

## 2. Cuándo utilizar un sistema experto

**Sobre la aplicación**, debe requerir:
- Conocimientos especializados.
- **Juicio** (capacidad de tomar decisiones sensatas/criterio).
- **Experiencia**.

**Sobre el problema:**
- Debe ser **heurístico** (sin solución algorítmica fácil).
- Debe estar **bien definido**.
- El área de experiencia debe estar bien definida y reconocida profesionalmente.

**Otros aspectos:**
- Necesidad de reclutar expertos cooperativos dispuestos a colaborar.
- Tamaño y complejidad manejables acorde a los recursos de la organización.
- Respaldo de la administración/dirección.

## 3. Conceptos clave

- **Experto**: persona con un alto nivel de habilidades que le permite hacer juicios en un dominio específico.
- **Pericia**: sabiduría práctica, experiencia y habilidad en una ciencia o arte. Incluye: conocimiento extenso del dominio, heurísticas que simplifican la resolución de problemas, **meta-conocimiento**, y comportamientos que mejoran el desempeño.
- **Conocimiento profundo**: conocimiento experto especializado (más allá del superficial).

## 4. Por qué construir un sistema experto

- **Preservar la experiencia** (jubilación, cambio de trabajo, escasez de expertos en un área).
- Mejorar la **productividad**.
- Hacer la experiencia **portable**.
- Obtener consejo experto que de otro modo sería imposible.

## 5. Construcción: adquisición y representación del conocimiento

1. **Adquisición del conocimiento** (*knowledge acquisition*): entrevistas y observación minuciosa del experto.
2. **Representación del conocimiento**, formas principales:
   - **Reglas de producción** (SI-ENTONCES): la forma más popular. Formato condición-acción.
     - Ejemplo: *SI el cliente requiere inversiones libres de riesgo ENTONCES debe considerar los bonos del Estado.*
   - **Marcos (frames)**: introducidos por **Minsky en 1975**. Estructuras de datos que representan situaciones estereotipadas mediante conceptos. Considerados predecesores de los **objetos** en programación orientada a objetos.
   - **Redes semánticas**: formalismo basado en relaciones (grafos) para representar conocimiento del dominio.

## 6. Componentes de un sistema experto

| Componente | Función |
|---|---|
| **Base de conocimiento** | Colección de hechos, reglas y procedimientos organizados en esquemas; contiene la información sobre el dominio (Reglas + Hechos). |
| **Motor de inferencia** | Contiene las estrategias de inferencia y controla la manipulación de la base de conocimiento y la de dominio. Recibe la consulta desde la interfaz de usuario. Es el "cerebro" del sistema experto. |
| **Interfaz de usuario** | Facilita la comunicación amigable entre usuario y ordenador. |
| **Subsistema de explicación** | Explica el razonamiento llevado a cabo y justifica las conclusiones alcanzadas. |
| **Memoria de trabajo** | Descripción del problema actual y almacenamiento de resultados intermedios. |
| **Base de datos de dominio** *(estructura alternativa)* | Información relevante sobre el dominio o área de interés. |
| **Sistema de gestión de BBDD** *(estructura alternativa)* | Controla entrada/gestión de las bases de datos de dominio y de conocimiento. |
| **Facilidad para la adquisición de conocimiento** *(estructura alternativa)* | Extracción/formulación de conocimiento derivado de fuentes (sobre todo expertos); en sistemas avanzados puede aprender de forma autónoma. |

## 7. Inferencia

- **Inferencia**: proceso de encadenar múltiples reglas basadas en los datos disponibles; lo lleva a cabo el motor de inferencia.
- **Encadenamiento hacia adelante** (*forward chaining*): búsqueda orientada por los **datos**. Si las premisas coinciden con la situación actual, se intenta afirmar la conclusión. La conclusión de una regla se convierte en un nuevo hecho que puede activar otras reglas.
- **Encadenamiento hacia atrás** (*backward chaining*): búsqueda orientada a **objetivos**. Comienza con la cláusula de acción de una regla y trabaja hacia atrás para verificar un conjunto de cláusulas.

## 8. Herramientas de desarrollo

**Proceso típico**: adquisición de conocimiento → representación del conocimiento → selección de herramientas de desarrollo → desarrollo del prototipo → evaluación → mejora/mantenimiento continuo.

**Criterios de selección de herramienta**: relación coste-beneficio, funcionalidad y flexibilidad, compatibilidad con la infraestructura existente, fiabilidad y soporte del proveedor.

**Opciones de desarrollo:**
- **Desde cero**: lenguajes de propósito general para IA — **LISP** y **Prólogo**. También lenguajes orientados a objetos (Smalltalk, Java) — dan más flexibilidad pero no orientan sobre cómo representar el conocimiento.
- **Entornos de desarrollo híbrido (shells)**: facilitan la construcción incluso a personas con poca experiencia; contienen todos los elementos esenciales de un sistema experto excepto el conocimiento específico del dominio; incluyen herramientas para interfaces de usuario.

**Ejemplos históricos de herramientas:**
- **VP-Expert**: fácil acceso a la base de conocimientos, incorporaba el algoritmo de encadenamiento hacia adelante.
- **Financial Advisor**: análisis de inversiones de capital.
- **Nexpert**: lenguaje de alto nivel, combinaba sistemas expertos + hipertexto (desarrollado en 1985).
- **Leonardo**: lenguaje orientado a objetos (Kontext), análisis de empresa/producto frente a la competencia.
- **Personal Consultant**: guía de vehículos en almacenes/plantas de manufactura.
- **EXSYS**: herramienta popular actual (Corbitt XXI), diseñada para usuarios no programadores, crea sistemas expertos en línea.

## 9. Áreas de aplicación

| Dominio | Ejemplos |
|---|---|
| **Ciencias / química** | **DENDRAL** (Feigenbaum, 1965) — deduce estructura molecular de compuestos orgánicos. |
| **Medicina** | **MYCIN** (Stanford) — diagnóstico y tratamiento de enfermedades bacterianas de la sangre. |
| **Negocios / configuración** | **XCON** (Digital Equipment Corporation, DEC) — configuración de sistemas; redujo el tiempo de procesamiento de pedidos de 25-30 min a 1 min. **XSEL**: versión orientada a ventas del mismo sistema; redujo la configuración de 3 horas a 15 minutos y bajó los errores del 30% al 1%. |
| **Finanzas** | Análisis de crédito, evaluación de seguros, prevención de fraude, evaluación de rendimiento. |
| **Marketing** | CRM, análisis y planificación de mercado. **Cover Story** — extrae información de marketing de bases de datos y redacta informes automáticamente. |
| **Recursos Humanos** | Evaluación de rendimiento, programación de personal, gestión de pensiones. |
| **Manufactura** | **ISIS-II** (Westinghouse Electric) — programación de órdenes de fabricación complejas. Planificación de producción, gestión de calidad, mantenimiento. |
| **Seguridad** | Evaluación de amenazas terroristas, detección de financiación de actividades ilícitas. |
| **Transporte/logística** | **Cargo Expert Systems** (Lufthansa) — determina la mejor ruta de envío. |
| **Telecomunicaciones** | **ACE** (AT&T) — mantenimiento de redes telefónicas. |
| **Crédito/finanzas** | **Authorizer's Assistant** (American Express) — reduce riesgos en concesión de crédito. |
| **Seguros/automoción** | **SHYSTER** *(nota: en el vídeo aparece como "skype"/similar)* (Ford Motor Company) — autorización y gestión de reclamaciones. |
| **Gestión financiera** | **Gurú** (Macro Data System) — soporte a la gerencia mediante hojas de cálculo. |

## 10. Beneficios de los sistemas expertos

- Capturan **conocimiento escaso** (útil ante jubilaciones, cambios de trabajo, falta de expertos).
- Trabajan **más rápido** que un ser humano.
- Ofrecen recomendaciones **consistentes**, reduciendo errores.
- Aumentan la **productividad** y calidad de las decisiones.
- Permiten incorporar el conocimiento de **múltiples expertos**.
- Pueden trabajar con información **incompleta, imprecisa e incierta**, como haría un humano.

## 11. Limitaciones de los sistemas expertos

- El conocimiento **no siempre está disponible**; puede ser difícil de extraer.
- Miedo de los expertos a compartir su conocimiento.
- Funcionan bien solo en un dominio **estrecho y bien definido**.
- Vocabulario técnico complejo.
- Posible **falta de confianza** del usuario final.
- A veces producen **recomendaciones incorrectas**.

## 12. Factores de éxito / fracaso

**Factores críticos de éxito:**
- Buen gestor del proyecto.
- Participación del usuario.
- Elemento de información/justificación adecuada del problema.
- Buena gestión de proyectos.
- Nivel de conocimiento suficientemente alto.
- Al menos un experto cooperativo.
- Problema principalmente **cualitativo** y de alcance limitado.
- Interfaz de usuario amigable, buena calidad en almacenamiento/manipulación del conocimiento.

**Estudio de 1995**: solo **1/3** de los sistemas expertos comerciales sobrevivía más allá de **5 años**. Causas típicas de fracaso: falta de aceptación por los usuarios, incapacidad de retener desarrolladores, problemas en la transición desarrollo→mantenimiento (falta de refinamiento), cambios en prioridades de la organización.

**Cita clave (Richard Barfus, presidente de MindBox):**
> *"Expert systems didn't really go away. They went undercover."*
> Los sistemas expertos no desaparecieron: evolucionaron y hoy se integran en **sistemas híbridos** combinados con otras técnicas de IA.

## 13. Caso: Sistema Experto para el cultivo de algodón (IoT)

*(Shahzadi, Ferzund, Tausif y Suryani, 2016 — Pakistán)*

- **Contexto**: Pakistán es un país agrícola; sufre pérdidas por siembra retrasada, peligros ambientales, plagas/enfermedades, riego no planificado, cosecha intempestiva, uso indebido de fertilizantes/insecticidas, falta de maquinaria, mal manejo de cultivos maduros.
- **Características del agricultor**: alto grado de analfabetismo → necesidad de orientación de expertos y agricultores experimentados.
- **IoT (Internet de las cosas)**: red de objetos físicos con tecnología integrada para comunicarse y detectar/interactuar con su entorno (definición de Gartner). Tecnologías clave: **RFID** y **WSN** (Wireless Sensor Network).
- **Uso del sistema propuesto**: manejo eficiente de cultivos y control de riego, advertencias/orientaciones medioambientales, uso óptimo de fertilizantes/insecticidas/pesticidas.
- **Arquitectura**: Sistema Experto basado en IoT = Despliegue de sensores + Sistema Experto + Recomendaciones.
  - **Sensores** (Waspmote agriculture sensor board): suelo, humedad, temperatura, humedad de la hoja.
  - **Comunicación**: XBee (802.15.4) hacia un Gateway, que envía los datos vía USB al servidor.
  - **Sistema experto (CLIPS)**: Base de conocimiento (reglas SI-ENTONCES) + Motor de inferencia (Agenda) + Memoria de trabajo (hechos) + Facilidad de explicación + Interfaz de usuario.
  - **Salida**: recomendación enviada al agricultor (condición del cultivo, problema detectado, tratamiento sugerido, consejo de riego, buenas prácticas).
- **Ejemplos de reglas SI-ENTONCES**:
  - Diagnóstico y tratamiento de plagas (ej. mosca blanca, trips, jassid) → recomendación de insecticidas específicos.
  - Diagnóstico y control de malezas (ej. *sedge*) → recomendación de herbicida, dosis y momento de aplicación.
  - Diagnóstico y control de gusanos en algodón (American Bollworm, Pink Bollworm, Spotted Bollworm) → recomendaciones de insecticidas según condiciones/síntomas.
  - Programación del riego según cultivo, área, temporada y textura del suelo.
- **Conclusiones del estudio**: evaluado en campo (jul-dic 2015), encuesta a 100 usuarios, **65% de aceptación**, recomendaciones sobre riego, plagas y malezas.
- **Trabajos futuros**: despliegue de actuadores en campo, mejora de servidor/escalabilidad, cámaras para diagnóstico visual.
- **Evolución 2016 → hoy**: de un sistema experto con IoT (reglas explícitas, trazables) hacia una **arquitectura híbrida de IA** que combina datos IoT + imágenes, modelos de IA/deep learning (detección de plagas), reglas/conocimiento experto e IA generativa (interfaz conversacional). *Lo nuevo no sustituye a las reglas: las complementa.*

## 14. Caso: Sistema Experto para predicción de inundaciones (Belief Rule Base)

*(Islam, Andersson y Hossain, 2015)*

- **Contexto histórico**: Gran inundación de China (1931) — 88.000 km² inundados, 80 millones sin hogar, 4 millones de fallecidos. Pantanada de Tous (Cuenca del Júcar, 20/10/1982). Inundaciones por el huracán Dorian en Bahamas.
- **Definición de inundación** (Geoscience Australia): condición general y temporal de anegación parcial o completa de áreas normalmente secas, por desbordamiento de aguas continentales o de marea debido a acumulación o escorrentía inusual y rápida.
- **Datos usados para la predicción**: cantidad de lluvia, duración de la lluvia, tasa de cambio en el flujo del río, nivel del agua del río, características de la cuenca de drenaje, actividades humanas.
- **Sistema experto basado en reglas de creencia (Belief Rule Base, BRB)**:
  - Diseñado para procesar la **incertidumbre** en datos de sensores: ignorancia, incompletitud, aleatoriedad, vaguedad, imprecisión.
  - **Base de conocimiento** = Base de reglas de creencia.
  - **Motor de inferencia** = Razonamiento probatorio (*evidential reasoning*, ER).
  - **Regla de creencia** (ejemplo): *SI Rainfall is Medium AND Rainfall Duration is High ENTONCES Meteorological Condition is {(Severe, 0), (Moderate, 0.4), (Low, 0.6)}* — nótese que la conclusión no es un valor único, sino una distribución de creencia sobre varios posibles resultados.
- **Fases del Evidential Reasoning (ER)**:
  1. Transformación de entrada.
  2. Activación de reglas.
  3. Actualización de creencias.
  4. Agregación de reglas.
- **Arquitectura del sistema (Web-BRBES)**: Input Module → Transformación → Reglas (R1...RL) → Inferencia usando ER → Configuration Module (conectividad de la base de conocimiento, configuración de reglas, configuración de inferencia ER, configuración de módulos de entrada) — todo sobre una Base de Conocimiento en MySQL y un servidor web en PHP.
- **Estructura jerárquica de la BRB** (variables): X7 = nivel de agua de inundación (variable objetivo), alimentada por factores meteorológicos (X8: lluvia inicial, lluvia prolongada), factores geológicos (X9: tipo de suelo, límite de saturación, tasa de infiltración), descarga del río (X10: profundidad, ancho, velocidad, pendiente, sedimentación), topografía (X11) y actividades humanas (X12: infraestructura no planificada, fallos de dique, deforestación, asentamientos en zonas inundables, reducción de cuencas).
- **Resultado de ejemplo**: nivel de agua de inundación previsto = **97.81 cm**, calculado agregando los valores de cada rama de la jerarquía de reglas.
- **Del BRB a la IA actual — enfoques complementarios**:
  - **BRB/Sistema experto**: conocimiento explícito + incertidumbre modelada; trazabilidad, explicabilidad, funciona con pocos datos.
  - **Machine Learning/Deep Learning**: aprendizaje a partir de datos, alta capacidad predictiva, pero menor interpretabilidad (caja negra) y requiere grandes volúmenes de datos.
  - **IA Generativa**: interacción en lenguaje natural, síntesis de información, comunicación de resultados, accesible a usuarios no expertos.
  - **Sistemas híbridos**: combinan fortalezas — las reglas justifican (explicabilidad), el ML predice (capacidad predictiva), la IA generativa comunica (interfaz y síntesis) → soluciones más robustas, precisas, trazables y accesibles.
