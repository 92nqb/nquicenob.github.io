---
title: Jugando con el contexto en JS, ¿Cuánto vale result?
---

Una de las cosas que al principio resultan más extrañas para un desarrollador que se aventura en JS es el valor que tiene **this** dentro de una función. Jugando con este tema elaboré un problema para picar a un amigo. Antes de ver el problema explicaré muy brevemente algunos conceptos: 

* **El valor de this** será siempre lo que esté justo antes del '.' a la hora de invocar una función, si no hubiera nada antes del '.', el valor de this pasaría a ser **global** o **windows** según el caso. Por ejemplo:

{% highlight js %}

obj.fn(); // this  es obj 
fn(); // this es global o window 

{% endhighlight %}

* **El ámbito de una variable en JS** (esto cambia con ecmascript 6) es global a la función que la contiene. Por ejemplo:

{% highlight js %}

(function(){
  var foo = 'global var';
  (function(){
    console.log(bar); // undefined
    if(true){
      var bar = 'local var';
    }
    console.log(bar); // local var
    console.log(foo); // global var
  })();
  console.log(bar); // throw err
  console.log(foo); // global var
})();

{% endhighlight %}

* **El operador new** crea una instancia nueva de this, entre otras cosas. Por ejemplo:

{% highlight js %}

function CreateA(){
  this.a = 'a';
}
var obj1 = CreateA();// obj es undefined, 
// window o global parsarian a tener la 
// propiedad a con valor 'a' 
var obj2 = new CreateA(); // obj es CreateA { a: 'a' }

{% endhighlight %}

* **Funciones como bind, apply y call** cambian el contenido de this:

{% highlight js %}

var obj1 = {
  a : 1,
  b : 2,
  sum : function(){
    return this.a + this.b;
  }
}
var obj2 = {
  a : -1,
  b : -2
}

obj1.sum(); // devuelve 3
obj1.sum.call(obj2); // devuelve -3

{% endhighlight %}

Explicado esto, os dejo el problema que debería ser resuelto solo con lápiz y papel.

{% highlight js %}

(function(){
  this.c = 0;

  function add(){
    return this.a + this.b;
  }

  function ClassObj(param1, param2, param3, param4){
    this.a = param1;
    this.b = param2;
    this.c = param3;
    this.result = (function(p4){
      var secret = p4;
      return function(ctx){
        return this.c + add.call(ctx) + secret;
      }
    })(param4);
    return this;
  }

  function changeCtx(cb){
    var self = this;
    return function(obj){
      return cb.call(self, obj);
    }
  }

  var obj1 = new ClassObj(1, 2, 3, 4);
  var obj2 = new ClassObj(-1, -2, -3, -4);
  var val1 = changeCtx(obj2.result)(obj1);
  var obj3 = ClassObj(4, 3, 2, 1);
  var val2 = changeCtx(obj2.result)(obj1);
  var val3 = changeCtx(obj1.result)(obj3);

  var result = val1 + val2 + val3;
  //¿Cuanto vale result?
})();

{% endhighlight %}


## Buscando la solución

Primero vamos a por **val1**:

{% highlight js %}

var val1 = changeCtx(obj2.result)(obj1);

// 1. changeCtx
function changeCtx(cb){
  var self = this; // guarda el valor de this en la 
  // variable self, como hemos visto antes no hay nada 
  // delante de la invocación de changeCtx, por lo tanto 
  // this será global
  return function(obj){ 
    return cb.call(self, obj); // Devuelve otra función 
    // que ejecuta el callback de la primera llamada 
    // con el contexto global, por lo tanto,
    // ejecutamos obj2.result con el contexto de self y
    // como parámetro obj1
  }
}

// 2. result
function ClassObj(param1, param2, param3, param4){
  // ..
  this.result = (function(p4){
    // esta función es un poco especial ya que es
    // auto-invocada y almacena param4 en la variable secret 
    // cuando se construye un nuevo objeto
    var secret = p4;
    return function(ctx){
      // Esta función es la más compleja, ya que trabaja 
      // con tres contextos distintos:
      // 1. this, que al ser invocada dentro de changeCtx pasa 
      // a ser self que a su vez es global
      // 2. ctx, que es el contexto que llega como parámetro y se usa 
      // para llamar a la función add
      // 3. El contexto del objeto al que pertenece result
      // Por lo tanto:
      
      // secret es -4, ya que es la única variable que pertenece al 
      // objeto propietario de result, en este caso obj2

      // this.c es 0, ya que es lo mismo que global.c

      // add ejecuta la suma de this.a + this.b, pero llamamos a 
      // add mediante call, que como sabemos cambia el 
      // contexto de la invocación, 
      // siendo este ctx que a su vez es obj1 {a: 1, b: 2, c: 3, d: 4}
      // así que add suma 1 + 2
    
      // 0 + (1 + 2) + (-4)
      return this.c + add.call(ctx) + secret;      
    }
  })(param4);
  // ..
}

// val1 = 0 + (1 + 2) + (-4)
var val1 = changeCtx(obj2.result)(obj1);

{% endhighlight %}

Ahora a por **val2**, de un primer vistazo podriamos pensar que es lo mismo que **val1** ya que:

{% highlight js %}

var val1 = changeCtx(obj2.result)(obj1);
// ...
var val2 = changeCtx(obj2.result)(obj1);

{% endhighlight %}

Pero como diría un profesor muy bueno que tuve de matemáticas, "no padre", básicamente por esta instrucción:

{% highlight js %}

var obj3 = ClassObj(4, 3, 2, 1);

{% endhighlight %}

Que se ejecuta justo antes de obtener val2. Como hemos visto antes no hemos puesto el operador **new**, por lo tanto las asignaciones a this dentro de **ClassObj** las estamos haciendo a **global**

{% highlight js %}

function ClassObj(param1, param2, param3, param4){
  this.a = param1;
  this.b = param2;
  this.c = param3;
  // ...
  return this;
}

// ...

var obj3 = ClassObj(4, 3, 2, 1);
// Ahora global pasa a tener las propiedades que se asignan 
// mediante this dentro de ClassObj, es decir, 
// global = { ..., a: 4, b:3, c:2 }
// y la operación tendría estos valores

// val2 = 2 + (1 + 2) + (-4)
var val2 = changeCtx(obj2.result)(obj1);
{% endhighlight %}

Por último a por **val3**:

{% highlight js %}

//val3 = 2 + (4+3) + 4
var val3 = changeCtx(obj1.result)(obj3);

{% endhighlight %}

Por lo tanto **result**:

{% highlight js %}

// result = -1 + 1 + 13
var result = val1 + val2 + val3;

{% endhighlight %}

Pd: He intentado explicarlo de la forma mas sencilla posible xD.

Más información

* [Understanding Javascript function invocation and this](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/ "Understanding Javascript function invocation and this")
* [Common Javascript gotchas](http://www.jblotus.com/2013/01/13/common-javascript-gotchas/ "Common Javascript gotchas")
* [MDN's documentation on apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply "MDN's documentation on apply")
* [MDN's documentation on call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call "MDN's documentation on call")
