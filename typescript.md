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

### Installing

Execute in bash `sudo apt install node-typescript`.
