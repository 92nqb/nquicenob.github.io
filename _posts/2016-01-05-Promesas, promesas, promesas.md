---
title: Promesas, promesas, promesas
---

Para poder desarrollar JS una de las cosas que tienes que entender y por entender me refiero a entender en profundidad, son las promesas.

### ¿Qué es un Promesa?

Una promesa es objeto que representa el estado de una operación asíncrona y permite asociar acciones al resultado de dicha operación.
Actualmente hay un [estándar](https://promisesaplus.com/) que especifica el funcionamiento de una promesa, dicho estándar cuenta con una serie de [test](https://github.com/promises-aplus/promises-tests) que deberá pasar una librería de promesas para demostrar que cumple con el. 

### Caracteristicas de las promesas

Como una promesa cuenta con un estado que representa una operación este puede ser:

* **pending**, cuando la promesa ha sido creada
* **fulfilled**, cuando la operación ha terminado y se ha resuelto la promesa
* **rejected**, cuando la operación ha fallado y la promesa ha sido rechazada

Los métodos que permiten cambiar de un estado a otro son:

* **resolve**, pasa el estado de la promesa a _fulfilled_
* **reject**, pasa el estado de la promesa a _rejected_

Es importante destacar que **una vez modificado el estado de la promesa a _fulfilled_ o _rejected_ este pasa a ser inmutable**, no tiene lógica que una operación se ejecute de manera exitosa y falle a la vez o justo después.

Veamos esto con código.

### Show me the code

Empecemos con esta función para hacer peticiones ajax que recibe un callback que ejecuta cuando la petición termina.

{% highlight js %}
var $ajaxRequest = (function () {
  function ajaxRequest(requestParams, cb) {
    var request = new XMLHttpRequest();
    request.onreadystatechange = function () {
      if (request.readyState === 4) {
        request.status === 200 ?
        cb(null, request.responseText) :
        cb(request.responseText)
      }
    }
    request.open(requestParams.method, requestParams.url);
    request.send();
  }

  function getRequestFn(method) {
    return function (requestParams, action) {
      requestParams.method = method;
      ajaxRequest(requestParams, action);
    }
  }

  return {
    'get': getRequestFn('GET'),
    'post': getRequestFn('POST'),
    'put': getRequestFn('PUT'),
    'delete': getRequestFn('DELETE')
  };
})();
{% endhighlight %}

Para realizar una petición haríamos algo así:

{% highlight js %}
$ajaxRequest.get({
  url: 'https://www.googleapis.com'
}, function () {
  console.log.apply(console, arguments);
});
{% endhighlight %}

Pasar una función como parámetro permite pasar una acción para ejecutar de manera asíncrona, en este caso cuando la petición termine.
Una descripción poco tecnica del código anterior sería: 
> haz una peticion a la url "_https://www.googleapis.com_" y cuando te devuelvan el resultado ejecuta esta función".

Ahora bien, si quisiéramos lanzar 3 peticiones que se encolen, de manera que no se realiza la siguiente hasta que no hayamos obtenido el resultado de la anterior, tendríamos que hacer algo así:

{% highlight js %}
$ajaxRequest.get({
  url: 'https://www.googleapis.com/0'
}, function(err, firstRes){
  //... process first response
  $ajaxRequest.get({
    url: 'https://www.googleapis.com/1'
  }, function(err, SecondRes){
    //... process second response
    $ajaxRequest.get({
      url: 'https://www.googleapis.com/2'
      }, function(err, LastRes){
      //... process last response
    });
  });
});
{% endhighlight %}

Como podemos ver, esto ya comienza a verse un poco feo pero puede ser peor, ya que pueden ser mas de 3 peticiones, pueden haber necesidades funcionales que requieran peticiones en paralelo mezcladas con peticiones encoladas además de la gestión de excepciones que ahora mismo no la estamos teniendo en cuenta.

Como vamos a ver a continuación estos problemas se simplifican con el uso de promesas, si nuestra función **$ajaxRequest** devolviera una promesa, el código anterior se escribiría de esta manera:

{% highlight js %}
$ajaxRequest.get({
  url: 'https://www.googleapis.com/0'
}).then(function(response){
  //... process first response
  return $ajaxRequest.get({url: 'https://www.googleapis.com/1'});
}).then(function(response){
  //... process second response
  return $ajaxRequest.get({url: 'https://www.googleapis.com/2'});
}).then(function(response){
  //... process last response
});
{% endhighlight %}

Podemos apreciar un código más elegante y legible, vamos a ver como funciona.

Primero vamos a modificar la función **$ajaxRequest** para que devuelva una promesa en lugar de trabajar con callback, para ello vamos  usar la librería [rsvp](https://github.com/tildeio/rsvp.js/) que cumple con el especificación A+.

{% highlight js %}
var $ajaxRequest = (function () {
  function ajaxRequest(requestParams) {
    // return a promise
    return new RSVP.Promise(function(resolve, reject){
      var request = new XMLHttpRequest();
      function handlerState(){
        if (this.readyState === 4) {
          // resolve o reject the promise
          this.status === 200 ? 
            resolve(this.responseText) : reject(this.responseText)
        }
      }
        
      request.onreadystatechange = handlerState;
      request.open(requestParams.method, requestParams.url);
      request.send();
    });
  }

  function getRequestFn(method) {
    return function (requestParams) {
      requestParams.method = method;
      return ajaxRequest(requestParams);
    }
  }

  return {
    'get': getRequestFn('GET'),
    'post': getRequestFn('POST'),
    'put': getRequestFn('PUT'),
    'delete': getRequestFn('DELETE')
  };
})();
{% endhighlight %}

Si recordamos el ejemplo de ejecución de **$ajaxRequest** trabajando con promesas haciamos esto:
{% highlight js %}
$ajaxRequest.get({
  url: 'https://www.googleapis.com/0'
}).then(function(response){
  //... process first response
  return $ajaxRequest.get({url: 'https://www.googleapis.com/1'});
})
//...
;
{% endhighlight %}

Para entender este código vamos a empezar por explicar el método **then** de un objeto promise.
  
### Then

El método **then** nos permite añadir acciones a una cola que se ejecutarán en el orden en el que las definimos cuando la promesa cambie su estado.

El método **then** por especificación recibe dos parámetros y devuelve una promesa, el primer parámetro es un callback(**onFulfilled**) que se ejecutará cuando la promesa cambie su estado a _fulfilled_ y el segundo parámetro es otro callback(**onRejected**) que se ejecutará en caso de que haya habido errores o la promesa cambie su estado a _rejected_.

Es importante entender que cada vez que llamemos al método then estamos añadiendo los callbacks(**onFulfilled**, **onRejected**) a una cola de manera que la ejecución de los mismos se producirá en orden secuencial.

Ahora vamos a ver en detalle las características de los callback **onFulfilled** y **onRejected**:

Los callback **onFulfilled** o **onRejected** esperan un parámetro de entrada y este puede ser:

* Si son los primeros callback de la cola, el parámetro que reciben será el argumento que se pasó al método _resolve_ o _reject_ de la promesa, que en nuestro caso es el resultado de la petición: 

{% highlight js %}
var $ajaxRequest = (function () {
  function ajaxRequest(requestParams) {
  // return a promise
    return new RSVP.Promise(function(resolve, reject){
      var request = new XMLHttpRequest();
      function handlerState(){
        if (this.readyState === 4) {
        // resolve o reject the promise
          this.status === 200 ? 
            resolve(this.responseText) : reject(this.responseText)
        }
        // ...
      }
      // ...
    });
  }

  function getRequestFn(method) {
    // ...
  }

  return {
    // ...
  };
})();
{% endhighlight %}

* En caso contrario el parámetro que reciben será el valor devuelto por la ejecución del último callback de la cola (cuando es el callback **onFulfilled**) o la razón del error (cuando es el callback  **onRejected**), veamos el ejemplo de código:

{% highlight js %}
function firstThen(response){
  //... first fulfillment handler in queue 
  return valueFirst;
}
function secondThen(valueFirst){
  //... second fulfillment handler in queue 
}
$ajaxRequest.get({
  url: 'https://www.googleapis.com/0'
}).then(firstThen)
  .then(secondThen);
{% endhighlight %}

La variable _valueFirst_ que recibe la función _secondThen_ es el valor devuelto por la función _firstThen_.

Hemos visto que pasa cuando devolvemos un valor, pero ¿qué pasa si devolvemos otra promesa como en nuestro ejemplo de código? Pues que el valor que recibimos es el resultado de la promesa que devolvió la ejecución del último callback de la cola, es decir:

{% highlight js %}
$ajaxRequest.get({
  url: 'https://www.googleapis.com/0'
}).then(function(firtsResponse){
  //... process first response
  return $ajaxRequest.get({url: 'https://www.googleapis.com/1'});
}).then(function(secondResponse){
  //... process second response
});
{% endhighlight %}

La variable **secondResponse** contendrá el valor devuelto por la petición a '_https://www.googleapis.com/1_' esto es llamado **assimilation** y es lo que nos permite encolar las peticiones ya que el siguiente callback de la cola no se ejecuta hasta que la promesa devuelta por el anterior haya cambiado de estado a _fulfilled_ o _rejected_.

Volviendo al ejemplo de código inicial:
{% highlight js %}
$ajaxRequest.get({
  url: 'https://www.googleapis.com/0'
}).then(function(response){
  //... process first response
  return $ajaxRequest.get({url: 'https://www.googleapis.com/1'});
})
  //...
;
{% endhighlight %}


Una explicación poco técnica podría ser:
> Realiza una petición a la url "_https://www.googleapis.com/0_" y devuelveme un objeto al que le digas el resultado, cuando el objeto sepa el resultado entonces ejecutará esta función y despues esta otra ... .

Por último es importante destacar que aunque la operación que realizamos sea síncrona la ejecución del callback del método then será siempre asíncrona, esto es descrito en la especificación A/+:
 
> Here “platform code” means engine, environment, and promise implementation code. In practice, this requirement ensures that onFulfilled and onRejected execute asynchronously, after the event loop turn in which then is called, and with a fresh stack. This can be implemented with either a “macro-task” mechanism such as setTimeout or setImmediate, or with a “micro-task” mechanism such as MutationObserver or process.nextTick. Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or “trampoline” in which the handlers are called.

Esto traducido a código significa que:

{% highlight js %}
var promise = new RSVP.Promise(function(resolve, reject){
  resolve("FIRST");
});
  
promise.then(function(result){
  console.log(result);
});
  
console.log("SECOND");
{% endhighlight %}

Lo primero que veremos en consola será:
> "SECOND" 
y lo siguiente 
> "FIRST"

Esta ejecución en asíncrono en el caso de la librería **rsvp** la realiza la función  _asap_ que elige la manera de encolar las peticiones en función de la plataforma, navegador, etc.

Esto también quiere decir que si hacemos esto:

{% highlight js %}
try{
  new RSVP.Promise(function(resolve, reject){
    resolve("FIRST");
  }).then(function(result){
    throw new Error("NO CATCH");
  });
}catch(err){
  console.log(err);
}
{% endhighlight %}

Nunca vamos a entrar en el catch, ya que el callback _onFulfilled_ pasado al método then se ejecuta en otro turno del **event loop**.

### Gestión de errores

Como hemos visto el método _then_ recibe como segundo parámetro un callback _onRejected_  que se ejecuta cuando salta una excepción o la promesa es rechazada. Para capturar la excepción del ejemplo anterior tendríamos que hacer algo asi:

{% highlight js %}
new RSVP.Promise(function(resolve, reject){
  resolve("FIRST");
}).then(function(result){
  throw new Error("NO CATCH");
}).then(null, function(err){
  console.log(err);
});
{% endhighlight %}

Hay que tener cuidado de no caer en el error de pensar que es correcto hacer esto:

{% highlight js %}
function errorHandler(reason){
  // ...
}
  
function successHandler(value){
  // ...
  throw new Error("NO CATCH");
}
  
new RSVP.Promise(function(resolve, reject){
  // resolve o reject promise
}).then(successHandler, errorHandler);
{% endhighlight %}

Ya que como vimos al principio, una promesa sólo puede cambiar de estado una vez, es decir, cuando ha pasado a **rejected** no puede pasar a **fulfilled** y viceversa, por lo tanto en el ejemplo de código anterior la función _errorHandler_ nunca se ejecutará si la excepción se produce en el la función _successHandler_.

### Otras cosas importantes

La gran mayoría de librerías de promesas cuentan con una serie de decoradores y funciones para facilitarnos la vida, estos permiten cosas como encapsular la ejecución de varias promesas en paralelo, métodos tipo **catch** que nos evitan tener que llamar a _then_ y pasarle un _null_, por ejemplo:

{% highlight js %}
new RSVP.Promise(function(resolve, reject){
  // resolve o reject promise
}).then(null, function(err){
  console.log(err);
});
{% endhighlight %}

Es lo mismo que esto:

{% highlight js %}
new RSVP.Promise(function(resolve, reject){
  // resolve o reject promise
}).catch(function(err){
  console.log(err);
});
{% endhighlight %}

Lo mejor es mirar la documentación de cada librería para aprovechar estas utilidades al máximo.

Como última recomendación utilizar **SIEMPRE** una librería que cumpla el estándar **A/+**.

Más información.

* [estándar A+](https://promisesaplus.com/)
* [Página](https://www.promisejs.org/) de información general sobre promesas
* [Articulo](http://www.mattgreer.org/articles/promises-in-wicked-detail/) par entender como funcionan las promesas por dentro
* Librería [rsvp](https://github.com/tildeio/rsvp.js)
* Diseño de la [librería q](https://github.com/kriskowal/q/blob/v1/design/README.js)
* [NodeSchool Workshopper - Promise It Won't Hurt](https://github.com/stevekane/promise-it-wont-hurt) (más que recomendable)










