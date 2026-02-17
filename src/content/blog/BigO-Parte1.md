---
title: 'Entendiendo Big-O en Español - Parte 1'
description: 'Exploramos las notaciones O(n) lineal y O(1) constante con ejemplos prácticos en JavaScript.'
pubDate: 'Jun 17 2022'
---

La notación Big-O es una herramienta fundamental para evaluar la eficiencia de los algoritmos. Aunque puede sonar compleja, es bastante sencilla de entender. Su propósito principal es medir la complejidad computacional y cómo los algoritmos se comportan a medida que aumenta el tamaño de los datos.

En esta primera parte, exploraremos dos de las notaciones más comunes: **O(n)** (lineal) y **O(1)** (constante), utilizando ejemplos prácticos.

---

## Expresiones Lineales

Imagina que tienes una lista de personas en un arreglo desordenado:

```javascript
const personas = ["Juan", "Pedro", "Pablo", "Damian", "Maria", "Diego", "Nelson", "Jose", "Monica", "Juana"];
```

Si quieres buscar a una persona específica en la lista, necesitas recorrer cada elemento uno por uno hasta encontrarla. Si la persona no está en la lista, tendrás que revisar todos los elementos.

El siguiente código ilustra este proceso:

```javascript
function encontrarPersona(nombre) {
  for (let persona of personas) {
    if (persona === nombre) {
      return "Persona Encontrada";
    }
  }
  return "Persona No Encontrada";
}
```

En este caso:

- **Mejor caso:** La persona está en la primera posición (O(1)).
- **Peor caso:** La persona no está o está al final de la lista (O(n)).
- **Caso promedio:** Se recorre la mitad de los elementos, pero en Big-O se simplifica como O(n).

En matemáticas, esta relación se expresa como:

```plaintext
f(x) = x
```

En notación Big-O, se representa así:

```plaintext
O(n)
```

Esto significa que, a medida que el número de elementos en el arreglo crece, el tiempo necesario para encontrar una persona también crece linealmente.

---

## Expresiones Constantes

Ahora supongamos que tienes una tabla hash (hashtable). Este tipo de estructura permite recuperar valores en base a una clave de forma casi instantánea.

Ejemplo de tabla hash:

```javascript
const colores = {
  "Blanco": "#FFFFFF",
  "Rojo": "#FF0000",
  "Negro": "#000000",
  "Amarillo": "#FFFF00",
  "Verde": "#008000"
};
```

Cuando intentas recuperar un valor por su clave, como en los siguientes ejemplos:

```javascript
const colorBlanco = colores["Blanco"]; // Resultado: "#FFFFFF"
const colorVerde = colores["Verde"];   // Resultado: "#008000"
const colorAzul = colores["Azul"];    // Resultado: undefined
```

El tiempo necesario para acceder a cualquier valor es constante, independientemente de cuántos elementos haya en la tabla hash.

En matemáticas, esta relación se expresa como:

```plaintext
f(x) = c
```

Y en notación Big-O:

```plaintext
O(1)
```

Esto significa que el tiempo de ejecución no depende del tamaño de la estructura de datos.

---

## Respondiendo Preguntas Comunes

### ¿Qué pasa si mi procesador es antiguo o muy rápido?
La notación Big-O mide la **tendencia** del tiempo de ejecución en relación con el tamaño de los datos, no el tiempo exacto. Por ejemplo:

- Si el procesador es más lento, podrías tener:
  ```plaintext
  f(x) = 2x
  ```
- Si es más rápido:
  ```plaintext
  f(x) = x / 2
  ```
- Si hay un retraso inicial antes de ejecutar:
  ```plaintext
  f(x) = x + 2
  ```

Sin embargo, en todos estos casos, la tendencia sigue siendo la misma: **f(x) = x** (lineal).

Lo mismo aplica para O(1), donde el tiempo puede variar, pero sigue siendo constante.

---

## Conclusión

En este post, hemos explorado las dos notaciones más simples de Big-O: **O(n)** (lineal) y **O(1)** (constante). Estas son esenciales para entender cómo los algoritmos escalan con datos más grandes.

En la próxima parte, hablaremos de **expresiones cuadráticas** y otros casos más complejos. ¡Prepárate para llevar tu comprensión de Big-O al siguiente nivel!
