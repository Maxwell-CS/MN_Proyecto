# Guía de estudio — Método de Runge-Kutta (para el Avance 2)

> Este método se propuso en la Metodología como el método numérico **no revisado en clase**, usado para integrar el sistema de EDOs del modelo caldera-turbina (Ecs. 1-3 del `main2.tex`). No se vio en el curso (el temario S3-S7 llega hasta interpolación, no toca integradores de EDOs), así que aquí va una explicación de cero.

---

## 1. ¿Qué problema resuelve?

Un **integrador numérico de EDOs** aproxima la solución de un problema de valor inicial:

```
x'(t) = f(t, x(t)),   x(t0) = x0
```

cuando no se puede (o no conviene) resolver la ecuación analíticamente. En nuestro caso, `x = (x1, x2, x3)` son los tres estados de la caldera y `f` son las Ecs. (1)-(3) del paper (que dependen de `u1, u2, u3` como entradas de control). El integrador nos da `x(t)` en instantes discretos `t0, t1, t2, ...` separados por un paso `h`.

**Por qué importa para el proyecto:** el agente de RL (y en esta etapa, el controlador PID con ganancias fijas) no "ve" la planta real, ve la trayectoria `x(t)` que produce el integrador. Si el integrador es impreciso, la señal de error/recompensa que se calcula a partir de esa trayectoria también lo es.

---

## 2. Punto de partida: método de Euler (el más simple)

Antes de Runge-Kutta, conviene tener claro el método más básico, porque RK es una mejora directa de esta idea:

```
x_{n+1} = x_n + h * f(t_n, x_n)
```

Idea geométrica: en el punto actual `(t_n, x_n)` calculo la pendiente `f(t_n, x_n)` (como la derivada) y avanzo en línea recta con esa pendiente durante un paso `h`. Es exactamente la misma lógica que la recta tangente de Newton-Raphson, pero aplicada para *avanzar en el tiempo* en vez de para *buscar una raíz*.

**Problema de Euler:** solo usa la pendiente al **inicio** del intervalo `[t_n, t_n+h]`. Si la función `f` cambia mucho dentro de ese intervalo (como pasa en nuestro modelo, que es no lineal), el error se acumula rápido. Se dice que Euler tiene **error local de truncamiento de orden `h²`** y **error global de orden `h`** (si divido el paso a la mitad, el error total solo se reduce a la mitad).

---

## 3. La idea de Runge-Kutta: promediar varias pendientes

Runge-Kutta (el método "clásico" de orden 4, **RK4**, que es el estándar de facto) mejora Euler evaluando la pendiente `f` **varias veces dentro del mismo paso** — en el inicio, en dos puntos intermedios, y en el final — y combina esas pendientes con un promedio ponderado. Así "space cubre mejor la curvatura de `f` en vez de asumir que es una recta durante todo el paso.

### Fórmula de RK4

Para avanzar de `x_n` (en `t_n`) a `x_{n+1}` (en `t_{n+1} = t_n + h`):

```
k1 = f(t_n,         x_n)
k2 = f(t_n + h/2,   x_n + (h/2) * k1)
k3 = f(t_n + h/2,   x_n + (h/2) * k2)
k4 = f(t_n + h,     x_n + h * k3)

x_{n+1} = x_n + (h/6) * (k1 + 2*k2 + 2*k3 + k4)
```

**Interpretación de cada `k`:**
- `k1`: pendiente al **inicio** del paso (igual que Euler).
- `k2`: pendiente en el **punto medio** del paso, usando `x` estimado con medio paso de `k1`.
- `k3`: otra pendiente en el **punto medio**, pero re-estimada con `k2` (una corrección de `k2`).
- `k4`: pendiente al **final** del paso, usando `x` estimado con un paso completo de `k3`.

El promedio `(k1 + 2 k2 + 2 k3 + k4) / 6` le da más peso a las dos estimaciones del punto medio (`k2`, `k3`), que es donde una parábola (aproximación de Simpson) concentra la información más representativa del intervalo. De hecho, RK4 está construido para que coincida con la **regla de Simpson** cuando `f` no depende de `x` (solo de `t`) — por eso ya conoces la lógica: es la misma idea de "usar el punto medio para estimar mejor un área/incremento" que ya viste en integración numérica, aplicada ahora paso a paso a una ecuación diferencial.

### Orden del método

- **Error local de truncamiento:** `O(h^5)` por paso.
- **Error global (acumulado):** `O(h^4)`.

Esto significa que si divides el paso `h` a la mitad, el error global se reduce en un factor de `2^4 = 16` (mucho mejor que Euler, que solo mejora en factor 2). Es el motivo por el que RK4 es el estándar en simulación: buena precisión sin tener que evaluar `f` demasiadas veces (4 evaluaciones por paso, contra 1 de Euler).

---

## 4. Cómo se aplica a un sistema vectorial (nuestro caso: 3 estados)

En el modelo caldera-turbina, `x` no es un escalar sino un vector `x = (x1, x2, x3)`, y `f` es un vector de 3 funciones (Ecs. 1-3 del paper). **RK4 no cambia** en absoluto: cada `k_i` (i=1..4) es ahora un **vector de 3 componentes**, calculado evaluando las 3 ecuaciones simultáneamente:

```
k1 = f(t_n, x_n, u_n)           # vector (k1_1, k1_2, k1_3)
k2 = f(t_n + h/2, x_n + h/2 * k1, u_n)
k3 = f(t_n + h/2, x_n + h/2 * k2, u_n)
k4 = f(t_n + h, x_n + h * k3, u_n)

x_{n+1} = x_n + h/6 * (k1 + 2 k2 + 2 k3 + k4)
```

(Aquí `u_n = (u1, u2, u3)` son las entradas de control, que se asumen constantes durante el paso `h` — esto es exactamente el supuesto de "las válvulas responden instantáneamente" que se declaró en Supuestos y simplificaciones.)

La única diferencia práctica frente al caso escalar es que cada `k_i` requiere evaluar las 3 ecuaciones (1)-(3) a la vez, así que se calcula con suma y multiplicación de vectores (como en Newton vectorizado de S4, donde ya trabajaste con `F(x)` vectorial).

---

## 5. Mini-ejemplo numérico (escalar, para practicar a mano)

Sea `x' = -2x`, `x(0) = 1` (solución exacta: `x(t) = e^{-2t}`). Con `h = 0.1`:

```
k1 = f(0, 1)        = -2(1)        = -2
k2 = f(0.05, 1 - 0.1) = -2(0.9)    = -1.8      # x + h/2 * k1 = 1 + 0.05*(-2) = 0.9
k3 = f(0.05, 1 - 0.09) = -2(0.91)  = -1.82     # x + h/2 * k2 = 1 + 0.05*(-1.8) = 0.91
k4 = f(0.1, 1 - 0.182) = -2(0.818) = -1.636    # x + h * k3 = 1 + 0.1*(-1.82) = 0.818

x1 = 1 + (0.1/6)(-2 + 2(-1.8) + 2(-1.82) + (-1.636))
   = 1 + (0.1/6)(-10.876)
   = 1 - 0.18127
   = 0.81873
```

Valor exacto: `e^{-0.2} = 0.81873`. **Coinciden hasta la 5ta cifra decimal con un solo paso** — así de bueno es RK4 comparado con Euler (que con el mismo `h=0.1` daría `x1 = 1 + 0.1(-2) = 0.8`, con error notablemente mayor).

---

## 6. Qué preguntas te pueden hacer en la sustentación

1. **¿Por qué no usar Euler?** Porque su error global es `O(h)`: para lograr la misma precisión que RK4 necesitarías un paso muchísimo más pequeño (más costo computacional), y en un sistema no lineal y acoplado como el nuestro el error se dispara rápido.
2. **¿Por qué RK4 y no un RK de orden más alto (RK5, RK8, etc.)?** Porque RK4 es el mejor compromiso costo/precisión para la mayoría de aplicaciones de simulación: cada orden adicional exige más evaluaciones de `f` por paso, con ganancias marginales decrecientes. Es el estándar de facto (equivalente a `ode45`/`RK45` de MATLAB en su núcleo).
3. **¿Qué pasa si el paso `h` es muy grande?** El error crece (más `h^4`) y en sistemas rígidos (*stiff*) el método puede volverse inestable (oscilar sin control), aunque el BTS de Åström-Bell no es un sistema extremadamente rígido.
4. **¿Cómo se relaciona con lo que sí vimos en clase?** Es la misma filosofía iterativa de "avanzar paso a paso" que Jacobi/Gauss-Seidel (S5) o Newton (S3-S4): una fórmula de recurrencia `x_{n+1} = g(x_n)` que se repite hasta cubrir el intervalo de tiempo deseado. La diferencia es que aquí no buscamos una raíz ni una solución estacionaria, sino que reconstruimos toda la trayectoria `x(t)`.
5. **¿Qué determina el "orden del integrador" del que habla la justificación del Avance 1?** Es justamente el exponente de `h` en el error global: orden 1 = Euler, orden 4 = RK4. A mayor orden, menor error para el mismo `h`, pero más costo por paso.

---

## 7. Referencias para profundizar (no citadas en el paper, son solo para estudiar)

- Cualquier libro de métodos numéricos del curso (Chapra & Canale, cap. de EDOs; Burden & Faires, cap. de problemas de valor inicial) trae la deducción completa de RK4 vía series de Taylor.
- El paper que sustenta la elección de RK4 en el Avance 2 es Bajrami, Bajrami y Lameski (2026), que compara Euler vs. RK4 como entorno de simulación para RL — ver el resumen de esa referencia en la respuesta del chat.
