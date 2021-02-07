# Curso "ES6 for everyone"

## 1.- Var vs let vs const
Hasta ES6 solo teníamos una forma de declarar variables, usando la palabra reservada `var`. Con las nuevas especificaciones del lenguaje, tenemos podemos usar además la sintaxis `let` y `const`, que tienen algunas diferencias. 

En primer lugar, la palabra `var` permitía crear variables globales, 
excepto cuando eran inicializadas dentro de una función, en cuyo caso quedaban vinculadas al ámbito concreto de esta función. Dicho de otro modo, era posible hacer esto: 
```javascript
function setWidth() {
  var width: = 100;
  console.log(width); // 100
}

setWidth() // 100
console.log(width) // Error, no está definido
``` 
Vemos que la variable `width` da error al ser invocada fuera de su ámbito. Este es un comportamiento normal y esperable, pero no lo es tanto si hacemos esto: 
```javascript
var age = 100;
if (age > 10) {
  var dogYears = age * 7;
  console.log(dogYears); // 70
}
console.log(dogYears); // 70
```
Al contrario que con las funciones, aquí si podemos acceder a una variable aunque fue inicializada y declarada dentro de un bloque, como pueden ser un `if` o un `for`. Esto no se podría hacer con `let` o `const`, así que la primera diferencia que encontramos es que **`var` y `const` tiene un ámbito de bloque** o *block scope* mientras que `var` tiene **ámbito de función** o *function scope*. 

Otra diferencia es a la hora de declarar. Con `var`, podía perfectamente declarar dos veces la misma variable, pero no puedo hacer eso con `let` y `const`. Ejemplo al canto: 
```javascript
  var points = 50;
  var points = 60; // OK

  let points = 50;
  let points = 60; // Error. "Points" ya ha sido declarado.
```
Además, al ser *block scope*, si actualizo la variable puedo crear otra variable dentro de un bloque (en este caso con el mismo nombre, para que sirva de ejemplo)
```javascript
let points = 50;
let winner = false;
if (points > 40) {
  let winner = true; // Inicializando una nueva variable, aunque con el mismo nombre que la anterior, lo cual puede inducir a confusión. Si utilizase var, se sobreescribiría
}
```
Por lo demás, la diferencia más obvia y que su propio nombre explica es que las variables declaradas con `let` pueden ser actualizadas mientras que las variables inicializadas con `const` no pueden ser reasignadas. Sin embargo, si podemos modificar valores de un objeto aunque éste esté declarado en forma de constante:
```javascript
  const person = {name: "Abel", age: 30}
  const person = {name: "Abel", age: 31} // Error. No se puede modificar una constante

  person.age = 31 // OK
```
Como pequeño truco final, para congelar un objeto e impedir que se actualicen sus propiedades podemos usar `Object.freeze(myObject)`, que no es una característica propia de ES6, sino que proviene de versiones anteriores. De esa forma, ya no podremos actualizar las propiedades.

## 2.- Arrow functions y mejoras en funciones
Las funciones flecha tienen varias ventajas. Además de una sintaxis más concisa y un `return` implícito, tampoco sobreescriben la palabra reservada `this` cuando son llamadas dentro de otra función. Algunas otras de sus características son: 
- Paso de un parámetro sin paréntesis. Siempre y cuando sea nuestra función flecha reciba un solo argumento, podemos prescindir de los paréntesis, que vuelven a ser necesarios si son dos los argumentos o si no hay ninguno. 
- Las funciones flecha son anónimas. Esto puede suponer un inconveniente a la hora de detectar errores, así que siempre podemos guardar la función dentro de una variable. 

Un comportamiento habitual al manejar el DOM y seleccionar un elemento con `QuerySelector()` o alguna función similar es utilizar la palabra `this` para referirse al propio elemento. Si usamos una función flecha, esto no será posible. Veamos un ejemplo: 
```javascript
const box = document.querySelector(".box");
box.addEventListener('click', function() {
  console.log(this); 
});
```
Este ejemplo mostrará por consola el elemento `box` que hemos seleccionado de nuestro HTML a través de una `class` de CSS. Sin embargo, si hacemos esto otro...
```javascript
const box = document.querySelector(".box");
box.addEventListener('click', () => {
  console.log(this); 
});
```
... por consola cada vez que clickemos en el elemento nos imprimirá el objeto global `window` de JavaScript. Esto ocurre porque las funciones flecha no reescriben el valor de `this`, sino que simplemente lo heredan del *scope* anterior, ya sea el ámbito global o una función.

Por lo anterior, tenemos que pensar cuidadosamente cuando vamos a usar una función flecha y cuando no, en función de si necesitamos acceder a `this`. Una solución usada bastante hasta ahora es guardar el contenido de `this` como una variable y luego ya podemos reutilizarla con cualquier función que herede el contexto. Así se haría: 
```javascript
const box = document.querySelector(".box");
box.addEventListener('click', function() {
  var that = this;
  that.classList.toggle('new-class');
  /* 
  Y a partir de aquí ya hacemos referencia
  a esta variable local que se ha guardado el contenido de this
  para cualquier transformación que necesitemos
  */
});
```
Sin embargo, podemos ahorrarnos todo este paso (y el código correspondiente) si hacemos un uso adecuado de las funciones flecha en momentos en los que la necesitemos. 

### Argumentos por defecto en funciones
Los parámetros o argumentos por defecto en una función nos permiten especificar un valor por defecto en la definición de la función que solo se tomará en cuenta en tiempo de ejecución si la función no recibe esos parámetros cuando es llamada. Ahí va un ejemplo: 
```javascript
function calculateBill(total, tax, tip) {
  return total + (total * tax) + (total * tip); 
}

const totalBill = calculateBill(100, 0.15, 0.20); // 135
```
Pero si llamo a la función indicando solo un parámetro obtendré un error: 
```javascript
const totalBill = calculateBill(100); // Devuelve NaN, porque no puede hacer las cuentas al no tener los valores de los dos parámetros que no hemos especificado. 
```
Una posible solución para esto sería darle unos valores a los parámetros dentro del cuerpo de la función, por ejemplo así: 
```javascript
function calculateBill(total, tax, tip) {
  if (tax === undefined) {
    tax = 0.15;
  }
  if (tip === undefined) {
    tip = 0.20;
  }
  return total + (total * tax) + (total * tip); 
}
```
Pero esta operación requiere añadir dos condicionales a nuestra función, cuando gracias a los parámetros por defecto podemos solventar el problema de forma más sencilla: 
```javascript
function calculateBill(total, tax = 0.15, tip = 0.20) {
  return ... 
}

// Y al llamar a la función: 
const TotalBill = calculateBill(100) // 135

// Incluso podemos pasar undefined como uno de los parámetros si solo concemos dos pero necesitamos respetar el orden en el que se pasan

const TotalBill = calculateBill(100, undefined, 0.30); // 145
``` 
### Cuándo no usar una función flecha: 
Hay varios casos en los que nos puede interesar no usar funciones flecha: 
- Cuando queremos mantener el ámbito de `this`, sin generar un nuevo *scope*. Como el ejemplo que vimos anteriormente.
- Cuando añadimos un método a un objeto nos interesará usar una función de toda la vida, porque `this` hará referencia al propio objeto.
- Cuando queremos añadir funcionalidad a objetos ya instanciados a través de `prototype`. El motivo es también por el `this`, por si hacemos referencia al propio objeto en el nuevo método. 
- Cuando queramos usar el objeto `arguments`. Es una posibilidad nativa de JS que nos permite acceder a los argumentos que recibe una función accediendo a su índice y con la palabra reservada `arguments`, pero que no es compatible con una función flecha.

## 3 y 4.- Template Strings y cadenas de texto
Hasta la llegada de ES6, en JavaScript no había una forma sencilla de intercalar variables dentro de una cadena de texto. La formá más habitual era esta:
```javascript
var name = "Abel";
var age = 30;
var city = "Madrid";

var presentacion = "Mi nombre es " + name + ", tengo " + age + " años y vivo en " + city";
```
Intercalar texto entre comillas y variables era una fuente potencial de errores, y si tenías un texto muy grande a veces era complicado saber donde habías cometido el error. Para eso se inventaron las plantillas literales o *template strings*: 
```javascript
const name = "Abel";
let age = 30;
let city = "Madrid";

const presentacion = `Mi nombre es ${name}, tengo ${age} años y vivo en ${city}`;
```
La sintaxis es más sencilla y más fácil de leer, además el resaltado 
de la sintaxis ayuda a evitar errores. Incluso dentro de la sintaxis `${}` podemos incluir expresiones JS para que sean evaluadas en tiempo de ejecución: 
```javascript
let age = 30;
const myAgeNextYear = `Next year I'll be ${age + 1}`; // 31
```
Un uso común es introducir directamente código HTML que queremos luego incorporar a nuestra página. Esto nos lo facilita la posibilidad de usar indentación y de incorporar fácilmente saltos de línea: 
```javascript 
const song = {
  name: 'Dying to live',
  artist: 'Tupac',
  featuring: 'Biggie Smalls'
}

const markup = `
  <div class="song">
    <p>
      ${song.name} - ${song.artist}
      ${song.featuring ? `Featuring ${song.featuring}` : '';
    </p>
  </div>
`
```
También en este ejemplo anterior vemos como es posible anidar unos `template strings` dentro de otros sin mayor problema. 

### Tagged template literals
Las *tagged templates* son una forma de evaluar nuestras *template literals* de forma que podamos utilizar una función para hacer cambios en una plantilla literal. Por ejemplo, podemos resaltar algunas partes, etc. Partiendo de este código... 
```javascript
function highlight() {...}

const name = 'Snickers';
const age = 100;
const sentence = `My dog's name is ${name} and he is ${age} years old`;
console.log(sentence);
```
Vamos a dotar de contenido a la función `highlight`:
```javascript
function highlight (strings, ...values)
```
Hay que señalar dos cosas aquí: la primera es que al ejecutar esta función ante un *tagged template literal*, el comportamiento que vamos a ver es el siguiente: la cadena de texto será analizada y el texto plano irá al primer parámetro, mientras que las variables irán al segundo parámetro, en ambos casos en forma de arrays.

Por si fuera poco, en la definición de esta función usamos los llamados **parámetros Rest**, que permiten pasar a una función un número de argumentos cualquiera sin necesidad de saber el número. La sintáxis es fácilmente identificable porque incluye tres puntitos antes `...values`. 

Como al ejecutarse la función serán arrays, podremos acceder a los parámetros y aplicarles métodos propios de arrays:
```javascript
function highlight (strings, ...values) {
  let str = '';
  strings.forEach((string, i) => {
    str += string + (values[i] || '');
  }); 
  return str;
}
```
En este código hay una cosa que nos puede sorprender: cuando aplicamos un `forEach()` sobre el *array* `strings` de nuestra función, coge la longitud de `strings`. Y esa longitud es una unidad mayor que la del segundo *array*, siempre. Como consecuencia, al iterar cogiendo la longitud del primer array, el segundo array tendrá una posición `undefined` porque accedemos a un índice que no existe. Esto no tiene que ocurrir siempre, y en este caso concreto lo solucionamos así: 
```javascript
strings.forEach((string, i) => {
  str += string + (values[i] || ''); // Si no existe el valor, pues string vacío
}); 
``` 
Pero esto es solo una excepción a un problema que podemos no tener siempre. Sigamos con el ejercicio y para ello vamos a destacar el texto de las variables, por ejemplo: 
```javascript
function highlight (strings, ...values) {
  let str = '';
  strings.forEach((string, i) => {
    str += `${string} <span class="highlight">${values[i] || ""}</span>
  }); 
  return str;
}
```
Y esto nos devolverá el texto de las variables subrayado. 

### Nuevas métodos para trabajar con cadenas de texto
ES6 incluye algunos métodos para *strings* que permiten simplificar el código y hacerlo más legible, evitando el uso incluso de expresiones regulares. Estos cuatro métodos son: 
- `startsWith()`: permite evaluar si la cadena de texto empieza con los caracteres que recibe la función por parámetro, y devuelve un `true` o `false` según el caso. Es sensible a mayúsculas y minúsculas, así que cuidado con esto. Si además le incorporamos un segundo parámetro, por ejemplo `string.startsWith("hey", 3)` se saltará el número de caracteres en la comprobación (en este caso, 3). 
- `endsWith()`: el funcionamiento es similar, evalúa si la cadena de texto termina con el texto que le indicamos en el parámetro. Si añadimos un segundo parámetro, limita el número total al valor que le pasemos. 
- `includes()`: busca el texto que reciba en el parámetro dentro de la cadena, en cualquier lugar. Pero, como el resto, tiene en cuenta las mayúsculas y las minúsculas, así que cuidado. 
- `repeat()`: permite repetir tantas veces como especifiquemos en el parámetro el string al que se aplica. 

## 5.- Destructuring
### Objetos
La "desestructuración" permite extraer valores de ciertos tipos de datos en JavaScript, como son objetos, *arrays*, *maps* y *sets* (estos dos últimos los vemos más adelante). Dado este código: 
```javascript
const person = {
  first: "Abel",
  last: "España", 
  country: "Spain",
  city: "Madrid",
  company: "Minsait"
}
```
Lo que normalmente haríamos para extraer estos valores en variables independientes:
```javascript
const first = person.first;
const last = person.last;
const country = person.country; 
// etc.
```
Con ES6 y el *destructuring*, en cambio, podemos hacer esto:
```javascript
const { first, last } = person;
```
Esto extraerá los valores del objeto con el nombre correspondiente y los almacenará en esas variables. Si resulta que apuntamos a una propiedad que no existe, generaremos un error como es de esperar.

Así a priori puede que no le veamos demasiada utilidad, pero se incrementa cuando tenemos que acceder a objetos más complejos, con más niveles por debajo.
```javascript 
const abel = {
  first: "Abel",
  last: "España",
  links: {
    social: {
      twitter: "www.twitter.com/abelespana",
      facebook: "www.facebook.com/fakeuser",
    }, 
    web: {
      personal: "www.abelespana.es",
      github: "www.github.com/abelespana",
    }
  }
}

// Puedo hacer esto, a la manera tradicional: 
const twitter = abel.links.social.twitter // e igual con el resto

// O puedo usar destructuring
const { twitter, facebook } = abel.links.social;
```
Puesdo incluso nombrar estas propiedades como quiera, añadiéndolo con dos puntos tras el nombre del objeto (a veces puede ser un nombre que ya esté usando en forma de variable o que simplemente no me guste): 
```javascript
const { twitter:tuiter, facebook:feisbuk } = abel.links.social;

console.log(tuiter);
console.log(feisbuk);
```
También se pueden poner valores por defecto al hacer *destructuring*, que funcionan igual que los parámetros por defecto en las funciones. El valor por defecto solo se aplica si no existe en el objeto que queremos desestructurar, de forma que nos ahorramos un posible `undefined`:
```javascript
const settings = {...properties here...}
const { width = 100, height = 100} = settings;
// Si el objeto al que apuntamos no tiene propiedades "width" y "height", entonces prevalecerá el valor por defecto. Si las tienen, ocurrirá lo contrario. 
```
### Arrays
Como en el caso anterior, a menudo accedemos a las propiedades de un array indicando su índice numérico, empezando por 0. Como en este ejemplo: 
```javascript
const details = ["Abel", 30, "Madrid"];
const name = details[0];
const age = details[1]; 
// etc. 
```
Y ahora...
```javascript
const details = ["Abel", 30, "Madrid"];
const [ name, age, city ] = details;
```
La única diferencia es que al ser un *array* y no tener las propiedades un identificador como el nombre, accede a los elementos en orden: la primera variable será el primer valor del array, la segunda el segundo, etc. 

Incluso si recibimos un *string* con un separador claro podemos desestructurarlo de esta forma: 
```javascript
const data = 'Basketball,Sports,90210,23';
const [ itemName, category, sku, inventory ] = data.split(','); 
```
La función `split()` no tiene demasiado que ver con *destructuring*, pero es útil porque divide el string buscando el caracter que recibe por parámetro y devuelve un array de todos los strings en los que trocee el string original. Y encima, después de trocear el *string*, descartará los valores que no hayan sido almacenados en variables.

Nos podemos encontrar con casos en los que queramos rescatar uno o dos valores del *array* y luego el resto, pero no sabemos cuántos son. Para esas situaciones, tenemos el operador *rest* que ya vimos antes en la parte de plantillas literales. Así funciona la cosa: 
```javascript
const team = ["Wes", "Harry", "Sarah", "Keegan", "Riker"];
const [ captain, assistant, ...players] = team;
```
Aquí la variable `captain` equivale al primer valor del array, `assistant` al segundo y `...players` es a su vez un *array* que contiene el resto, sean cuantos sean.  
### Intercambio de variables 
Veamos un ejemplo práctico en el cual nos puede ser útil el *destructuring* de arrays: tenemos dos valores, `player1` y `player2`, y queremos intercambiarlos por el motivo que sea: 
```javascript
let player1 = "Messi";
let player2 = "Cristiano Ronaldo";

console.log(player1) // Messi
console.log(player2) // Cristiano Ronaldo

// Para intercambiar estos valores de forma sencilla:
[player1, player2] = [player2, player1];

console.log(player1) // Cristiano Ronaldo
console.log(player2) // Messi
```
### Destructuring en funciones
Todos estos mecanismos que hemos visto para trocear *arrays* u objetos y pasarlos como parámetros a funciones, también es posible aplicarlos a funciones. Aquí va un ejemplo sencillo: 
```javascript
function convertCurrency(amount) {
  const converted = {
    USD: amount * 0.76,
    GPB: amount * 0.53,
    AUD: amount * 1.01,
    MEX: amount * 13.3
  };
  return converted
}

const currency = convertCurrency(100);
console.log(currency.USD);
console.log(currency.GPB); 
console.log("etc.");

// Con una simple variable, conseguiremos el efecto de devolver múltiples returns de una función, aunque lo que en realidad devolvemos es un objeto que luego desestructuramos: 
const { USD, GPB, AUD, MEX } = convertCurrency(100)

// Y como es un objeto y va a acceder por propiedades, ni siquiera tenemos que preocuparnos por el orden. Estos es perfectamente válido: 
const { AUD, MEX, USD, GPB } = convertCurrency(100);
```
¿Otro ejemplo? Ahora pasamos los parámetros a una función en forma de objeto, usando también los parámetros por defecto que vimos anteriormente, y de esa forma vemos como lo desestructura:
```javascript
// Antes
function tipCalculator(total, tip = 0.15, tax = 0.13) {...}

// Ahora
function tipCalculator({ total, tip = 0.15, tax = 0.13 }) {
  return total + (total * tip) + (tax * total);
}

// Al llamar a la función
const bill = tipCalculator({ total: 200, tip: 0.20, tax: 0.13 })
console.log(bill);
```
Lo que conseguimos así es desestructurar los parámetros, de forma que podemos pasarlos en otro orden o incluso omitir alguno, porque la función va a buscar los parámetros por el mismo nombre:
```javascript
const bill = tipCalculator({ total: 200 });
// E incluso cambiando el orden. 
const bill = tipCalculator({ tax: 0.38, total: 200 });
```
Pero cuidado: esto me obliga a pasar un parámetro siempre. Si llamo a la función sin parámetro alguno
```javascript
const bill = tipCalculator(); // Error

// Solución: reformular la función así: 
function tipCalculator({ total, tip = 0.15, tax = 0.13 } || {} ) {...}
```
Es decir, los parámetros que se pasan, o si no un objeto vacío (y ya coge los parámetros por defecto)

## 6.- Iterables y bucles
### El bucle *for of*
De sobra conocemos el bucle `for` de toda la vida, y también el bucle `forEach`, que permite lo mismo aunque no podemos establecer la condición de parada, ni saltar un valor por ejemplo. 
```javascript
for (let i = 0; i > array.length; i++) {...} 

array.forEach(function(element) {
  ...
});

// O la variante de ES6, con sintaxis más sencilla. 
array.forEach(element => {...});
```
También tenemos el bucle `for... in`, que devuelve un índice y no un valor: 
```javascript
const values = [...];
for (const index in values) {
  console.log(values[index])
}
```
Sin embargo, este bucle tiene una gran desventaja, y es que itera sobre todo el *array*. Incluso si añadimos métodos o propiedades al array mediante `array.prototype` los veremos también aquí, así que no es demasiado buen negocio. 

El bucle `for... of` es una nueva especificación de ES6, y su funcionamiento es similar al `for... in` que acabamos de ver, aunque en este caso solo muestra el contenido del array: 
```javascript
const numbers = ["one", "two", "three", "four"];
for (const number of numbers) {
  console.log(number)
}
```
Y además permite dos cosas interesantes, que es establecer fácil una parada en el bucle o incluso saltar un valor: 
```javascript
const numbers = ["one", "two", "three", "four"];

// Para parar el bucle en un punto concreto
for (const number of numbers) {
  console.log(number); // one, two
  if (number === "three") {
    break; 
  }
}

// Para saltarnos un valor en concreto del array
for (const number of numbers) {
  if (number === "three") {
    continue; 
  }
  console.log(number); // one, two, four
}
```
E incluso, usando la propiedad `array.entries()` podemos iterar sobre el array obteniendo el índice a la vez (lo podríamos necesitar para insertar una tabla en el DOM, por ejemplo): 
```javascript
const names = ["Abel", "Cristina", "David", "Sergio"];

for (const name of names.entries()) {
  console.log(name); // [0, "Abel"], [1, "Cristina"], etc. 
}
```
Al usar `array.entries()` nos devuelve un array con un número de índice y con el valor. Podemos usar el *destructuring* que ya vimos para tener un código más legible: 
```javascript
const names = ["Abel", "Cristina", "David", "Sergio"];

for (const [index, name] of names.entries()) {
  console.log(`${name} is the number ${index}`) // Abel is the number 0, Cristina is the number 1, etc...
}
```
Se puede aplicar sobre ciertos objetos (veremos más sobre ello luego) y sobre *strings*: 
```javascript
// Strings
const name = "Abel España";
for (const character of name) {
  console.log(character); // A, b, e, l, E...
}
```
También se puede usar con el pseudo array (que en realidad es un objeto) `arguments`y con los `NodeList` que nos devuelve la función `querySelector()`, por ejemplo. Es decir, que se puede usar con *maps*, *sets*, *nodeList*, los propios *arrays*, *strings*... Muy potente. 

En cambio, si queremos hacer un `for... of` sobre un objeto...
```javascript
const apple = {
  color: "red", 
  size: "medium",
  weight: 50,
  sugar: 10
}

for (const property of apple) {
  console.log(property) // Error. 
}
```
En su lugar, podemos usar las funciones nativas del estandar ES2017 junto con nuestro `for... of`. 
```javascript
// Object.keys devuelve un array con las keys del objeto. 
for (const property of Object.keys(apple)) {
  const value = apple[prop];
  console log(`${value}: ${property}); "Color: red", "size: medium", etc. 
}
```

## 7.- Mejoras en arrays
### `Array.from( )`
Estos métodos no están en el `prototype` de un array, pero son métodos que podemos aplicar para convertir cosas en arrays. Un ejemplo muy claro es cuando seleccionamos muchos elementos del DOM y queremos iterar sobre ellos (para extraer o insertar valores, por ejemplo): 
```html
<!-- index.html -->
<div class="personas">
  <p> Luis </p>
  <p> Julio </p>
  <p> Manuel </p>
</div>
```
```javascript
// index.js
const personas = document.querySelectorAll(".personas p");
console.log(personas); // 3 elementos <p>, pero no es un array
```
Creemos que lo que nos devuelve `querySelectorAll()` es un array porque se parece a un array y se escribe como un array, pero no es un array. Si descubrimos sus propiedades, vemos que su prototipo es un `NodeList`. Tiene algunos métodos como `forEach()`, tiene algunas propiedades como `length`... pero no es un array. Para convertirlo en un array: 
```javascript
const arrayPersonas = Array.from(personas); // Prototype: array 
```
Incluso puedo, a este método, pasarle un segundo parámetro que sea una función:
```javascript
const arrayPersonas = Array.from(personas, function(persona) {
  console.log(persona);
}
```
### `Array.of()`
Sirve para construir un array partiendo de los argumentos que le pasemos, funciona un poco al revés que `array.from()` pero hace lo mismo:
```javascript
const ages = Array.of(16, 22, 34, 59, 81). 
console.log(ages) // [16, 22, 34...]
```
### `Array.find()`
Este método sirve para buscar dentro de un array, incluso si el array está compuesto de objetos, que es como normalmente suelen venir los datos provenientes de una `API`. Por ejemplo, imaginemos que tenemos un array de códigos y lo buscamos: 
```javascript
const posts = [{...}, {...}, {...}, {...}]; // Datos de una API, mismamente
const code = "VBgtGQcSf";

const myPost = posts.find(function(post){
  post.code === code // true or false
});
```
La función `findIndex()` devuelve el índice de un array. Tan fácil: 
```javascript
const posts = [{...}, {...}, {...}, {...}];
const code = "VBgtGQcSf";

const postIndex = posts.findIndex(post => post.code === code) 
console.log(postIndex); // 22 (o el número de índice que sea)
```
### `Array.some()` y `Array.every()`
Estos dos métodos no son del estándar ES6, pero molan, así que ahí van un par de ejemplos que prácticamente no necesitan explicación: 
```javascript
// ¿Hay al menos algún adulto en el grupo?
const ages = [15, 16, 32, 29];
const isAdultPresent = ages.some(age => age >= 18) // true

// ¿Tiene todo el mundo edad para beber?
const canEveryoneDrink = ages.every(age => age > 18) // false
```

## 8.- Spread operator (operador de propagación)
El *spread operator* u **operador de propagación** es aplicable a cualquier iterable, entre ellos, aunque normalmente lo usaremos con *arrays*. Por ejemplo, para unir un array con otro podíamos hacerlo así:
```javascript
const featuredPizzas = ["Barbacoa", "Pepperoni", "Hawaiana"];
const specialPizzas = ["Seis Quesos", "Mexicana", "Carnívora"];

// Para unir los dos arrays, una solución podría ser:
let allThePizzas = featuredPizzas.concat(specialPizzas); // 6 pizzas

// Si quería añadir una nueva pizza entre medias, el proceso se complicaba un poco: 
allThePizzas = featuredPizzas.push("Vegana"); // 4 pizzas
allThePizzas = featuredPizzas.concat(specialPizzas); // 7 pizzas
```
Con el *spread operator*, es mucho más sencillo. Lo podemos reconocer por los tres puntitos (`...`) que preceden a una variable o array. Y el ejemplo anterior lo haríamos así: 
```javascript
const featuredPizzas = ["Barbacoa", "Pepperoni", "Hawaiana"];
const specialPizzas = ["Seis Quesos", "Mexicana", "Carnívora"];

// Para hacer una unión de dos arrays en otro
let allThePizzas = [...featuredPizzas, ...specialPizzas];

// Para hacer una unión  de dos arrays añadiendo un valor:
allThePizzas = [...featuredPizzas, "vegana", ...specialPizzas];
```
El operador de propagación también nos ayuda con otras operaciones que hacemos con arrays. Por ejemplo, normalmente para copiar un array haríamos esto: 
```javascript
const superheroes = ["Batman", "SuperMan", "Spiderman", "Hulk"];
const newSuperheroes = superheroes; 
```
Simplemente apunto el contenido del nuevo array al contenido del otro array. El problema es que los arrays no son un tipo primitivo, y su contenido se pasa por referencia y no por valor. Así que si luego modifico un superhéroe en el nuevo array...
```javascript
newSuperheroes[0] = "Ant-Man";
console.log(superheroes) // ["Ant-Man", "SuperMan"...]
```
... de un plumazo nos hemos cargado el array anterior, al modificar un valor del nuevo. Con nuestro nuevo amigo el *spread operator*, la cosa se haría así: 
```javascript
const superheroes = ["Batman", "SuperMan", "Spiderman", "Hulk"];
const newSuperheroes = [...superheroes]; 
```
Y ya puedo hacer todos los cambios que quiera en el array copiado, que el original no se verá afectado. 

### Operador de propagación en funciones
También podemos usar el *spread operator* para pasar argumentos a funciones
```javascript
const name = ["Abel", "España"]

function sayHi(nombre, apellido) {
  alert(`Hola, soy ${nombre} ${apellido}`)
}

// Y al llamar a la función
sayHi(...name) // Hola, soy Abel España
```

## 9.- Mejoras en objetos literales
A la hora de trabajar con objetos, a veces necesitamos construirlo desde 0 con varias variables, como en este caso: 
```javascript
const name = "Abel";
const surname = "España";
const age = 30;
const city = "Madrid";

// A la antigua usanza
var person = {
  name: name,
  surname: surname,
  age: age,
  job: "Front-end developer"
}

// Desde ES6
const person = {
  name,
  surname,
  age, 
  city, 
  job: "Front-End developer"
}
```
E incluso sirve para crear métodos dentro de objetos:
```javascript
// Antes
const modal = {
  create: function() {...},
  open: function() {...},
  close: function() {...}
}

// Ahora
const modal = {
  create() {...},
  open(content) {...}, // Paso de parámetros incluido
  close() {...}
}
```
Conviene recordar que al declarar métodos de clases o funciones dentro de objetos, quizás las *arrow functions* no sean nuestra forma de sintaxis más aliada, porque podemos tener problemas con el valor de `this`.

Un tercer añadido son las *computed properties*, que es la posibilidad de añadir lógica para calcular propiedades de un objeto, como en este par de ejemplos sencillos: 
```javascript
const key = 'pocketColor';
const value = "#ffc600";

// Añadiendo esas propiedades al objeto de forma dinámica, de modo que si cambio el contenido de la variable cambiaré también el objeto. Esto no es nuevo de ES6. 
const tShirt = {
  [key]: value
};

// Y ahora las computed properties en acción

function invertirColor(color) {...};
const key = 'pocketColor';
const value = "#ffc600";

const newTshirt = {
  [key]: value,
  [`$k{key}Opposite`]: invertirColor(value)
}
```
Con este código y la ayuda de una pequeña función que nos permite invertir colores (cuyo contenido no nos importa), lo que conseguimos es generar una nueva variable a la que llamamos de forma opuesta al valor que tenemos (el nombre es lo de menos) y calculamos el valor de forma dinámica, con esa función `invertirColor`. 

Otro ejemplo: 
```javascript
const key = "Name";
const value = "Abel"
  
function calculateAge(birthYear, currentYear) { 
  return currentYear - birthYear;
}

const person = {
  [key]: value,
  age: calculateAge(1988, 2019) // Propiedad computada, calculada de forma dinámica
}
```

## 10.- Promesas
Las *promesas* son una forma gestionar la asincronía en JavaScript, lo que hace que sean comunmente usadas para llamadas a APIs y AJAX en general, porque es el claro ejemplo donde tenemos que controlar que recibimos la respuesta antes de continuar con el flujo de nuestro programa, que seguramente incluya hacer algo con los datos que provienen de esa respuesta.

Vamos a ver primero la forma tradicional de crear promesas, y después alguna librería que las integra:
```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Ya está hecho");
  }, 2000);
});
```
Establecemos que nuestra promesa tendrá dentro un temporizador (esto para falsear el tiempo que tardaría una llamada a una API o similar) y una vez que está, llamamos al método `resolve()` de nuestra promesa instanciada. Esto devuelve la promesa y ya podemos encadenarle un nuevo proceso, de esta forma: 
```javascript
promise.then(data => console.log(data)); // "Ya está hecho", tras dos segundos

// O un error si algo fue mal
const promise = new Promise((resolve, reject) => {
  setTimeout(() {
    reject("Se fue todo al carajo");
  }, 2000); 
});

promise
  .then(data => console.log(data)); // O una función, o lo que queramos hacer aquí. 
  .catch(error => console.error(`Error: ${error}`)); // "Error: se fue todo al carajo" tras dos segundos
```
### Promesas encadenadas y control de flujo
Es posible encadenar promesas y usarlas para controlar el flujo de nuestra aplicación. Las promesas también nos dan métodos que podemos usar para gestionar estos, como `promise.all()`, `promise.race()` o `promise.finally()`. En el archivo de ejercicios adjuntos vemos algunos de ellos.

## 11.- Símbolos (Symbols)
Además de `string`, `number`, `object`, `boolean`, `null` y `undefined`, ES6 añade un nuevo **tipo primitivo**: `symbol`. Los símbolos tienen una utilidad interesante cuando queremos nombrar propiedades o elementos, o para guardar datos privados por la particularidad que tienen al acceder a ellos. Por ejemplo, en un objeto, podría ser que tuviéramos:
```javascript
const classRoom = {
  'Mark': {grade: 50, gender: 'male'},
  'Olivia': {grade: 80, gender: 'female'},
  'Olivia': {grade: 80, gender: 'female'}
}
```
Al ser ejecutado, este código sobreescribirá a la primera `Olivia` con la última, y prevalecerá esta última. Como eso no es algo que queremos, podemos usar un `Symbol` para evitar estos conflictos de nombrado, porque los símbolos son únicos y siempre van a ser `false` si lo comparamos con otro `symbol`. La cosa quedaría así: 
```javascript
const classRoom = {
  [Symbol('Mark')]: {grade: 50, gender: 'male'},
  [Symbol('Olivia')]: {grade: 80, gender: 'female'},
  [Symbol('Olivia')]: {grade: 80, gender: 'female'}
}
```
Los símbolos no son iterables, no puedo aplicarles un `for... of` o un `for... in`. Pero `Object` tiene un método para ver los símbolos que contiene un objeto: 
```javascript
const mySymbols = Object.getOwnPropertySymbols(classRoom);
console.log(mySymbols) // Devuelve un array con los símbolos, pero no tienen nada dentro, no son un objeto. 
```

## 12.- Código de calidad con ESLint
Eslint es una herramienta que permite escanear nuestro código para no solo descubrir errores, sino también para aplicar buenas prácticas o convenciones que queramos usar. Por ejemplo,nos avisará si tenemos variables declaradas pero no usadas, o espacios donde no queremos, etc. 

Para usar ESLint, primero debemos instalarlo globalmente en nuestro equipo a través de NPM
```npm
  npm install -g eslint
```
Una vez hecho, podemos establecer unas opciones de configuración. Lo más habitual es hacer esto a nivel de proyecto, porque no en todos los proyectos utilizamos las mismas reglas o los mismos lenguajes o *frameworks*. Para crearlo, simplemente añadimos un fichero `.eslintrc` en la carpeta del proyecto y dentro, en formato `json`, establecemos las condiciones. Ver el ejemplo adjunto. 

### Algunas particularidades de ESLint son: 
- En nuestra configuración, podemos usar `env` para fijar los entornos de desarrollo. Por ejemplo, si vamos a usar Node, o si nuestro código se ejecutará en el escritorio, si vamos a usar ES6, o si vamos a incorporar la librería jQuery por ejemplo.
- También podemos usar el comando `extends` para importar todo un set de reglas.
- El uso de reglas nos permite incorporar condiciones. Por ejemplo, una de las reglas es `no-unused-vars`, que rastrea variables no usadas y nos invita a eliminarlas.
- Las reglas nos permiten, en su mayoría, establecer el nivel de severidad que van a aplicar, a elegir entre `off`, `warn`, `error`.
- A menudo nos pasa que queremos usar una librería externa, y que en algún momento de nuestro código llamamos a un método concreto (por ejemplo, para inicializar un carrusel), pero esas funciones a las que llamemos en nuestro código no van a estar definidas. Lo van a estar en otras librerías que integremos, bien en local o bien a través de un enlace a una CDN, y no vamos a tener errores al ejecutar el código, pero sí al analizarlo con ESlint. Para solucionarlo, **al inicio del fichero JS** podemos establecer un comentario de bloque (no de línea) y añadir esas variables que usamos pero no definimos acompañadas de la palabra clave `globals`, lo que indica que van a ser variables globales o de `window`.  
```javascript
  /* Globals fn1 fn2 */ 
  // Sin errores de linter
  fn1(); 
  fn2(); 
```
- A veces necesitamos que una regla de nuestro fichero ESlint no se aplique a un fichero concreto por algún motivo. Un ejemplo puede ser si añadimos algo al `prototype` de un fichero. Es una práctica no recomendada, pero a veces no tenemos más remedio para hacer `polyfills`, así que al inicio de ese fichero JS concreto añadimos una línea en comentario de bloque que diga lo siguiente:
```javascript
/* eslint-disable nombre-de-la-regla */
```
- Incluso es posible hacer que ignore un bloque de código concreto, si lo incluimos entre dos comentarios: 
```javascript
/* eslint-disable nombre-de-la-regla */
console.log("Este código no será testeado con la regla anterior");
/* eslint-enable nombre-de-la-regla */
```
- Y si no ponemos nada después de `eslint-disable`y `eslint-enable`, entonces quita todas las reglas de todo el fichero en todo ese bloque que introduzcamos entre comillas. 
- ESLint se puede extender mediante el uso de plugins hecho por la comunidad. Por supuesto tenemos para los principales *frameworks*, pepro también para HTML, MarkDown, y lo que se nos ocurra. Estos paquetes se instalan aparte, normalmente a través de NPM. 

## 13.- Uso de ES Modules con NPM y Webpack
ES6 ya está bastante extendido y cuenta con soporte entre todos los navegadores principales, pero no así los módulos que incorpora, de forma que es necesario recurrir a herramientas para compilar estos módulos y que el archivo resultante sea compatible. No tenemos que recordar como se instalan paquetes desde 0, ni la utilidad que tiene el fichero `package.json`, etc.

Utilizando la configuración por defecto de Webpack conseguimos compilar todos nuestros ficheros a uno solo, normalmente llamado `bundle.js`, optimizado para producción (minificado y tal). Lo habitual es copiar la configuración por defecto en cualquier proyecto inicial, aunque existen diferentes opciones y crear una propia también puede ser buena idea. De esta forma, podremos incorporar paquetes de terceros o librerías como `lodash` o `jquery`.

Incluso podemos crear nuestros própios módulos, exportar funciones y variables e importarlas en otro archivo para usarlas. De paso nos servirá para mantener estos datos privados, por decirlo de alguna forma, ya que no pertenecerán a un *scope* que no sea el suyo. Por ejemplo: 
```javascript
// Archivo de configuración config.js
const apiKey = "abc123";
export default apiKey; 

// Archivo principal app.js
import apiKey from './src/config';
console.log(apiKey);
```
La principal diferencia es que la exportación por defecto permite que demos un nombre a la variable al importarla, porque solo puede haber una exportación por defecto en un documento. La exportación donde especificamos el nombre, en cambio, puede ser múltiple y necesitamos conocer el nombre y especificarlo al importarlo
```javascript
// Archivo de configuración config.js
const apiKey = "abc123";
export default apiKey; 

// Archivo principal app.js
import cualquierCosa from './src/config';
console.log(cualquierCosa); // OK porque es export default, pongo el nombre que quiera
```
Si el `export` tiene nombre, entonces debemos importarlo entre llaves:
```javascript
// Archivo de configuración config.js
export const apiKey = "abc123";

// Archivo principal app.js
import { apiKey } from './src/config';
console.log(apiKey); // Importado entre llaves porque puede haber más de uno
```
Se puede renombrar...
```javascript
import { apiKey as passkey} from './src/config';
```
... y exportar varias cosas en una sola línea, con la misma sintáxis más o menos:
```javascript
const age = 30;
const name = "Abel";

export { name, age };
// O exportar renombrando
export { name, age as old } // Name y old
``` 

## 14.- Otras herramientas de ES6
Además de Webpack para hacer un compilado de nuestro código, ES6 y otras herramientas de JavaScript nos ofrecen otras alternativas, algunas de ellas son:
- Babel: permite transpilar ES6 y posteriores versiones a ES5, compatible con un mayor número de navegadores. Se le pueden añadir *presets* para ciertos lenguajes y navegadores y también ciertos *plugins* para ayudarnos a hacer cosas concretas, así como *polyfills*. 
- Pollyfill.io es una web que nos sirve para cubrir generar *pollyfills* dinámicamente, de modo que incrustando el `script` con el enlace a su librería en nuestro HTML, se ejecuta al cargar la página y ya inyecta el código JS necesario para que cargue solo los *polyfills* que necesita. 
- System.js. Utilidad que permitía cargar paquetes de NPM también de forma dinámica en tiempo de ejecución, sin necesidad de compilarlos previamente en un `bundle.js`, a costa de mayores tiempos de carga. No es muy útil precisamente por eso.

## 15.- Herencia con prototipos y clases
En JS ya teníamos forma de hacer herencia a través de prototipos, la posibilidad de añadir métodos y propiedades a una función constructora a través de su prototipo. Código de recordatorio: 
```javascript
function Dog(name, breed) {
  this.name = name;
  this.breed = breed;
}
// Añade un método a la clase o función constructora. Disponible para todas las instancias de esa clase. 
Dog.prototype.bark = function() {
  console.log(`Bark Bark! My name is ${this.name}`);
}

const snickers = new Dog('Snickers', 'King Charles');
const sunny = new Dog('Sunny', 'Golden Doodle');

// Sobreescribimos el método atacando de nuevo al prototipo
Dog.prototype.bark = function() {
  console.log(`Bark Bark! My name is ${this.name} and I'm a ${this.breed}`);
}
```
Cuando instanciamos una clase usando el constructor `new`, creamos una instancia de ese elemento y luego modificamos su `prototype`, lo que nos permite dotarle de propiedades que no tenía, y se aplican a todas las instancias. 
```javascript
snickers.bark() // "Bark Bark!...
sunny.bark()
```
Si mostramos estos elementos por consola y los inspeccionamos con detalle, vemos que el nuevo método `bark()` no pertenece a los objetos `snickers` o `sunny`, sino al prototipo de ambos que en este caso es un `Object`. 

### Clases
Las clases sirven para hacer lo mismo que la herencia por prototipos, pero quizás con una sintaxis un poco más accesible: 
```javascript
class Dog {
  constructor(name, breed) {
    this.name = name;
    this.breed = breed;
  }

  bark() {
    console.log(`Bark Bark! My name is ${this.name}`);
  }
  cuddle()) {
    console.log("I love you owner");
  }
}

const snickers = new Dog('Snickers', 'King Charles');
const sunny = new Dog('Sunny', 'Golden Doodle');
```
Así sería más o menos el ejemplo anterior. Los métodos están ahora expresados con otra sintaxis, sin incluir la palabra `function`, y a continuación del `constructor`, sin necesidades de poner una coma. Además, tenemos la posibilidad de hacer métodos `static`, que son aquellas que no son accesibles desde la clase instanciada sino desde la clase formulada: 
```javascript
static info() {
  console.log("A dog is better than a cat by 10 times");
}

// En la consola:
snickers.info() // Error
Dog.info() // "A dog is better than..."
```
Además, tenemos `get` y `set` para establecer propiedades, cada una con un nombre bastante descriptivo. Funcionan así: 
```javascript
get description() {
  return `${this.name} is a ${this.breed} type of dog`;
}
set nicknames(value) {
  this.nick = value
}
get nicknames() {
  return this.nick;
}
```
Esto, aunque tengan esa pinta, no son métodos sino propiedades. No se accede a ellas con paréntesis `snickers.nicknames()`, como si ejecutásemos un método, sino como propiedades. Es lo que se llama **propiedades computadas**, porque pueden evaluar código JS como doblar un número, o convertir un texto o lo que sea.

### Extendiendo clases
Consideremos este ejemplo de una clase genérica `Animal`:
```javascript
class Animal {
  constructor(name) {
    this.name = name;
    this.thirst = 100;
    this.belly = [];
  }
  drink() {
    this.thirst -= 10;
    return this.thirst;
  }
  eat(food) {
    this.belly.push(food);
    return this.belly;
  }
}
```
Y ahora creamos un nuevo animal, un rinoceronte mismamente:
```javascript
const rhino = new Animal('Rhiney');
rhino.drink() // rhino.thirst = 90
rhino.eat("pizza") // rhino.belly = ["pizza"]
```
Esto funciona porque creamos una clase genérica y luego la usamos específicamente en `rhino`. Usamos sus métodos, sus propiedades, y ningún problema. Sin embargo, puede que queramos crear una nueva clase con nuevos métodos pero aprovechar los métodos de la clase anterior. Está todo pensado, así se haría: 
```javascript
class Dog extends Animal {
  constructor(name, breed) {
    super(name); 
    this.breed = breed;
  }
}

const snickers = new Dog('Snickers', 'King Charles');
```
Solo con usar la palabra reservada `extends` no nos sirve, ya que debemos además llamar a un método `super()` en el constructor, para que sea efectiva la herencia de clases. Al llamar a `super()`, la nueva clase ya sabe lo que tiene que hacer con el parámetro que le enviamos, y también le incorpora a la nueva clase los métodos que tenga su padre. Además de otra clase, también podemos heredar de otros tipos de objetos, como un `Array`. 

## 16.- Generadores
Cuando nuestro intérprete de JavaScript ejecuta una función, lo hace de arriba abajo y desde el inicio hasta el fin. Con los `generators` de ES6 conseguimos vitaminar de alguna forma estas funciones para que puedan detenerse, o podamos pasarles parámetros posteriormente. Veamos en detalle algunas de estas opciones: 
```javascript
function* listPeople() {
  yield 'Wes';
  yield 'Kait';
  yield 'Snickers';
}

const people = listPeople(); 
console.log(people);
// Generator, WTF?
```
Sobre el papel, el `generator` lo que hace es detener la función en el punto que lo indicamos con la palabra reservada `yield`, de forma que para avanzar en el código debemos llamar a la función de nuevo. A diferencia de un `return`, que siempre detiene la función en el mismo punto. Sin embargo, si ejecutamos la función y sacamos por consola el valor que nos devuelve, vemos un código extraño y que su `prototype` es `Generator`. 

![Generadores](capturas/generators.png)

¿Qué demonios es todo esto? ¿Como extraemos el valor que queremos que devuelva la función? Pues debemos llamar a una función concreta: 
```javascript
people.next()

// En consola
Object {value: "Wes", done: false}
```
Un momento, un momento. ¿Y qué es ese `done: false` que no está en ningún lado de mi función? Asi aparece hasta que se ejecutan todos los valores + 1 (así que el último es `undefined`). Esto nos sirve para controlar el flujo de ejecución de nuestra función. Por ejemplo, para iterar un array si queremos hacer algo con cada valor, algo que sería más verboso hacer con un `map`. Va otro ejemplo más práctico: 
```javascript
const inventors = [
  { first: 'Albert', last: 'Einstein', year: 1879 },
  { first: 'Isaac', last: 'Newton', year: 1643 },
  { first: 'Galileo', last: 'Galilei', year: 1564 },
  { first: 'Marie', last: 'Curie', year: 1867 },
  { first: 'Johannes', last: 'Kepler', year: 1571 },
  { first: 'Nicolaus', last: 'Copernicus', year: 1473 },
  { first: 'Max', last: 'Planck', year: 1858 },
];

function* loop(array) {
  for (const item of array) {
    yield item;
  }
}

const inventorsGenerator = loop(inventors);
console.log(inventors.next()) // Albert Einsten
console.log(inventors.next()) // Isaac Newton
console.log(inventors.next()) // etc, etc...
```
Por último, los generadores se pueden usar para hacer control de flujo en una llamada Ajax, cuando la ejecución de las siguientes lineas depende de datos recibidos en las anteriores, por ejemplo. Es un comportamiento similar a las promesas, aunque aquí nos ahorramos el uso de *callbacks*. Además, los generadores se pueden recurrir con un bucle *for ...of* y tendrían el mismo comportamiento. 

## 17.- Proxies
Los *proxies* nos permiten controlar o directamente sobreescribir el comportamiento de un objeto, modificar la forma en la que normalmente interactuamos con ellos:
```javascript
const person = { name: "Abel", age: 30 };
const personProxy = new Proxy(person, {
  get(target, name) {
    return target[name].toUppperCase();
  },
  set(target, name, value) {
    if(typeof value === 'string') {
      target[name] = value; 
    }
  }
});

personProxy.name = 'Wesley'; 
```
La nueva variable `Proxy` que instanciamos en esta operación admite dos parámetros, siendo el primero el objeto para el cual queremos aplicar el proxy (`person` en este caso) y el segundo es un objeto llamado `handler`(manejador) en el que establecemos todas las modificaciones que queremos hacer. Al usar `set` y `get` podemos aplicar estas transformaciones al usar estos métodos. 

## 18.- Sets y Weaksets
Un `set` en ES6 es una estructura de datos similar a un array, pero con algunas particularidades (por ejemplo, no podemos acceder a los elementos por su índice). Podemos añadir contenido, eliminarlo o iterar sobre lo que tenga dentro, y además cuenta con algúnos métodos bastante molones para manejarlo:
```javascript
const people = new Set();
people.add('Wes'); // Propiedad única, no se puede meter la misma más de una vez. 
people.add('Kait');
people.add('Snickers');

console.log(people.size) // 3 // No people.length como si fuese un array

// Otros métodos
people.delete('Wes'); // Borra uno, sin especificar el índice
people.clear(); // Borra todos
people.has("Kait"); // true
```
Por debajo, el `Set` funciona como un generador, es decir que podemos ir usando el método `next` (por ejemplo, `people.next()`) para ir pasando de un valor a otro. También es iterable
```javascript
for (const person of people) {
  console.log(person);
}
```
Un nuevo ejemplo de para que podemos usar un `set`: 
```javascript
const brunch = new Set();
brunch.add("Wes");
brunch.add("Sarah");
brunch.add("Simone");

const line = brunch.values(); // Iterador
console.log(line.next().value); // Wes
console.log(line.next().value); // Sarah

// Y podemos seguir añadiendo gente al Set e iterando sobre la marcha
brunch.add("Heather");
brunch.add("Snickers");
console.log(line.next().value) // Simone, y no tengo que refrescar el iterador
```

### Weaksets
El `WeakSet` es una derivación del `set`, con algunas limitaciones o ventajas (según se mire). Para empezar, solo puede incluir objetos como propiedades en el interior, ni arrays, ni números, ni *strings*... solo Objetos. Tampoco se puede iterar sobre ellos, no tienen iteradores. Tiene los métodos de `has`, `add` y `delete` entre otros, pero no el método `clear()` porque la particularidad que tiene es que realiza su propia eliminación de variables no usadas. Por ejemplo: 
```javascript
let dog1 = { name: "Snickers", age: 3 };
let dog2 = { name: "Sunny", age: 1 };

const weakSauce = new WeakSet([dog1, dog2]);
console.log(weakSauce); // 2 perros
dog 1 = null
console.log(weakSauce); // 1 perro solo, automáticamente. 
```

## 19.- Maps y weakMaps
La diferencia entre un `set`y un `map` es como la diferencia entre un *array* y un objeto: el primero admite puede admitir solo valores y el segundo necesita que especifiquemos una clave y un valor. Respecto a la API que nos permite interactuar con el, también es sencilla. Ejemplo al canto: 
```javascript
const dogs = new Map();
dogs.set('Snickers', 3);
dogs.set('Sunny', 2);
dogs.set('Hugo', 10);

// Podemos usar forEach
dogs.forEach((val, key) => console.log(key, val)); // "Snickers" 3

// Podemos usar for... of
for (const dog of dogs) {
  console.log(dog); // Devuelve un array. 
}
```
El **weakMap** tiene las mismas particularidades que el *weakSet*: no es enumerable y no se puede vaciar con un método porque ya se encarga el *garbage collector* de limpiarlo cuando lo necesite. 

## 20.- Async / await
Sabemos que JavaScript es asíncrono por naturaleza. El intérprete de JS de los navegadores no se detiene al procesar nuestro código, lo hace de forma continua e ininterrumpida, con algunos condicionantes (las variables se suben al inicio por ejemplo). Existen algunas excepciones (como los `alert`), pero en general cuando más notamos ese comportamiento es en las llamadas AJAX. Toda petición a una API o otra web toma un tiempo, y si procesamos el resultado de la petición antes de que esta finalice nos encontraremos con comportamientos inesperados. 

Anteriormente vimos el uso de Promesas, que son una forma que incorpora ES6 para gestionar la asincronía. Versiones posteriores de JavaScript añaden además `async/await`. Una función asíncrona tiene esta pinta: 
```javascript
async function go() {
  await breathe(600);
}

// Para las funciones flecha
const go = async () => {...}
```
Lo que nos permite `async/await` es no tener que encadenar tantos `then()` cuando queremos avanzar en una promesa, resultando en una sintaxis más clara y que se lee más fácil: 
```javascript
function breathe(amount) {
  return new Promise((resolve, reject) => {
    if (amount < 500) {
      reject(`${amount} is too small of a value`);
    }
    setTimeout(() => resolve(`Done for ${amount} ms`), amount);
  })
}
async function go() {
  console.log("Start");
  const res = breathe(1000); // Devuelve promesa porque falta el await
  console.log(res);
  const res2 = await breathe(600); // Espera a que se resuelva la promesa
  console.log(res2);
  const res3 = await breathe(750);
  console.log(res3);
  const res4 = await breathe(900);
  console.log(res4);
  console.log("end");
}

go(); 
```
La explicación de este código es la siguiente: en nuestra función `go()` establecemos que va a esperar el resultado de una promesa, lo hacemos con la palabra reservada `await` al llamar a la función `breathe()`. Si olvidamos el `await`, entonces la función `breathe()` devuelve una promesa sin resolver, y de esta forma encadenamos promesas (cada llamada a la función `breathe()` solo se ejecuta cuando la anterior se ha resuelto) sin necesidad de usar múltiples veces `then()`. 

### Gestión de errores con `async/await`
Al no tener una serie de `.then()` encadenados a una promesa, tampoco podemos tener el `.catch()` final que nos ayudaba con la gestión de los errores de cualquiera de las promesas. Al usar `async/await`, usamos una forma parecida que se llama `try/catch` y que tiene esta pinta
```javascript
async function go() {
  try {
    console.log("Start");
    const res = breathe(1000); // Devuelve promesa porque falta el await
    console.log(res);
    const res2 = await breathe(600); // Espera a que se resuelva la promesa
    console.log(res2);
    const res3 = await breathe(750);
    console.log(res3);
    const res4 = await breathe(900);
    console.log(res4);
    console.log("end");
  } catch (error) {
    console.error(err);
  }
}
```
Si tenemos muchas funciones que usan `async/await`,  nos encontramos con que dentro tenemos que incorporar muchos `try/catch` para gestionar los posibles errores en cada una de ellas. Para solucionar esto, podemos crearnos una sola función que ejecute las funciones con promesas en su interior y se encargue de los errores si se producen:
```javascript
function catchErrors(fn) {
  return function() {
    return fn().catch(error => {
      console.error(`Se ha producido un error: ${error}`);
    });
  }
}
```
Esta tipo de función se llama Función de Alto Orden (o *High Order Function* en el orginal), y el funcionamiento es el siguiente: recibe una función por parámetro (`fn`), la que hemos definido con promesas en su interior. Si esta función da errores, el fallo se propagará hacia arriba y será recogido por el `catch()` de nuestra función de alto orden. El ejemplo completo sería así: 
```javascript
function catchErrors(fn) {
  return function(...args) {
    return fn(...args).catch(error => {
      console.error(`Se ha producido un error: ${error}`);
    });
  }
}

function breathe(amount) {
  return new Promise((resolve, reject) => {
    if (amount < 500) {
      reject(`${amount} is too small of a value`);
    }
    setTimeout(() => resolve(`Done for ${amount} ms`), amount);
  })
}

async function go(son, father) { // Ya sin try/catch dentro
  console.log(`Hola ${son}, hijo de ${father}`);
  const res = await breathe(1000); 
  console.log(res);
  const res2 = await breathe(600);
  console.log(res2);
  const res3 = await breathe(750);
  console.log(res3);
  const res4 = await breathe(300);
  console.log(res4);
  console.log("end");
}

const wrappedFunction = catchErrors(go);
wrappedFunction("Abel", "Alfonso");
```
Una última cosa que no explicamos es, en nuestra función `catchErrors()`, usamos `...rest` y `...spread` para gestionar el paso de parámetros. De esta forma, sabemos que entren los parámetros que entren serán tratados correctamente. 

### Sacando más rendimiento de `async/await` con `Promise.all()`
Un problema de rendimiento que tenemos al usar `async/await` es que cada bloque que marquemos como `await` sólo va a ejecutarse cuando termine el anterior. Pero puede que queramos que todos empiecen a la vez, porque si cada uno tarda 5 segundos y espera al anterior, al final aumentamos el tiempo de carga que percibe el usuario. 

Una buena práctica se ve en este código: 
```javascript
async function go() {
  const p1 = fetch("https://api.github.com/users/wesbos");
  const p2 = fetch("https://api.github.com/users/stolinski");

  // Esperando a las dos promesas
  const response = await Promise.all([p1, p2]);
  console.log(response);

  go()
}
```
### Refactorizando una función con *callbacks* a una con `async/await`:
La nueva forma de gestionar asincronía `async/await` está muy bien, pero a veces tenemos código heredado que necesitamos convertir a promesas. Aquí tenemos un ejemplo sencillo, que utiliza la geolocalización del navegador. 
```javascript
navigator.geolocation.getCurrentPosition(function(pos) {
  console.log("It worked");
  console.log(pos);
}, function (error) {
  console.log("it failed");
  console.log(error);
}
})
```
Esta función recibe dos parámetros, una función de éxito y una función de fallo, y ejecuta una u otra según vaya el proceso. Si lo *promisificamos*, quedaría así: 
```javascript
function getCurrentPosition(...args) {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(resolve, reject, ...args)
  });
}

const positionOptions = {
  enableHighAccuracy: true,
  timeout: 5000,
  maximumAge: 0
};

async function go() {
  console.log("Buscando ubicación");
  const position = await getCurrentPosition();
  console.log(position);
  console.log("Ubicación encontrada");
}

go();
```
Y de esta forma convertimos una función basada en *callbacks* a una basada en promesas. Existen muchas APIs propias del navegador que aún funcionan a base de *callbacks*

## 21.- ES7, ES8 y lo que queda por venir
A continuación algunos puntos claves de las novedades de las últimas versiones de JavaScript, que llegaron después de ES6 (2015) con una periodicidad más fija pero con menos novedades en general. 
- La primera característica son las llamadas `class properties`. Cuando tenemos una clase, a veces necesitamos que tenga unas propiedades de partida al instanciarse, y podemos ponerla en el constructor. Pero quizás no en todas las instancias necesitemos usar todas, así que es tontería crearlas si no las vamos a usar. Por eso, se suelen poner en un objeto, como hace React con `this.state = {}` y luego se compilan a través de Babel.
- Existen dos nuevos métodos de strings: `padStart()` y `padEnd()`. Sirven para que un string pase a ocupar más carácteres. El número total de carácteres será el que pasemos por parámetro, se colocarán al inicio o al final según el método y serán espacios en blanco por defecto, aunque podemos decidir si queremos otro carácter simplemente pasándole un segundo parámetro. 
- Operador exponencial: `3 ** 3` y el resultado es `27`.
- Dos nuevos métodos de objetos: `Object.entries()` y `Object.values()`, que se suman a `Object.keys()`. Aquí hay que ver un ejemplo completo: 
```javascript
const inventory = {
  backpacks: 10,
  jeans: 23,
  hoodies: 4,
  shes: 11,
}

const keys = Object.keys(inventory) // ["Backpacks", "jeans", "hoodies", "shoes"]

const values = Object.values(inventory) // [10, 23, 4, 11]

const entries = Object.entries(inventory) // Devuelve un array de arrays con el par de clave y valor. 

// Podríamos pintar este array destructurándolo:
Object.entries(inventory).forEach(([key, value]) => {
  console.log(key, val); // Backpacks 10, jeans 23, etc.
});

