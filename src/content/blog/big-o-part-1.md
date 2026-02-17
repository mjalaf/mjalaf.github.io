---
title: 'Big-O en Español Parte 1'
description: 'Introducción a la notación Big-O: expresiones lineales y constantes explicadas con ejemplos prácticos.'
pubDate: 'Jun 17 2022'
---

Durante mis años en la universidad nunca había escuchado sobre la notación Big-O, no es nada nuevo ni tampoco complejo de entender, es simplemente una notación para medir complejidad.

Para explicarlo, lo más fácil es mediante ejemplos.

## Expresiones Lineales

Supongamos que tenemos una lista de 10 personas en un arreglo, y no estamos teniendo en cuenta el orden.

```
const personas = ["Juan","Pedro", "Pablo", "Damian", "Maria","Diego", "Nelson", "Jose", "Monica","Juana"]
```

Si queremos encontrar a una persona de la lista, tendremos que iterar la lista y cuando encontremos a la persona, devolver un mensaje de Persona Encontrada, o en caso de que la persona no esté en la lista, comunicar Persona No encontrada.

Ejemplo de Código:

```ts
function EncontrarPersona(nombre: string) {
    for (let index = 0; index < personas.length; index++) {
        const persona = personas[index];

        if (persona === nombre)
            return "Persona Encontrada";
    }
    return "Persona No Encontrada";
}
```

Como se puede ver, es una simple iteración, donde en el mejor de los casos, la persona se ubicará en la primera posición del arreglo y en el peor de los casos, la persona no se encontrará en el arreglo.

En expresiones Matemáticas esta es la expresión Lineal:

`f(x)=x`

En notación Big-O se expresa de la siguiente manera:

`O(n)`

Como lo expresan las ecuaciones, cuanto más elementos tenga el arreglo, más tiempo va a demorar en encontrar personas.

## Constantes

Supongamos que tenemos una Tabla Hash (Hashtable) cuya responsabilidad es devolver el valor en base a una clave.

Entonces si tenemos la siguiente Tabla Hash:

```
const colores = ["Blanco", "#FFFFFF"], ["Rojo", "#FF0000"], ["Negro", "#000000"], ["Amarillo", "#FFFF00"], ["Verde", "#008000"]
```

Por cada vez que intente recuperar un valor, en base a una clave, el tiempo de respuesta será siempre el mismo:

```ts
const colorBlanco = colores["Blanco"]
// resultado #FFFFFF
const colorVerde = colores["Verde"]
// resultado "#008000"
const colorAzul = colores["Azul"]
// resultado Undefined
```

En expresiones Matemáticas esta es la expresión Constante:

`f(x)=c`

En notación Big-O se expresa de la siguiente manera:

`O(1)`

Las gráficas de estas dos notaciones se pueden ver a continuación:

![Big-O Linear vs Constant](/images/BigO/BigO-1.jpg)

Algunas preguntas típicas que se hacen sobre estas anotaciones:

- ¿Qué sucede si mi procesador es antiguo?
- ¿Qué sucede si tengo un procesador muy rápido?
- ¿Qué sucede si mi procesador tarda en cargar, y luego arranca la ejecución?

La respuesta a estas preguntas son las mismas para las tres:

La ecuación no cambia, tal vez:

- f(x)=x demorando la mitad, será ==> f(x)=1/2x
- o demorando el doble será ==> f(x)= 2x
- O demorando en iniciar, será ==> f(x) = 2+x

Pero siempre la ecuación es f(x)=x

Con lo cual se la considera lineal y lo mismo aplica para la constante, puede ser f(x) = 1 o f(x) = 1/2 o f(x)=2 pero siempre será f(x)=c.

Estas son las dos notaciones más simples de entender de Big O.

En el próximo Post voy a hablar de las expresiones cuadráticas y de cualquier exponente.
