---
title: 'Big-O en Español Parte 3'
description: 'Expresiones logarítmicas, exponenciales y factoriales en la notación Big-O con ejemplos prácticos.'
pubDate: 'Jun 30 2022'
---

Este es el tercer post de una serie de 4 sobre la notación Big-O. En este artículo, exploraremos las **expresiones logarítmicas**, **exponenciales** y **factoriales**.

---

## Posts Anteriores

- **Parte 1:** Expresiones Lineales y Constantes
- **Parte 2:** Expresiones Cuadráticas, Cúbicas y de Otras Potencias

---

## Expresiones Logarítmicas

Un ejemplo clásico de una expresión logarítmica es la **búsqueda binaria**, uno de los primeros algoritmos que aprendemos en estructuras de datos. Este método requiere un arreglo **ordenado** para funcionar.

Ejemplo de arreglo:

```javascript
const vector = [3, 6, 7, 12, 15, 16, 24, 25, 32, 35, 39];
```

### Explicación

Supongamos que queremos buscar el número **7**. En la búsqueda binaria:

1. Dividimos el arreglo por la mitad.
2. Comparamos el número buscado con el valor en la posición central.
3. Si el número buscado es menor, seguimos con la mitad izquierda; si es mayor, usamos la mitad derecha.

**Resultado:** El número **7** está en el arreglo.

Si buscamos el número **34**, aplicamos la misma técnica, pero el número no se encontrará después de iterar.

### Código de Búsqueda Binaria

```javascript
function busquedaBinaria(arregloNumeros, numEncontrar) {
  let izquierda = 0;
  let derecha = arregloNumeros.length - 1;

  while (izquierda <= derecha) {
    const medio = Math.floor((izquierda + derecha) / 2);

    if (arregloNumeros[medio] === numEncontrar) return medio;
    if (numEncontrar < arregloNumeros[medio]) derecha = medio - 1;
    else izquierda = medio + 1;
  }

  return -1; // Número no encontrado
}
```

### Complejidad Matemática

La búsqueda binaria es eficiente porque no revisa todos los elementos, sino que divide el arreglo iterativamente. Matemáticamente, se representa como:

```plaintext
f(x) = Log x
```

En notación Big-O:

```plaintext
O(Log n)
```

### Gráfica

La gráfica de estas expresiones es:

![BigO-Log](/assets/img/BigO/BigO-5.jpg)

---

## Expresiones Exponenciales

Un ejemplo típico de una expresión exponencial es el cálculo recursivo de la **serie de Fibonacci**.

### Fórmula de Fibonacci

```plaintext
F(n) = F(n-1) + F(n-2)
```

### Código

```javascript
var serieFibonacci = function(numero) {
    if (numero <= 1) return numero;
    return serieFibonacci(numero - 2) + serieFibonacci(numero - 1);
};
```

Este algoritmo calcula la serie recursivamente y, debido a que realiza muchas llamadas repetidas, es muy ineficiente para valores altos de **n**.

### Complejidad Matemática

Matemáticamente, se representa como:

```plaintext
f(x) = 2^x
```

En notación Big-O:

```plaintext
O(2^n)
```

### Gráfica

La gráfica para este tipo de expresiones es:

![BigO-Exponential](/assets/img/BigO/BigO-6.jpg)

---

## Expresiones Factoriales

Las **expresiones factoriales** son las de mayor complejidad y se observan en algoritmos que prueban todas las combinaciones posibles, como los **ataques de fuerza bruta**.

### Ejemplo: Ataques de Fuerza Bruta

Imagina que queremos adivinar una contraseña probando todas las combinaciones de letras y números. Este enfoque es ineficiente y puede tardar días, semanas o meses dependiendo de la longitud de la contraseña.

Aunque los ataques de fuerza bruta son ineficientes, en ocasiones se diseñan algoritmos similares para explorar combinaciones donde el tiempo no es una restricción crítica.

### Complejidad Matemática

Matemáticamente, se representa como:

```plaintext
f(x) = x!
```

En notación Big-O:

```plaintext
O(n!)
```

### Gráfica

La gráfica de este tipo de algoritmos muestra un crecimiento extremadamente rápido:

![BigO-Factorial](/assets/img/BigO/BigO-7.jpg)

---

## Conclusión

En este post, hemos explorado:

- Expresiones logarítmicas (búsqueda binaria): **O(Log n)**
- Expresiones exponenciales (Fibonacci recursivo): **O(2^n)**
- Expresiones factoriales (ataques de fuerza bruta): **O(n!)**

En la próxima y última parte, exploraremos técnicas para **optimizar algoritmos** y evitar complejidades altas. ¡No te lo pierdas!
