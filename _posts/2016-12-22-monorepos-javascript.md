---
title: Monorepos en Javascript
---

Empresas de la talla de Facebook y Google tiene todo su desarrollo en un único y gigantesco repositorio, esto no es casualidad, tener un sistema de control de versiones monolítico, comúnmente conocidos como *"monorepos"*, es algo que nos puede aportar mucho valor como veremos a lo largo de este post.

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
* Issues tracking unificado.

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

Tenemos que ser conscientes de que no hay definido un patrón sobre cómo organizar y gestionar un monorepo, podemos basarnos en las aproximaciones seguidas por algunos proyectos open source de la comunidad, yo he encontrado 3 modelos diferentes que veremos a continuación:

<a name="monorepo-monolitico"></a>

### Monorepo "monolítico"


> Un Monorepo "monolítico" es un repositorio que contiene un módulo con mucha funcionalidad que es fragmentado en módulos independientes para su distribución.

Para ver un ejemplo algo más práctico, observemos un poco el funcionamiento de [Lodash](https://lodash.com/).

Lodash es una librería con un montón de funcionalidad que tiene como objetivo simplificar el desarrollo en JS, si queremos usar lodash en un proyecto basta con hacer un `npm install lodash --save` y después con un `require('lodash')` tendremos acceso a toda la funcionalidad de la librería bajo un *namespace*, hasta aquí es igual al resto de librerías, ahora bien, imaginemos que estamos desarrollando una aplicación front donde el peso de la misma es algo muy importante y queremos usar lodash, si solo usamos un par de utilidades de lodash incluir la librería entera nos supondría mucho código inútil en nuestro proyecto, para no cargarnos con este peso, lodash nos da la opción de utilizar sus [microlibrerías](https://www.npmjs.com/browse/keyword/lodash-modularized), estas microlibrerías son el resultado de fragmentar y distribuir la funcionalidad de lodash, dicha fragmentación se realiza mediante un proceso de "build" con el que parten el monolito en micromódulos que contiene solo una funcionalidad, lo que convierte a Lodash en un monorepo monolítico; recordemos el [objetivo](#que-es-un-monorepo):  *un único repositorio(repositorio de lodash), varios entregables(Lodash y sus microlibrerías)*.  

<a name="monorepo-compuesto-por-modulos-totalmente-independientes"></a>

### Monorepo compuesto por módulos independientes

> Un monorepo compuesto por módulos independientes es un repositorio en el que se desarrollan simultáneamente varios módulos, dichos módulos NO tiene la obligatoriedad de pertenecer al mismo repositorio.

Primero aclaremos que debe tener un módulo de npm para ser independiente:

* Su propio `packages.json`.
* Sus propia definición de dependencias.
* Sus propios "npm scripts".

Cumpliendo lo anterior, podemos observar que cualquiera de los módulos que componen el repositorio podría ser extraído del monorepo teniendo garantías de poder tener continuidad en su ciclo de vida(install, build, testing, etc).

Como aproximaciones a este modelo, aunque no totales podemos encontrar con proyectos de la talla de [Babel](https://github.com/babel/babel/) o [Jest](https://github.com/facebook/jest), digo no totales porque en ambos se opta por unificar el *"testing"* y el *"building"* de los módulos que los componen (en ambos casos unificar dichos procesos trae consigo muchísimos beneficios). Conviene señalar que en ambos proyectos la gestión de los módulos es realizada con [lerna](https://lernajs.io/).

**Lerna** es una herramienta opensource que nos permite gestionar proyectos con múltiples `packages.json`, con ella podemos:

* Asociar dependencias entre los módulos de nuestro monorepo.
* Ejecutar scripts de npm o bash para cada módulo.
* Unir a nuestro monorepo módulos externos.
* Gestionar la publicación de los módulos de nuestro monorepo en npm de forma conjunta o independiente.

Veamos un poco como trabaja Lerna en la práctica:

Lo primero que tenemos que saber es que el contrato con Lerna es un directorio `./packages` en el raiz de nuestro proyecto, dicho directorio contendrá los módulos que gestionará Lerna, partamos del siguiente ejemplo:

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
</pre>

Ahora imaginemos que `./packages/module-3/package.json` tiene el siguiente contenido:

{% highlight json %}
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
{% endhighlight %}

Como vemos en él están definidas 2 dependencias internas (`module-1` y `module-2`) y una dependencia externa (`lodash`). Si ejecutamos:

<pre>
$ lerna bootstrap
</pre>

Lerna recorrerá todos los directorios que estén dentro del directorio `./packages` e instalará sus dependencias, en el caso de que las dependencias sean internas creará `symlinks` dentro del directorio `node_modules` del módulo, en el caso de las dependencias sean externas a nuestro monorepo, lerna se descargara las mismas. Con el comando anterior conseguimos la siguiente estructura:

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
│               ├── module-1 # symlink
│               ├── module-2 # symlink
│               └── lodash
└── README.md
</pre>

Como vimos anteriormente también podemos ejecutar scripts de npm por cada módulo, por ejemplo con este comando:

<pre>
$ lerna run test
</pre>

Ejecutaremos `npm run test` en cada módulo.

Para más información sobre cómo trabajar con lerna os recomiendo su [documentación](https://github.com/lerna/lerna#readme) y aquí os dejo un ejemplo en [este repositorio](https://github.com/nquicenob/monorepo-lerna-example).

<a name="monorepo-basado-en-el-alle-model"></a>

### Monorepo basado en el "Alle model"

> Un monorepo basado en el "Alle model" es una variación de un "monorepo compuesto por módulos independientes" con la diferencia de que en este tipo de monorepo los módulos están  obligados a pertenecer al repositorio y a desarrollarse bajo un directorio `/node_modules` obteniendo como ventajas la posibilidad de definir dependencias globales entre los mismos y unicidad y simpleza en sus ficheros de configuración.

Este tipo de monorepo es usado en [pouchdb](https://github.com/pouchdb/pouchdb), veamos un ejemplo más práctico:

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
</pre>

Donde `./packages/node_modules/module-3/package.json` tiene el siguiente contenido:

{% highlight json %}
{
  "name": "module-3",
  "version": "0.0.1",
  "description": "is module-3",
  "main": "index.js"
}
{% endhighlight %}

Y el fichero `./packages/node_modules/module-3/index.js`:

{% highlight js %}
import module2 from 'module-2';
import module3 from 'module-2';

// ... code
{% endhighlight %}

De lo anterior podemos observar:

* Los módulos se encuentran bajo un directorio `./packages/node_modules`.
* El módulo `module-3` **NO** tiene definidas dependencias en su `package.json`.
* El módulo `module-3` en su fichero `ìndex.js` importa los módulos `module-2`  y `module-1`.

Ahora nos preguntaremos:

**¿Cómo puede funcionar el módulo `module-3` si no hay definidas dependencias pero vemos que se hacen `imports` en el código?.**

Debemos pensar en el comportamiento del [algoritmo de resolución de dependencias de  node](https://nodejs.org/api/modules.html#modules_all_together) que al detectar que se encuentra bajo un directorio `node_modules` resuelve las dependencias buscándolas en el mismo directorio. Por lo tanto tenemos directorios que representan módulos de nuestro monorepo pero con un gran acoplamiento.

**¿Cómo resolvemos dicho acoplamiento para transformarlos en entregables?.**

Lo que mejor hacemos en javascript es transpilar y unir módulos para construir *bundles*, y esto es lo que haremos para construir los entregables, para ello podemos hacer uso de herramientas como [rollup](http://rollupjs.org/) con su plugin [node resolve](https://github.com/rollup/rollup-plugin-node-resolve).

[Aquí](https://github.com/nquicenob/monorepo-alle-example) os dejo un ejemplo con componentes de [vue](https://github.com/vuejs/vue) y [aquí](https://gist.github.com/nolanlawson/457cdb309c9ec5b39f0d420266a9faa4) podéis ver un poco el razonamiento de cuándo  es mejor usar este modelo que basar la gestión en Lerna.

## Más información

### Info

* [Why is Babel a monorepo?](https://github.com/babel/babel/blob/master/doc/design/monorepo.md).
* [Model "Alle"](https://github.com/boennemann/alle).
* [Monorepo, scaling projects](https://kikobeats.com/monorepo/).
* [Advantages of monolithic version control](http://danluu.com/monorepo/).
* [Why we dropped Lerna from PouchDB](https://gist.github.com/nolanlawson/457cdb309c9ec5b39f0d420266a9faa4).

### Librerías usadas como ejemplo

* [Lodash](https://github.com/lodash/lodash).
* [pouchdb](https://github.com/pouchdb/pouchdb).
* [Babel](https://github.com/babel/babel).
* [Jest](https://github.com/facebook/jest).
* [React](https://facebook.github.io/react/).
* [Meteor](https://github.com/meteor/meteor).

### herramientas

* [Lerna](https://github.com/lerna/lerna).
* [Rollup](https://github.com/rollup/rollup).
* [Webpack](https://webpack.github.io/).
