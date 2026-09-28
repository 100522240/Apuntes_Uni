# Guía de Estudio: Algoritmos Evolutivos

*Inteligencia Artificial en las Organizaciones — basada en los 5 vídeos del tema (4.1 a 4.5) y en las diapositivas de la sesión magistral.*

## Índice
1. [Evolución biológica y computación evolutiva](#1-evolución-biológica-y-computación-evolutiva)
2. [Elementos: analogía biología ↔ computación](#2-elementos-analogía-biología--computación)
3. [Estructura básica de un algoritmo evolutivo](#3-estructura-básica-de-un-algoritmo-evolutivo)
4. [Los operadores genéticos en detalle](#4-los-operadores-genéticos-en-detalle)
5. [El algoritmo evolutivo como proceso de búsqueda](#5-el-algoritmo-evolutivo-como-proceso-de-búsqueda)
6. [Taxonomía de los algoritmos evolutivos](#6-taxonomía-de-los-algoritmos-evolutivos)
7. [Cómo aplicar un algoritmo genético: las dos fases](#7-cómo-aplicar-un-algoritmo-genético-las-dos-fases)
8. [Ejemplo de los vídeos: el *scorer* bancario](#8-ejemplo-de-los-vídeos-el-scorer-bancario)
9. [Caso de las diapositivas 1: reconocimiento facial visible + térmico](#9-caso-de-las-diapositivas-1-reconocimiento-facial-visible--térmico)
10. [Caso de las diapositivas 2: pose de sentado del robot NAO](#10-caso-de-las-diapositivas-2-pose-de-sentado-del-robot-nao)
11. [Otras aplicaciones y vigencia actual](#11-otras-aplicaciones-y-vigencia-actual)
12. [Resumen rápido y errores típicos de test](#12-resumen-rápido-y-errores-típicos-de-test)

---

## 1. Evolución biológica y computación evolutiva

### Contexto biológico
- Hasta el siglo XVIII se creía que la naturaleza era **inmutable**; el descubrimiento de los **fósiles** contradijo la teoría de la inmutabilidad de las especies.
- **Siglo XIX**: **Charles Darwin y Alfred Wallace**, tras décadas de observación en distintos lugares del planeta, llegaron a la misma conclusión: *las especies evolucionan a lo largo del tiempo*. Presentaron sus conclusiones conjuntamente en **1858**.
- **1859**: Darwin publica en solitario ***El origen de las especies***, considerada la obra más influyente en la historia de la ciencia.
- Hoy la evolución no se discute; lo que se debate son los **mecanismos** por los que ocurre.
- **Años 40 del siglo XX**: **teoría de síntesis evolutiva moderna**, que integra:
  - Evolución por **selección natural** (Darwin y Wallace).
  - **Teoría genética** como base de la herencia biológica (Gregor **Mendel**).
  - **Mutación genética** como base de la variación.
  - **Genética de poblaciones**.

### Computación evolutiva (CE)
- Está basada en la teoría de síntesis evolutiva moderna: **emula conceptos de esa teoría** para obtener respuestas a determinados problemas.
- Integra **tres conceptos básicos** de la evolución biológica:

| Concepto | Idea |
|---|---|
| **Selección** | Los mejor adaptados tienen más probabilidad de reproducirse |
| **Herencia** | Los descendientes heredan material genético de los padres |
| **Variación** | Aparecen cambios (mutación, recombinación) |

- Desde el punto de vista computacional, su implementación y combinación da lugar a un algoritmo que supone un **proceso de búsqueda**.
- Se aplican **principalmente a problemas de diseño y de optimización** cuya respuesta se puede obtener mediante búsqueda y donde otros tipos de búsqueda (**fuerza bruta**, búsqueda heurística) **no son factibles** en la práctica.

### Historia
| Época | Hito |
|---|---|
| **Años 50** | Prototipos computacionales de los modelos biológicos de la evolución (para su análisis y validación) |
| **Años 60** | Se trasladan a la computación los conceptos de la síntesis evolutiva; primeras aproximaciones: **algoritmos genéticos, estrategias evolutivas y programación evolutiva** |
| **Años 70-80** | Dos décadas con apenas avances significativos |
| **Principios de los 90** | La comunidad científica se organiza: primeras ediciones de conferencias (que aún se celebran); más soporte teórico; renovado impulso hasta hoy |

### Características de la CE
- Alternativa viable cuando la **elevada complejidad computacional** de otros algoritmos impide obtener respuesta en el tiempo requerido, o cuando **no se puede aplicar una heurística** para obtener la solución (óptima).
- Ofrece **flexibilidad**, **adaptabilidad** y un **desempeño robusto** (flexible **y** robusto, no una cosa sin la otra).
- Lleva a cabo **búsqueda global en paralelo con búsqueda local**.
- El objetivo hoy es **resolver problemas** de diseño y optimización; no se limita a modelar/validar la evolución biológica (que era el objetivo de los primeros prototipos de los años 50).

---

## 2. Elementos: analogía biología ↔ computación

| Evolución biológica | Computación evolutiva |
|---|---|
| **Entorno natural** | El **problema** que se quiere resolver |
| **Individuo** | Una **posible solución** al problema |
| — | Cada individuo = un **punto del espacio de búsqueda** (definido por el problema y todas sus posibles respuestas) |
| **Población** | Conjunto (limitado) de posibles soluciones que se manejan; se corresponde con los puntos del espacio de búsqueda que se van **visitando** |
| **Adecuación (fitness)**: capacidad de reproducirse (probabilidad de transferir material genético a la siguiente generación) | **Valor de adecuación**: lo determina el **problema** (ej.: % de acierto al clasificar operaciones de tarjeta como fraudulentas; ahorro de combustible del diseño aerodinámico de un coche). Debe **distinguir muy bien** una buena solución de una muy mala |
| **Cromosoma**: material donde se codifican las características físicas de un ser vivo | **Cromosoma**: mucho **más sencillo**; en muchos casos, el conjunto de valores de los parámetros que definen una solución |
| **Gen** (definición compleja) | **Gen** = unidad de información **más pequeña** dentro de un cromosoma |
| Reproducción sexual y mutaciones | **Operadores genéticos**: **sobrecruzamiento (recombinación)** y **mutación** |
| **Selección natural** (el mejor adaptado) | **Continuar la búsqueda a partir de las mejores soluciones** encontradas hasta el momento |

### Puntos clave sobre el cromosoma
1. Es **más sencillo** que en biología.
2. Su diseño es **específico para cada problema**: si se aplica un AE a un problema nuevo, hay que **diseñar el cromosoma desde cero**.
3. Los valores de los parámetros se pueden representar **de diferentes maneras**: valor en base 10, código binario, números reales, caracteres, etc. Un parámetro puede ocupar **un solo elemento** del cromosoma o **varios**.

> **Implementación en un lenguaje de programación:** individuo = *array* (de 0/1, de caracteres o de *float*) y población = *matriz* del mismo tipo. **Todas son válidas, depende del problema.**

### Puntos clave sobre el gen
- Un parámetro de la solución puede corresponderse con **un solo gen** o con **una secuencia de genes**.
- El gen es la **unidad mínima** sobre la que actúan algunos operadores: se modifica su valor (mutación) o se corta el cromosoma justo donde empieza/termina (sobrecruzamiento).

### Operadores genéticos (visión general)
- Implementan **saltos dentro del espacio de búsqueda** (determinan los siguientes puntos a visitar).
- **Sobrecruzamiento / recombinación**: combina el material genético de **dos** individuos (padres) para obtener nuevos individuos (hijos).
- **Mutación**: pequeños cambios sobre el material genético de un individuo.
- Existen múltiples versiones y, según el problema, puede ser necesario modificar el diseño general.
- Las soluciones que **no son las mejores no se descartan**: simplemente su probabilidad de generar saltos/descendencia debe ser **menor**.

---

## 3. Estructura básica de un algoritmo evolutivo

Todos siguen más o menos una estructura básica, aunque se pueden hacer modificaciones para mejorar **eficacia** (mejores respuestas) y **eficiencia** (menos generaciones o menos individuos).

### Pseudocódigo
```
inicio AE
  población <- inicializarPoblacionAleatoriamente()
  obtenerValorAdecuacionPoblacion(población)

  MIENTRAS (condiciónTerminación == false)
      padres        <- seleccionarPadresParaReproduccion(población)
      nuevaPoblación <- reproducirPadres(padres)   // sobrecruzamiento + mutación
      obtenerValorAdecuacionPoblacion(nuevaPoblación)
      numGeneración <- numGeneración + 1
      población     <- nuevaPoblación               // reemplazo
  FIN_MIENTRAS
fin AE
```

### Pasos
1. **Población inicial**: lo habitual es generarla de forma **aleatoria** (en algunos casos podría no serlo). Cada individuo es una posible respuesta.
2. **Evaluación** de cada individuo (valor de adecuación).
3. Se comprueba la **condición de terminación**; si se cumple, fin.
4. Si no, dentro del bucle de cada generación:
   1. **Selección** de los progenitores (padres).
   2. **Sobrecruzamiento** y **mutación** → descendientes (hijos).
   3. **Reemplazo** → se obtiene la población de la siguiente generación.
   4. **Evaluación** de la nueva población y se incrementa el nº de generación.

> **Orden de los operadores: Selección → Sobrecruzamiento → Mutación → Reemplazo.**

### Condición de terminación
- Alcanzar un **número determinado de generaciones**, o
- Que la población cumpla determinada condición (relativa a un individuo, a un subconjunto o a toda la población). Ejemplos: que el fitness **medio** supere un umbral, o que el **mejor 10 %** supere un umbral.

### Caso especial: población de tamaño 1
| Operador | ¿Aplicable? | Motivo |
|---|---|---|
| Selección | ✗ | No hay entre qué elegir |
| Sobrecruzamiento | ✗ | Necesita **dos** padres |
| Mutación | ✓ | Se muta el único individuo |
| Reemplazo | ✓ | Se decide si el hijo sustituye al padre |

---

## 4. Los operadores genéticos en detalle

### 4.1 Selección
- Elige los cromosomas que se van a **reproducir**.
- **Principio general**: los cromosomas con **buen fitness** tienen **mayor probabilidad** de ser seleccionados.
  - Buen fitness = valor **alto** si se **maximiza**, valor **bajo** si se **minimiza**.
- Hay muchas versiones (ej.: **ruleta**, usada en el caso de reconocimiento facial); cada problema tendrá la más adecuada.

### 4.2 Sobrecruzamiento (recombinación)
- Toma los cromosomas de **dos individuos distintos** y genera nuevos recombinando trozos/genes.
- Se aplica con una **probabilidad** determinada; si no se aplica, los dos padres se **clonan**.
- Los trozos pueden resultar de dividir cada cromosoma en **un punto** (el mismo en ambos) o en **varios puntos**.
- Las soluciones resultantes deben **seguir teniendo sentido** (ser viables) como solución al problema.
- Realiza **búsqueda global**: permite alcanzar desde los padres **regiones distantes** del espacio de búsqueda.

### 4.3 Mutación
- Realiza **pequeños cambios** en el cromosoma = **pasos pequeños** en el espacio de búsqueda → explora áreas que no se alcanzan con el sobrecruzamiento.
- **Previene la saturación** de la población con cromosomas similares.
- Cada gen se examina y su **alelo (valor)** cambia con una **tasa de mutación** (pequeña o muy pequeña).
  - Tasa **demasiado alta** → la búsqueda se **desorienta**.
  - Tasa **demasiado pequeña** → la búsqueda se **estanca**.
  - Valor óptimo (según ciertos estudios): **1 / longitud del cromosoma** (existen otras fórmulas).
- Realiza **búsqueda local**.

### 4.4 Reemplazo
Estrategias:
- **Generacional**: se reemplaza **toda** la población.
- **Estado estacionario**: el reemplazo es parcial. Criterios:
  1. Reemplazar solo los individuos con **peor fitness**.
  2. Reemplazar a los **progenitores**.
  3. Reemplazar los individuos **más parecidos entre sí** a nivel del cromosoma.

---

## 5. El algoritmo evolutivo como proceso de búsqueda

- **Paralela**: se manejan simultáneamente varias alternativas (los individuos de la población).
- **Estocástica**: tiene componentes aleatorios (dónde saltar y desde dónde). No es inherente a los AE que el **cálculo del fitness** de un individuo sea aleatorio.

### Exploración vs. explotación
| | Qué es | Cómo |
|---|---|---|
| **Exploración** | Saltos **en parte aleatorios** (probabilísticos) dentro del espacio de búsqueda | Saltos aleatorios |
| **Explotación** | Saltos realizados **a partir de los mejores individuos** | Individuos con **mejor** valor de adecuación |

### Búsqueda global vs. local (se llevan a cabo **simultáneamente**)
| | Global | Local |
|---|---|---|
| Idea | **Grandes saltos** | **Pequeños saltos** |
| Se implementa con | **Población inicial** (intenta cubrir todo el espacio) y **sobrecruzamiento** (alcanza regiones distantes) | **Mutación** (pequeñas modificaciones sobre cada solución) |

> La **selección natural** no es un operador de búsqueda local: determina desde qué individuos se sigue buscando (explotación), pero no genera nuevos puntos.

---

## 6. Taxonomía de los algoritmos evolutivos

Todos comparten el **principio común**: evolución de una población de soluciones mediante **variación** (mutación, cruce) y **selección** para buscar progresivamente mejores soluciones. Los enfoques están **abiertos**: se pueden **modificar y combinar** entre sí (incluso crear un AE *ad hoc*).

| Enfoque | Autor / época | Qué evoluciona | Operadores | Usos típicos |
|---|---|---|---|---|
| **Algoritmos genéticos (AG)** | **John Holland**, fundamentos teóricos **1975** ("padre de la CE") | Soluciones codificadas (**cromosomas**) | Selección, **cruce** y **mutación** | Optimización de parámetros, selección de características, diseño de soluciones, problemas combinatorios |
| **Estrategias evolutivas (EE)** | **Rechenberg y Schwefel**, finales de los 60 / principios de los 70 | **Parámetros reales** (float) | Solo **selección y mutación** | Optimización continua, diseño de ingeniería, control y robótica, entrenamiento de modelos |
| **Programación evolutiva (PE)** | **Lawrence Fogel**, años 60 *(en la transcripción aparece como "Lawrence Roberts")* | Directamente los **parámetros/estructura de la solución** | Solo **mutación** y selección, **sin cruce** | Optimización continua, control adaptativo, sistemas dinámicos, diseño de controladores |
| **Programación genética (PG)** | Ampliación del enfoque de los AG | **Programas** (expresiones, árboles, reglas, código fuente) | Como AG | Descubrimiento de expresiones, modelado simbólico, diseño automático de estrategias, robótica y control |

### Ampliaciones de los algoritmos genéticos
- **AG basados en orden**: el valor de adecuación depende del individuo **y de su posición/orden** dentro de la población.
- **Sistemas clasificadores**: el cromosoma codifica **sistemas de reglas** (ej.: ayudar a tomar una decisión o controlar un sistema).
- **Programación genética**: el cromosoma codifica **programas, algoritmos o código fuente** (no un vector de números).

> "Sistemas Genéticos" **no** existe en la taxonomía.

### Algoritmos genéticos (AG): características
- Paradigma **más completo, más estudiado y más aplicado**.
- Estructura clásica: la vista en el apartado 3. Mantienen una población de posibles soluciones, sometida a **transformaciones** y a un proceso de **selección** que favorece a los mejores.
- Son un **método estocástico de búsqueda ciega de soluciones cuasi-óptimas** (no "informada", no "óptimas").
- Requieren inicialmente **poco conocimiento** del problema, pero son **muy flexibles**: es **fácil incorporar información específica** del problema.
- Muy **fáciles de implementar** (sencillos de entender y modificar).
- Operan **directamente sobre el genotipo** (el cromosoma).

### Estrategias evolutivas (EE)
- Ideadas especialmente para **optimización numérica**.
- El cromosoma contiene los **valores de los parámetros** de un sistema, codificados como **números reales (float)**.
- Solo **selección y mutación**.
- **Selección determinista**: se toman solo los **mejores** de la población para la siguiente generación y se desestima el resto (a diferencia de los AG, donde es probabilística).
- **Mutación**: un gen = un único valor float; el nuevo valor sigue una **distribución gaussiana (normal)** (no uniforme).

### Programación evolutiva (PE)
- **No confundir** con programación genética.
- Surge del intento de crear/emular **inteligencia artificial**; se concibió como la **evolución de máquinas de estados** (autómatas finitos) mediante aplicaciones sucesivas **solo del operador de mutación**.
- La solución **no tiene por qué ser una cadena de tokens**: se puede representar tal cual se implementa (ej.: una red neuronal). La mutación se aplica sobre la **matriz de pesos** o **añadiendo/eliminando conexiones** → se aplica sobre el **fenotipo**, no sobre el cromosoma (genotipo).
- **No hay recombinación/sobrecruzamiento**: cada individuo representa una **especie** que evoluciona solo por mutación.

---

## 7. Cómo aplicar un algoritmo genético: las dos fases

### Fase 1 — Modelar el problema (dependiente del problema, *ad hoc*)
Muy importante para la eficiencia y eficacia del AG:
1. **Diseñar el cromosoma**: qué información se incluye y **cómo se codifica**.
2. **Determinar la función de adecuación (fitness)**: debe permitir distinguir de forma significativa si una solución es mejor que otra.

### Fase 2 — Determinar el algoritmo genético completo
Se puede aplicar el esquema general o modificaciones (hay abundante bibliografía):
- Cuerpo del bucle de cada generación.
- **Rutina de selección**.
- **Operadores de sobrecruzamiento y mutación** (independientes del cromosoma diseñado).
- **Parámetros**: **tasa de mutación**, **tamaño de la población**, etc.
- **Criterio de terminación**.

> Orden correcto: **primero** cromosoma y fitness, **después** operadores y parámetros.

---

## 8. Ejemplo de los vídeos: el *scorer* bancario

### Problema
- Para un banco, un préstamo es un **activo** (ingresos netos positivos) pero con **riesgo de impago**: interesa **minimizar el dinero perdido** en la concesión de créditos.
- Se consideran atributos del cliente: edad, estado civil, nº de hijos, tipo de entidad en la que trabaja, salario, deudas, antigüedad en la empresa, etc.
- Un ***scorer*** es un sistema que recibe la información de un cliente y devuelve un **valor numérico asociado al riesgo** (según el cual se decide conceder o no el préstamo y a qué interés).

### Modelado
- El *scorer* se compone de un **conjunto de reglas** (tipo **IF-THEN**), representadas como una **matriz**: una **fila por regla**, columnas = características del cliente, **última columna = salida** (conceder / no conceder).
- **Cromosoma / individuo = TODA la matriz de reglas** (no una única regla).
- **Codificación**: código para cada *token* de información y sus posibles valores (nº de bits variable según el nº de valores posibles) → se unen en una cadena de 0/1 → se concatenan las cadenas de cada regla/fila.
- **Población** = conjunto de matrices.
- **Fitness**: se aplica el conjunto de reglas sobre un **histórico de créditos concedidos en el pasado**, donde se sabe si hubo **impago** o no → fitness = **dinero perdido** (se **minimiza**). No lo asigna un experto; se calcula automáticamente. Solo sirve el histórico de préstamos **concedidos** (los no concedidos no permiten saber si habría impago).

### Operadores aplicados al ejemplo
Selección, sobrecruzamiento, mutación y reemplazo, con las características vistas en el apartado 4.

---

## 9. Caso de las diapositivas 1: reconocimiento facial visible + térmico

*Hermosilla, Gallardo, Farias y San Martin (2015), "Fusion of Visible and Thermal Descriptors Using Genetic Algorithms for Face Recognition Systems", Sensors.*

### Idea
- Sistema de reconocimiento facial que **fusiona** información del **espectro visible** y del **térmico**, usando un **AG para optimizar los pesos de fusión**.

| | Espectro visible | Espectro térmico |
|---|---|---|
| Aporta | Información de apariencia; muy usado en reconocimiento facial | Información térmica; menor dependencia de la iluminación visible |
| Limitación | Sensible a cambios de **iluminación** | Sensible a condiciones **ambientales y fisiológicas** |

### Descriptores (métodos locales de correspondencia)
- Visibles y térmicos: **LBP** (Local Binary Pattern), **WLD** (Weber Law Descriptor), **GJD** (Gabor Jet Descriptor).
- Visibles: **LDP** (Local Derivative Pattern), **HOG**.
- En la fusión propuesta se usan combinaciones **LDP–LBP** y **LBP–LBP** (LDP de 3.er orden con 256 divisiones/8 sectores; LBP con 32 regiones).

### Requisitos
- **Funcionamiento en tiempo real**.
- Utilizar **solo una imagen en galería**.
- **Alto rendimiento**.

### Modelado del AG
- **Cromosoma = 32 pesos visible + 32 pesos térmico = 64 genes** (un peso por región de la imagen y por espectro).
- Descriptor de fusión: `FD = [ w_visible_i · HI(VD_i^{j,k}) , w_thermal_i · HI(TD_i^{j,k}) ]`
  - *i*: región de la imagen · *j*: imagen de prueba · *k*: imagen de galería · *VD/TD*: descriptor visible/térmico · *HI*: función de **intersección de histograma**.
- **Fitness = tasa de reconocimiento** del individuo (calculada con el valor de similaridad SV = suma de los FD de las 64 componentes, sobre la base de datos de galería y de prueba).

### Ciclo del AG
| Paso | Detalle |
|---|---|
| **Inicialización** | **100 individuos**, cada uno con **64 pesos aleatorios en [0, 1]**; pesos **complementarios** para las regiones en la población inicial |
| **Evaluación** | Cada individuo se evalúa sobre las bases de datos de galería y prueba |
| **Selección** | 2 padres, **método de la ruleta** |
| **Cruce** | **25 %** de probabilidad de cruce **en cada punto**; se obtienen **2 nuevos individuos** |
| **Mutación** | **25 %** de probabilidad, en **3 puntos aleatorios** |
| **Reemplazo** | Se calcula el fitness de la nueva población; se **reemplaza un individuo aleatorio** si el nuevo tiene **mejor fitness** (estado estacionario) |
| **Fin** | Al llegar a **100.000 iteraciones** |

### Bases de datos
- **Equinox**: espectro visible + infrarrojo (cercano, onda media, onda larga); **18.629 imágenes por espectro**, 240×320 px; conjunto galería de **81 individuos** (iluminación frontal y lateral, tres expresiones); preprocesado a **81×150 px**; individuos con gafas se almacenan dos veces; **15 conjuntos de datos = 6 de galería + 9 de test**; en cada galería solo hay **una imagen por individuo**.
- **PUCV-VTF**: **76 individuos**, **12.160 imágenes** en ambos espectros (visible 640×480, infrarrojo de onda larga 640×512); **5 subconjuntos / 5 expresiones** (normal, fruncido, gafas, sonrisa, vocal); el grupo **"normal"** es la galería; imágenes a 81×150 px con 42 px entre los ojos, normalizadas.

### Experimentos
- Equinox: **entrenamiento con AE sobre galería** y test sobre el resto; se comparan los descriptores individuales y la fusión (la fusión obtiene tasas de reconocimiento **superiores** a los descriptores individuales).
- PUCV-VTF: se reutiliza la **mejor población** del experimento anterior; se usa el criterio **"top"** para la tasa de reconocimiento.

### De 2015 a hoy (visión multimodal)
- Hoy se usan **modelos profundos** (CNN, Vision Transformers, modelos visión-lenguaje) para integrar múltiples modalidades (RGB, IR, profundidad, texto).
- **Los algoritmos evolutivos siguen siendo relevantes** como herramienta de optimización complementaria: diseño y ajuste de modelos (arquitecturas, selección de características, **hiperparámetros**), problemas **no diferenciables o con restricciones**, nuevos escenarios (modelos generativos, multiobjetivo, pocos datos).
- **Idea clave**: la forma de construir el sistema ha cambiado, pero el **problema de optimización permanece**; los AE no sustituyen al modelo de visión, pero son útiles para buscar buenas soluciones en **espacios complejos**, sobre todo cuando la optimización **no es fácilmente diferenciable**.

---

## 10. Caso de las diapositivas 2: pose de sentado del robot NAO

*Al-Hami y Lakaemper (2014), "Sitting pose generation using genetic algorithm for NAO humanoid robots", IEEE ARSO.*

### Contexto y objetivo
- Creciente uso de **robots humanoides** para imitar al ser humano.
- **Objetivo**: encontrar una **postura óptima para sentarse**, basada en el estado físico, que satisfaga los parámetros del objeto sobre el que se sienta.

### Robot NAO
- **25 grados de libertad**.
- Capacidades limitadas a: **detección de colisión**, **estimación de distancia** y **motor físico**.

### Función de fitness
- Se usan **tres distancias**: cabeza–suelo, caderas–superficie del objeto y caderas–superficie del suelo.
- `F : S → L ∈ U` — *F* = función de fitness; *S* = todas las poses posibles para sentarse; *L* = valor de fitness; *U* = **seis niveles monotónicamente crecientes**.
- Pose óptima: `p̂ = argmax_{p ∈ S} F(p)`.

### Descubrimiento de la altura del objeto
- **Metas**: detectar colisiones y colocar las caderas del robot **sobre** el objeto.
- Se usa una **ruta de movimiento estable predefinida**; el proceso ajusta articulaciones concretas: **LKneePitch, RKneePitch, LHipPitch, RHipPitch**.

### Representación (cromosoma)
- Bloques de articulaciones: **brazo izquierdo (5)**, **brazo derecho (5)**, **pierna izquierda (6)**, **pierna derecha (6)**.
- El cromosoma se organiza por estos **bloques** (valores en grados de cada articulación).

### Algoritmo genético
- **CrossRate = 0,9** · **MutRate = 0,6** · la mutación es **como máximo de 5 grados**.
- Generación de un descendiente: cruce a nivel de **bloques/genes** de dos padres y posterior mutación de algún gen (ver ejemplo Parent1/Parent2 → Crossover → Mutation → Child en las diapositivas).
- Validación en el simulador **V-REP** con objetos de distinta forma y tamaño (cubos de 12 y 15 cm de altura; esferas de 5 y 7,5 cm de radio), y también con el robot real.

### NAO: optimización robótica en la era generativa
| | |
|---|---|
| **Ayer** | AG aplicados al **control de posturas** (poses estables); optimización de movimientos con **restricciones físicas** |
| **Hoy** | **IA generativa** genera escenarios sintéticos de interacción y simulaciones realistas; **Deep Learning** en visión y percepción |
| **Todavía vigente** | Los **algoritmos evolutivos siguen siendo esenciales** para la **optimización motora** en robots. **Complementariedad**: la IAG *imagina/simula*, la IA clásica *optimiza y controla*. NAO es un ejemplo didáctico de cómo las organizaciones deben integrar **enfoques híbridos** |

---

## 11. Otras aplicaciones y vigencia actual

Los AE se pueden aplicar a muchos problemas, por ejemplo:
- Parámetros para el diseño del **ala de un avión** o de las **hélices de un aerogenerador**.
- Generación de **trayectorias de robots**.
- Diseño de **redes neuronales artificiales** (incluidas las *deep neural networks*).
- Diseño de **sistemas de diagnóstico de fallos**.
- Ejemplos de las diapositivas ("tareas de optimización"): dibujar una imagen (torre Eiffel) mediante evolución, marcha de robots cuadrúpedos/bípedos, simulación de un vehículo que evoluciona para recorrer un terreno.

---

## 12. Resumen rápido y errores típicos de test

### Datos para memorizar
- **3 conceptos** de la evolución que usa la CE: **selección, herencia, variación**.
- Problemas típicos: **diseño y optimización** (no clasificación/regresión, no planificación, no reconocimiento de patrones).
- Orden de los operadores: **Selección → Sobrecruzamiento → Mutación → Reemplazo**.
- **Sobrecruzamiento = búsqueda global** · **Mutación = búsqueda local**.
- AG = búsqueda **estocástica**, **ciega**, de soluciones **cuasi-óptimas**; requieren **poco** conocimiento inicial y son **flexibles**.
- Fases del AG: **1) cromosoma y fitness → 2) operadores y parámetros**.
- Tasa de mutación óptima aproximada: **1 / longitud del cromosoma**.
- Holland **1975** (AG) · Rechenberg y Schwefel (EE) · Fogel (PE).

### Trampas frecuentes
| Afirmación falsa típica | Realidad |
|---|---|
| El libro de Darwin es *El origen de la vida*, de 1759 | ***El origen de las especies*, 1859** |
| Un cromosoma en AE es tan complejo como en biología | Es **más sencillo** y específico de cada problema |
| El cromosoma solo puede ser una secuencia de 0 y 1 | Puede ser binario, base 10, reales, etc. |
| Las soluciones no óptimas no pueden tener descendencia | **Sí pueden**, con **menor probabilidad** |
| Un AE es flexible pero NO robusto (o al revés) | Es **flexible y robusto** |
| El único objetivo de la CE es modelar/validar la evolución biológica | Sirve para **resolver problemas** de diseño y optimización |
| La selección natural equivale a búsqueda local | La búsqueda local es la **mutación** |
| La explotación usa los individuos con **menor** adecuación | Usa los de **mejor** adecuación |
| Los AG requieren mucho conocimiento del problema | Requieren **poco**; es fácil **incorporar** el específico |
| El AG es búsqueda **informada** de soluciones **óptimas** | Búsqueda **ciega** de soluciones **cuasi-óptimas** |
| Programación genética = programación evolutiva | Son **distintas** (PG: programas/código; PE: mutación sobre máquinas de estados/fenotipo) |
| En EE la mutación usa distribución uniforme | Usa distribución **gaussiana (normal)** |
| En PG el cromosoma es un vector de números | Es un **programa / algoritmo / código fuente** |
| En PE se aplica recombinación | **No**: solo mutación |
| En el *scorer*, un cromosoma es una sola regla | Es **toda la matriz de reglas** |
| El fitness del *scorer* lo asigna un experto o usa préstamos no concedidos | Se calcula sobre un **histórico de préstamos concedidos** con su resultado (impago o no) |
| "Sistemas Genéticos" es una ampliación de los AG | No existe; son **AG basados en orden, sistemas clasificadores y programación genética** |
| Con población de 1 individuo se puede aplicar selección y/o sobrecruzamiento | Solo **mutación** y **reemplazo** |
