# Guía de Estudio: Inteligencia Artificial en las Organizaciones

## Índice
1. [Redes de Neuronas Artificiales (RNA)](#redes-de-neuronas-artificiales-rna)
2. [Sistemas Expertos](#sistemas-expertos)

---

# Redes de Neuronas Artificiales (RNA)

## 1. Historia y origen

Las RNA surgen en los años **40** con el objetivo de crear sistemas artificiales que modelen el funcionamiento del cerebro.

| Año | Autor(es) | Aportación |
|---|---|---|
| 1943 | **McCulloch y Pitts** | Primer modelo matemático de neurona artificial. Basado en impulsos binarios. Introduce la función de paso por umbral. Sin capacidad de aprendizaje. |
| 1949 | **Hebb** | Dos ideas clave (base psicofisiológica): (1) el aprendizaje se localiza en las sinapsis; (2) la información se representa mediante neuronas activas/inactivas. Aportación: **aprendizaje hebbiano**. |
| 1957 | **Rosenblatt** | Generaliza el modelo anterior añadiendo aprendizaje → crea el **perceptrón**, la red neuronal más antigua. Introduce una capa oculta (3 niveles) pero no logra entrenarla. Limitación: no resuelve problemas no separables linealmente (ej. XOR). |
| 1969 | **Minsky y Papert** | Crítica severa al perceptrón por su naturaleza lineal → caída de la investigación ("época gris" durante los 70). |
| Años 80 | **Grupo PDP** (Rumelhart, McClelland, Hinton) | Procesamiento Paralelo Distribuido. Resurgimiento del interés en RNA. |
| 1982 | **Hopfield** | Función de energía para redes monocapa en tiempo discreto. Entrenar = reducir la energía de los estados. |
| 1984+ | **Kohonen** | Memorias asociativas, matrices de correlación. Aportación destacada: **mapas auto-organizativos (SOM)** — no supervisados, usan función de vecindad, rejilla 2D (rectangular/hexagonal). También el **LVQ** (aprendizaje competitivo). |
| — | **Grossberg y Carpenter** | Teoría de la **Resonancia Adaptativa (ART)**: redes no supervisadas capaces de seguir aprendiendo en uso. |
| 1992 | **LeCun** | Redes neuronales **convolucionales** (deep learning), refinadas en 2012 por el grupo de **Ciresan**. |

**Idea clave:** Las RNA tienen un origen relativamente antiguo (años 40) pero se usan de forma extensa hoy en día; siguen siendo objeto de investigación activa.

### Evolución hacia la IA actual
Perceptrón (modelo lineal) → Perceptrón multicapa/MLP (red feedforward) → Deep Learning (redes profundas) → Arquitecturas especializadas (CNN, RNN/LSTM, Transformers) → IA Generativa (modelos de lenguaje, generación de imágenes/audio, modelos multimodales).
**Los modelos actuales de IA generativa son una evolución de las redes neuronales, no una tecnología independiente.**

---

## 2. Conceptos básicos

- **Neurona artificial**: unidad de proceso de una o varias entradas con una o varias salidas.
- **Pesos**: definen el impacto de una entrada sobre la neurona de la capa siguiente.
  - Peso **positivo** → excitación.
  - Peso **negativo** → inhibición.
- El **conocimiento** se almacena en los **pesos de las conexiones (sinapsis)**, no en la distribución de las capas ocultas.
- **Sinapsis**: punto de conexión/unión entre dos neuronas (no es "un componente" que envía señales, es la conexión en sí).
- **Arquitectura o patrón de interconectividad**: estructura de interconexión entre neuronas (ej. feedforward).
- El **cerebro** funciona con cómputo **masivamente paralelo** (las neuronas biológicas son más **lentas** que los circuitos eléctricos, pero el paralelismo compensa).

### Arquitecturas más comunes (Feed forward vs Feed backward)
- **Feed forward**: Single layer perceptron, Multi layer perceptron, Radial Basis Function network.
- **Feed backward**: Bayesian Regularised NN (BRANN), Kohonen's SOM, Hopfield networks, Competitive networks, ART models.

---

## 3. Fases de trabajo con una RNA

1. **Construir** la red (diseñarla y entrenarla).
2. **Validar** dicha RNA.
3. **Utilizarla** con datos que la RNA **no ha visto antes** (para comprobar su capacidad de generalización).

## 4. Tipos de aprendizaje

- **Supervisado**: los datos de entrenamiento son pares (entrada, salida deseada). Se ajustan los pesos según la diferencia entre salida real y deseada (incremento positivo o negativo, no siempre positivo).
- **No supervisado**: solo se dispone de los patrones de entrada (sin salida asociada). Ej: SOM para clustering/segmentación de clientes.

## 5. Parámetros de diseño de una RNA

- **Arquitectura**: nº de neuronas de entrada (una por variable/atributo), nº de capas ocultas y neuronas por capa (regla habitual: el doble que la entrada si no se tiene claro), nº de neuronas de salida.
- **Entrenamiento**: algoritmo de aprendizaje (ej. retropropagación/backpropagation), tipo de aprendizaje (supervisado/no supervisado), % de división train/test (ej. 60/40, 50/50), nº de ciclos/épocas (típico 100–500).
- **Decisión/clasificación**: valor umbral (threshold) para clasificar la salida en una categoría u otra.

## 6. Ventajas y desventajas

**Ventajas:**
- Resultados excelentes en clasificación, representación y predicción (tasas de acierto 98-99%).
- Modelan y predicen comportamientos **no lineales** (a diferencia de muchos modelos estadísticos clásicos).
- **Robustas**: manejan datos incompletos, ruido, e incluso ejemplos contradictorios.
- Adecuadas para el procesamiento de información financiera y de negocios.

**Desventaja principal — Síndrome de "caja negra":**
- Las RNA no son capaces de explicar qué información les ha llevado a tomar una decisión concreta. (No confundir con la capacidad de aprender con datos incompletos, que es una ventaja distinta).

## 7. Líneas de investigación actuales

- **Conjuntos de clasificadores / mezcla de expertos (ensembles)**: combinar varias RNA suele dar mejores resultados que un clasificador individual.
- **Nuevos algoritmos de aprendizaje** (más allá de backpropagation, incluidas versiones más rápidas).
- **Diseño automático de arquitecturas**: para reducir la necesidad de conocimiento experto.
- **Nuevos modelos**: redes convolucionales / deep learning.

## 8. Aplicaciones y casos reales

### Ámbito financiero y empresarial
- **Banco Mellon**: sistema de detección de fraude basado en RNA; recuperó la inversión en **6 meses** gracias al ahorro en fraudes detectados.
- **Visa y Mastercard**: monitorizan más de **12 millones de cuentas** diariamente con RNA.
- **Ernst & Young**: uso en gestión (consultora de alto nivel).
- Otras aplicaciones: predicción de ganancias, momentos óptimos para auditorías, gestión de estrategias operativas.

### Predicción de bancarrotas
- Se combinan RNA con modelos clásicos, como el **modelo de Altman** (1968, análisis multivariante), para potenciar la capacidad predictiva.
- Hitos clásicos: Beaver (1966, técnicas estadísticas), Altman (1968, referencia), Deakin (1972, mezcla multivariante + tendencias), Freeman/Altman/Ko (1985, particionamiento recursivo).
- **Caso 1994/1995**: 20 empresas (~20M$, 10 solventes / 10 en quiebra), 18 parámetros (razones financieras: liquidez, solvencia/apalancamiento, endeudamiento, cobertura, actividad, márgenes, rentabilidad) con datos de 2 años.
  - Salida: valor entre 0 y 1, umbral = **0.4**.
  - Resultado: 3 tipos de empresas → saludables, en quiebra, **en riesgo** (tercer tipo, de gran interés).
  - Acierto del 100% en empresas que quebraron; error del 30% en empresas identificadas como riesgo que no quebraron realmente.

### Perfil de solvencia bancaria (concesión de hipotecas)
- Perceptrón multicapa, algoritmo backpropagation.
- Entradas: datos del cliente (ingresos, créditos, compras, tarjeta, descubiertos, profesión, nº hijos, estado civil...).
- División de datos: 60% entrenamiento / 40% test.
- Salida: solvencia del cliente.

### Marketing dirigido por datos (data-driven marketing)
- RNA **no supervisadas** para segmentar clientes (más eficientes que clustering clásico, ej. k-means).
- Caso: sociedad inmobiliaria, 5 millones de clientes, campaña de prueba a 50.000 clientes → 2% de respuesta positiva (1.000 clientes). Objetivo: duplicar al 4% (40.000 nuevos inversores).
- Variables: `time_ace` (antigüedad de la cuenta en años) y `avebal` (balance/actividad promedio últimos 3 meses, en libras).
- Arquitectura: perceptrón multicapa, 2 neuronas de entrada, 4 en capa oculta, 1 de salida (puntuación 0-1).
- 2.000 datos (1.000 positivos / 1.000 negativos), divididos en 500/500 para train y 500/500 para test.
- Algoritmo: backpropagation, entre 100 y 500 pasadas (épocas).
- Resultado: la RNA discrimina mejor que el método clásico (ej. método CHAID/site) en la curva de ganancia.

### Advertencia de colisión trasera (ADAS - caso conducción)
- Uno de los accidentes más comunes en autopistas: la **colisión trasera**, debida a juicio erróneo del conductor (percepción inexacta, subestimación del frenado necesario, tiempo de reacción — PRT).
- **Enfoques clásicos** de sistemas de advertencia de colisión trasera (RE-CWS):
  - **Enfoque perceptivo**: basado en el TTC (*Time To Collision*) = (h − L) / (V_F − V_P). Ej. algoritmo Honda.
  - **Enfoque cinemático**: variables observables en entorno V2I excepto el PRT. Ej. algoritmos Stopping Distance, Berkeley, Mazda.
  - **Enfoque basado en RNA**: perceptrón multicapa con entradas Gap, V_F, V_P, a_F, a_P → salida = nivel de advertencia de colisión trasera. No depende del PRT.
- Sistema propuesto: **MCWA**, con entrenamiento y predicción en tiempo real mediante **periodos rodantes** (rolling periods): mientras se predice el presente con el modelo del periodo anterior, se entrena el modelo para el periodo siguiente.
- Datos: NGSIM, U.S. Route 101, segmento de 640m, 5 carriles, frecuencia 0.1s, 15/06/2005, 7:50-8:35h.
- Tras filtrar (camiones/motos excluidos): 322 casos de seguimiento de vehículos, 53 con al menos un frenado brusco.
- **Evaluación comparativa** (curva ROC, área bajo la curva):
  - Honda: 0.636
  - Berkeley (τ=0.5): 0.683
  - Berkeley (τ=1.5): 0.710
  - Berkeley (τ=2.5): 0.704
  - **MCWA (RNA): 0.810** — mejor desempeño de todos.
- Contexto: se estima que una advertencia 0.5s antes evita el 60% de colisiones traseras, y con 1s extra, el 90%. Tiempo mínimo de advertencia adoptado en el estudio: 1.5s.

---

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
