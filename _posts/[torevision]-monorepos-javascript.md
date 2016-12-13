---
title: Monorepos en Javascript - ¿WTF?
---


Si te digo que empresas de la talla de Facebook y Google tiene todo su desarrollo en un único y gigantesco repositorio, ¿que es lo primero que piensas?. Mi primera reacción fue de sorpresa, acto seguido pensé "¿que puede tener eso de bueno?", lo cierto es que mucho, este modelo llamado **Monorepo** está lleno de ventajas, ventajas que veremos a lo largo de este post.

<a name="que-es-un-monorepo"></a>

## ¿Que es un monorepo?

**Un monorepo podría entenderse como un enorme repositorio que contiene todo el código y funcionalidad que nos sea posible incluir**, como vemos en esta definición entran casi cualquier cosa, en este post nos centraremos en los monorepos que tiene como objetivo *conseguir desde un único repositorio(monorepo) obtener varios entregables independientes(librerías, aplicaciones, paquetes, etc)* y trabajan con **Javascript**.

<a name="que-tiene-de-bueno"></a>

## ¿Que tiene de bueno un monorepo?

Sobre los beneficios se ha debatido mucho en internet y entre ellos se citan:

* Comodidad y facilidad a la hora de desarrollar, lo que se traduce en productividad.
* Facilidad a la hora de coordinar las distintas aplicaciones o módulos (según aplique).
* Unicidad del testing de la funcionalidad del monorepo.
* Unicidad del deploy y el setup del enviroment de desarrollo.
* Issues tracketing unificado.

<a name="y-de-malo"></a>

## ¿Y de malo?

Los principales inconvenientes son los que se pueden asociar a un repositorio gigante:

* Lo intimidante que puede resultar.
* El tamaño del repositorio. Aquí conviene aclarar que herramientas como Git y mercurial son increíblemente buenas gestionando repositorios grandes.

Otros posibles inconvenientes se pueden derivar de algunas de las ventajas, por ejemplo, unificar el deploy en determinados escenarios pueden traducirse a un proceso más complejo o dependiente y eso no siempre juega en nuestro favor.

<a name="quien-usa-monorepos"></a>

## ¿Quien usa monorepos?

La curiosidad por este tema me entro al leer [este](https://github.com/babel/babel/blob/master/doc/design/monorepo.md) markdown sobre el diseño de babel, en él podemos ver que en proyectos open source del mundo JS los monorepos son algo habitual, proyectos de la talla de [React](https://facebook.github.io/react/) o [Meteor](https://github.com/meteor/meteor) usan este modelo.

<a name="monorepos-en-la-practica"></a>

## Monorepos en la práctica.

Tenemos que ser conscientes de que no hay definido un patrón sobre cómo organizar y gestionar un monorepo, podemos basarnos en las aproximaciones seguidas por algunos proyectos open source de la comunidad de Javascript, tenemos dos formas principales de organización:

<a name="monorepo-monolitico"></a>

### Monorepo "monolítico"

**Este modelo se puede entender como un gran monolito que es fragmentado para su distribución mediante un proceso de "build"**. Para ver algo mas practico, observemos un poco el funcionamiento de [Lodash](https://lodash.com/).

Lodash es una librería con un montón de funcionalidad que tiene como objetivo simplificar el desarrollo en JS, si queremos usar lodash en un proyecto basta con hacer un `npm install lodash --save` y después un `require('lodash')`, y ya tendremos acceso a la versión monolítica, hasta aquí es igual al resto de librerías, ahora bien, imaginemos que estamos desarrollando una aplicación front donde el peso de la misma es algo muy importante y queremos usar lodash, si solo usamos un par de utilidades de lodash, incluir el monolito entero nos supondría mucho código inútil en nuestro proyecto, para no cargarnos con este peso, lodash nos da la opción de utilizar sus [microlibrerías](https://www.npmjs.com/browse/keyword/lodash-modularized), estas microlibrerías son el resultado de fragmentar y distribuir la funcionalidad de lodash, dicha fragmentación se realiza mediante un proceso de "build" con el que parten la librería monolítica. Recordemos el [objetivo](#que-es-un-monorepo)  *un único repositorio(repositorio de lodash), varios entregables(Lodash y sus microlibrerías)*.  

<a name="monorepo-compuesto-por-modulos-totalmente-independientes"></a>

### Monorepo compuesto por módulos totalmente independientes

Este modelo se basa en tener un monorepos compuesto por módulos totalmente independientes, es decir, **cada uno de los módulos que componen el monorepo podría ser extraídos del monorepo y debería garantizarse la continuidad de su ciclo de vida(build, testing, etc)**.

Primero aclaremos que debe tener un módulo para ser independiente:

* Su propio `packages.json`.
* Su propias dependencias.
* Sus propios scripts.
* Su propio registro en npm.

Como aproximaciones a este modelo, aunque no totales podemos encontrar con proyectos de la talla de [Babel](https://github.com/babel/babel/) o [Jest](https://github.com/facebook/jest), digo no totales porque en ambos se opta por unificar el "testing" y el "build" los módulos que los componen (en ambos casos unificar dichos procesos trae consigo muchísimos beneficios). conviene señalar que en ambos proyectos la gestión de los módulos es realizada con [lerna](https://lernajs.io/).

**Lerna** es una herramienta opensource para la línea de comandos que nos permite gestionar proyectos con múltiples `packages.json`, con ella podemos:

* Asociar dependencias entre los módulos de nuestro monorepo
* Ejecutar scripts de npm o bash para cada módulos de nuestro monorepo.
* Unir a nuestro monorepo módulos externos.
* Gestionar la publicación de los módulos de nuestro monorepo en npm de forma conjunta o independiente.

Veamos un poco como trabaja Lerna en la práctica, lo primero que tenemos que saber es que tendremos como contrato un directorio `./packages` dentro de nuestro proyecto que contendrá los módulos que administrará Lerna, partamos del siguiente ejemplo:

<pre>
.
├── lerna.json
├── LICENSE
├── node_modules
├── package.json # package.json del modulo padre
├── packages # directorio que contendra los submodulos
│       ├── module-1 # modulos del monorepo
│       │   ├── LICENSE
│       │   ├── package.json
│       │   ├── README.md
│       │   └── index.js
│       ├── module-2 # modulos del monorepo
│       │   ├── LICENSE
│       │   ├── package.json
│       │   ├── README.md
│       │   └── index.js
│       └── module-3 # modulos del monorepo
│           ├── LICENSE
│           ├── package.json
│           ├── README.md
│           ├── test.js
│           └── index.js
└── README.md
<pre>

Ahora imaginemos que `module-3` tiene el siguiente `package.json`

```json
{
  "name": "module-3",
  "version": "0.0.1",
  "description": "is module-3",
  "main": "index.js",
  "scripts": {
    "test": "node ./test.js"
  },
  "dependencies": {
    "module-1": "*",
    "module-2": "*",
    "lodash": "^4.0.0"
  }
}
```

Como vemos en él están definidas 2 dependencias internas (`module-1` y `module-2`) y una dependencias externa, pues bien si ejecutamos:

```bash
$ lerna bootstrap
```

Lerna recorrerá todos los módulos dentro del directorio `./packages` e instalará sus dependencias, en caso de que sean dependencias internas creará `symlinks` y en caso de las dependencias externas a nuestro monorepo, lerna se descargara las mismas. Pasaremos a tener la siguiente estructura:

<pre>
.
├── lerna.json
├── LICENSE
├── node_modules
├── package.json # package.json del modulo padre
├── packages # directorio que contendra los submodulos
│       ├── module-1 # modulos del monorepo
│       ├── module-2 # modulos del monorepo
│       └── module-3 # modulos del monorepo
│           ├── LICENSE
│           ├── package.json
│           ├── README.md
│           ├── test.js
│           ├── index.js
│           └── node_modules
│               ├── module-1
│               ├── module-2
│               └── lodash
└── README.md
<pre>

Como vimos anteriormente también podemos ejecutar scripts de npm por cada módulo, así podríamos ejecutar el script de test:

```bash
$ lerna run test
```

Para más información sobre cómo trabajar con lerna te recomiendo su [documentación](https://github.com/lerna/lerna#readme) y aquí os dejo un ejemplo en [este repositorio](https://github.com/nquicenob/lerna-example).


<a name="monorepo-basado-en-el-alle-model"></a>

### Monorepo basado en el "Alle model"

Este modelo es usado en [pouchdb](https://github.com/pouchdb/pouchdb) y esta un poco a medio camino entre un [monorepo monolítico](#monorepo-monolitico) y un [monorepo compuesto por módulos totalmente independientes](#monorepo-compuesto-por-modulos-totalmente-independientes), en el se tiene **módulos semi independientes**, es decir, **son módulos que se distribuyen de forma independientes pero necesitan de otros módulos del monorepo para su funcionamiento**, veamos en qué consiste.

Partamos de un proyecto con la siguiente estructura:

<pre>
.
├── LICENSE
├── package.json # package.json con los scripts y las dependencias globales para los modulos
├── packages
│   └── node_modules # directorio donde almacenaremos nuestros modulos
│       ├── module-1 # modulos del monorepo
│       │   ├── LICENSE
│       │   ├── package.json
│       │   ├── README.md
│       │   └── index.js
│       ├── module-2 # modulos del monorepo
│       │   ├── LICENSE
│       │   ├── package.json
│       │   ├── README.md
│       │   └── index.js
│       └── module-3 # modulos del monorepo
│           ├── LICENSE
│           ├── package.json
│           ├── README.md
│           └── index.js
├── README.md
└── tests
<pre>

Donde `./packages/node_modules/module-3/package.json` tiene el siguiente contenido:

```json
{
  "name": "module-3",
  "version": "0.0.1",
  "description": "is module-3",
  "main": "index.js"
}
```

Y el fichero `./packages/node_modules/module-3/index.js`:

```js
import module2 from 'module-2';
import module3 from 'module-2';

// ... code
```

De lo anterior podemos observar:

* Los módulos se encuentran bajo un directorio `./packages/node_modules`.
* El módulo `module-3` no tiene definidas sus dependencias en su `package.json`.
* El módulo `module-3` usa dependencias internas del monorepo.

¿Como funciona esto? sencillo, debemos pensar en el funcionamiento del [algoritmo de resolución de dependencias de  node](https://nodejs.org/api/modules.html#modules_all_together).

Con los anterior podemos tener en nuestro entorno de desarrollo módulos sin dependencias pero con un fuerte acoplamiento, ¿como conseguimos que estos módulos se transformen en entregables? lo que mejor hacemos en javascript es construir bundles de módulos y es lo que haremos para construir nuestros entregables, para ellos se pueden usar herramientas tipo [rollup](http://rollupjs.org/) o [webpack](https://webpack.github.io/).

[Aquí](https://github.com/nquicenob/alle-example) os dejo otro ejemplo de un monorepo gestionado por este modelo y [aquí](https://gist.github.com/nolanlawson/457cdb309c9ec5b39f0d420266a9faa4) podéis ver un poco el razonamiento de porqué abandonar lerna por este modelo.


## Más información

### Info

* [Why is Babel a monorepo?](https://github.com/babel/babel/blob/master/doc/design/monorepo.md).
* [Model "Alle"](https://github.com/boennemann/alle).
* [Monorepo, scaling projects](https://kikobeats.com/monorepo/)
* [Advantages of monolithic version contro](http://danluu.com/monorepo/)
* [Why we dropped Lerna from PouchDB](https://gist.github.com/nolanlawson/457cdb309c9ec5b39f0d420266a9faa4)

### Librerías usadas como ejemplo

* [Lodash](https://github.com/lodash/lodash)
* [pouchdb](https://github.com/pouchdb/pouchdb)
* [Babel](https://github.com/babel/babel)
* [Jest](https://github.com/facebook/jest)
* [React](https://facebook.github.io/react/)
* [Meteor](https://github.com/meteor/meteor)

### herramientas

* [Lerna](https://github.com/lerna/lerna)
* [Rollup](https://github.com/rollup/rollup)
* [Webpack](https://webpack.github.io/)
