---
title: Promesas, promesas, promesas
---

Para poder desarrollar JS una de las cosas que tienes que entender y por entender me refiero a entender en profundidad, son las promesas.

### ¿Qué es un Promesa?

Una promesa es objeto que representa el estado de una operación asincrona y permite asociar acciones al resultado de dicha operación.
Actualmente hay un [estandar](https://promisesaplus.com/) que especifica el funcionamiento de una promesa, dicho estandar cuenta con una serie de [test](https://github.com/promises-aplus/promises-tests) que deberá pasar una libreria de promesas para demostrar que cumple con el. 

### Caracteristicas de las promesas

Como una promesa cuenta con un estado que representa una operación este puede ser:

* **pending**, cuando la promesa ha sido creada
* **fulfilled**, cuando la promesa ha sido resuelta
* **rejected**, cuando la promesa ha sido rechazada

Los metodos que permiten trancisionar de un estado a otro son:

* **resolve**, pasa el estado de la promesa a _fulfilled_
* **reject**, pasa el estado de la promesa a _rejected_

Es importante destacar que **una vez modificado el estado de la promesa a _fulfilled_ o _rejected_ este pasa a ser inmutable**, la lógica de esto está en que estamos representando el estado de una operación asincrona y no tiene sentido que esta falle y termine exitosamente a la vez o justo después.

Veamos un poco de código para entender por que son buenas las promesas.

### show me the code

Si tenemos la siguiente función para hacer peticiones ajax.

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

Para realizar una petición hariamos algo así:

{% highlight js %}
$ajaxRequest.get({
  url: 'https://www.googleapis.com'
}, function () {
  console.log.apply(console, arguments);
});
{% endhighlight %}

Una descripción poco tecnica del codigo anterior sería: 
> haz una peticion a la url "_https://www.googleapis.com_" y cuando te devuelvan el resultado ejecuta esta función".

Ahora bien, si quisieramos lanzar 3 peticiones que se encolen, de manera que no se realiza la siquiente hasta que no hayamos obtenido el resultado de la anterior, tendriamos que hacer algo así:

{% highlight js %}
$ajaxRequest.get({url: 'https://www.googleapis.com/0'}, function(err, firstRes){
  //... process first response
  $ajaxRequest.get({url: 'https://www.googleapis.com/1'}, function(err, SecondRes){
    //... process second response
    $ajaxRequest.get({url: 'https://www.googleapis.com/2'}, function(err, LastRes){
      //... process last response
    });
  });
});
{% endhighlight %}

Como podemos ver, esto ya comienza a verse un poco feo pero puede ser peor, ya que pueden ser mas de 3 peticiones, pueden haber necesidades funcionales que requieran peticiones en paralelo mezcladas con peticiones encoladas ademas de la gestión de excepciones que ahora mismo no la estamos teniendo en cuenta.

Si nuestra función **$ajaxRequest** devolviera una promesa, el código anterior se escribiría así:

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

Podemos apreciar un código mas elegante y legible, vamos a ver como funciona.

Primero vamos a modificar la funcion **$ajaxRequest** para que devuelva una promesa en lugar de un callback, para ello vamos  usar la librería [rsvp](https://github.com/tildeio/rsvp.js/) que cumple con el especificación A+.

{% highlight js %}
var $ajaxRequest = (function () {
  function ajaxRequest(requestParams) {
    // return a promise
    return new RSVP.Promise(function(resolve, reject){
      var request = new XMLHttpRequest();
      function handlerState(){
        if (this.readyState === 4) {
          // resolve o reject the promise with the request result,
          // this action is asynchronously
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

Si recordamos el ejemplo de ejecución de **$ajaxRequest** devolviendo una promesas haciamos esto:
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

Para entender este código vamos a empezar por explicar el metodo **then** de un objeto promise.
  
### Then

El método **then** nos permite añadir acciones a una cola que se ejecutarán en el orden en el que las definimos cuando la promesa cambie su estado.

El metodo **then** por especificación recibe dos parámetros y devuelve una promesa, el primer parametro es un callback(**onFulfilled**) que se ejecutara cuando la promesa cambie su estado a _fulfilled_ y el segundo parametro es otro callback(**onRejected**) que se ejecutara en caso de que haya habido errores o la promesa cambie su estado a _rejected_.

Es importante entender que cada vez que llamemos al metodo then estamos añadiendo los callbacks(**onFulfilled**, **onRejected**) a una cola de manera que la ejecución de los mismos se producira en orden secuencial.

Ahora vamos a ver en detalle las caracteristicas de los callback **onFulfilled** y **onRejected**:

Para los callback **onFulfilled** o **onRejected** esperan un parámetro de entrada y este puede ser:

* Si son los primeros callback de la cola, el parametro que reciben será el argumento que se paso al metodo _resolve_ o _reject_ de la promesa, que en nuestro caso es el resultado de la petición: 

{% highlight js %}
var $ajaxRequest = (function () {
  function ajaxRequest(requestParams) {
  // return a promise
    return new RSVP.Promise(function(resolve, reject){
      var request = new XMLHttpRequest();
      function handlerState(){
        if (this.readyState === 4) {
        // resolve o reject the promise with the request result,
        // this action is asynchronously
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

* En caso contrario el parámetro que reciben será el valor devuelto por la ejecución del ultimo callback de la cola (cuando es el callback **onFulfilled**) o la razon del error (cuando es el callback  **onRejected**), veamos el ejemplo de código:

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

La variable _valueFirst_ que recibe la funcion _secondThen_ es el valor devuelto por la funcion _firstThen_.

Hemos visto que pasa cuando devolvemos un valor, pero ¿que pasa si devolvemos otra promesa como en nuestro ejemplo de código? Pues que el valor que recibimos es el resultado de la promesa que devolvio la ejecución del ultimo callback de la cola, es decir:

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

La variable **secondResponse** contendrá el valor devuelto por la petición a '_https://www.googleapis.com/1_' esto es llamado **assimilation** y es lo que nos permite encolar las peticiones ya que el siguiente callback no se ejecuta hasta que la promesa devuelta por el anterior haya cambiado de estado a fulfilled o rejected.

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

La explicación menos técnica podría ser:
> Realiza una petición a la url "_https://www.googleapis.com/0_" y devuelveme un objeto al que le digas el resultado, cuando el objeto sepa el resultado entonces ejecutara esta función y despues esta otra ... .

Por último es importante destacar que aunque la operación que realizamos sea sincrona la ejecucion del callback del metodo then será simpre asincrona, esto es descrito en la especificación A/+:
 
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

Esta ejecución en asincrono en el caso de la libreria **rsvp** la realizá la funcion  _asap_ que elige la manera de encolar las peticiones en función de la plataforma, navegador, etc.

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

Nunca vamos a entrar en el catch, ya que el callback _onFulfilled_ pasado al metodo then se ejecuta en el siguiente turno del **event loop**.

### Gestión de errores

Como hemos visto el metodo then recibe como segundo parámetro un callback _onRejected_  que se ejecuta cuando salta una excepcion o la promesa es rechazada. Para capturar la excepcion del ejemplo anterior tendríamos que hacer algo asi:

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

Ya que como vimos al principio, una promesa solo puede cambiar de estado una vez, es decir, cuando ha pasado a **rejected** no puede pasar a **fulfilled** y viseversa, por lo tanto en el ejemplo de código anterior la función _errorHandler_ nunca se ejecutará si la excepción se produce en el la funcion _successHandler_.

### Otras cosas importantes

La gran mayoria de librerias de promesas cuentan con una serie de decoradores y funciones para facilitarnos la vida, estos permiten cosas como encapsular la ejecución de varias promesas en paralelo, metodos tipo **catch** que nos evitan tener que llamar a _then_ y pasarle un _null_, por ejemplo:

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

Lo mejor es mirar la documentación de cada libreria para aprovechar estas utilidades al maximo.

Como última recomendación utilizar **SIEMPRE** una libreria que cumpla el estandar **A/+**.













