# Acknowledgments

Most of the materials used in this short summary is taken from the great Mozilla documentation, see [Mozilla JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

# Task for today

Inspired by [mozilla prime numbers example](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Introducing_workers) implement simple user api in html which as an input takes an integer `n` and as a result shows all primes smaller or equal to `n` (simple two html div tags for input and output will suffice + some button to fire calculations). If `n` is prime the output should contain "`n` is a prime" (instead of printing `n`). The function calculating required primes should be non-blocking (notice that such a procedure might be time consuming for large numbers - we do not want out browser to freeze). Some requirements

- stick to general coding principles
- throw custom errors (for how to create one, see e.g. [this link](https://javascript.info/custom-errors)) e.g. if user inputs invalid data
- handle appropriately these errors (do not allow program to crush). A user might be malicious.
- use some caching to speed up the upcoming user requests.
- prepare two asynchronous versions: one based on promises (equivalently, `async` functions) and the other based on (dedicated) `workers`.

# Understanding the concurrency of js

Is it really executed by one thread? It is an oversimplification.
[see e.g. this article](https://blog.avenuecode.com/understanding-the-javascript-concurrency-model)

```javascript
var callback = function() {
    console.log('B');
};

var foobar = function() {
    console.log('A');
    setTimeout(callback, 1000);
    console.log('C')
}
```
keywords: callstack, task queue (message queue), event loop, web api

`Message queue`  is a list of messages to be processed. Each message has an associated function that gets called to handle the message.


What happens under the hood when `setTimeout` is reached?
Firstly, `callback` is moved to Web API where it is executed.  When some task is completed by the Web API (in our case `callback`()) then it is pushed to the Task Queue. Now the last piece of puzzle comes in - the event loop. It has a simple job, it looks at the call stack and the message queue and if the stack is empty, it takes the oldest message from the queue and its corresponding function is called with the message as an input parameter (as usual by creating new stack frame for that function).

Properties:
1. Never blocking
2. Event loop (see [mozilla event loop](https://developer.mozilla.  org/en-US/docs/Web/JavaScript/Event_loop)) looks like
    ```javascript
    while (queue.waitForMessage()) {
      queue.processNextMessage();
    }
    ```

    In particular, each message is processed completely before any other message is processed.

    This offers some nice properties when reasoning about your program, including the fact that whenever a function runs, it cannot be preempted and will run entirely before any other code runs (and can modify data the function manipulates).

3. Several runtimes communicating together.
A web worker or a cross-origin iframe has its own stack, heap, and message queue. Two distinct runtimes can only communicate through sending messages via the postMessage method. This method adds a message to the other runtime if the latter listens to message events.


# Memory management

In order to not bother the programmer with allocations, JavaScript will automatically allocate memory when values are initially declared.
```javascript
const s = "azerty"; // allocates memory for a string
function f(a) {
  return a + 2;
} // allocates a function (which is a callable object)
const d = new Date(); // allocates a Date object
```

You do not have to worry about releasing memory,  js uses garbage collection see [Mark-and-sweep algorithm](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management#mark-and-sweep_algorithm)


# Object

It is a data type. Nearly all objects in JavaScript are instances of `Object`; a typical object inherits properties (including methods) from `Object.prototype`, although these properties may be shadowed (a.k.a. overridden). The only objects that don't inherit from `Object.prototype` are those with `null` prototype, or descended from other `null` prototype objects.



# Properties
Every property in JavaScript objects can be classified by three factors:

- Enumerable or non-enumerable. Enumerable properties are those properties whose internal enumerable flag is set to true, which is the default for properties created via simple assignment or via a property initializer. Properties defined via Object.defineProperty and such are not enumerable by default. Most iteration means (such as for...in loops and Object.keys) only visit enumerable keys.

- String or symbol.

- Own property or inherited property from the prototype chain. Ownership of properties is determined by whether the property belongs to the object directly and not to its prototype chain.


All properties, enumerable or not, string or symbol, own or inherited, can be accessed with dot notation or bracket notation.

```javascript
const person1 = {};
person1['firstname'] = 'Mario';
person1['lastname'] = 'Rossi';

console.log(person1.firstname);
// Expected output: "Mario"

const person2 = {
  firstname: 'John',
  lastname: 'Doe',
};

console.log(person2['lastname']);
// Expected output: "Doe"
```

<table>
  <thead>
    <tr>
      <th></th>
      <th>Enumerable, own</th>
      <th>Enumerable, inherited</th>
      <th>Non-enumerable, own</th>
      <th>Non-enumerable, inherited</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable"><code>propertyIsEnumerable()</code></a></td>
      <td><code>true ✅</code></td>
      <td><code>false ❌</code></td>
      <td><code>false ❌</code></td>
      <td><code>false ❌</code></td>
    </tr>
    <tr>
      <td><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty"><code>hasOwnProperty()</code></a></td>
      <td><code>true ✅</code></td>
      <td><code>false ❌</code></td>
      <td><code>true ✅</code></td>
      <td><code>false ❌</code></td>
    </tr>
    <tr>
      <td><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn"><code>Object.hasOwn()</code></a></td>
      <td><code>true ✅</code></td>
      <td><code>false ❌</code></td>
      <td><code>true ✅</code></td>
      <td><code>false ❌</code></td>
    </tr>
    <tr>
      <td><a href="/en-US/docs/Web/JavaScript/Reference/Operators/in"><code>in</code></a></td>
      <td><code>true ✅</code></td>
      <td><code>true ✅</code></td>
      <td><code>true ✅</code></td>
      <td><code>true ✅</code></td>
    </tr>
  </tbody>
</table>

You can delete  a property e.g.
```javascript
const Employee = {
  firstname: 'Maria',
  lastname: 'Sanchez',
};

console.log(Employee.firstname);
// Expected output: "Maria"

delete Employee.firstname;

console.log(Employee.firstname);
// Expected output: undefined

```

# This and Bind
In most cases, the value of `this` is determined by how a function is called (runtime binding). It can't be set by assignment during execution, and it may be different each time the function is called.
Inside a function, the value of `this` depends on how the function is called. Think about `this` as a hidden parameter of a function — just like the parameters declared in the function definition, `this` is a binding that the language creates for you when the function body is evaluated.
For a typical function, the value of `this` is the object that the function is accessed on. In other words, if the function call is in the form obj.f(), then this refers to obj. For example:

Note that the value of `this` is not the object that has the function as an own property, but the object that is used to call the function. You can prove this by calling a method of an object up in the prototype chain.

```javascript
const obj3 = {
  __proto__: obj1,
  name: "obj3",
};

console.log(obj3.getThis()); // { name: 'obj3' }
```

In the `strict-mode` - if the function is called without being accessed on anything, this will be undefined.

```javascript
function getThisStrict() {
  "use strict"; // Enter strict mode
  return this;
}
console.log(typeof getThisStrict()); // "undefined"
```
In` arrow function`s, `this` retains the value of the enclosing lexical context's `this`. In other words, when evaluating an` arrow function`'s body, the language does not create a new `this` binding.


`Arrow functions` create a closure over the `this` value of its surrounding scope, which means `arrow functions` behave as if they are "auto-bound" — no matter how it's invoked, this is bound to what it was when the function was created (in the example above, the global object). The same applies to `arrow functions` created inside other functions: their `this` remains that of the enclosing lexical context.

```javascript
const obj = {
  getThisGetter() {
    const getter = () => this;
    return getter;
  },
};

const fn = obj.getThisGetter();
console.log(fn() === obj); // true

const fn2 = obj.getThisGetter;
console.log(fn2()() === globalThis); // true in non-strict mode
```
## `this` in DOM event handlers
When a function is used as an event handler, its `this` parameter is bound to the DOM element on which the listener is placed.
```javascript
// When called as a listener, turns the related element blue
function bluify(e) {
  // Always true
  console.log(this === e.currentTarget);
  // true when currentTarget and target are the same object
  console.log(this === e.target);
  this.style.backgroundColor = "#A5D9F3";
}

// Get a list of every element in the document
const elements = document.getElementsByTagName("*");

// Add bluify as a click listener so when the
// element is clicked on, it turns blue
for (const element of elements) {
  element.addEventListener("click", bluify, false);
}
```

When the code is called from an inline event handler attribute, its `this` is bound to the DOM element on which the listener is placed.
```javascript
<button onclick="alert(this.tagName.toLowerCase());">Show this</button>
```

Note, however, that only the outer scope has its `this` bound this way.
```html
<button onclick="alert((function () { return this; })());">
  Show inner this
</button>
```
In `this` case, the `this` parameter of the inner function is bound to `globalThis`.



## Bind
Bind what for?
```javascript
const module = {
  x: 42,
  getX: function () {
    return this.x;
  },
};

const unboundGetX = module.getX;
console.log(unboundGetX()); // The function gets invoked at the global scope
// Expected output: undefined

const boundGetX = unboundGetX.bind(module);
console.log(boundGetX());
// Expected output: 42
```

Just like with regular functions, the value of `this` within `methods` depends on how they are called. Sometimes it is useful to override this behavior so that `this` within classes always refers to the class instance. To achieve this, bind the class methods in the constructor.

```javascript
class Car {
  constructor() {
    // Bind sayBye but not sayHi to show the difference
    this.sayBye = this.sayBye.bind(this);
  }

  sayHi() {
    console.log(`Hello from ${this.name}`);
  }

  sayBye() {
    console.log(`Bye from ${this.name}`);
  }

  get name() {
    return "Ferrari";
  }
}

class Bird {
  get name() {
    return "Tweety";
  }
}

const car = new Car();
const bird = new Bird();

// The value of 'this' in methods depends on their caller
car.sayHi(); // Hello from Ferrari
bird.sayHi = car.sayHi;
bird.sayHi(); // Hello from Tweety

// For bound methods, 'this' doesn't depend on the caller
bird.sayBye = car.sayBye;
bird.sayBye(); // Bye from Ferrari
```


# Classes

Classes are templates for creating objects. They encapsulate data with code to work on that data. Classes in JS are built on prototypes but also have some syntax and semantics that are unique to classes. Classes are in fact "special functions". Code within class is always in the strict mode.

A class element can be characterized by three aspects:

- `Kind`: Getter, setter, method, or field
- `Location`: Static or instance
- `Visibility`: Public or private

```javascript
class Rectangle /*extends Shape*/ {
  static displayName = "Rectangle";
 
  #height = 0;
  #width = 0;
 
  constructor(height, width) {
    // super(...)
    this.#height = height;
    this.#width = width;
  }
  // Getter
  get area() {
    return this.calcArea();
  }
  // Method
  calcArea() {
    return this.#height * this.#width;
  }
  *getSides() {
    yield this.#height;
    yield this.#width;
    yield this.#height;
    yield this.#width;
  }
}

const square = new Rectangle(10, 10);

console.log(square.area); // 100
console.log([...square.getSides()]); // [10, 10, 10, 10]
console.log(Rectangle.displayName)
```
Note that `getSides()` method is a generator. For more information see [mozilla generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators?ref=sfeir.dev).

## Inheritance

JavaScript objects are dynamic "bags" of properties (referred to as own properties). JavaScript objects have a link to a prototype object. When trying to access a property of an object, the property will not only be sought on the object but on the prototype of the object, the prototype of the prototype, and so on until either a property with a matching name is found or the end of the prototype chain is reached.

[[Prototype]]  should not be confused with the func.prototype property of functions, which specifies the [[Prototype]] to be assigned to all instances of objects created by the given function when used as a constructor.

Why prototypes? To share common things.

```javascript
 Object.getPrototypeOf()
 ```
Example with shadowing
```javascript
const o = {
  a: 1,
  b: 2,
  // __proto__ sets the [[Prototype]]. It's specified here
  // as another object literal.
  __proto__: {
    b: 3,
    c: 4,
    __proto__: {
      d: 5,
    },
  },
};
```
What will be printed?
console.log(a,b,c,d)

## Inheriting  methods

```javascript
const parent = {
  value: 2,
  method() {
    return this.value + 1;
  },
};

console.log(parent.method()); // 3
// When calling parent.method in this case, 'this' refers to parent

const child = {
  __proto__: parent,
};
console.log(child.method()); // 3
// When child.method is called, 'this' refers to child.
// So when child inherits the method of parent,
// The property 'value' is sought on child. However, since child
// doesn't have an own property called 'value', the property is
// found on the [[Prototype]], which is parent.value.

child.value = 4; // assign the value 4 to the property 'value' on child.
// This shadows the 'value' property on parent.
// The child object now looks like:
// { value: 4, __proto__: { value: 2, method: [Function] } }
console.log(child.method()); // 5
// Since child now has the 'value' property, 'this.value' means
// child.value instead
```

## Constructor function
Compare the following:
```javascript
const boxes = [
  { value: 1, getValue() { return this.value; } },
  { value: 2, getValue() { return this.value; } },
  { value: 3, getValue() { return this.value; } },
];
```
vs
```javascript
const boxPrototype = {
  getValue() {
    return this.value;
  },
};

const boxes = [
  { value: 1, __proto__: boxPrototype },
  { value: 2, __proto__: boxPrototype },
  { value: 3, __proto__: boxPrototype },
];
```
vs


```javascript
// A constructor function
function Box(value) {
  this.value = value;
}

// Properties all boxes created from the Box() constructor
// will have
Box.prototype.getValue = function () {
  return this.value;
};

const boxes = [new Box(1), new Box(2), new Box(3)];
```
vs

```javascript
class Box {
  constructor(value) {
    this.value = value;
  }

  // Methods are created on Box.prototype
  getValue() {
    return this.value;
  }
}
```
Some prototypes are asssigned implicitly, e.g.
```javascript
const array = [1, 2, 3];
Object.getPrototypeOf(array) === Array.prototype; // true
```

Question, could/should we
```javascript
Array.prototype.myMethod = function () {...}
```

Compare
```javascript
function Base() {}
function Derived() {}
// Set the `[[Prototype]]` of `Derived.prototype`
// to `Base.prototype`
Object.setPrototypeOf(Derived.prototype, Base.prototype);

const obj = new Derived();
// obj ---> Derived.prototype ---> Base.prototype ---> Object.prototype ---> null
```
vs class approach

```javascript
class Base {}
class Derived extends Base {}

const obj = new Derived();
// obj ---> Derived.prototype ---> Base.prototype ---> Object.prototype ---> null
```

Warning: an arrow function does not have a default prototype! Every other function does!



# Promises and async
See [Mozilla documentation on Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) and [Mozilla documentation on async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) and [Mozilla documentation on await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await).

A `Promise` is a proxy for a value not necessarily known when the promise is created. It allows you to associate handlers with an asynchronous action's eventual success value or failure reason. This lets asynchronous methods return values like synchronous methods: instead of immediately returning the final value, the asynchronous method returns a promise to supply the value at some point in the future.

   Let us recall that promise can be in one of three states:

- `pending`: the promise has been created, and the asynchronous function it's associated with has not succeeded or failed yet. This is the state your promise is in when it's returned from a call to fetch(), and the request is still being made.
- `fulfilled`: the asynchronous function has succeeded. When a promise is fulfilled, its then() handler is called.
- `rejected`: the asynchronous function has failed. When a promise is rejected, its catch() handler is called.

A promise is said to be `settled` if it is either fulfilled or rejected, but not pending.


```javascript
const fetchPromise = fetch(
        "bad-scheme://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
    );

    fetchPromise
        .then((response) => {
            if (!response.ok) {
                throw new Error(`HTTP error: ${response.status}`);
            }
            return response.json();
        })
        .then((data) => {
            console.log(data[0].name);
        })
        .catch((error) => {
            console.error(`Could not get products: ${error}`);
        });

```


```javascript
const fetchPromise1 = fetch(
        "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
    );
    const fetchPromise2 = fetch(
        "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/not-found",
    );
    const fetchPromise3 = fetch(
        "https://mdn.github.io/learning-area/javascript/oojs/json/superheroes.json",
    );

    Promise.all([fetchPromise1, fetchPromise2, fetchPromise3])
        .then((responses) => {
            for (const response of responses) {
                console.log(`${response.url}: ${response.status}`);
            }
        })
        .catch((error) => {
            console.error(`Failed to fetch: ${error}`);
        });
```

# Basics
1. "use strict". Why? Strict mode makes it easier to write "secure" JavaScript.
    E.g. In the normal JavaScript, mistyping a variable name creates a new global variable.
    In strict mode, this will throw an error, making it impossible to accidentally create a global variablepi = 3.14;
2.  `var`, `let`, `const`
```javascript
const case_sensitive_unicode_read_only = 1; /*statements sep by semicolon*/
var declare_variable_with_opt_value = "abc";
let block_scoped = 1;
```
What are differences between const, let,var? Let is scoped var's scope is function, var is hoisted and initialized, let is hoisted but not initialized, let does not create a property on the global object. In strict mode var variable can be redeclared, let does not allow this to happen. Example of problems with var:
```javascript
// Logs 3 thrice, not what we meant.
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```
const variables cannot be reassigned. Are the completely const? No e.g.
```javascript
// You can create a constant array:
const cars = ["Saab", "Volvo", "BMW"];
// You can change an element:
cars[0] = "Toyota";
// You can add an element:
cars.push("Audi");
```

3. Functions are hoisted with definitions - no need to forward declare.

4.  Closures,
```javascript
function init() {
  var name = "Mike";
  function displayName() {
    // displayName() is the inner function, that forms the closure
    console.log(name); // use variable declared in the parent function
  }
  displayName();
}
init();
```
5. What types are considered false?
```javascript
if (false || undefined || null || 0 || NaN || "") {
        console.log("no log")
    }
```

6. Exceptions handling. `try` `catch` `finally`
```javascript
 try {
        throw Error("myException"); // generates an exception
    } catch (err) {
        // statements to handle any exceptions
        console.error(`exception was thrown: ${err.name}, ${err.message}`); // pass exception object to error handler
    }
    finally {
       console.log("after try catch e.g. close file"); // Always close the resource
    }
```
7. For loop
```javascript
let obj = {abc : "abc", efg : 2};
    for (const i in obj){
        console.log(`i = ${i}\n, obj[i] = ${obj[i]}\n`);
    }
    let arr = [1,2,3];
    arr.foo = "hello";
    for (const val of arr){
        console.log(`val = ${val}\n`);
    }
    const sayings = new Map();
    sayings.set("dog", "woof");
    sayings.set("cat", "meow");
    sayings.set("elephant", "toot");
    sayings.size; // 3
    sayings.get("dog"); // woof
    sayings.get("fox"); // undefined
    sayings.has("bird"); // false
    sayings.delete("dog");
    sayings.has("dog"); // false

    for (const [key, value] of sayings) {
        console.log(`${key} goes ${value}`);
    }
// "cat goes meow"
// "elephant goes toot"
```

8. Arguments of function [by value or by reference?](https://stackoverflow.com/questions/518000/is-javascript-a-pass-by-reference-or-pass-by-value-language).  call-by-sharing. An item is passed in by value. But the item that is passed by value is itself a reference.
```javascript
function changeStuff(a, b, c)
    {
        a = a * 10;
        b.item = "changed";
        c = {item: "changed"};
    }

    var num = 10;
    var obj1 = {item: "unchanged"};
    var obj2 = {item: "unchanged"};

    changeStuff(num, obj1, obj2);

    console.log(num);
    console.log(obj1.item);
    console.log(obj2.item);
```
what will be printed?

BTW What about assignment operator? See e.g. [how js references work](https://www.sitepoint.com/how-javascript-references-work/).
In short, when primitives are assigned, they are assigned by value whereas references types (like  objects) are assigned by reference (or more precisely they're assigned a copy of the reference).
```javascript
function addABC(foo) {
   foo.abc = 10;
}

var x = {};
var y = x;
addABC(x);
console.log(x.abc, y.abc);
```

or what is `x.a` at the end of
```javascript
    var x = { a: 1 };
    var y = x;
    y = {}; ////value is reassigned (create new reference)
```

