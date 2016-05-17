---
title: Escribir código ES6 con Babel
---

Recientemente se ha liberado la version la [version 6.0 de Node](https://nodejs.org/en/blog/release/v6.0.0/) que cuenta con un **93%** de compatibilidad con **ES2015** :), como no siempre gozamos con la posibilidad de trabajar con versiones de node que tengan tanta compatibilidad o simplemente porque proyecto en el que estamos trabajando exige garantizar compatibilidad con motores js antiguos, nos vemos obligados a recurrir a herramientas como **Babel**.

**Babel** es un compilador *source-to-source* también llamado *transpilador*, en lo práctico quiere decir que podemos escribir código en ES6 y decirle a Babel que nos transpile a código javascript compatible (ej: ES5.1).
Después de esta breve explicación, vamos a ver cómo trabajar con Babel.

Para este ejemplo, primero vamos a crear la estructura del proyecto:

<pre>
.
├── /lib/            # Temp folder for compiled output
├── /node_modules/   # 3rd-party libraries and utilities
├── /src/            # ES2015 source code
└── package.json     # Project settings
</pre>

Como podemos ver nuestro código en ES6 lo escribiremos en el directorio `/src` y Babel nos volcara el código transpilado en el directorio llamado `/lib`, también es habitual para este directorio el nombre de `/build` o de `/dist`.

Una vez creada nuestra estructura de directorios, añadimos Babel a las `devDependencies` de nuestro `package.json`.

<pre>
$ npm install babel-cli --save-dev
</pre>

`babel-cli` trae consigo un varias utilidades, entre ellas:

* **babel.js** que nos da como salida el código transpilado. En la versión **^6.0** de `babel-cli`, `babel.js` se encuentra en `/node_modules/babel-cli/bin/babel.js`.
* **babel-node** que es un interprete, es decir, ejecuta el código sin transpilar previamente. En la versión **^6.0** de `babel-cli` podemos encontrarlo en `./node_modules/babel-cli/bin/babel-node.js`.

Para las pruebas de este post partiremos de un archivo llamado `/main.js` ubicado en `/src`:

<pre>
.
├── /lib/                  # Temp folder for compiled output           
├── /node_modules/         # 3rd-party libraries and utilities
├── /src/                  # ES2015 source code
|   └── /main.js           # test code written in ES6
└── package.json           # Project settings
</pre>

Que contiene el siguiente código:

{% highlight js %}
// arrow fn
(() => {
  console.log('hello world');
})();

// Object assign
console.log(Object.assign({}, {
  a: 'a',
  b: 'b'
}));

// generator
function* gen(){
  let num = 0;
  while(true) {
    yield num;
    num++
  }
}

const genFn = gen();
console.log(genFn.next());
console.log(genFn.next());
console.log(genFn.next());
console.log(genFn.next());
{% endhighlight %}

Si en este punto tratamos de interpretar con `babel-node` nuestro archivo `/main.js`:

<pre>
$ ./node_modules/babel-cli/bin/babel-node.js src/main.js // Error
</pre>

Veremos que nos devuelve un error, esto es porque Babel es una herramienta de propósito general y necesita plugins y una configuración que le definan qué hacer exactamente. Para ello instalamos el `present` para ES6:

<pre>
$ npm install babel-preset-es2015 --save-dev
</pre>

Un `present` es un conjunto de plugins.

Una vez terminada la instalación, le decimos a Babel que haga uso del mismo, para ello creamos un archivo `/.babelrc` con la configuración que tiene que usar al transpilar, esta configuración también puede ir por parámetro (no recomendado). El contenido de `/.babelrc` ha de ser el siguiente:

{% highlight json %}
{
  "presets": ["es2015"]
}
{% endhighlight %}

Si volvemos a ejecutar:

<pre>
$ ./node_modules/babel-cli/bin/babel-node.js src/main.js
</pre>

Veremos la ejecución correcta de nuestro `/src/main.js`:

<pre>
hello world
{ a: 'a', b: 'b' }
{ value: 0, done: false }
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
</pre>

Vamos a probar a transpilar, para ello ejecutamos:

<pre>
$ ./node_modules/babel-cli/bin/babel.js src/main.js -o lib/main.js
</pre>

Se ha debido de crear un archivo llamado `/lib/main.js`. El comando anterior sería conveniente llevarlo a un script en nuestro `package.json`:

{% highlight json %}
{
  "scripts": {
    "compile": "babel src -d lib"
  }
}
{% endhighlight %}

El parámetro `-d` le indica a babel que tiene que transpilar el directorio completo. Tampoco estaría de más otro script que limpie el directorio `/lib` previamente.

{% highlight json %}
{
  "scripts": {
    "clean": "rm -rf lib/*",
    "compile": "npm run clean && babel src -d lib"
  }
}
{% endhighlight %}

Como ya tenemos nuestro primer archivo transpilado, vamos a tratar de ejecutarlo:

<pre>
$ node lib/main.js
</pre>

Deberíamos ver en nuestro terminal algo como:

<pre>
ReferenceError: regeneratorRuntime is not defined
  # ...
</pre>

Sin embargo, cuando ejecutamos `babel-node` pudimos ver en consola la salida de nuestro archivo `/src/main.js`, esto es porque `babel-node` utiliza `babel-polyfill` para hacer las transformaciones de código a nivel global. Vamos a añadir `babel-polyfill` a nuestras `dependencies`:

<pre>
$ npm install babel-polyfill --save
</pre>

Y lo importamos en nuestro `/src/main.js`

{% highlight js %}
import "babel-polyfill";

// ... js code
{% endhighlight %}

Volvemos a transpilar con el script que hemos añadido en los pasos anteriores:

<pre>
$ npm run compile
</pre>

Si ahora ejecutamos

<pre>
$ node lib/main.js
</pre>

Tenemos que ver en consola la salida correcta de nuestro `/main.js`:

<pre>
hello world
{ a: 'a', b: 'b' }
{ value: 0, done: false }
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
</pre>

Ahora bien, con `babel-polyfill` como hemos dicho tendríamos a nivel global estas transformaciones de código, las cosas globales no suelen ser de buen gusto, si queremos disponer de estas transformaciones sólo a nivel de nuestro `/main.js` debemos hacer uso de otros plugins.

Primero eliminamos a `babel-polyfill` de nuestras `dependencies`:

<pre>
$ npm uninstall babel-polyfill --save
</pre>

También borramos la línea añadida a nuestro `/src/main.js` donde hacíamos `import "babel-polyfill"`.

En reemplazo de `babel-polyfill` vamos a añadir el plugin de `babel-plugin-transform-runtime`. Para ellos ejecutamos:

<pre>
$ npm install babel-plugin-transform-runtime --save-dev
$ npm install --save babel-runtime
</pre>

Este plugin nos permite tener las transformaciones de `babel-polyfill` sin manchar nuestro contexto global.

Ahora tenemos que decirle a Babel que utilice este plugin en la transpilación, para ello modificamos nuestro archivo `/.babelrc` añadiendo:

{% highlight json %}
{
  "presets": ["es2015"],
  "plugins": ["transform-runtime"] // new plugin
}
{% endhighlight %}

Volvemos a transpilar:

<pre>
$ npm run compile
</pre>

Y volvemos a ejecutar:

<pre>
$ node lib/main.js
</pre>

Veremos en la consola la salida correcta:

<pre>
hello world
{ a: 'a', b: 'b' }
{ value: 0, done: false }
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
</pre>

Si abrimos nuestro archivo transpilado en `lib/main.js`, veremos cómo importa de `babel-runtime` alguna funcionalidad, en lugar de ser añadida de manera global a los objetos.
Espero que sea de utilidad :).


Más información.

* [Babel](http://babeljs.io/)
* [Manual de usuario de babel](https://github.com/thejameskyle/babel-handbook)
* [Módulos con babel y mas herramientas](https://medium.com/@tarkus/how-to-build-and-publish-es6-modules-today-with-babel-and-rollup-4426d9c7ca71#.k59zua4)
