# Task for today

Upgrade your previous js project to ts.

# Acknowledgements

These notes use materials/examples from [typescript handbook](https://www.typescriptlang.org/docs/handbook/intro.html).

# Problems with pure js

Consider the following code
```javascript
function fn(x) {
  return x.flip();
}
```
We can observe by reading the code that this function will only work if given an object with a callable flip property, but JavaScript doesn’t surface this information in a way that we can check while the code is running. The only way in pure JavaScript to tell what fn does with a particular value is to call it and see what happens. This kind of behavior makes it hard to predict what the code will do before it runs, which means it’s harder to know what your code is going to do while you’re writing it.

Similarly,
```javascript
const message = "hello!";
 
message();
```
Note that we got error: `This expression is not callable. Type 'String' has no call signatures.`. Imagine a situation in which we did not thoroughly tested our program and this piece
of code was never run. Since js can detect this error only at the run time, we will never notice this bug.

If our class does not have a field we would like to be notified of this fact. However, js returns `undefined` instead.
```javascript
const user = {
  name: "Daniel",
  age: 26,
};
user.location; // returns undefined
```

Static typing property of ts solves many problems of such types. E.g. it can detect typos when calling methods,  check if operands of `operator<` are comparable, check if a method exists (in particular code completion or error checking can be enabled in your editor) and many more.

# Installing

Execute in bash `sudo apt install node-typescript`.


# Running
If you are using WebStorm read [this](https://www.jetbrains.com/help/webstorm/compiling-typescript-to-javascript.html#ts_create_tsconfig_json). In case you are going to use terminal firstly save 
```javascript
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date}!`);
}
 
greet("Brendan");
```
as `example_with_bug.ts` and try to compile it using `tsc example_with_bug.ts`.  See what happens. Note that js file was generated despite the fact that ts compiler informed us about an error. This behaviour allows you to copy correct (working) js file into ts one and compile/run it even if type checking produces errors (every js code is valid ts code). If you want to be more restrictive and do not generate js file in case when compilation fails use flag `--noEmitOnError`.

# Explicit types, type annotations
You can tell ts what are the types of variables. Take for example:
```typescript
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
```
Is this correct code? Will it compile?
```typescript
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
 
greet("Maddison", Date());
```
<details>
  <summary>Solution</summary>
  Change Date() to new Date()
</details>

We do not have always write type annotations, e.g.
```typescript
a = "string"
```
In this case ts guesses that `a:string = "string"`. Similarly for arguments of anonymous functions
```typescript
const names = ["Alice", "Bob", "Eve"];
 
// Contextual typing for function - parameter s inferred to have type string
names.forEach(function (s) {
  console.log(s.toUpperCase());
});
 
// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUpperCase());
});
```
In this example, parameter `s:string` (the process  of guessing the type of `s` is called `contextual typing` because the context that the function occurred within informs what type it should have).



You can annotate the return type of a function as well:
```typescript
async function getFavoriteNumber(): Promise<number> {
  return 26;
}
```
# Type erasure
Notice that js cannot execute ts with type annotations. To solve this ts needs compilation to erase types. E.g. ts on our (corrected) example produces
```javascript
"use strict";
function greet(person, date) {
    console.log("Hello ".concat(person, ", today is ").concat(date.toDateString(), "!"));
}
greet("Maddison", new Date());

```
Note that there are no type annotations and concatenation method is used. In this example the code after compilation was downleveled to old ECMA standard (that's why concat method appeared). If you target new ECMA standard by using flag `--target es2015` you will get
```javascript
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
greet("Maddison", new Date());
```
Remember to always use up-to-date ecma standard [see wiki list](https://en.wikipedia.org/wiki/ECMAScript_version_history) target unless you have very good reason not to (e.g. compability with older browsers). 
Remember that type annotation never changes the runtime behaviour of your program!


# Strictness flags

In `tsconfing.json` file you can set how strict ts type checking should be. E.g. you can as strict as possible `"strict": true`. Otherwise you can choose what ts should check for you by launching some options e.g. `"noImplicitAny": true` or `"strictNullChecks": true`. 

## noImplicitAny
Sometimes ts does not infer type is lenient as assigns type `any` (plain js type). This flag notifies you about such cases.

## strictNullChecks

By default, values like `null` or `undefined` are assignable to any other type. This flag makes handling these cases more explicit (in case we forgot to handle them).

# Types
1. `string` ("...") ,
2. `number` (int or float),
3. `boolean` (true or false),
4. `array` (e.g. number[], Array&lt;number&gt;, [1,2,3]),
5. `any` (no type checking),
6. `null` (represents the intentional absence of any object value. It is one of js's primitive values and is treated as falsy for boolean operations),
7. `undefined`  (a variable that has not been assigned a value is of type undefined),
8. `void` (type which indicates that a function returns nothing),
9. `object` (refers to any value that isn’t a primitive (`string`, `number`, `bigint`, `boolean`, `symbol`, `null`, or `undefined`); is different from the empty object type { }; is different from the global type Object),
10. `unknown` (type-safe counterpart of `any`). Anything is assignable to `unknown`, but `unknown` isn't assignable to anything but itself and any without a type assertion or a control flow based narrowing. Likewise, no operations are permitted on an `unknown` without first asserting or narrowing to a more specific type. You use it e.g. when you want to describe the least-capable type in TypeScript;  this is useful for APIs that want to signal “this can be any value, so you must perform some type of checking before you use it”. This forces users to safely introspect returned values. For more see [stackoverflow](https://stackoverflow.com/questions/51439843/unknown-vs-any)
11. `never`, ( represents values which are never observed), a function can return `never` type if it in fact never returns a value (e.g. throws an error). Later on you will learn that it can represent the type left when there is nothing left in a union type.

# Type Assertions
Add information about the type of a value that TypeScript can’t know about. Example calling js object method,
```typescript
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```
or equivalently
```typescript
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

Beware, since type assertions are removed at compile-time, there is no runtime checking associated with a type assertion. There won’t be an exception or null generated if the type assertion is wrong.
## object type

This is any JavaScript value with properties, e.g. arg of a function can be of the form `point: { x: number; y: number }`. Object can have optional properties
```typescript
function printName(obj: { first: string; last?: string }) {
  // ...
}
// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```
Note that when you read from an optional property, you’ll have to check for undefined before using it.
```typescript
function printName(obj: { first: string; last?: string }) {
  // Error - might crash if 'obj.last' wasn't provided!
  console.log(obj.last.toUpperCase());
  //'obj.last' is possibly 'undefined'.
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
 
  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
```
## Conditional type
```typescript
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}
 
type Example1 = Dog extends Animal ? number : string;
```
## Union type

The allow you to combine types e.g. `number | string`. 
Note that deduced type can be of union form,
```typescript
let x = Math.random() < 0.5 ? 10 : "hello world!";
```
where `x` is of type `string | number`.
### How to handle union types?
You can handle each type separately (`narrowing`) using  `typeof` or `Array.isArray`. 
For example
```typescript
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```
The part `typeof padding === "number"` is called `type guard`. TypeScript follows possible paths of execution that our programs can take to analyze the most specific possible type of a value at a given position. It looks at these special checks (called type guards) and assignments, and the process of refining types to more specific types than declared is called `narrowing`.

Watch out. The code below might not work as you think (at first glance)
```typescript
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
  //'strs' is possibly 'null'.
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```
<details><summary>Reason</summary>the typeof `null` is "object". You can guard agaist null` using if (strs && typeof strs === "object")`</details>

You can narrow types using operator `in` (checks if an object contains a property) ,
```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }
 
  return animal.fly();
}
```

You can narrow types using `instanceof` (checks if a type contains a prototype of given type),
```typescript
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
               
  //(parameter) x: Date
  } else {
    console.log(x.toUpperCase());           
  //(parameter) x: string
  }
}
```

You can narrow types using operator `===`,
```typescript
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase();
    y.toLowerCase();
  } else {
    console.log(x);
//(parameter) x: string | number
    console.log(y);
//(parameter) y: string | boolean
  }
}
```
## Alias type

To create new types. E.g.
```typescript
type Point = {
  x: number;
  y: number;
};
```
or `type ID = number | string;`.
Then you can annote variables with e.g. Point type,  
```typescript
// Exactly the same as the earlier example
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```
You can extend a type via intersections
```typescript
type Animal = {
  name: string;
}

type Bear = Animal & { 
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
```

## Interface  
Used to name an object type of a specific structure.
```typescript
interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```
Note that ts is only concerned with the structure of the value we passed to `printCoord `- it only cares that it has the expected properties. Being concerned only with the structure and capabilities of types is why we call TypeScript a `structurally typed` type system.


You can extend interface via
```typescript
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
```

## Alias vs interface

The key distinction is that a type cannot be re-opened to add new properties vs an interface which is always extendable. Hence you can do 
```typescript
interface Window {
  title: string;
}

interface Window {
  ts: TypeScriptAPI;
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```
but not 
```typescript
type Window = {
  title: string;
}

type Window = {
  ts: TypeScriptAPI;
}
 // Error: Duplicate identifie# Literal typesr 'Window'.        
```

# Literal types number and string
Created using `const` declaration, e.g.
```typescript
const constantString = "Hello World";
// Because `constantString` can only represent 1 possible string, it
// has a literal type representation
```
They can be useful, e.g.
```typescript
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
```
Note that you will get error: Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
Similarly,
```typescript
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```
How to change object properties into literals?
```typescript
const req = { url: "https://example.com", method: "GET" as "GET" };
handleRequest(req.url, req.method as "GET");
```
or make all properties const (converting numbers and strings into literals) via
```typescript
req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

## Enum
By example
```typescript
enum Direction {
  Up = 1,
  Down,
  Left,
  Right,
}
```

## BigInt
From ES2020 onwards, there is a primitive in JavaScript used for very large integers
```typescript
// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);
 
// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;
```

## symbol
Symbols are immutable, and unique.
```typescript
let sym2 = Symbol("key");
let sym3 = Symbol("key");
sym2 === sym3; // false, symbols are unique
```
Just like strings, symbols can be used as keys for object properties.
```typescript
const sym = Symbol();
let obj = {
  [sym]: "value",
};
console.log(obj[sym]); // "value"
```
# Using type predicates
Recall that the `as` keyword is a Type Assertion in TypeScript which tells the compiler to consider the object as another type than the type the compiler infers the object to be.
```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```
`pet is Fish` is our type predicate in this example. A predicate takes the form `parameterName is Type`, where `parameterName` must be the name of a parameter from the current function signature.

Any time `isFish` is called with some variable, TypeScript will narrow that variable to that specific type if the original type is compatible. Moreover, below in the `else` branch compiler knows that the variable type is not Fish,
```typescript
let pet = getSmallPet();
 
if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```
You can use predicates to filter arrays,
```typescript
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// or, equivalently
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];
```
# Never type
When narrowing, you can reduce the options of a union to a point where you have removed all possibilities and have nothing left. In those cases, TypeScript will use a `never` type to represent a state which shouldn’t exist. Using the fact that you can assign to `never` type only `never` type, `never` can serve as a check for unreachable code. Indeed, by assigning to `never` union type object compiler with throw an error if there is some type you did not handle previously. For example 
```typescript
type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
// Type 'Triangle' is not assignable to type 'never'.
      return _exhaustiveCheck;
  }
}
```


# Function Type Expressions
```typescript
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}
```

Note that the parameter name is required. The function type `(string) => void` means a function with a parameter named string of type `any`!

# Call Signatures

In JavaScript, functions can have properties in addition to being callable. However, the function type expression syntax doesn’t allow for declaring properties. If we want to describe something callable with properties, we can write a call signature in an object type:

```typescript
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
 
function myFunc(someArg: number) {
  return someArg > 3;
}
myFunc.description = "default description";
 
doSomething(myFunc);
```

Note that the syntax is slightly different compared to a function type expression - use `:` between the parameter list and the return type rather than `=>`.

# Construct Signatures

JavaScript functions can also be invoked with the `new` operator. TypeScript refers to these as constructors because they usually create a new object. You can write a construct signature by adding the `new` keyword in front of a call signature:

```typescript
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

Some objects, like JavaScript’s `Date` object, can be called with or without new. You can combine call and construct signatures in the same type arbitrarily:
```typescript
interface CallOrConstruct {
  (n?: number): string;
  new (s: string): Date;
}
```

# Generic Functions

We can write functions using `type parameter` (in this case named `Type`),
```typescript
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
```
`Type` is deduced (no need to specify `Type` directly when calling this function).
```typescript
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
```
or 
```typescript
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
 
// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

We can constrain types using `extends` keyword. For example let us write a function that returns the longer of two values. To do this, we need a length property that’s a number. We constrain the type parameter to that type by writing an extends clause:

```typescript
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
```

You can manually specify generic types when calling a function
```typescript
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```
Note that without `<string | number>` this code would not compile.

# More on functions
 You can have `optional arguments`, e.g. `declare function f(x?: number): void;`. You can have `default parameters`, e.g. `function f(x = 10)`. You can overload functions. Remember that when you overload in TypeScript, you only have one implementation with multiple signatures,
 ```typescript
class Foo {
    myMethod(a: string);
    myMethod(a: number);
    myMethod(a: number, b: string);
    myMethod(a: string | number, b?: string) {
        alert(a.toString());
    }
}
```
The implementation signature must be compatible with all the overloads. As JavaScript doesn't have types, compiled ts cannot end up creating two functions taking same number of arguments. Hence the following will not compile
```typescript
class A {
constructor(){}
create(a:number) {}
create(b:string) {}
}
```
For more read [stackoverflow](https://stackoverflow.com/questions/12688275/how-to-do-method-overloading-in-typescript/12689054#12689054).


# Rest Parameters and Arguments

We can define functions that take an unbounded number of arguments using rest parameters, e.g.
```typescript
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```
Conversely, we can provide a variable number of arguments from an iterable object (for example, an array) using the spread syntax. For example, the push method of arrays takes any number of arguments:

```typescript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
Try
```

# Parameter destructing - unpacking objects

```typescript
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```


# Objects

## Optional Properties
```typescript
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}
 
function paintShape(opts: PaintOptions) {
  // ...
}
 
const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

You can do unpacking with the default values
```typescript
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions);
```


## readonly Properties 
```typescript
interface SomeType {
  readonly prop: string;
}
```
Now  if you have an object of type `SomeType` you cannot assign to `prop`. On the other hand you can modify `name` and `age` of `resident` in 
```typescript
interface Home {
  readonly resident: { name: string; age: number };
}
```
objects.

# Index Signatures
```typescript
interface StringArray {
  [index: number]: string;
}
 
const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
```

This index signature states that when a `StringArray` is indexed with a `number` , it will return a `string`. 

Only some types are allowed for index signature properties: `string` (dictionary), `number` (array), `symbol`, `template string patterns`, and `union types consisting only of these`.

While string index signatures are a powerful way to describe the “dictionary” pattern, they also enforce that all properties match their return type. This is because a string index declares that `obj.property` is also available as `obj["property"]`. In the following example, name’s type does not match the string index’s type, and the type checker gives an error:
```typescript
interface NumberDictionary {
  [index: string]: number;
 
  length: number; // ok
  name: string; //error
}
```
On the other hand if you change `[index: string]: number;` to `[index: string]: string | number;` this example will work.

# Generic interface
As with function interface can be "templated",
```typescript
interface Box<Type> {
  contents: Type;
}
let box: Box<string>;
```

A built-in examples are `Array<T>` or `ReadonlyArray<T>`.


# Tuple types

A `tuple type` is another sort of `Array` type that knows exactly how many elements it contains, and exactly which types it contains at specific positions. Note that if we try to index past the number of elements, we’ll get an error.



```typescript
type StringNumberPair = [string, number];
```

You can "unpack" tuple via
```typescript
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;
// const hash: number
//const inputString: string
  console.log(inputString);
  console.log(hash);
}
```

Types can be optional e.g. `type Either2dOr3d = [number, number, number?];`.
Tuples can use rest parameters 
```typescript
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```

Fancy example
```typescript
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```
is basically equivalent to
```typescript
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```

# Classes

1. Fields can have default values (`initializers`)
```typescript
class Point {
  x = 0;
  y = 0;
}
 ```
2. Configuration `--strictPropertyInitialization` sets controls whether class fields need to be initialized in the constructor.

3. A field can be `readonly`. This prevents assignments to the field `outside of the constructor`.

4.  You can overload constructors.

5. If you have a base class then ts will notify you if you forget to call base constructor via `super(...)`.
6. You can use getters (`get`) and setters (`set`). If get exists but no set, the property is automatically readonly. If the type of the setter parameter is not specified, it is inferred from the return type of the getter. Getters and setters must have the same Member Visibility.
7. Since TypeScript 4.3, it is possible to have accessors with different types for getting and setting.
```typescript
class Thing {
  _size = 0;
 
  get size(): number {
    return this._size;
  }
 
  set size(value: string | number | boolean) {
    let num = Number(value);
 
    // Don't allow NaN, Infinity, etc
 
    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }
 
    this._size = num;
  }
}
```
8. Classes can declare `index signatures`; these work the same as Index Signatures for other object types:
```typescript
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);
 
  check(s: string) {
    return this[s] as boolean;
  }
}
```
9. You can use an `implements` clause to check that a class satisfies a particular `interface`. An error will be issued if a class fails to correctly implement it:
```typescript
interface Pingable {
  ping(): void;
}
 
class Sonar implements Pingable {
  ping() {
    console.log("ping!");
  }
}
```
Note that implements does not change the type of class `Sonar`.

10. Classes may `extend` from a `base` class. A derived class has all the properties and methods of its base class, and can also define additional members.
```typescript
class Animal {
  move() {
    console.log("Moving along!");
  }
}
 
class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}
 
const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```
11. Can `override` methods.
```typescript
class Base {
  greet() {
    console.log("Hello, world!");
  }
}
 
class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}
 
const d = new Derived();
d.greet();
d.greet("reader");
```
It’s important that a derived class follow its base class contract. Remember that it’s very common (and always legal!) to refer to a derived class instance through a base class reference:

```typescript
// Alias the derived instance through a base class reference
const b: Base = d;
// No problem
b.greet();
```
12. When `target >= ES2022` or `useDefineForClassFields` is true, class fields are initialized after the parent class constructor completes, overwriting any value set by the parent class. 
13. You can set `Member Visibility` to `public` (everybody can use), `protected` (subclasses can use) or `private` (only the class), e.g.
```typescript
class Greeter {
  public greet() {
    console.log("hi!");
  }
}
const g = new Greeter();
g.greet();
```
Beware! Like other aspects of TypeScript’s type system, `private` and `protected` are only enforced during type checking.
This means that JavaScript runtime constructs like in or simple property lookup can still access a private or protected member.
Moreover, `private` allows access using bracket notation during type checking. This makes private-declared fields potentially easier to access for things like unit tests, with the drawback that these fields are `soft private` and don’t strictly enforce privacy. Unlike TypeScripts’s private, JavaScript’s private fields (#) remain private after compilation and do not provide the previously mentioned escape hatches like bracket notation access, making them `hard private`.
14. Classes can have `static` fields and methods. Watch out! Some names are reserved, e.g.
```typescript
class S {
  static name = "S!"; //error
  //Static property 'name' conflicts with built-in property 'Function.name' of constructor function 'S'.
}
```
15.  Classes can be generic, however, fields in generaic class cannot be generic (remember that code is compiled into js!)
```typescript
class Box<Type> {
  static defaultValue: Type;
  //Static members cannot reference class type parameters.
}
```
16. You can use `abstract` class with `abstract methods` (methods which are not implmented in the class but are in the derived ones). You cannot create an `abstract` class using `new`.
17. Relationships Between Classes. In most cases, classes in TypeScript are compared structurally, the same as other types.
For example, these two classes can be used in place of each other because they’re identical:
```typescript
class Point1 {
  x = 0;
  y = 0;
}
 
class Point2 {
  x = 0;
  y = 0;
}
 
// OK
const p: Point1 = new Point2();
```
Similarly, subtype relationships between classes exist even if there’s no explicit inheritance:
```typescript
class Person {
  name: string;
  age: number;
}
 
class Employee {
  name: string;
  age: number;
  salary: number;
}
 
// OK
const p: Person = new Employee();
```

Note that `empty classes` have no members. In a structural type system, a type with no members is generally a supertype of anything else. So if you write an empty class (don’t!), anything can be used in place of it. 
```typescript
class Empty {}
 
function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}
 
// All OK!
fn(window);
fn({});
fn(fn);
```

# Modules
In TypeScript, just as in ECMAScript 2015, any file containing a top-level import or export is considered a module.
```typescript
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
export interface Dog {
  breeds: string[];
  yearOfBirth: number;
}

// @filename: maths.ts
export const pi = 3.14;
export default class RandomNumberGenerator {}
 
// @filename: app.ts
import type { Cat, Dog } from "./animal.js";
// or import { type Cat, type Dog } from "./animal.js";


import RandomNumberGenerator, { pi as π } from "./maths.js";
 
RandomNumberGenerator;
         
(alias) class RandomNumberGenerator
import RandomNumberGenerator
 
console.log(π);
```
# Namespaces
 You can use namespace to better organize you code. E.g. 
 ```typescript
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
  const lettersRegexp = /^[A-Za-z]+$/;
  const numberRegexp = /^[0-9]+$/;
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s);
    }
  }
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}
// Some samples to try
let strings = ["Hello", "98052", "101"];
// Validators to use
let validators: { [s: string]: Validation.StringValidator } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();
// Show whether each string passed each validator
for (let s of strings) {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? "matches" : "does not match"
      } ${name}`
    );
  }
}
```
You can split namespaces across multiple files. But then you must use `--outFile` flag when compiling: `tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts` to put all input files into one js file.
