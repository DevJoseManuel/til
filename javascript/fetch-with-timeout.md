# Fetch with timeout

> **Contexto**: Es posible que se nos presente a escenario en el que tengamos que hacer desde el navegador una petición con `fetch` que queremos cancelar pasada una cierta cantidad de tiempo.

Partiremos con una función como la siguiente que recibe la _ulr_ a la que vamos a hacer la petición y la cantidad de tiempo en milisegundos a partir de la cual deberemos cancelar la petición en el caso de que se tenga que cancelar la petición.

```ts
async function fetchWithTimeout(url, timeout = 5_000) {
  const response = away fetch(url)
  return response.json()
}
```

Para lograr nuestro objetivo vamos a apoyarnos en la clase `AbortController` que nos proporcina la [API del navegador](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) la cual está pensada para poder resolver situaciones como la que estamos cubriendo. Así pues, comenzaremos creando una instancia de esta clase:

```ts
async function fetchWithTimeout(url, timeout = 5_000) {
  const controller = new AbortController()

  const response = away fetch(url)
  return response.json()
}
```

y lo que haremos será obtener una referencia al atributo `signal` que tiene definido puesto que es el que nos va a permitir realizar la cancelación como veremos un poco más adelante.

```ts
async function fetchWithTimeout(url, timeout = 5_000) {
  const controller = new AbortController()
  const signal = controller.signal

  const response = away fetch(url)
  return response.json()
}
```

¿Qué tenemos que hacer con este `signal`? Pues simplemente pasárselo en el objeto con todas las opciones con las que se invoca a la función `fetch` ya que en el caso de que se tenga que cancelar la petición la implementación de `fetch` se enterará de ello puesto que `signal` le estará informando.

```ts
async function fetchWithTimeout(url, timeout = 5_000) {
  const controller = new AbortController()
  const signal = controller.signal

  const response = away fetch(url, { signal })
  return response.json()
}
```

Hasta aquí tenemos el esqueleto de lo que pretendemos construir pero ¿cómo logramos que este `signal` se capaz de comunicar a `fetch` que ha cambiado? Pues aquí es donde entran en juego dos cosas:

- en primer lugar el método `abort` que nos proporciona la clase `AbortController` y que sirve precisamente para lograr notificar que la operación se tiene que cancelar.

- en segundo lugar el uso de un `timeout` de tal manera que este método sea invocado en el momento en el que transcurren los milisegundos que se han recibido en el segundo de los parámetros.

Sabiendo esto el código de nuestra función hasta este momento será algo parecido a lo siguiente:

```ts
async function fetchWithTimeout(url, timeout = 5_000) {
  const controller = new AbortController()
  const signal = controller.signal

  const timeoutId = setTimeout(() => {
    controller.abort()
  }, timeout)

  const response = away fetch(url, { signal })
  return response.json()
}
```

y simplemente nos quedará una última cosa por realizar y es asegurarnos de que el `timeout` que hemos definido será eliminado ya sea porque o bien se ha cancelado la petición, porque la llamada a `fetch` finaliza antes de que se alcance el tiempo de _timeout_ o bien porque se produce un error y para ello nos vamos a apoyar en la estructura del tipo `try-finally` puesto que en el bloque `finally` vamos a llamar a `clearTimeout` y de esta manera lograremos el objetivo que estamos persiguiendo:


```ts
async function fetchWithTimeout(url, timeout = 5_000) {
  const controller = new AbortController()
  const signal = controller.signal

  const timeoutId = setTimeout(() => {
    controller.abort()
  }, timeout)

  try {
    const response = away fetch(url, { signal })
    return response.json()
  } finally {
    clearTimeout(timeoutId)
  }
}
```




