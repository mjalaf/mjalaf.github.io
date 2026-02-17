---
title: 'Big-O en Español Parte 4'
description: 'Resumen final de Big-O: gráfico de complejidad, tabla comparativa, estructuras de datos y combinación de complejidades.'
pubDate: 'Jul 05 2022'
---

Este es el último de los Posts sobre la notación Big-O, vamos a revisar los distintos tipos de rendimientos de cada uno de los tipos de expresión y vamos a ver cómo combinar las diferentes complejidades.

Los posts anteriores fueron:
- [Parte 1 - Expresiones Lineales y Constantes](/blog/big-o-part-1/)
- [Parte 2 - Expresiones Cuadráticas, cúbicas y de otras potencias](/blog/big-o-part-2/)
- [Parte 3 - Expresiones Logarítmicas, Exponenciales y Factoriales](/blog/big-o-part-3/)

## Gráfico de complejidad de Big-O

![Chartset](/images/BigO/BigO-8.jpg)

Referencia: [Bigocheatsheet](https://www.bigocheatsheet.com/)

Como se muestra en el análisis y lo fuimos revisando en los posts anteriores, las expresiones constantes y logarítmicas son las de mejor rendimiento y las exponenciales y factoriales las de peor rendimiento.

Veamos la complejidad de estos algoritmos con números:

| Notación Big O | 10 elementos | 100 elementos | 1000 elementos |
| --- | --- | --- | --- |
| O(1) | 1 | 1 | 1 |
| O(n) | 10 | 100 | 1000 |
| O(log n) | 1 | 2 | 3 |
| O(n²) | 100 | 10,000 | 1,000,000 |
| O(2^n) | 1,024 | 1.267e+30 | 1.071e+301 |
| O(n!) | 3,628,800 | 9.33e+167 | Infinito |

Como se puede ver y se revisó en los posts anteriores, a mayor cantidad de elementos, el algoritmo demora mucho más tiempo. Cabe destacar que estos no son números expresados en segundos; de hecho, en la actualidad, los procesadores son capaces de procesar algoritmos mal diseñados en cuestión de milisegundos, pero siempre hay que tener en cuenta el número de elementos y el impacto que puede tener en nuestros algoritmos.

## Operaciones según el tipo de estructura de datos

La siguiente tabla muestra el tipo de Notación Big-O aplicada a cada una de las estructuras de datos y cuándo se utilizan:

![Chartset](/images/BigO/BigO-9.jpg)

Referencia: [Bigocheatsheet](https://www.bigocheatsheet.com/)

Si comparamos un Arreglo con una Pila (stack) y con una Cola (queue) podemos ver (según la tabla) que, tanto en el peor de los casos como en el mejor de los casos, el acceso en un arreglo es más óptimo que en una cola y una pila, pero insertar y eliminar elementos es más óptimo en una pila y una cola, la búsqueda tiene la misma complejidad para los tres O(n) - Lineal.

Es importante entender este tipo de aspectos al trabajar con las diferentes estructuras de datos.

## ¿Qué pasa cuando en un código se aplican varias complejidades?

Supongamos que en nuestro algoritmo tenemos:

```
O(1)
O(n)
O(n²)

O = O(1) + O(n) + O(n²) ==> O(n²)
```

Siempre se considera la mayor complejidad cuando evaluamos un código. De hecho, si convertimos esta notación en una función matemática:

`f(x) = 1 + x + x²`

La gráfica nos quedará de la siguiente manera:

![Combined complexity](/images/BigO/BigO-10.jpg)
