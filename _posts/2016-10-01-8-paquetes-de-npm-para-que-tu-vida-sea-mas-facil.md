---
title: 8 paquetes de npm para que tu vida sea más fácil
---

El ecosistema de **node js** es una auténtica locura, es increíble el ritmo al que se mueve la comunidad y la cantidad de librerías y herramientas que se publican todos los días. Aquí os traigo un listado de algunos paquetes de _npm_ que no son los típicos (_webpack, babel, eslint, etc_) y que pueden ayudar a mejorar nuestra **"Developer Experience" (DX)**.

### 1. ["rimaraf"](https://github.com/isaacs/rimraf). Borrar archivos y directorios como 'rm -rf'.

Según esta el patio, un proyecto JS no es molón si no tiene por lo menos un _transpilador_, un _compilador_ o un _minificador_, esto nos obliga a tener casi siempre un directorio para archivos temporales que debemos borrar de manera repetitiva. Con [rimaraf](https://github.com/isaacs/rimraf) podemos  automatizar esta tarea, su funcionamiento es simple, tiene la misma funcionalidad que el comando `rm -rf` de sistemas unix.

### 2. ["mkdirp"](https://github.com/substack/node-mkdirp). Crear directorios recursivamente

Igual que borramos directorios y archivos, muchas veces tenemos que crearlos, con [mkdirp](https://github.com/substack/node-mkdirp) podemos crear directorios recursivamente igual que con el comando `mkdir -p`.

### 3. ["cross-env"](https://github.com/kentcdodds/cross-env). Variables de entorno en 'Unix style' para cualquier SO (incluido windows)

Aunque sea duro de decir, existe gente que no puede o quiere trabajar en sistemas operativos, véase [windows](https://www.microsoft.com/es-es/windows)   >:P, y esto nos crea la necesidad de poder definir variables de entorno de manera independiente a la plataforma que estemos usando. Por suerte para nosotros llega [cross-env](https://github.com/kentcdodds/cross-env), con esta micro librería podamos definir variables de entorno independientemente del SO en el que estemos ejecutando nuestra app.

### 4. ["shelljs"](https://github.com/shelljs/shelljs). Comandos Unix para nodejs en cualquier SO (incluido windows)

Si nuestro proyecto tiene una gran carga de tareas repetitivas, querremos automatizarlas, para ello podemos elegir un gestor de tareas tipo gulp o venirnos arriba y hacer scripting. El problema del scripting es el de siempre, la portabilidad entre diferentes SO, con [shelljs](https://github.com/shelljs/shelljs) podemos hacer uso de un gran abanico de comandos _unix_ desde _node_ y con la garantía de que funcionarán independientemente del SO en el que se ejecuten.

### 5. ["minimist"](https://github.com/substack/minimist). Parámetros amigables.

A la hora de pasar variables por línea de comando a nuestra app en _node_, nos vemos con el problema de entendernos con el objeto `process.argv`, con [minimist](https://github.com/substack/minimist) tendremos una abstracción simple para hacer de `process.argv` usable.

### 6. ["npm-run-all"](https://github.com/mysticatea/npm-run-all). Ejecutar scripts de npm en paralelo o de forma secuencial.

En algunos proyectos, reducir el tiempo que tardan ciertas tareas de scripting (_build_, _test_, _lint_, _etc ..._) puede ser algo fundamental, una manera de optimizar nuestros scripts es paralelizar tareas, con [npm-run-all](https://github.com/mysticatea/npm-run-all) tendrás una interfaz para el terminal con la que poder ejecutar scripts en paralelo o de manera secuencial.
Si quieres una alternativa [concurrently](https://github.com/kimmobrunfeldt/concurrently) puede ser una opción.

### 7. ["precommit"](https://github.com/observing/pre-commit). Instalar un script en el hook 'pre-commit' de git.

Instalando el modulo [precommit](https://github.com/observing/pre-commit) en nuestro proyecto, podemos definir por configuración la ejecución de un _script_ en el susodicho _hook_ de _git_. De esta manera podemos impedir, por ejemplo, hacer un _commit_ si no cumplimos con las reglas de nuestro _lint_ o si no pasamos los _test_ de manera exitosa.

### 8. ["require-all"](https://github.com/felixge/node-require-all). Hacer un require() de todos los archivos de un directorio.

Si queremos ejecutar todos los archivos de un directorio de nuestro proyecto, filtrados por un patrón, [require-all](https://github.com/felixge/node-require-all) es nuestro módulo. Admite directorios recursivos y expresiones regulares en el filtrado.


Mucha más información en:

* [awesome-nodejs](https://github.com/sindresorhus/awesome-nodejs)
