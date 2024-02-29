t# Problems with pure js

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
In this case ts guesses that `a:string = "string"`. 
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
