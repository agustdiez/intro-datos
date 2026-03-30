---
title: Introducción a probabilidad
layout: default
parent: Probabilidad y estadistica
nav_order: 1
has_toc: true
---

# Introducción a probabilidad
{: .no_toc }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Definiciones básicas

A continuación, daremos algunas definiciones que usaremos más adelante respecto a términos de probabilidad.


**Espacio muestral** $\Omega$: conjunto de todos los posibles resultados de un experimento.

**Evento**: subconjunto de $\Omega$.

**Evento simple**: subconjunto con un único elemento.

**Operaciones entre eventos** (heredadas de teoría de conjuntos):

- Unión: $A \cup B$
- Intersección: $A \cap B$
- Complemento: $A^c$
- Leyes de De Morgan: $(A \cup B)^c = A^c \cap B^c$ y $(A \cap B)^c = A^c \cup B^c$

### Demosle contexto...

Supongamos que un experimento sea tirar un dado.

En ese caso, el espacio muestral $\Omega$ se comprende de cada número del 1 al 6. Un evento simple sería el `4` por ejemplo.

Los espacios muestrales no tienen que ser necesariamente discretos. Si nuestro experimento es `[Precio del barril de petroleo]`, el espacio muestral estaría comprendido por los números positivos.

### Frecuencia relativa

Si pensamos en un experimento como por ejemplo `tirar dos dados`, sabemos de antemano como se compone el espacio muestral (es decir, todos los posibles eventos simples posibles) y además presuponemos que ciertos eventos tienen mayor posibilidad de suceder. 

>Sumar 7 con dos dados se puede realizar de **múltiples maneras**.
>
>Sumar 12 con los dados puede hacerse de **una sola manera**

$$f_A = \text{Frecuencia Relativa de } A = \frac{\text{cantidad de veces que ocurre } A}{n}$$

Si repetimos el experimento $n$ veces (>30), el teorema de la frecuencia relativa nos permite aproximar la probabilidad a dicha frecuencia.

Podemos entender entonces a la probabilidad de forma intituiva como la frecuencia relativa del evento dentro de una repetición grande del experimento.

---

## Axiomas de Probabilidad

De lo expuesto, podemos desprender ciertos axiomas de la probabilidad, que los podemos interpretar para cada ejemplo que se les ocurra en un espacio de eventos.

{. important}
>Los axiomas de probabilidad son las condiciones mínimas que deben verificarse para que una función definida sobre un conjunto de sucesos determine consistentemente sus probabilidades.


Una función $P$ es una probabilidad si cumple:

- **A1** $P(A) \geq 0$ para todo evento $A$
> No existen probabilidades negativas
- **A2** $P(\Omega) = 1$
> Observen la definición de frecuencia relativa, pensando en el experimento del dado. ¿Cuál es la cantidad de veces que ocurre $\Omega$ = {1,2,3,4,5,6} en las n veces que tiro un dado? Siempre, por lo que n/n = 1.
- **A3** Si $A \cap B = \emptyset$, entonces $P(A \cup B) = P(A) + P(B)$
> Si A=par y B=impar, no se intersecan y la sumatoria de sacar par o impar se suma, y vale lógicamente 1.

Estos axiomas se dan por válidos, y a partir de allí se construye la probabilidad. Se dice que $P$ es una función de probabilidad que actúa sobre $\Omega$ para aquellas funciones que verifican los 3 axiomas. La función devuelve siempre un número que está dentro de los números reales.

### Propiedades derivadas

Estas propiedades se pueden deducir de los axiomas, pero no se expondrá su deducción si no que simplemente buscaremos ejemplificarlas.

- $P(\emptyset) = 0$
> Un evento que no existe dentro del espacio muestral no tiene probabilidad asociada
- $0 \leq P(A) \leq 1$
> Un evento siempre está dentro de $\Omega$
- $P(A \cup B) = P(A) + P(B) - P(A \cap B)$
> Si los eventos no son disjuntos (por ejemplo, que el dado sea impar (A) y que el dado sea {1} (B))
- $P(A^c) = 1 - P(A)$
> El complemento del evento y el evento en sí mismo siempre suman 1, ya que componen el espacio muestral.

### Espacios equiprobables

Si todos los eventos simples tienen igual probabilidad:

$$P(A) = \frac{\#A}{\#\Omega}$$

El $\#$ nos indica la cantidad de elementos.

---

### ¿Cómo calculo la probabilidad de un evento?

Sigamos con el evento de los dados, ¿cómo se calcula exactamente la probabilidad de que el dado sea par?

$$P(A) = \sum_{a_i \in A} P(\{a_i\})$$

Siendo $A = \text{Dado es par}$

Suponiendo que los dados estén equilibrados (espacio equiprobable), sabemos que la cantidad de eventos dentro del espacio tienen la misma probabilidad, siendo:

$$P(\Omega) = 1  | \Omega = \{1,2,3,4,5,6\}$$ 

por lo que $p=1/6$, y al ser $A=\{2\}\cup\{4\}\cup\{6\}$ y siendo todos disjuntos, $P(A) = 1/2$

---


## Probabilidad Condicional

Hasta ahora estamos pensando en experimentos del estilo `tirar dados`, `medir una persona`, pero siempre sin una situación previa o un evento anterior.

La probabilidad condicional se trata de querer determinar como afecta la probabilidad de un evento A el hecho de que previamente ocurrió un evento B. 

> Por ejemplo, intuimos que la probabilidad de que tengamos 10mm de precipitación hoy no es la misma que sabiendo que los días previos llovió cierta cantidad.

>¿Si salió en la ruleta el rojo 3 veces antes, cuánto vale la probabilidad de que salga rojo la próxima vez?

Dado $P(B) > 0$, la probabilidad de $A$ dado que ocurrió $B$:

$$P(A \mid B) = \frac{P(A \cap B)}{P(B)}$$

### Teorema de Multiplicación

$$P(A \cap B) = P(A \mid B) \cdot P(B)$$

### Teorema de Probabilidad Total

Si $A_1, \ldots, A_k$ es una partición de $\Omega$:

$$P(B) = \sum_{i=1}^{k} P(B \mid A_i) \cdot P(A_i)$$

Veamos con un ejemplo la aplicación del teorema

>Un reconocido hospital tiene tres sedes llamadas Almagro, Belgrano y San Martín. En la sede Almagro el 13 % de los pacientes diabéticos están en estado crítico. En la sede Belgrano el 15 % de los pacientes diabéticos están en estado crítico mientras que en la sede San Martín el 23 % de los pacientes diabéticos está en estado crítico. 
>
>Además, de la poblacion total de pacientes diabéticos del hospital un 27 % se atiende en la sede Almagro, un 33 % en la sede Belgrano y un 40 % en la sede San Martín. Si se toma al azar un paciente diabético del hospital, **cualcular la probabilidad de que esté en estado crítico**.

Tenemos en este problema el evento `estar en estado crítico`, peor no tenemos la información total si no que se brinda por partes. Nuestro espacio muestral se reduce a `pacientes diábeticos`. Nuestras particiones del espacio muestral en este caso son las sedes.

Entendemos a $B$ = `el paciente está en estado crítico` de acuerdo a la definición del teorema:

$$A_1 = \text{Almagro}, \quad A_2 = \text{Belgrano}, \quad A_3 = \text{San Martín}$$

Con los datos del problema:

$$P(B \mid A_1) = 0{,}13 \quad P(B \mid A_2) = 0{,}15 \quad P(B \mid A_3) = 0{,}23$$

$$P(A_1) = 0{,}27 \quad P(A_2) = 0{,}33 \quad P(A_3) = 0{,}40$$

Aplicando la expresión de arriba:

$$P(B) = \sum_{i=1}^{3} P(B \mid A_i) \cdot P(A_i)$$

$$P(B) = (0{,}13)(0{,}27) + (0{,}15)(0{,}33) + (0{,}23)(0{,}40)$$

$$P(B) = 0{,}0351 + 0{,}0495 + 0{,}0920 = 0{,}1766$$

La probabilidad de que un paciente diabético elegido al azar esté en estado crítico es aproximadamente **17,66%**.



### Teorema de Bayes

$$P(A_j \mid B) = \frac{P(B \mid A_j) \cdot P(A_j)}{P(B)}$$


Siguiendo con el ejemplo anterior, si se toma un paciente diabético del hospital al azar y resulta que está en estado crítico, **¿cuál es la probabilidad de que se atienda en la sede San Martín?**


Buscamos $P(A_3 \mid B)$, es decir, la probabilidad de que el paciente pertenezca a la sede San Martín dado que está en estado crítico. Aplicando la expresión de Bayes:

$$P(A_3 \mid B) = \frac{P(B \mid A_3) \cdot P(A_3)}{P(B)}$$

Usando los valores del ejercicio anterior, donde ya calculamos $P(B) = 0{,}1766$:

$$P(A_3 \mid B) = \frac{(0{,}23)(0{,}40)}{0{,}1766} = \frac{0{,}0920}{0{,}1766} \approx 0{,}5209$$

La probabilidad de que el paciente en estado crítico se atienda en la sede San Martín es aproximadamente **52,09%**. Tiene cierto sentido, ya que San Martín concentra la mayor proporción de pacientes diabéticos y la tasa de criticidad más alta por centro.

---

## Independencia

$A$ y $B$ son **independientes** si:

$$P(A \cap B) = P(A) \cdot P(B)$$

Equivalentemente (cuando $P(B) > 0$): $P(A \mid B) = P(A)$.

**Propiedades**: si $A$ y $B$ son independientes, también lo son $A$ y $B^c$, $A^c$ y $B$, $A^c$ y $B^c$.

---

## Técnicas de Conteo

Varias veces a lo largo de la materia estaremos realizando distintos experimentos (tuning de modelos) donde una celda de código puede estar corriendo un largo período de tiempo, pero es bueno conocer distintas técnicas de conteo que pueden aparecer en cualquier tipo de experimento.

### Principio de Multiplicación

Si hay $m$ maneras de hacer la tarea 1 y $n$ maneras de hacer la tarea 2, entonces ambas tareas combinadas pueden hacerse de $m \cdot n$ formas.

### Número Factorial

Dados $n$ objetos distintos, la cantidad de formas de ordenarlos es:

$$n! = n \cdot (n-1) \cdot \ldots \cdot 2 \cdot 1$$

**Ejemplo:** ¿De cuántas formas pueden ordenarse 4 libros en una estantería?

$$4! = 4 \cdot 3 \cdot 2 \cdot 1 = 24 \text{ formas}$$

### Variaciones (orden importa)

Cantidad de formas de elegir **ordenadamente** $m$ objetos entre $n$:

$$V(n, m) = n \cdot (n-1) \cdots (n-m+1) = \frac{n!}{(n-m)!}$$

**Ejemplo:** ¿De cuántas formas pueden asignarse el podio entre 8 pilotos?

$$V(8, 3) = \frac{8!}{(8-3)!} = \frac{8!}{5!} = 8 \cdot 7 \cdot 6 = 336 \text{ formas}$$

### Número Combinatorio (orden no importa)

Cantidad de formas de elegir $m$ objetos entre $n$ sin importar el orden:

$$\binom{n}{m} = \frac{n!}{m!(n-m)!}$$

**Ejemplo:** ¿De cuántas formas pueden elegirse 3 pilotos de un grupo de 8?

$$\binom{8}{3} = \frac{8!}{3! \cdot 5!} = \frac{8 \cdot 7 \cdot 6}{3 \cdot 2 \cdot 1} = 56 \text{ formas}$$



### Muestreo con y sin reposición

| Tipo | Resultados posibles |
|---|---|
| Con reposición (k-upla ordenada) | $n^k$ |
| Sin reposición (k-upla ordenada) | $n \cdot (n-1) \cdots (n-k+1)$ |

**Ejemplo:** Se extrae una muestra de $k = 2$ cartas de un mazo de $n = 4$ cartas (A, B, C, D):

- **Con reposición:** $4^2 = 16$ resultados posibles (la primera carta se devuelve antes de extraer la segunda).
- **Sin reposición:** $4 \cdot 3 = 12$ resultados posibles (la primera carta no se devuelve).

---


## Variables aleatorias. ¿Qué son?

Entendamos en este curso a una variable aleatoria como un valor numérico que está afectado por el azar. Definida una variable (supongamos `altura de adolescentes de 16 a 18 años`), no es posible conocer con certeza que valor tomará hasta ser medida, pero sí sabemos que existirá una distribución de probabilidad asociada al conjunto de valores posibles (el $\Omega$ definido en los apartados anteriores). 

Sin saber aún lo que es una distribución, es natural pensar que es raro que alguien de esa edad mida 190cm como también lo sería que mida 50cm. La variable a medir ya nos presupone a anticipar valores esperados de la variable.

Tenemos de dos tipos en función de su naturaleza: **discretas o continuas**.

## Variables Aleatorias Discretas

Una **variable aleatoria** es una función $X: \Omega \to \mathbb{R}$.

Es **discreta** si su rango $R_X$ es finito o numerable.

### Función de probabilidad puntual

$$p_X(x) = P(X = x)$$

Propiedades: $p_X(x) \geq 0$ y $\sum_{x \in R_X} p_X(x) = 1$.

### Función de distribución acumulada

$$F_X(x) = P(X \leq x) = \sum_{y \leq x,\, y \in R_X} p_X(y)$$

Es monótona no decreciente, continua a derecha, con $\lim_{x \to -\infty} F_X(x) = 0$ y $\lim_{x \to +\infty} F_X(x) = 1$.

---

## Distribuciones Discretas

A continuación se dan algunas distribuciones discretas famosas de probabilidad. 

### Bernoulli — $X \sim \text{Ber}(p)$

Un único ensayo con éxito (1) o fracaso (0):

$$P(X = 1) = p, \quad P(X = 0) = 1-p$$

### Binomial — $X \sim \text{Bi}(n, p)$

Cantidad de éxitos en $n$ ensayos de Bernoulli independientes:

$$P(X = k) = \binom{n}{k} p^k (1-p)^{n-k}$$

{: .important}
>La distribución Bernoulli es un caso particular de la Binomial con un único ensayo

### Geométrica — $X \sim G(p)$

Número de repeticiones hasta el primer éxito:

$$P(X = k) = (1-p)^{k-1} p$$

### Hipergeométrica — $X \sim H(n, N, D)$

Cantidad de defectuosos en una muestra de $n$ extraída sin reposición de una población de $N$ con $D$ defectuosos:

$$P(X = k) = \frac{\binom{D}{k}\binom{N-D}{n-k}}{\binom{N}{n}}$$

### Poisson — $X \sim \text{Po}(\lambda)$

$$P(X = k) = \frac{e^{-\lambda} \lambda^k}{k!}, \quad k \in \mathbb{N}$$

Todas estas distribuciones se explican aplicando lo visto en librerías en la notebook de ejercitación. Ver [Notebooks](../notebooks/index.md) para detalle.


---

## Variables Aleatorias Continuas

Una variable es **continua** si entre dos valores de su rango hay infinitos valores posibles. No tiene función de probabilidad puntual; se describe mediante una **función de densidad** $f$.

### Función de densidad

$f$ es densidad de $X$ si:

- $f(x) \geq 0$
- $\int_{\mathbb{R}} f(x)\,dx = 1$
- $P(X \in (a, b)) = \int_a^b f(x)\,dx$

La densidad es la función que aproxima al histograma de frecuencia relativa de la variable.

### Distribución Uniforme — $X \sim U(a, b)$

$$f_X(x) = \frac{1}{b-a}, \quad a \leq x < b$$


### Normal — $X \sim \mathcal{N}(\mu, \sigma^2)$

La distribución más importante para que recuerden. Su densidad tiene forma de campana simétrica centrada en $\mu$.

$$f_X(x) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}$$

- $\mu$ controla el centro de la campana.
- $\sigma$ controla el ancho: mayor $\sigma$ → más achatada.

{: .important}
> La Normal es el resultado asintótico del TCL: la media muestral de cualquier población se aproxima a una Normal cuando $n$ crece.

### Exponencial — $X \sim \text{Exp}(\lambda)$

Modela el tiempo de espera hasta el primer evento de un proceso de Poisson. Es la versión continua de la Geométrica.

$$f_X(x) = \lambda e^{-\lambda x}, \quad x \geq 0$$

- Mayor $\lambda$ → eventos más frecuentes → tiempos de espera más cortos.
- Media: $E(X) = 1/\lambda$

---

## Esperanza y Varianza

La **esperanza** de una variable aleatoria es su valor promedio teórico, ponderado por las probabilidades. Intuitivamente, es el valor que esperaríamos obtener si repitiéramos el experimento muchas veces.

Para variables **discretas**:

$$E(X) = \sum_{x \in R_X} x \cdot p_X(x)$$

Para variables **continuas**:

$$E(X) = \int_{-\infty}^{\infty} x \cdot f_X(x)\,dx$$

La **varianza** mide la dispersión respecto a la esperanza:

$$\text{Var}(X) = E\!\left[(X - E(X))^2\right]$$

Que puede calcularse también como:

$$\text{Var}(X) = E(X^2) - [E(X)]^2$$

**Propiedades útiles** para $a, b$ constantes:

- $E(aX + b) = aE(X) + b$
- $\text{Var}(aX + b) = a^2\,\text{Var}(X)$



| Distribución | $E(X)$ | $\text{Var}(X)$ |
|---|---|---|
| $\text{Ber}(p)$ | $p$ | $p(1-p)$ |
| $\text{Bi}(n,p)$ | $np$ | $np(1-p)$ |
| $G(p)$ | $1/p$ | $(1-p)/p^2$ |
| $\text{Po}(\lambda)$ | $\lambda$ | $\lambda$ |
| $U(a,b)$ | $(a+b)/2$ | $(b-a)^2/12$ |
| $\mathcal{N}(\mu,\sigma^2)$ | $\mu$ | $\sigma^2$ |
| $\text{Exp}(\lambda)$ | $1/\lambda$ | $1/\lambda^2$ |

---
[↑ Volver al índice](./index.md){: .btn .btn-outline }


