# Guía de estudio — Las 10 referencias del Avance 2

Para cada referencia: la cita completa (APA), un resumen de qué hace el paper, **dónde** se cita en `main2.tex` y **para qué** se usa exactamente en nuestro argumento (no solo "de qué trata", sino "qué trabajo le hace a nuestro texto"). Orden alfabético, igual que en la sección de Referencias del documento.

---

## 1. Ajjarapu, V., & Christy, C. (1992)
**The continuation power flow: A tool for steady state voltage stability analysis.** *IEEE Transactions on Power Systems*, 7(1), 416–423. https://doi.org/10.1109/59.141737

**De qué trata:** Es un paper clásico (de los más citados) de sistemas eléctricos de potencia. El problema que resuelven: al calcular el flujo de potencia (*power flow*) de una red eléctrica con Newton-Raphson normal, a medida que la carga del sistema sube hacia su límite de estabilidad de voltaje, el Jacobiano se acerca a ser singular y Newton-Raphson deja de converger justo en el punto más interesante (el límite crítico). Proponen el **"continuation power flow"**: en vez de resolver cada nivel de carga desde cero, usan la solución de un nivel de carga como **punto semilla** de Newton para el siguiente nivel (más un paso de predicción para no perder la trayectoria), y así logran seguir toda la curva de soluciones incluso cerca de la singularidad.

**Dónde se cita:** Sección Metodología, 2.3 *Métodos numéricos propuestos*, primer párrafo (línea 163 de `main2.tex`), justo después de explicar la estrategia de continuación para el método de Newton.

**Para qué la usamos:** Es la referencia que **respalda técnicamente** nuestra decisión de no resolver cada punto de equilibrio del BTS desde cero, sino encadenar las soluciones (equilibrio de un nivel de carga → semilla del siguiente). Es exactamente la misma idea que ellos aplican a redes eléctricas, aplicada acá a un sistema caldera-turbina: ambos son sistemas no lineales fuertemente acoplados donde Newton "a ciegas" puede fallar si el punto semilla está lejos de la raíz.

---

## 2. Åström, K. J., & Bell, R. D. (2000)
**Drum-boiler dynamics.** *Automatica*, 36(3), 363–378. https://doi.org/10.1016/S0005-1098(99)00171-5

**De qué trata:** Es el paper que **origina el modelo** que usamos en todo el proyecto. Desarrolla, a partir de primeros principios físicos (balances de masa y energía), un modelo dinámico no lineal de tercer orden para una caldera de tambor de circulación natural. El modelo queda reducido a un conjunto pequeño de parámetros físicos y fue validado con datos reales de plantas en Suecia y Australia. Desde entonces es el modelo de referencia (*benchmark*) estándar del área — prácticamente todos los demás papers que citamos (Tan, Garrido, Jalali, Ghabraei, Liu, Kruthika) usan este mismo modelo como punto de partida.

**Dónde se cita:** Es la referencia más citada del documento (7 veces): en la Introducción (antecedentes, justificación, objetivo general) y en toda la Metodología (Modelo del sistema, Supuestos y simplificaciones).

**Para qué la usamos:** Es la fuente **directa** de las Ecuaciones (1)-(8) de nuestra sección "Modelo del sistema" — literalmente el modelo matemático que vamos a resolver numéricamente. También sustenta las simplificaciones que asumimos (parámetros concentrados, no distribución espacial) porque son las mismas que adopta el modelo original.

---

## 3. Bajrami, E., Bajrami, E., & Lameski, P. (2026)
**Modelling and quantifying numerical integration errors in deep reinforcement learning for propulsion dynamics.** *Aerospace Science and Technology*, 177, Artículo 112209. https://doi.org/10.1016/j.ast.2026.112209

**De qué trata:** Paper muy reciente. Entrenan controladores de aprendizaje por refuerzo (PPO, SAC y **TD3** — el mismo algoritmo base del paper de Kruthika y Paneerselvam) sobre un modelo de dinámica de propulsión (un sistema de segundo orden), y comparan qué pasa si el entorno de simulación se discretiza con **Euler** (a distintos tamaños de paso) o con **Runge-Kutta de 4to orden (RK4)**. Corren más de 50,000 episodios de simulación y muestran que la precisión del integrador numérico afecta de forma significativa la estabilidad del entrenamiento y el desempeño final del controlador.

**Dónde se cita:** Sección Metodología, 2.3 *Métodos numéricos propuestos*, tercer párrafo (línea 167), justificando el uso de Runge-Kutta para integrar la dinámica del BTS.

**Para qué la usamos:** Es la evidencia empírica más directa de por qué **el integrador numérico importa** en nuestro proyecto. El agente de RL que ajusta el PID no observa la planta real sino la trayectoria que produce el integrador; si el integrador es de baja fidelidad (como Euler con paso grande), la señal de recompensa/error que ve el agente está distorsionada, igual que muestra este paper. Justifica por qué elegimos RK4 en vez de un método de un solo paso más simple.

---

## 4. Gao, J., & Budman, H. M. (2005)
**Design of robust gain-scheduled PI controllers for nonlinear processes.** *Journal of Process Control*, 15(7), 807–817. https://doi.org/10.1016/j.jprocont.2005.02.003

**De qué trata:** Proponen un método para diseñar controladores PI de "ganancia programada" (*gain-scheduled*) para procesos no lineales: en vez de un solo PI global, diseñan varios controladores lineales en distintos puntos de operación del proceso y los **interpolan** entre sí para cubrir todo el rango operativo, con garantías de robustez frente a la incertidumbre del modelo en las zonas intermedias (donde no hay un controlador diseñado explícitamente).

**Dónde se cita:** Sección Metodología, 2.3 *Métodos numéricos propuestos*, segundo párrafo (línea 165), justo después de proponer la interpolación polinomial entre los puntos de operación tabulados.

**Para qué la usamos:** Es el precedente directo de nuestra idea de **interpolar entre los 7 puntos de operación del Cuadro 2** para poder evaluar el modelo (y, más adelante, el controlador) en niveles de carga intermedios que el paper base no reporta explícitamente. Ellos interpolan ganancias de PI; nosotros interpolamos puntos de equilibrio del modelo — mismo principio matemático (interpolación entre soluciones locales conocidas), aplicado un escalón antes en la cadena (al modelo, no directamente al controlador).

---

## 5. Garrido, J., Morilla, F., & Vázquez, F. (2009)
**Centralized PID control by decoupling of a boiler-turbine unit.** En *Proceedings of the European Control Conference (ECC 2009)* (pp. 4007–4012). EUCA.

**De qué trata:** Sobre el mismo modelo de Åström-Bell, proponen un esquema de control PID **centralizado con desacoplamiento**: construyen una red de desacoplamiento (una matriz que "cancela" las interacciones cruzadas del sistema, calculada sobre una linealización del modelo) para que cada lazo PID controle "su" variable con menos interferencia de las otras dos.

**Dónde se cita:** Introducción, en Antecedentes (línea 99) y en Justificación (línea 101).

**Para qué la usamos:** Es uno de los ejemplos que usamos para armar el argumento central de la Justificación: es una técnica que funciona, pero **se apoya en una matriz de transferencia linealizada** — es decir, fija sus parámetros fuera de línea sobre una aproximación lineal del sistema. Junto con Jalali, Ghabraei y Liu, forma el conjunto de antecedentes que comparten esa misma limitación estructural (ninguno ajusta sus ganancias en función del desempeño observado durante la operación), que es precisamente el vacío que después llena Kruthika y Paneerselvam con MARL.

---

## 6. Ghabraei, S., Moradi, H., & Vossoughi, G. (2020)
**Multivariable robust regulation of an industrial boiler-turbine with model uncertainties.** *2020 9th International Conference on Modern Circuits and Systems Technologies (MOCAST)* (pp. 1–4). IEEE. https://doi.org/10.1109/MOCAST49295.2020.9200289

**De qué trata:** Proponen una regulación **robusta** multivariable para el BTS que explícitamente considera que el modelo tiene incertidumbre (no coincide exactamente con la planta real). El control robusto se diseña para funcionar bien incluso en el peor caso dentro de esa incertidumbre.

**Dónde se cita:** Introducción, Antecedentes (línea 99) y Justificación (línea 101).

**Para qué la usamos:** Es el ejemplo que usamos para señalar el **costo** de la robustez clásica: al diseñar para el peor caso dentro de una incertidumbre acotada, el controlador se vuelve conservador y su desempeño en el punto de operación nominal (el caso "normal", no el peor caso) es peor de lo que podría ser. Es el contraste que preparamos antes de presentar el enfoque adaptativo (MARL) de Kruthika y Paneerselvam, que en lugar de "blindarse" contra la incertidumbre, ajusta sus ganancias según lo que observa en tiempo real.

---

## 7. Jalali, A. A., & Golmohammad, H. (2012)
**An optimal multiple-model strategy to design a controller for nonlinear processes: A boiler-turbine unit.** *Computers & Chemical Engineering*, 46, 48–58. https://doi.org/10.1016/j.compchemeng.2012.06.005

**De qué trata:** En vez de un solo modelo lineal global, usan **varios modelos lineales locales**, cada uno válido cerca de un punto de equilibrio distinto. Usan la **distancia de Vinnicombe** (una métrica que compara qué tan diferentes son dos modelos lineales) para decidir cuántos modelos lineales hacen falta como mínimo y dónde ubicar los puntos de linealización, de forma que se cubra bien todo el rango de operación sin usar modelos de más.

**Dónde se cita:** Introducción, Antecedentes (línea 99) y Justificación (línea 101).

**Para qué la usamos:** Es el antecedente que más se parece, en espíritu, a nuestro propio enfoque numérico (usamos varios puntos de equilibrio, no uno solo). La diferencia que resaltamos en la Justificación es que ellos trasladan el problema hacia la **conmutación** entre modelos/controladores (¿cuándo cambio de un modelo lineal al siguiente?), mientras que nuestro enfoque con Newton + continuación + interpolación mantiene el modelo no lineal completo y solo usa los puntos de equilibrio como referencias, sin necesidad de conmutar entre controladores distintos.

---

## 8. Kruthika, U., & Paneerselvam, S. (2025)
**Novel multiagent reinforcement learning framework using twin delayed deep deterministic policy gradient for adaptive PID control in boiler turbine systems.** *Scientific Reports*, 15(1), Artículo 34558. https://doi.org/10.1038/s41598-025-17928-9

**De qué trata:** Es **el paper base de todo el proyecto** (el que se está reproduciendo/evaluando críticamente). Sobre el modelo de Åström-Bell, proponen usar aprendizaje por refuerzo multiagente para ajustar en tiempo real las ganancias de un controlador PID. Presentan dos variantes del algoritmo TD3 (Twin Delayed DDPG): **SCMA-TD3** (crítico compartido entre los 3 agentes) e **ICMA-TD3** (crítico individual por agente), y comparan su desempeño contra DDPG estándar usando métricas de error (ITAE, ISE, IAE) bajo distintas condiciones de carga y perturbaciones. Concluyen que ICMA-TD3 es superior.

**Dónde se cita:** Es la segunda referencia más citada (6 veces): en todos los tramos clave de la Introducción (contexto, antecedentes, justificación, objetivo general) y en Modelo del sistema (de ahí sacamos el Cuadro 2 de puntos de operación).

**Para qué la usamos:** Cumple dos roles distintos:
1. **Fuente de datos concretos** — de aquí tomamos el Cuadro 2 (los 7 puntos de operación) y confirmamos que el modelo que vamos a implementar es exactamente el que ellos usan como entorno de simulación para entrenar sus agentes.
2. **Objeto de nuestra crítica/contribución** — en la Justificación señalamos que sus resultados no han sido verificados de forma independiente, y que la ventaja de ICMA-TD3 sobre SCMA-TD3 no tiene una justificación teórica clara para un problema de 3 agentes con recompensa acoplada. Esto es lo que motiva nuestro objetivo general: construir el modelo numérico que permita, más adelante, evaluar esa propuesta con rigor.

---

## 9. Liu, X., & Cui, J. (2018)
**Economic model predictive control of boiler-turbine system.** *Journal of Process Control*, 66, 59–67. https://doi.org/10.1016/j.jprocont.2018.03.004

**De qué trata:** Proponen un **Control Predictivo basado en Modelo Económico (Economic MPC)**: en vez de solo perseguir un punto de consigna, el controlador optimiza en línea un costo económico (relacionado con eficiencia/costo de operación) sobre una familia de modelos lineales identificados por mínimos cuadrados en distintos niveles de carga. Para que la optimización en línea no sea demasiado costosa computacionalmente, parametrizan la señal de control con **funciones de Laguerre**, lo que reduce mucho el número de variables de decisión del problema de optimización.

**Dónde se cita:** Introducción, Antecedentes (línea 99) y Justificación (línea 101).

**Para qué la usamos:** Es el ejemplo que usamos para mostrar el **costo computacional** como la limitación de fondo de los enfoques predictivos clásicos: al optimizar en línea sobre un horizonte futuro, el MPC necesita trucos matemáticos (como las funciones de Laguerre) solo para mantener el cálculo manejable en tiempo real — un problema que un enfoque de control PID con ganancias pre-aprendidas (como MARL) evita, porque toda la parte "cara" del aprendizaje ocurre fuera de línea.

---

## 10. Tan, W., Marquez, H. J., Chen, T., & Liu, J. (2005)
**Analysis and control of a nonlinear boiler-turbine unit.** *Journal of Process Control*, 15(8), 883–891. https://doi.org/10.1016/j.jprocont.2005.03.007

**De qué trata:** Usan la **métrica de brecha (*gap metric*)** — una forma de medir qué tan "lejos" está un sistema no lineal de su aproximación lineal en una región de operación dada — para cuantificar el nivel de no linealidad del BTS. Su hallazgo principal: aunque el sistema es globalmente muy no lineal, en regiones de operación bien elegidas se puede controlar eficazmente con un único controlador lineal. También identifican las restricciones físicas de los actuadores (saturaciones) como una fuente secundaria de no linealidad y aplican técnicas *anti-windup* para mitigarla.

**Dónde se cita:** Introducción, Antecedentes (línea 99) y Justificación (línea 101).

**Para qué la usamos:** Es la referencia que **cuantifica** (con la gap metric) el fenómeno que da pie a toda la Justificación: un controlador lineal único solo es válido dentro de una región de operación acotada. Es el primer eslabón de la cadena de antecedentes que va mostrando, uno por uno, cómo cada autor confirma la misma limitación (dependencia de una aproximación lineal local) desde un ángulo distinto.

---

## Resumen visual: quién hace qué en el argumento

```
Åström & Bell (2000)  →  EL MODELO (de aquí sale todo lo demás)
        │
        ├─ Tan et al. (2005)        → cuantifica la no linealidad (gap metric)
        ├─ Garrido et al. (2009)    → control lineal con desacoplamiento
        ├─ Jalali & Golmohammad     → varios modelos lineales + conmutación
        ├─ Ghabraei et al. (2020)   → control robusto (conservador)
        ├─ Liu & Cui (2018)         → Economic MPC (caro computacionalmente)
        │        ↓ (todos comparten: parámetros fijados fuera de línea)
        └─ Kruthika & Paneerselvam (2025) → MARL/TD3, llena ese vacío
                 ↓ (nuestro proyecto reproduce/evalúa esto)
   NUESTRA METODOLOGÍA NUMÉRICA
        ├─ Ajjarapu & Christy (1992)  → respalda la continuación de Newton
        ├─ Gao & Budman (2005)        → respalda la interpolación entre puntos
        └─ Bajrami et al. (2026)      → respalda la elección de RK4
```
