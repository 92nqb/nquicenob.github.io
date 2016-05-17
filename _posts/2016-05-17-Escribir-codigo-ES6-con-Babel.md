---
Escribir código ES6 con Babel
---

Recientemente se ha liberado la version la [version 6.0 de Node](https://nodejs.org/en/blog/release/v6.0.0/) que cuenta con un **93%** de compatibilidad con **ES2015** :). Como no siempre gozamos con la posibilidad de trabajar con versiones de node que tengan tanta compatibilidad o simplemente por requisitos del proyecto hay que garantizar compatibilidad con motores js antiguos, nos vemos obligados a recurrir a herramientas como **Babel**.

**Babel** es un compilador *source-to-source* tambien llamado *transpilador*, esto para entendernos quiere decir que podemos escribir código en ES6 y decirle a Babel que nos los compile a código javascript compatible (ej: ES5.1).

Despues de esta breve explicación, vamos a ver como trabajar con Babel :).

Primero, creamos la estructura del proyecto:

```
.
├── /lib/                 # Temp folder for compiled output           
├── /node_modules/         # 3rd-party libraries and utilities
├── /src/                  # ES2015 source code
└── package.json           # Project settings
```

Como podemos ver nuestro código en ES6 lo escribiremos en el directorio `/src` y le diremos a Babel que nos vuelque el codigo transpilado en el directorio llamado `lib`, tambien es habitual para este directorio el nombre de `/build` o de `/dist`.

Una vez creada nuestra estructura de directorio, añadimos Babel a las `devDependencies` de nuestro `package.json`.

```
$ npm install babel-cli --save-dev
```

`babel-cli` trae consigo un monton de utilidades, entre ellas:

* **babel** (valga la redundancia) que nos vuelca una salida con el codigo transpilado. Podemos verlo en `./node_modules/babel-cli/bin/babel.js` y recibe como parámetro el archivo que contiene el código a transpilar.
* **babel-node** que interpreta el código y lo ejecuta, es decir, no hace falta una transpilar previamente. Se encuentra en `./node_modules/babel-cli/bin/babel-node.js`.

Para las pruebas de este post partiremos de un archivo llamado `/main.js` ubicado en `/src`.

```
.
├── /lib/                  # Temp folder for compiled output           
├── /node_modules/         # 3rd-party libraries and utilities
├── /src/                  # ES2015 source code
|   └── /main.js           # test code written in ES6
└── package.json           # Project settings
```

Que contiene el siguiente código:

```js
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
```

Si en este punto tratamos de interpretar con `babel-node` nuestro archivo `main.js`:

```
$ ./node_modules/babel-cli/bin/babel-node.js src/main.js // Error
```

Veremos que nos devuelve un error, esto es debido a que Babel por si solo no sabe que tipo de código de entrada espera. Para ello instalamos su extensión para ES6:

```
$ npm install babel-preset-es2015 --save-dev
```

y creamos un archivo `/.babelrc` que le diga a Babel la configuración que tiene que usar al transpilar, esta configuración tambien puede ir por parámetro (no recomendado), el contenido de `/.babelrc` ha de ser el siguiente:

```
{
  "presets": ["es2015"]
}
```

si volvemos a ejecutar:

```
$ ./node_modules/babel-cli/bin/babel-node.js src/main.js
```

Veremos la ejecución correcta de nuestro `/src/main.js`;

```js
// hello world
// { a: 'a', b: 'b' }
// { value: 0, done: false }
// { value: 1, done: false }
// { value: 2, done: false }
// { value: 3, done: false }
```

Vamos a probar a  transpilar, para ello ejecutamos:

```
$ ./node_modules/babel-cli/bin/babel.js src/main.js -o lib/main.js
```

Se ha creado un archivo llamado `/lib/main.js`. El comando anterior sería conveniente llevarlo a un script en nuestro `package.json`:

```js
// package.json
{
  // ...
  "scripts": {
    "compile": "babel src -d lib"
  },
  // ...
}
```

El parámetro `-d` le indica a babel que tiene que transpilar el directorio completo.

Ya tenemos nuestro primer archivo transpilado, pero con alguna pega xD, si tratamos de ejecutar el archivo que acabamos de transpilar:

```
$ node lib/main.js
```

Deberiamos ver en nuestro terminal algo como:

```
ReferenceError: regeneratorRuntime is not defined
  # ...
```

Sin embargo, cuando ejecutamos `babel-node` pudimos ver en consola la salida de nuestro archivo `/src/main.js`, esto es por que `babel-node` utiliza `babel-polyfill` para hacer las transformaciones de código a nivel global. Vamos a añadir `babel-polyfill` a nuestras `dependencies`:

```
$ npm install babel-polyfill --save
```

y lo importamos en nuestro `/src/main.js`

```js
import "babel-polyfill";

// ... js code
```

Volvemos a transpilar con el script que hemos añadido en los pasos anteriores:

```
$ npm run compile
```

Si ahora ejecutamos

```
$ node lib/main.js
```

tenemos que ver en consola algo así:

```js
// hello world
// { a: 'a', b: 'b' }
// { value: 0, done: false }
// { value: 1, done: false }
// { value: 2, done: false }
// { value: 3, done: false }
```

Que es la salida correcta de nuestro `/src/main.js` :).

Ahora bien, con `babel-polyfill` como hemos dicho tendriamos a nivel global estas transformaciones de código y las cosas globales no suelen ser de buen gusto, si queremos disponer de estas transformaciones solo a nivel de nuestro `/main.js` debemos ir por otro camino.

Primero eliminamos a `babel-polyfill` de nuestras `dependencies`:

```
$ npm uninstall babel-polyfill --save
```

Tambien borramos la linea añadida a nuestro `/src/main.js` donde haciamos `import "babel-polyfill"`.

En remplazo de `babel-polyfill` vamos a añadir un plugin. Para ellos ejecutamos:

```
$ npm install babel-plugin-transform-runtime --save-dev
$ npm install --save babel-runtime
```

Este plugin nos permite tener las transformaciones de `babel-polyfill` sin manchar nuestro contexto global.

Ahora tenemos que decirle a Babel que utilice este plugin en la transpilación, para ello modificamos nuestro archivo `/.babelrc` añadiendo:

```js
{
  "presets": ["es2015"],
  "plugins": ["transform-runtime"] // new plugin
}
```

Volvemos a transpilar:

```
$ npm run compile
```

Y si volvemos a ejecutar:

```
$ node lib/main.js
```

Veremos en la consola la salida correcta:

```js
// hello world
// { a: 'a', b: 'b' }
// { value: 0, done: false }
// { value: 1, done: false }
// { value: 2, done: false }
// { value: 3, done: false }
```

Si abrimos nuestro archivo transpilado en `lib/main.js`, veremos como obtiene del `babel-runtime` alguna funcionalidad, en lugar de ser añadida de manera global a los objetos. 
Espero que sea de utilidad :).
