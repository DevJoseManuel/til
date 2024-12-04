# Promise.all vs Promise.allSettled

La clase `Promise` nos ofrece los métodos `all` y `allSettled` que funcionan de una forma similar a la hora de controlar las promesas que estemos ejecutando dentro de nuestro código pero el comportamiento de los mismos es a su vez un tanto diferente por lo que es necesario que seamos capaces de elegir el que mejor se ajuste a nuestras necesidades.

Para poder entender la diferencia que existe entre ambos vamos a suponer que tenemos una función como la siguiente en la que, sin mostrar el código porque no es relevante para la explicación, sabemos que apróximadamente el 50% de las veces que la llamamos falla lanzando un error:

```ts
async function riskyBusiness() {}
```

Supongamos ahora que tenemos el siguiente código que se encargará de crear un array de varias promesas donde cada una de ellas será el resultaod de la invocación de esta función:

```ts
async function riskyBusiness() {}

const promises = Array.from({ lenght: 3 }, riskyBusiness)
```

## `Promise.all`

Vamos a ahora a hacer uso del métood `Promise.all` para ejecutar todas las promesas de nuestro array como sigue donde simplemente rodeamos a la invocación de este método con un bloque `try-catch` para ver cuál es el resultado que obtenemos:

```ts
try {
  const results = await Promise.all(promises)
  console.log(results)
} catch (err) {
  console.log('We reached the catch block')
}
```

Si ahora desde la terminal del sistema ejecutamos nuestro código la salida que vamos a obtener va a ser algo parecido a lo que podemos ver a continuación:

```bash
$ node main.ts
We reached the catch block
```

Es decir que se estará mostrando el mensaje de error puesto que una de las promesas que están formando parte del array estará lanzando un error.

## `Promise.allSettled`

Si ahora cambiamos el código anterior para que pase a usar el método `allSettled`:

```ts
try {
  const results = await Promise.allSettled(promises)
  console.log(results)
} catch (err) {
  console.log('We reached the catch block')
}
```

y volvemos a ejecutar el código, podemos ver que en la salida esta vez no se estará mostrando el texto que está recogido en el bloque `catch` sino que realmente se está mostrando un mensaje como el siguiente:

```bash
$ node main.ts
res:
  {
    status: 'rejected',
    reason: Error.failure
      at riskyBusiness ....
      [cause]: 'Random number was too high'
  },
  { status: 'fulfilled', value: 'Success' },
  {
    status: 'rejected'
    ...
  }
```

donde lo importante aquí es entender que se está mostrando el valor que está recogido en la variable `result` que será un array con el resultando de la ejecución de las tres promesas que hemos definido anteriormente. Es más, cada uno de los elementos de este array recogerá el estado en el que se encuentra dicha promesa lo que quiere decir que la primera de ellas (el elemnto cero del array) habrá fallado, la segunda habrá terminado correctamente y la tercera también habrá fallado.

