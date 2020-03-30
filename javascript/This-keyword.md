# The `this` keyword

- See:

   - [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)
   - [The Complete JavaScript Course](https://www.udemy.com/course/the-complete-javascript-course/)

## Basic idea

- Every function has access to a `this` reference
- The `this` reference is assigned a value when creating the *execution context* for an execution of the function (see [Scope and closures](./Scope-closures.md))
   - Note that this means that a function's  `this` can point to different things based on how it's executed!
   - Run-time binding, not write-time binding! In that sense, it's a bit the opposite of lexical scoping (see [Scope and closures](./Scope-closures.md))
- What the `this` reference points to is determined by a number of rules

## Rules for binding `this`

### Default binding

This is the most basic case, where the function is directly called without doing anything special with it

Example:

```javascript
function logA() {
    console.log(this.a);
}

var a = "test";

logA();
```

What `this` is bound to depends on *strict mode*:

- If *strict mode* is not enabled, the function's `this` points to the global object. Since the global object has a variable `a` with value `test`, the code will print `test`
- If *strict mode* is enabled, the function's `this` is not allowed to point to the global object. Instead, `this` points to `undefined`, and the code throws a `TypeError` because we tried to access `undefined.a`

### Implicit binding

*Implicit binding* occurs when the function is called through an object that holds a reference to it. In this case, that object is called the *context object* and the function's `this` points to the object.

Example:

```javascript
function logA() {
    console.log(this.a);
}

const theObject = {
    a: "test",
    logA: logA
};

const secondObject = {
    a: "second",
    doSomething: logA
};

theObject.logA(); // test
secondObject.doSomething(); // second
```

One tricky thing: it's easy to lose that implicit binding!

```javascript
function logA() {
    console.log(this.a);
}

const theObject = {
    a: "test",
    logA: logA
};

const theFunction = theObject.logA; // theFunction now just points to logA
theFunction(); // logs undefined (non-strict mode) or throws TypeError (strict mode)
```

As we see above, assigning the function to a variable and then calling it doesn't use *implicit binding* anymore. Instead, we fall back to *default binding*.

One case where we need to be careful with that is when passing functions around as callbacks. Example:

```javascript
function logA() {
    console.log(this.a);
}

const theObject = {
    a: "test",
    logA: logA
};

// here, we assign the function to an argument and setTimeout calls it later
// logs undefined (non-strict mode) or throws TypeError (strict mode)
setTimeout(theObject.logA, 1000);

// here, we make sure to call logA through theObject
// logs 'test'
setTimeout(function () {
    theObject.logA();
}, 1000);
```

### Explicit binding

As the name suggests, *explicit binding* is more explicit about what the function's `this` reference is bound to

Example:

```javascript
function logA() {
    console.log(this.a);
}

const theObject = {
    a: "test",
    logA: logA
};

// call and apply differ in how to specify the function's arguments (not relevant here)
logA.call(theObject); // test
logA.apply(theObject); // test

const theFunction = theObject.logA;
theFunction(); // logs undefined (non-strict mode) or throws TypeError (strict mode)
const boundFunction = theObject.logA.bind(theObject);
boundFunction(); // test

setTimeout(theObject.logA.bind(theObject), 1000); // test
```

Some things to notice:

- `call` and `apply` can be used to call a function while explicitly specifying what `this` should point to. You can use these to dynamically call a function with any `this` reference you like.
- `bind` returns a new function that is identical the original function except that its `this` is hard-wired to the argument you passed to `bind`. One use case is passing the function as a callback (see example code above)

Other use case: *monkey patching* (extending the behavior of a function defined on an existing object):

```javascript
const existingObject = {
    a: "test",
    logA: function () {
        console.log(this.a);
    }
};

const oldFunction = existingObject.logA;

existingObject.logA = function () {
    console.log("before");
    oldFunction.call(this);
    console.log("after");
}

existingObject.logA(); // before, test, after
```

### `new` binding

The `new` binding occurs when using the keyword `new` to call a function. Invoking a function using the `new` keyword has the following effect:

- A new object is created
- The new object's `__proto__` points to the function (see [Object prototypes and classes](./Object-prototypes-classes.md))
- The new object is set as the `this` binding for that call to the function
- Unless the function returns something itself, the function call will automatically return the new object

Important: `new` does not require any specific kind of function to be called on! It can be used on any function, and when it does, it modifies the function's behavior according to the above. In this case, the function is sometimes called a *constructor*, but it is still just a regular function. The only thing making it behave differently is the `new` keyword.

Example:

```javascript
// the PascalCase name is a convention for function intended to be called with new
// JavaScript itself doesn't care about this
function Test(a) {
    this.a = a;
}

const one = new Test("one");
const two = new Test("two");

console.log(one.a); // one
console.log(two.a); // two
```

## Priority of the rules

From highest to lowest:

- `new`  binding (overrides all the rest)
- Explicit binding
- Implicit binding
- Default binding

## Lexical this

One exception to the behavior of `this` inside a function occurs when using *arrow functions*. Arrow function's don't follow the rules above. Instead, the `this` inside an arrow function points to the `this` of the surrounding scope.

One use case: calling other function on same object from a callback

```javascript
const theObject = {
    testA: function () {
        console.log("testA");

        // naive approach, ends up using default binding
        setTimeout(function () {
            this.testB(); // TypeError: this.testB is not a function
        }, 1000);

        // solution without arrow functions
        const self = this;

        setTimeout(function () {
            self.testB(); // test
        }, 2000);

        // solution with arrow functions
        setTimeout(() => {
            this.testB();
        }, 3000);
    },
    testB: function () {
        console.log("testB");
    }
};

theObject.testA();
```

Also note that, because of the fact that arrow functions don't have their own `this`, JavaScript does not allow you to call them using the `new` keyword:

```javascript
const Test = () => {
    this.a = "test";
}

const theObject = new Test(); // TypeError
```