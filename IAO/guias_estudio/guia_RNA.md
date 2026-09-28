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
