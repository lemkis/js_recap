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

If our class does not have a field we would like to be notified of this. However, js returns `undefined` instead.
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

Save 
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
Remember to always use `es2015` target unless you have very good reason not to (e.g. compability with older browsers). 
Remember that type annotation `never` change the runtime behaviour of your program!


# Strictness flags

In `tsconfing.json` file you can set how strict ts type checking should be. E.g. you can as strict as possible `"strict": true`. Otherwise you can choose what ts should check for you by launching some options e.g. `"noImplicitAny": true` or `"strictNullChecks": true`. 

## noImplicitAny
Sometimes ts does not infer type is lenient as assigns type `any` (plain js type). This flag notifies you about such cases.

## strictNullChecks

By default, values like `null` or `undefined` are assignable to any other type. This flag makes handling these cases more explicit (in case we forgot to handle them).

# Types
`string` ("...") , `number` (int or float), `boolean` (true or false), `array` (e.g. number[], Array&lt;number&gt;, [1,2,3]), `any` (no type checking)

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
'obj.last' is possibly 'undefined'.
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
 
  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
```

## Union type

The allow you to combine types e.g. `number | string`. You can handle each type separately (`narrowing`) using  `typeof` or `Array.isArray`.

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

 // Error: Duplicate identifier 'Window'.

        
```
