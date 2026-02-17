---
title: 'Big-O en Español Parte 3'
description: 'Expresiones logarítmicas, exponenciales y factoriales en notación Big-O. Búsqueda binaria y Fibonacci.'
pubDate: 'Jun 30 2022'
---

Este es el tercer Post de una serie de 4 posts hablando sobre la notación Big-O, en este post vamos a hablar específicamente de las expresiones Logarítmicas, las expresiones exponenciales y factoriales.

Los posts anteriores fueron:
- [Parte 1 - Expresiones Lineales y Constantes](/blog/big-o-part-1/)
- [Parte 2 - Expresiones Cuadráticas, cúbicas y de otras potencias](/blog/big-o-part-2/)

## Expresiones Logarítmicas

Todos los que hemos estudiado Ingeniería en Sistemas o carreras afines, lo primero que aprendemos en las materias de algoritmos y estructura de datos son los algoritmos de búsqueda, y entre ellos, uno de los primeros que se estudia es la búsqueda dicotómica o binaria.

Un ejemplo de una expresión logarítmica es la búsqueda dicotómica o binaria.

Prerrequisitos: para poder usar esta técnica de búsqueda debemos tener un arreglo o vector ordenado.

Ejemplo:

`const vector = [3, 6, 7, 12, 15, 16, 24, 25, 32, 35, 39]`

Supongamos que queremos buscar el número 7:

![BinarySearch](/images/BigO/BigO-3.jpg)

En este caso como muestra la imagen, tenemos que dividir el arreglo por la mitad, y comparar si el número buscado es mayor o menor que la mitad, si es menor, se utiliza la parte izquierda del arreglo para seguir dividiendo en mitades y comparando y si es mayor se utiliza el lado derecho del arreglo.

En este caso, el número 7 está dentro del arreglo.

Supongamos que queremos buscar el número 34:

En este caso vamos a usar la parte derecha del arreglo, dividirlo por mitades y comparar.

![BinarySearch](/images/BigO/BigO-4.jpg)

Como se puede ver, en este caso no se encontró el número 34.

Ejemplo de código:

```ts
function busquedaBinaria(arregloNumeros: number[], numEncontrar: number): number {
    let izquierda: number = 0;
    let derecha: number = arregloNumeros.length - 1;

    while (izquierda <= derecha) {
        const medio: number = Math.floor((izquierda + derecha) / 2);

        if (arregloNumeros[medio] === numEncontrar) return medio;
        if (numEncontrar < arregloNumeros[medio]) derecha = medio - 1;
        else izquierda = medio + 1;
    }

    return -1; // Numero no Encontrado
}
```

El objetivo de este post es hablar de la notación Big-O para expresiones Logarítmicas, con lo cual no hacemos foco en agregar mucho detalle a la explicación de la búsqueda dicotómica o binaria, sino simplemente un refresco de cómo se utiliza.

Como se puede ver en este ejemplo, si bien tenemos un arreglo, pero nunca lo recorremos entero, lo vamos recorriendo por mitades, esto hace que el tiempo de búsqueda sea más eficiente que una búsqueda secuencial.

Este tipo de expresiones se las llama logarítmicas:

```
f(x) = Log x
```

En notación Big-O se expresa de la siguiente manera:

`O(Log n)`

La gráfica de este tipo de expresiones es la siguiente:

![BigO-Log](/images/BigO/BigO-5.jpg)

## Expresiones Exponenciales

Un ejemplo típico de este tipo de expresiones podría ser el cálculo recursivo de una serie de Fibonacci, pueden ver ejemplos en el siguiente [sitio](https://www.omnicalculator.com/math/fibonacci#what-is-the-fibonacci-sequence).

El algoritmo que típicamente se utiliza para este cálculo es el siguiente:

La Fórmula: `F(n) = F(n-1) + F(n-2)`

Algoritmo para calcular la serie de Fibonacci:

```ts
var serieFibonacci = function(numero) {
    if (numero <= 1) return numero;
    return serieFibonacci(numero - 2) + serieFibonacci(numero - 1);
};
```

Este algoritmo es recursivo y se expresa:

Expresión Matemática: `f(x)=2^x`

En notación Big-O: `O(2^n)`

La gráfica de este tipo de expresión tiene la siguiente forma:

![BigO-Exponential](/images/BigO/BigO-6.jpg)

## Expresiones Factoriales

El último de los tipos de notaciones que vamos a ver son los factoriales.

Estas expresiones son las de mayor complejidad.

El ejemplo más simple para explicar este tipo de algoritmos son los ataques de fuerza bruta.

Por ejemplo, para saber una contraseña se prueban con todas las combinaciones de letras (mayúsculas y minúsculas) y números del abecedario, hasta dar con la combinación. Los ataques de fuerza bruta pueden demorar días, semanas o meses intentando todas las combinaciones (dependiendo de la longitud de la Password).

Este tipo de algoritmos no son muy comunes de ver, ya que son ineficientes en la práctica. Cabe destacar que las búsquedas por fuerza bruta no son solo utilizadas para fines delictivos, muchas veces es necesario probar todas las combinaciones posibles y se diseñan algoritmos con este fin, donde se sabe de antemano que la demora puede ser muy larga.
