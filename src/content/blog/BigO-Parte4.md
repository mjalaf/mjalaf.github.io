---
title: 'Big-O en Español Parte 4'
description: 'Análisis del rendimiento de cada tipo de expresión Big-O y cómo combinar diferentes complejidades.'
pubDate: 'Jul 05 2022'
---

Este es el último post de la serie sobre la notación Big-O. En este artículo, analizaremos el rendimiento de cada tipo de expresión y aprenderemos cómo combinar diferentes complejidades en un algoritmo.

---

## Posts Anteriores

- [Parte 1 - Expresiones Lineales y Constantes](https://mjalaf.github.io/2022/06/17/big-o.html)
- [Parte 2 - Expresiones Cuadráticas, Cúbicas y de Otras Potencias](https://mjalaf.github.io/2022/06/24/big-o-2.html)
- [Parte 3 - Expresiones Logarítmicas, Exponenciales y Factoriales](https://mjalaf.github.io/2022/06/30/big-o-3.html)

---

## Gráfico de Complejidad de Big-O

![Chartset](/assets/img/BigO/BigO-8.jpg)

**Referencia:** [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)

Como vimos en los posts anteriores, las expresiones constantes (**O(1)**) y logarítmicas (**O(Log n)**) son las de mejor rendimiento, mientras que las exponenciales (**O(2^n)**) y factoriales (**O(n!)**) son las menos eficientes.

---

## Complejidad de Algoritmos con Números

La siguiente tabla ilustra cómo aumenta el tiempo de procesamiento a medida que crece el número de elementos:

| Notación Big-O   | 10 elementos | 100 elementos     | 1000 elementos      |
|------------------|--------------|-------------------|---------------------|
| **O(1)**         | 1            | 1                 | 1                   |
| **O(n)**         | 10           | 100               | 1000                |
| **O(Log n)**     | 1            | 2                 | 3                   |
| **O(n²)**        | 100          | 10,000            | 1,000,000           |
| **O(2^n)**       | 1024         | 1.27e+30          | 1.07e+301           |
| **O(n!)**        | 3,628,800    | 9.33e+157         | Infinito            |

**Nota:** Estos valores no representan tiempos en segundos. Gracias a los procesadores modernos, incluso los algoritmos mal diseñados pueden ejecutarse en milisegundos para volúmenes de datos pequeños. Sin embargo, es crucial entender cómo el crecimiento exponencial o factorial puede impactar significativamente nuestros sistemas cuando los datos aumentan.

---

## Operaciones Según el Tipo de Estructura de Datos

La siguiente tabla muestra la notación Big-O asociada a diferentes estructuras de datos:

![Chartset](/assets/img/BigO/BigO-9.jpg)

**Referencia:** [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)

### Comparación de Arreglo, Pila (Stack) y Cola (Queue)

- **Acceso:** Los arreglos tienen mejor rendimiento en acceso aleatorio (**O(1)**) en comparación con pilas y colas.
- **Inserción y Eliminación:** Las pilas y colas son más eficientes (**O(1)**) para estas operaciones.
- **Búsqueda:** Para los tres casos, la búsqueda tiene una complejidad lineal (**O(n)**).

Es esencial comprender estas diferencias para tomar decisiones informadas al seleccionar estructuras de datos.

---

## ¿Qué Pasa Cuando un Código Tiene Múltiples Complejidades?

Si un algoritmo combina diferentes complejidades, siempre se considera la **mayor** al evaluar el rendimiento general.

Por ejemplo:

```plaintext
O(1) 
O(n) 
O(n²)

O = O(1) + O(n) + O(n²) = O(n²)
```

En términos matemáticos:

```plaintext
f(x) = 1 + x + x²
```

### Gráfica

La gráfica de esta función muestra cómo domina la complejidad cuadrática en comparación con las otras:

![Chartset](/assets/img/BigO/BigO-10.jpg)

---

## Conclusión

Con este artículo, cerramos la serie sobre Big-O. Ahora tienes una visión integral de cómo analizar y optimizar algoritmos basándote en su complejidad computacional. 

¡Gracias por leer! Si te gustó esta serie, no dudes en compartirla con otros desarrolladores interesados en mejorar su comprensión de algoritmos.
