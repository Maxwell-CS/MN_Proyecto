# Métodos Numéricos — Resumen de Clases (Sem 3 a Sem 7)

> UTEC — Ciencias. Profesores: Jimmy Mendoza, Rósulo Pérez, Marco Cuyubamba, Patricia Reynoso, Roger Ponce.
> Bibliografía base: Chapra & Canale, *Métodos numéricos para ingenieros* (7a ed.); Burden & Faires, *Análisis numérico* (7a ed.).
> Herramienta de laboratorio: **MATLAB** (Symbolic Math Toolbox, `jacobian`, `matlabFunction`, `eig`, `diag`, `tril`, `triu`).

Este documento sirve como base de conocimiento para saber **qué métodos se vieron, cómo se aplican, con qué criterios de convergencia/parada y hasta qué profundidad** (teoría + algoritmo + ejemplo + código MATLAB + tipo de examen).

---

## Panorama general

| Semana | Tema | Métodos principales | Laboratorio |
|--------|------|--------------------|-------------|
| S3 | Ecuaciones no lineales (1 variable) | Gráfico, Bolzano, Bisección, Newton, Punto Fijo | — |
| S4 | Sistemas de ecuaciones no lineales (SENL) | Newton vectorizado (Jacobiano), Punto Fijo vectorial | MATLAB: `newton2v`, `newtonVec`, `pfijos` |
| S5 | Métodos iterativos para SEL (lineales) | Jacobi, Gauss-Seidel, convergencia | MATLAB: `jacobi`, `gausseidel`, `diagdom` |
| S6 | Casos de SEL + Valores propios | Kirchhoff, masa-resorte, discretización EDO; `eig`, Gershgorin, Potencia, QR | MATLAB: `gershgorin`, `potencia`, `gs_c` |
| S7 | Interpolación polinomial | Vandermonde, Lagrange, Diferencias Divididas/Newton, error | — |

**Profundidad general alcanzada:** definición formal → interpretación geométrica → teorema de convergencia → algoritmo pseudocódigo → ejemplo numérico iterativo → implementación MATLAB → aplicación a problemas de ingeniería → ejercicios tipo examen (EP/EL). No se profundiza en demostraciones completas (salvo la de Jacobi con matriz definida positiva) ni en estabilidad numérica avanzada.

---

# S3 — Ecuaciones No Lineales

**Problema:** dada `f: R → R`, hallar `x̂` tal que `f(x̂) = 0`.

**Objetivos:** localizar/delimitar raíces (gráfico y analítico); resolver con Bisección, Newton y Punto Fijo; determinar convergencia y eficiencia.

## 1. Métodos cerrados

### 1.1 Método gráfico
- Se usa para **obtener un valor inicial** para otros métodos.
- Ejemplo: garantizar existencia de raíz positiva de `e^(x−1) − cos(πx/4) = 0`.

### 1.2 Teorema de Bolzano
Sea `f` continua en `[a,b]` con `f(a)·f(b) < 0` ⇒ existe al menos un `x̂ ∈ [a,b]` con `f(x̂)=0`.
- La continuidad es esencial. Dentro del intervalo puede haber más de una raíz.
- Si `f(a)f(b) > 0` no se puede concluir (puede haber 0, 1 o 2 raíces).

### 1.3 Método de la Bisección
Búsqueda incremental: se divide el intervalo a la mitad; se conserva el subintervalo con cambio de signo.

**Algoritmo**
```
Input: a, b con f(a)f(b)<0, Tol, N (máx iteraciones)
for i = 0 to N:
    c = (a+b)/2
    if f(c)=0: terminar
    if f(c)f(a) > 0: a = c
    else:            b = c
return c
```

**Convergencia:** si `f` es continua y cambia de signo en `[a,b]`, la sucesión converge a una raíz.

**Error y tolerancia (general a iterativos):**
- Error absoluto: `ξn = |xn − x(n−1)|`
- Error relativo: `δn = ξn / |xn|`
- Tolerancia: `ξn ≤ Tol` (o `δn ≤ Tol`)

**Mínimo número de iteraciones:**
```
(b − a) / 2^(n+1) ≤ Tol
n = ⌈ ln( (b−a)/(2·Tol) ) / ln 2 ⌉
```
Ejemplo EP 2023-2: `(e^x − x^3)/√x = 0`, `[1,2]`, `Tol=1e-3` ⇒ `n = 9`.

**Ejemplo aplicado (Rezagados 2024-1):** `i(t)=10e^(−t/2)cos(3t)` mA, hallar primer instante en que `i=2` mA con `[0, 0.5]`, 2 iteraciones.

---

## 2. Métodos abiertos

Requieren uno o dos valores iniciales que **no necesariamente encierran la raíz**. Convergen más rápido pero son menos estables.

### 2.1 Método de Newton–Raphson
Tangente en `(xi, f(xi))`; donde corta al eje x es la nueva aproximación.

**Fórmula:**
```
x(i+1) = xi − f(xi) / f'(xi)
```
No funciona si `f'(xi) = 0`.

**Algoritmo**
```
Input: f, x0, Tol (error relativo)
Obtener f'(x)
while δr > Tol:
    if f'(xk)=0: terminar con error
    x(k+1) = xk − f(xk)/f'(xk)
    δr = |x(k+1) − xk| / |x(k+1)|
return x(k+1)
```

**Convergencia (cuadrática):** existe raíz `x*`; `f' ≠ 0` en `I=[x*−δ, x*+δ]`; `f''` continua en `I`; `x0 ∈ I` suficientemente cercano. Si `M·δ < 1` con `M = max |f''(x)| / (2|f'(x)|)`, entonces
```
|x(k+1) − x*| ≤ M |xk − x*|²
```

**Ejemplo:** `f(x)=e^(x−1) − cos(πx/4)`, `f'(x)=e^(x−1) + (π/4)sin(πx/4)`, con `x0=0.1` (y también `x0=1.5`), 4 iteraciones + error relativo. Converge a `≈0.7924`.

### 2.2 Método del Punto Fijo
Punto fijo de `g`: `g(p)=p`. Equivale a resolver `g(x)=x`, es decir raíz de `f(x)=g(x)−x`.
- Reescribir `f(x)=0` como `x = g(x)`. La elección de `g` **no es única** (unas convergen, otras divergen).
- Geométricamente: intersección de `y=g(x)` con `y=x`.

**Algoritmo**
```
Input: g, x0, Tol (error relativo)
while δr > Tol:
    x(k+1) = g(xk)
    δr = |x(k+1) − xk| / |x(k+1)|
return x(k+1)
```

**Convergencia:** `g` continua en `[a,b]` y derivable en `⟨a,b⟩` tal que
1. `g(x) ∈ [a,b]` para todo `x ∈ [a,b]` (existencia), y
2. `|g'(x)| < 1` en `⟨a,b⟩` (contracción)

⇒ existe un **único** punto fijo `p` y la sucesión converge a `p`.

**Ejemplo:** `x³ + 4x² − 10 = 0` con `x0=1.8`. Dos despejes: `g1 = x − x³ − 4x² + 10` (diverge) y `g2 = (10/(4+x))^(1/2)` (converge a `≈1.36523`).
- Ejemplo de verificación: `x² − x = 0` en `[0.64,1.44]`, `g(x)=√x`; `max|g'| = 0.625 < 1`; `g([0.64,1.44]) = [0.8,1.2] ⊂ [0.64,1.44]`.
- Ejercicio EP 2023-2: veracidad de convergencia con `f(x)=e^x − x(e−1) − 1`, `g(x)=(e^x − 1)/(e−1)` en `[−1,1]`.

**Conclusiones S3:** métodos abiertos más rápidos pero menos estables; la convergencia depende del valor inicial; control de tolerancia/error es clave.

---

# S4 — Sistemas de Ecuaciones No Lineales (SENL)

**Problema:** dadas `f, g: R² → R`, hallar `(x̂,ŷ)` con `f=0` y `g=0`.
- Idea gráfica: intersección de las curvas de nivel cero (ej. parábola vs coseno).

## 1. Método de Newton para SENL
Se generaliza con el **desarrollo de Taylor** y la **matriz Jacobiana**.

```
F(x) ≈ F(x0) + J(x0)(x − x0)
x1 = x0 − J⁻¹(x0) F(x0)
```
Sin inversa (más práctico): `x1 = x0 − Δx`, donde `J(x0) Δx = −F(x0)`.

**Iteración general:**
```
x(k+1) = x(k) − J⁻¹(x(k)) F(x(k)),  k = 0,1,2,...
```
Jacobiano (2 ecuaciones):
```
J = [ ∂f/∂x  ∂f/∂y ;
      ∂g/∂x  ∂g/∂y ]
```

**Algoritmo vectorizado**
```
Input: F(x), x(0), Tol
Calcular J(x)
while δr > Tol:
    if det(J(x(k)))=0: terminar con error
    x(k+1) = x(k) − J⁻¹(x(k)) F(x(k))
    δr = ||x(k+1) − x(k)||∞ / ||x(k+1)||∞
return x(k+1)
```

**Norma infinito:**
- Vector: `||x||∞ = max_i |xi|`
- Matriz: `||A||∞ = max_i Σ_j |aij|`

**Ejemplo:** `x² + xy − 10 = 0`, `3xy² + y − 57 = 0`, semilla `(1.5, 3.5)`, 3 iteraciones. Jacobiano `[[2x+y, x],[3y², 6xy+1]]`.
- Inversa 2×2: `1/(a11a22 − a21a12) · [[a22, −a12],[−a21, a11]]`.

**Ejercicios tipo examen:** EP 2023-2 (V/F unicidad con `sec(x) − e^(y²)=0`, `4x²+5y=0`); EP 2024-2 (flujo uniforme, hallar `[V α]ᵀ`).

## 2. Método de Punto Fijo para SENL
Punto fijo de `G: R²→R²`: `p = G(p)`, con `G=(u,v)`:
```
x = u(x,y),  y = v(x,y)
```
Cualquier punto fijo de `G` es raíz del SENL original.

**Convergencia:** `G` continua en `D ⊆ Rⁿ` con `G(x) ∈ D`; derivadas parciales continuas y constante `K<1` con `|∂gj/∂xi| ≤ K/n` en `D` ⇒ único punto fijo y `x(k)=G(x(k−1))` converge.

**Ejemplo trabajado:** `x1² −10x1 + x2² +8=0`, `x1x2² + x1 −10x2 +8=0` transformado a
`x1=(x1²+x2²+8)/10`, `x2=(x1x2²+x1+8)/10`; verificar existencia y contracción con `K=0.9` en `D=[0,1.5]²`.

**Criterio de parada:** `||x(k+1) − x(k)||∞ / ||x(k+1)||∞ < Tol`.

**Ejemplo 1:** mismo sistema, semilla `(0.3, 1.25)`, 3 iteraciones, con dos despejes distintos.

## 3. Laboratorio S4 (MATLAB)
- `syms x y`, `jacobian(F(x,y),[x,y])`, `matlabFunction`, evaluación `dFfun(3,4)`.
- `newton2v(F,x0,Tol)`: calcula Jacobiano simbólico y aplica Newton con error relativo infinito.
- `newtonVec(F,x0,Maxiter)`: versión genérica n×n usando `syms x [n 1]`.
- `pfijos(G,x0,Tol)`: iteración de punto fijo vectorial.
- Ejercicios: sistema 3×3 (`3x−cos(yz)=1/2`, etc.) con `Tol=1e-4`; y un sistema de 2 circunferencias que se cruzan (EL1 2022-2) usando Newton con `Tol=1e-5`, reportando `n`, `w`, `|F(w)|`.

**Conclusiones S4:** Newton es eficiente pero requiere Jacobiano e inversa en cada iteración (costoso) y buen punto semilla; Punto Fijo requiere construir `G` que garantice contracción.

---

# S5 — Métodos Iterativos para SEL

**Problema:** resolver `Ax = b` (A n×n, solución única) de forma iterativa.
**Ventajas frente a métodos directos:** controlan mejor la amplificación de errores; eficientes para matrices **esparsas** y de gran tamaño (ingeniería, fluidos, estructuras).

**Esquema de punto fijo para SEL:** transformar `Ax=b` en `x = Tx + c`, con iteración
```
x(k+1) = T x(k) + c,   k = 0,1,2,...
```

## Descomposición A = D − L − U
- `D`: diagonal de A.
- `L`: triangular inferior (negativos de los elementos bajo la diagonal).
- `U`: triangular superior (negativos de los elementos sobre la diagonal).

## 1. Método de Jacobi
Despejando `xi` de la i-ésima ecuación y usando **solo valores de la iteración anterior**:
```
x_i^(k+1) = ( b_i − Σ_{j≠i} a_ij x_j^(k) ) / a_ii,  a_ii ≠ 0
```
**Forma matricial:**
```
Tj = D⁻¹(L + U)        (matriz de iteración de Jacobi)
cj = D⁻¹ b             (vector de Jacobi)
x(k+1) = Tj x(k) + cj
```

## 2. Método de Gauss–Seidel
Usa los valores **ya actualizados** `x1..x(i−1)` de la misma iteración:
```
x_i^(k+1) = ( b_i − Σ_{j<i} a_ij x_j^(k+1) − Σ_{j>i} a_ij x_j^(k) ) / a_ii
```
**Forma matricial:**
```
Tgs = (D − L)⁻¹ U
cgs = (D − L)⁻¹ b
x(k+1) = Tgs x(k) + cgs
```

**Ejemplo común:** sistema 4×4 (coefs `−6,8,5,8`), semilla `(20,5,30,10)`, comparar Jacobi vs Gauss-Seidel. También sistema 4×4 con solución `x*=[1 2 −1 1]ᵀ` y `Tol=1e-3`.

## 3. Convergencia

### Diagonal estrictamente dominante (DED)
A es DED por filas si `|a_ii| > Σ_{j≠i} |a_ij|`.
**Teorema:** si A es DED, Jacobi y Gauss-Seidel **convergen para cualquier vector inicial**.
> Nota: el criterio DED se aplica a la **matriz A**; si no es DED no se puede afirmar nada (hay que usar radio espectral). Es posible reordenar filas para lograr DED.

### Radio espectral
```
ρ(T) = max{|λ| : λ valor propio de T}
det(T − λI) = 0
```
**Teorema:** `x(k+1)=Tx(k)+c` converge a la solución única **si y sólo si** `ρ(T) < 1`.
> Se aplica a la **matriz de iteración T** de cada método (Tj o Tgs).

### Matriz definida positiva
Si A es simétrica y definida positiva (`xᵀAx > 0`, todos los valores propios positivos) ⇒ Jacobi converge. Para `Tj = I − D⁻¹A`, `λi(Tj) = 1 − λi(A)/λi(D)`, luego `|λi(Tj)| < 1`.

**Propiedad:** cuando ambos convergen, **Gauss-Seidel converge más rápido** que Jacobi. Existen casos en que Jacobi converge y Gauss-Seidel no.

**Ejemplo 4:** `3x1+x2=7`, `2x1+5x2=9` → hallar `Tgs`, DED, `ρ(Tgs)=2/15`, convergencia, 2 iteraciones.

## 4. Laboratorio S5 (MATLAB)
- `D=diag(diag(A))`, `L=-tril(A,-1)`, `U=-triu(A,1)`.
- `jacobi(A,b,x0,Tol)`, `gausseidel(A,b,x0,Tol)` (error relativo infinito).
- `diagdom(A)` → devuelve 1/0 si A es DED.
- Radios espectrales: `max(abs(eig(Tj)))`, `max(abs(eig(Tgs)))`.
- Ejercicio tipo EL1 2024-2: sistema con parámetro `c ∈ [−15,−3]`; contar casos convergentes, guardar soluciones `xsol` e iteraciones `niterv`.
- Actividad: hallar Tj y Tgs en función de `p,q` y analizar convergencia.

## 5. Aplicaciones (en S5 y S6)
- **Masa-resorte (EP 2023-2):** 3 bloques / 4 resortes; sistema tridiagonal `[[3k,−2k,0],[−2k,3k,−k],[0,−k,k]]x = [m1g,m2g,m3g]ᵀ`. Resolver con Jacobi/Gauss-Seidel y comparar con radio espectral.
- **Barra de calor (EP 2023-2):** discretización da `−T(i−1) + (2+ch²)T_i − T(i+1) = ch²Ta`, `c=0.001 cm⁻²`, `L=200`, `h=50`, 3 incógnitas. Formular Ax=b DED, analizar convergencia, hallar Tgs/cgs, 2 iteraciones con semilla `[10,20,30]ᵀ`.

---

# S6 — Casos de SEL y Valores Propios

## Parte A: Aplicaciones de SEL (formulación de modelos)

### Caso 1: Circuitos eléctricos (Leyes de Kirchhoff)
- **Ley de nodos:** suma de corrientes entrantes = salientes.
- **Ley de lazos:** suma de cambios de voltaje en un recorrido cerrado = 0 (signos según +→− o −→+).
- Ejemplo 3 ecuaciones: `i1 − i2 + i3 = 0`; `(R1+R4)i1 + R2i2 = V1−V2`; `R2i2 + R3i3 = V3−V2`.
- Ejercicios tipo EP 2024-2: circuito con R1..R13 y V1..V5; plantear `Ax=b` con `b1,b3>0` y `b2,b4<0`; analizar convergencia de Jacobi por radio espectral; 2 iteraciones con semilla `[3,−3,3,−3]ᵀ`.

### Caso 2: Sistema masa-resorte
- Balance de fuerzas (ley de Hooke) para cada masa.
- Ejemplo 3 masas: matriz tridiagonal (arriba).
- Ejercicio 4 resortes en serie, `F=2000 N`, `k1..k4`; formular y resolver por método directo e iterativo.
- Ejercicio EP 2024-2 con `K11..K22`, `F`, diagonal positiva, convergencia Jacobi, 2 iteraciones + error.

### Caso 3: Discretización de una EDO
- Aproximación de **diferencia finita central** de la 2da derivada:
```
d²T/dx² |_{xi} = (T(i−1) − 2T_i + T(i+1)) / Δx²
```
- Ecuación de calor: `d²T/dx² + c(Ta − T) = 0` ⇒
```
−T(i+1) + (2 + c h²) T_i − T(i−1) = c h² Ta,  i = 1..n−1
```
- **Condiciones de frontera:** Dirichlet (valor de T), Neumann (flujo/derivada), Mixtas/Robin (convección).
- Ejercicios: barra 10 m, `Ta=20`, `T(0)=40`, `T(10)=200`, `c=0.02`, `n=5`; EP 2024-1 pared plana con generación de calor `q`, `k=0.5`, `q=10⁴`, `L=10 cm`, `T0=20`, `T4=50`, 3 nodos; discretizar, 2 iteraciones Gauss-Seidel matricial, convergencia por radio espectral.

## Parte B: Valores propios (Laboratorio S6)

### Definición
`λ` valor propio de A si `Av = λv`, `v ≠ 0`. Polinomio característico `p(λ)=det(A−λI)`; sus raíces son los valores propios. Vectores propios al resolver `Av=λv`.

### MATLAB: `[V,D] = eig(A)`

### Teorema de Gershgorin
Cada valor propio `λ` satisface, para algún `i`:
```
|λ − a_ii| ≤ Σ_{j≠i} |a_ij|
```
es decir, cae en algún **disco de Gershgorin** `D_i` (centro `a_ii`, radio suma de no diagonales). Código MATLAB `gershgorin(A)` dibuja los discos.

### Aplicación: PageRank
- Grafo dirigido de páginas (nodos = páginas, aristas = enlaces).
- `r_i` proporcional a la suma de clasificaciones de páginas que enlazan a `i`.
- Se plantea `r = α A r` ⇒ `A r = λ r` con `λ = 1/α`. Se toma el **único valor propio real dominante**, `α = 1/λ`, y `r` = vector propio normalizado (`r = r/sum(r)`).

### Método de la Potencia
Aproxima el **valor propio dominante**:
```
Input: A, x0, Maxiter
for k=1:Maxiter
    y1 = A*x0
    [maxi,pos] = max(abs(y1))
    u  = y1(pos)      % valor propio
    x1 = y1/u         % vector propio normalizado
    x0 = x1
```
Ejemplo: `A=[[2,−12],[1,−5]]`, `x0=[1;1]`.

### Método de Descomposición QR
Estima todos los valores propios de A:
1. Factorizar `A = Q1 R1` (Gram-Schmidt).
2. `A1 = R1 Q1`, factorizar `A1 = Q2 R2`.
3. Repetir `A_{k} = R_k Q_k`.
La sucesión converge a una matriz **triangular (o casi)** cuyos elementos diagonales son los valores propios. Requiere muchas iteraciones (ej. 500).
- `gs_c(A)`: factorización QR por Gram-Schmidt modificado.
- Caso real: `A=[[1,−1,4],[3,2,−1],[2,1,−1]]`, `λ = 3,−2,1`.
- Caso complejo: bloque `T(2:3,2:3)` con `eig` para valores complejos.

**Conclusiones S6:** valores propios clave en PageRank; Gershgorin localiza valores propios; método de la potencia halla el dominante (útil en convergencia de iterativos y estabilidad).

---

# S7 — Interpolación Polinomial

**Objetivos:** construir polinomios interpolantes a partir de datos; estimar cota de error.
**Contexto:** pasar de datos discretos (tablas/experimentos) a un modelo continuo `f(x)`.

## 1. Planteamiento
Dados `(x0,y0),...,(xn,yn)`, hallar `P` con `P(xi)=yi`. El interpolador puede ser:
- **Polinomio** `Pn(x)` (grado ≤ n).
- **Spline** `S(x)` (polinomio de grado ≤ 3 a tramos).

**Teorema (unicidad):** si `x0,...,xn` son distintos, existe un **único** polinomio de grado ≤ n que interpola los `n+1` puntos.

## 2. Forma algebraica (Vandermonde)
Plantea el sistema con la **matriz de Vandermonde**:
```
[ x0^n ... x0 1 ] [an]   [y0]
[ x1^n ... x1 1 ] [..] = [y1]
[ ...            ] [a0]   [...]
```
**Ejemplo:** puntos `(−2,−27), (0,−1), (1,0)` ⇒ `P2(x) = −4x² + 5x − 1`.

## 3. Interpolación de Lagrange
Polinomios básicos:
```
Lk(x) = Π_{i≠k} (x − xi)/(xk − xi)
Lk(xj) = 1 si k=j, 0 si k≠j ;  grado n
```
Polinomio interpolante:
```
Pn(x) = Σ_{k=0}^{n} yk Lk(x)
```
**Ejemplo:** mismos 3 puntos; `L0 = (1/6)x² − (1/6)x`, `L1 = −(1/2)x² − (1/2)x + 1`, `L2 = (1/3)x² + (2/3)x` ⇒ `P2 = −4x² + 5x − 1`.

**Ejercicio:** puntos `(0,1), (0.8,0.4), (2,0.2)`.

## 4. Diferencias divididas y forma de Newton
- 1er orden: `f[xi,xi+1] = (f(xi+1) − f(xi)) / (xi+1 − xi)`
- 2do orden: `f[xi,xi+1,xi+2] = (f[xi+1,xi+2] − f[xi,xi+1]) / (xi+2 − xi)`
- Orden k: `f[xi,...,xi+k] = (f[xi+1,...,xi+k] − f[xi,...,xi+k−1]) / (xi+k − xi)`

Se organizan en una **tabla de diferencias divididas**.

**Forma de Newton:**
```
Pn(x) = f[x0] + Σ_{k=1}^{n} f[x0,...,xk] (x−x0)(x−x1)...(x−x(k−1))
```
Ejemplo con los 3 puntos: `P2(x) = −27 + 13(x+2) − 4(x+2)(x)` = `−4x²+5x−1`.

## 5. Error de interpolación
```
En(x,f) = f(x) − Pn(x)
```
**Teorema:** si `f` tiene `n+1` derivadas continuas en `[a,b]`, existe `ξ(x)` con
```
En(x,f) = [ f^(n+1)(ξ(x)) / (n+1)! ] · (x−x0)(x−x1)...(x−xn)
```
La **cota de error** es `max_{[a,b]} |En(x,f)|`. Se acota en dos pasos:
1. `max |f^(n+1)(x)|`
2. `max |Π(x−xi)|`

**Ejemplo:** `f(x)=cos(x)`, `x0=0, x1=0.6, x2=0.9`, aproximar `f(0.45)`. Error exacto `≈0.0023`; cota `|E2| ≤ (|sin(0.9)|/6)·0.05704 ≈ 0.0074468`.

## 6. Regla del término siguiente
Cuando `f` es desconocida y no se conoce `f^(n+1)`:
```
f^(n+1)(ξ(x))/(n+1)! = f[x0,...,xn,x(n+1)]
En(x,f) = f[x0,...,xn,x(n+1)] · (x−x0)...(x−xn)
```
**Ejemplo:** `f(x)=x²e^(−x/2)`, puntos `1.1, 2, 3.5, 5, 7.1`; `P1(x)=0.6981+0.8593(x−1.1)`; error estimado en `x=1.75`: `E1 = −0.1755(0.65)(−0.25) = 0.02852`.

**Actividad:** estimar variable de proceso a las 10h y 15h con datos horarios, usando Lagrange; comparar con interpolación lineal / mínimos cuadrados.

---

## Criterios de parada usados (resumen)

| Contexto | Criterio |
|----------|----------|
| Raíces 1D (error absoluto) | `ξn = \|xn − x(n−1)\| ≤ Tol` |
| Raíces 1D / SENL (error relativo) | `δr = \|x(k+1) − xk\| / \|x(k+1)\| ≤ Tol` |
| SENL / SEL (norma infinito) | `\|\|x(k+1) − x(k)\|\|∞ / \|\|x(k+1)\|\|∞ ≤ Tol` |
| Bisección (nº iteraciones) | `(b−a)/2^(n+1) ≤ Tol` |
| SEL convergencia | `ρ(T) < 1` y/o A diagonal estrictamente dominante |

---

## Mapa de profundidad por método

| Método | Teoría | Algoritmo | Ejemplo resuelto | MATLAB | Examen |
|--------|:------:|:---------:|:----------------:|:------:|:------:|
| Gráfico / Bolzano | ✅ | — | ✅ | — | — |
| Bisección | ✅ (convergencia + nº iter) | ✅ | ✅ | — | ✅ EP2023-2, Rez2024-1 |
| Newton 1D | ✅ (convergencia cuadrática) | ✅ | ✅ | — | — |
| Punto Fijo 1D | ✅ (contracción) | ✅ | ✅ | — | ✅ EP2023-2 |
| Newton SENL | ✅ (Jacobiano) | ✅ | ✅ | ✅ | ✅ EP2023-2, EP2024-2 |
| Punto Fijo SENL | ✅ (K<1) | ✅ | ✅ | ✅ | — |
| Jacobi SEL | ✅ | ✅ | ✅ | ✅ | ✅ EP/EL |
| Gauss-Seidel SEL | ✅ | ✅ | ✅ | ✅ | ✅ EP/EL |
| DED / radio espectral | ✅ | ✅ | ✅ | ✅ | ✅ |
| Gershgorin | ✅ | ✅ | ✅ | ✅ | — |
| Potencia / QR | ✅ | ✅ | ✅ | ✅ | — |
| PageRank | ✅ (aplicación) | ✅ | ✅ | ✅ | — |
| Lagrange | ✅ | ✅ | ✅ | — | actividad |
| Diferencias divididas / Newton | ✅ | ✅ | ✅ | — | — |
| Error de interpolación | ✅ (teorema + cota) | ✅ | ✅ | — | actividad |

---

## Fórmulas clave (chuleta)

```
Bisección:        c = (a+b)/2 ;  n = ⌈ ln((b−a)/(2Tol))/ln2 ⌉
Newton:           x(k+1) = xk − f(xk)/f'(xk)
Punto fijo:       x(k+1) = g(xk)          |g'|<1
Newton SENL:      x(k+1) = x(k) − J⁻¹(x(k)) F(x(k))
Jacobi:           Tj  = D⁻¹(L+U),  cj  = D⁻¹b
Gauss-Seidel:     Tgs = (D−L)⁻¹U,  cgs = (D−L)⁻¹b
Convergencia:     ρ(T) = max|λ| < 1 ;  DED: |aii| > Σ|aij|
Lagrange:         Pn(x) = Σ yk Lk(x),  Lk = Π (x−xi)/(xk−xi)
Newton interp.:   Pn(x) = f[x0] + Σ f[x0..xk] Π(x−xi)
Error interp.:    En = f^(n+1)(ξ)/(n+1)! · Π(x−xi)
```
