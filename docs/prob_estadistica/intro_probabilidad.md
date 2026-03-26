---
title: Introducción a probabilidad
layout: default
parent: Conceptos de probabilidad y estadística
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

## Técnicas de Conteo

### Principio de Multiplicación

Si hay $m$ maneras de hacer la tarea 1 y $n$ maneras de hacer la tarea 2, entonces ambas tareas combinadas pueden hacerse de $m \cdot n$ formas.

### Número Factorial

Dados $n$ objetos distintos, la cantidad de formas de ordenarlos es:

$$n! = n \cdot (n-1) \cdot \ldots \cdot 2 \cdot 1$$

### Variaciones (orden importa)

Cantidad de formas de elegir **ordenadamente** $m$ objetos entre $n$:

$$V(n, m) = n \cdot (n-1) \cdots (n-m+1) = \frac{n!}{(n-m)!}$$

### Número Combinatorio (orden no importa)

Cantidad de formas de elegir $m$ objetos entre $n$ sin importar el orden:

$$\binom{n}{m} = \frac{n!}{m!(n-m)!}$$

### Muestreo con y sin reposición

| Tipo | Resultados posibles |
|---|---|
| Con reposición (k-upla ordenada) | $n^k$ |
| Sin reposición (k-upla ordenada) | $n \cdot (n-1) \cdots (n-k+1)$ |

---

## Espacios Muestrales y Eventos

**Espacio muestral** $\Omega$: conjunto de todos los posibles resultados de un experimento.

**Evento**: subconjunto de $\Omega$.

**Evento simple**: subconjunto con un único elemento.

**Operaciones entre eventos** (heredadas de teoría de conjuntos):

- Unión: $A \cup B$
- Intersección: $A \cap B$
- Complemento: $A^c$
- Leyes de De Morgan: $(A \cup B)^c = A^c \cap B^c$ y $(A \cap B)^c = A^c \cup B^c$

---

## Axiomas de Probabilidad

Una función $P$ es una probabilidad si cumple:

- **A1** $P(A) \geq 0$ para todo evento $A$
- **A2** $P(\Omega) = 1$
- **A3** Si $A \cap B = \emptyset$, entonces $P(A \cup B) = P(A) + P(B)$

### Propiedades derivadas

- $P(\emptyset) = 0$
- $0 \leq P(A) \leq 1$
- $P(A \cup B) = P(A) + P(B) - P(A \cap B)$
- $P(A^c) = 1 - P(A)$

### Espacios equiprobables

Si todos los eventos simples tienen igual probabilidad:

$$P(A) = \frac{\#A}{\#\Omega}$$

---

## Probabilidad Condicional

Dado $P(B) > 0$, la probabilidad de $A$ dado que ocurrió $B$:

$$P(A \mid B) = \frac{P(A \cap B)}{P(B)}$$

### Teorema de Multiplicación

$$P(A \cap B) = P(A \mid B) \cdot P(B)$$

### Teorema de Probabilidad Total

Si $A_1, \ldots, A_k$ es una partición de $\Omega$:

$$P(B) = \sum_{i=1}^{k} P(B \mid A_i) \cdot P(A_i)$$

### Teorema de Bayes

$$P(A_j \mid B) = \frac{P(B \mid A_j) \cdot P(A_j)}{P(B)}$$

---

## Independencia

$A$ y $B$ son **independientes** si:

$$P(A \cap B) = P(A) \cdot P(B)$$

Equivalentemente (cuando $P(B) > 0$): $P(A \mid B) = P(A)$.

**Propiedades**: si $A$ y $B$ son independientes, también lo son $A$ y $B^c$, $A^c$ y $B$, $A^c$ y $B^c$.

---

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

### Bernoulli — $X \sim \text{Ber}(p)$

Un único ensayo con éxito (1) o fracaso (0):

$$P(X = 1) = p, \quad P(X = 0) = 1-p$$

### Binomial — $X \sim \text{Bi}(n, p)$

Cantidad de éxitos en $n$ ensayos de Bernoulli independientes:

$$P(X = k) = \binom{n}{k} p^k (1-p)^{n-k}$$

### Geométrica — $X \sim G(p)$

Número de repeticiones hasta el primer éxito:

$$P(X = k) = (1-p)^{k-1} p$$

### Hipergeométrica — $X \sim H(n, N, D)$

Cantidad de defectuosos en una muestra de $n$ extraída sin reposición de una población de $N$ con $D$ defectuosos:

$$P(X = k) = \frac{\binom{D}{k}\binom{N-D}{n-k}}{\binom{N}{n}}$$

### Poisson — $X \sim \text{Po}(\lambda)$

$$P(X = k) = \frac{e^{-\lambda} \lambda^k}{k!}, \quad k \in \mathbb{N}$$

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

---

## Esperanza y Varianza de Sumas

Para $X_1, \ldots, X_n$ variables aleatorias:

$$E\!\left(\sum X_i\right) = \sum E(X_i)$$

Si además son independientes:

$$\text{Var}\!\left(\sum X_i\right) = \sum \text{Var}(X_i)$$

Para $n$ variables i.i.d. con media $\mu$ y varianza $\sigma^2$, definiendo $S_n = \sum X_i$ y $\bar{X}_n = S_n/n$:

$$E(S_n) = n\mu, \quad \text{Var}(S_n) = n\sigma^2$$

$$E(\bar{X}_n) = \mu, \quad \text{Var}(\bar{X}_n) = \frac{\sigma^2}{n}$$




