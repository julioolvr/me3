+++
title="Lazy JS method evaluation"
date=2013-02-17
+++

El otro día, mirando contra mi voluntad el código de
[CKEditor](https://github.com/ckeditor/ckeditor-dev), me encontré con un patrón
para evaluación lazy de los métodos de un objeto JS bastante canchero (y
probablemente conocido).

Por ejemplo, digamos que un método de un objeto tiene una parte excesivamente
costosa, representada convenientemente por una función llamada
`doSomethingExpensive`.

```js
function doSomethingExpensive() {
  console.log("expensive!");
  return 42;
}

function doSomethingCheap(answer) {
  return answer + 1;
}

function objeto() {}

objeto.prototype.metodo = function () {
  var expensive = doSomethingExpensive();
  doSomethingCheap(expensive);
};

var obj = new objeto();
obj.metodo(); // logs 'expensive!'
obj.metodo(); // logs 'expensive!' again
```

Bien, con cada llamada a `metodo` se ejecuta `doSomethingExpensive`. Si el
resultado de esa función varía con cada llamada al método, no hay mucho que
hacer. Pero si el resultado puede cachearse, o si es necesario porque el
resultado necesita ser compartido entre sucesivas llamadas al método, entonces
una primera forma de cambiarlo sería procesarlo cuando se declara el método:

```js
objeto.prototype.metodo = (function () {
  var expensive = doSomethingExpensive();

  return function () {
    doSomethingCheap(expensive);
  };
})();

var obj = new objeto(); // logs 'expensive!'
obj.metodo(); // no loguea nada
obj.metodo(); // no loguea nada
```

La
[IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)
se ejecuta en el momento de declarar el método, evalúa `doSomethingExpensive`,
almacena el resultado y devuelve una función que usa ese valor almacenado. Esto
es un avance, pero presenta la desventaja de que ejecuta `doSomethingExpensive`
incluso si `metodo` nunca se llama. Dependiendo del caso, esto puede ser
importante, ya sea que se vayan a inicializar muchos objetos como para retrasar
el tiempo de inicio de la aplicación, o porque el resultado de
`doSomethingExpensive` solamente es significativo en el momento en que se
ejecuta `metodo`, no en el momento en el que se declara. En ese caso, la
alternativa, que es la que vi en el código de CKEditor (en particular
[acá](https://github.com/ckeditor/ckeditor-dev/blob/master/core/dom/document.js#L237)
al momento de escribir esto, el método `getWindow` de `document`), es que el
método haga la evaluación, y luego se reemplace a sí mismo por una copia que use
el valor ya calculado:

```js
objeto.prototype.metodo = function () {
  var expensive = doSomethingExpensive();
  this.metodo = function () {
    return doSomethingCheap(expensive);
  };
  return this.metodo();
};

var obj = new objeto(); // no loguea nada
obj.metodo(); // logs 'expensive!'
obj.metodo(); // no loguea nada
```

Esto combina lo mejor de los dos mundos, no ejecuta `doSomethingExpensive` si
`metodo` no se ejecuta nunca, y por otro lado si lo hace, lo hace una sola vez y
comparte el resultado entre las sucesivas llamadas a `metodo`. Aprovechando que
el valor de retorno de una asignación es el valor asignado, se puede hacer una
versión más corta:

```js
objeto.prototype.metodo = function () {
  var expensive = doSomethingExpensive();
  return (this.metodo = function () {
    return doSomethingCheap(expensive);
  })();
};
```
