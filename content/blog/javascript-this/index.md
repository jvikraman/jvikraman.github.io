---
title: "🧗 Getting a grip on Javascript's `this` keyword"
date: "2020-08-05T15:19:00.000Z"
description: "Let's look in to the intricacies of javascript's `this` keyword."
---

### A few mental models on the intricacies around javascript's _this_ keyword.

### Javascript's _this_ context

The javascript `js÷this` keyword behaves _differently in different contexts_. It's _not the same in browser vs node environments_. For e.g. when used outside of a function, _this_ points to the _global_ object - in browser environments, that would be the _window_ object.

```js {2,7,10-11}
// In `browser` environment.
this === window         // true (both in strict & non-strict mode)

// In `node` environment.

// Inside node's repl.
this === global         // true (both in strict & non-strict mode)

// Outside node's repl.
this === global         // false 
this === module.exports // true
```

### _this_ in function calls

In _plain functions_, the _this_ value is determined by _how the function is called_. For e.g. In _non-strict mode_, the _this_ value will point to the _global_ object. However, in _strict mode_, the _this_ value will be _undefined_.

```js {4,13-14}
// File1.js
// Function in non-strict mode.
function firstFunc() {
  console.log(this === global);     // true
}

firstFunc();

// File2.js
// Function in strict mode.
"use strict";
function secondFunc() {
  console.log(this === global);     // false
  console.log(this === undefined);  // true
}

secondFunc();
```
In the below example, since the function is under _non-strict mode_ the _this_ value will be pointing to the _global_ object and the _names get added_ to the _global_ object.

```js {9}
// Function in non-strict mode.
function Person(first, last) {
  this.first = first;
  this.last = last;
}

// Calling the function directly without 
// using the `new` keyword.
const person = Person("John", "Doe");
console.log(person);          // undefined
console.log(global.first);    // John
console.log(global.last);     // Doe
```

Similarly, under _strict mode_ the program will _error out_ indicating `js÷Cannot set property 'first' of undefined!`, the reason being _this_ value set to _undefined_.

So, the _correct way_ to call the _Person_ function is to _construct_ it with a `js÷new` keyword like below:

```js {2}
// ...
const person = new Person("John", "Doe");
console.log(person);    // Person { first: 'John', last: 'Doe' }
```

### _this_ in method calls

When a _method_ is called on an _object_, the _this_ value is _set_ to the _object_ the method is called on. That _object_ is also known as the _intented receiver_ of the method. For e.g.:

```js {10}
const person = {
  first: "John",
  sayHello() {
    console.log(`Hello! my name is ${this.first}.`);
  }
}

// Here, the `person` object is the intented
// receiver of the `sayHello` method.
person.sayHello();    // Hello! my name is John.
```

This still works even if the _sayHello_ method is _defined separately_ and _later attached_ to the _person_ object like below:

```js {numberLines:true} {1-3,9-10}
function sayHello() {
  console.log(`Hello! my name is ${this.first}.`);
}

const person = {
  first: "John"
}

person.sayHello = sayHello;
person.sayHello();          // Hello! my name is John.

// In the below nested property chain, the `person` object
// is still the `intented receiver` of the method.
foo.baz.person.sayHello();
```

**What it means to _lose_ the intented receiver**: It's _common in functions to lose the intented receiver_. In the below example, under _non-strict mode_ if you store the _sayHello_ method's reference to a variable like _greet_ and then call _greet()_, the _this_ value points to the _global_ object and results in an output where the _variable first_ is _undefined_.

Under _strict mode_, the program _throws an error_ stating `js÷Cannot read property 'first' of undefined`.

```js {numberLines:true} {8-9}
const person = {
  first: "John",
  sayHello() {
    console.log(`Hello! my name is ${this.first}.`);
  }
}

const greet = person.sayHello;
greet();    // Hello! my name is undefined.
```

This is also _common in scenarios_ where you use the `js÷setTimeout()` function. For e.g.:

```js {numberLines:true} {5}
const person = {
  // ...
}

setTimeout(person.sayHello, 1000);   // Hello! my name is undefined.
```

**A few work-arounds to not losing the intented receiver:**

```js {numberLines:true} {7-9,12}
const person = {
  // ...
}

// Having the `person.sayHello()` function invoked as a method
// will prevent losing the receiver.
setTimeout(function() {
  person.sayHello();
}, 1000);

// Or, you could `bind` the `this` value to the `person` object.
setTimeout(person.sayHello.bind(person), 1000);
```

### specifying _this_ using .call() or .apply()

Both `js÷.call()` and `js÷.apply()` allow you to specify a custom _this_ value via their _first_ parameter the _thisArg_. In the below example, using _.call()_ and _.apply()_ you can call the _sayHello_ function _without attaching_ it to the _Person_ object.

```js {numberLines:true} {1-3,9,11}
function sayHello() {
  console.log(`Hello! my name is ${this.first}.`);
}

const person = {
  first: "John"
}

sayHello.call(person);    // Hello! my name is John.

sayHello.apply(person);   // Hello! my name is John.
```

**One easy way to remember the diff. b/w _.call()_ & _.apply()_**: _**(c)**all_ accepts a _**(c)**omma_ separated list of arguments, whereas _**(a)**pply_ accepts an _**(a)**rray_ of arguments.

```js {numberLines:true} {3,7,10}
const names = ['John', 'Sally', 'Megan', 'Blake', 'Trevor'];

const firstSlice = names.slice(1, 4);

// The above line is just syntactic sugar for `.call()`
// with arguments shown below.
const secondSlice = names.slice.call(names, 1, 4);

// This can also be written using `.apply()` like so.
const thirdSlice = names.slice.apply(names, [1, 4]);

console.log(firstSlice);    // [ 'Sally', 'Megan', 'Blake' ]
console.log(secondSlice);   // [ 'Sally', 'Megan', 'Blake' ]
console.log(thirdSlice);    // [ 'Sally', 'Megan', 'Blake' ]
```

**Caveat**: 

- In _non-strict mode_, if you _pass a value like null or undefined_ in either _.call()_ or _.apply()_, the javascript engine will _ignore those values_ and will set the _thisArg_ to the _global_ object
- To the _contrary_, in _strict mode_, the JS engine will respect those values

```js {numberLines:true} {3,6-7,9-10}
// Function in non-strict mode!
function myFunc() {
  console.log(this === global);   // true
}

myFunc.call(null);
myFunc.call(undefined);

myFunc.apply(null);
myFunc.apply(undefined);
```

### Hard-binding _this_ using _.bind()_ method

You can _hard-bind_ a function's _this_ value (or) _create a bound function_ using the `js÷.bind()` method.

```js {numberLines:true} {10-11,19}
const person = {
  first: "John"
  sayHello() {
    console.log(`Hello! my name is ${this.first}.`);
  }
}

// Hard-bind the `sayHello()` method to `person` object
// using the `.bind()` method.
const greet = person.sayHello.bind(person);
greet();                      // Hello! my name is John.

// Once the above function is bound, it cannot change
// even with a `.call()` or `.apply()` method.
const anotherPerson = {
  first: "Jane"
}

greet.call(anotherPerson);    // Hello! my name is John.
```

### _this_ inside arrow functions

Inside _arrow functions_, the _this_ context is decided by the _function's execution context_. In the below example, the _outerThis_ context and the arrow function's inner _this_ context are same.

```js {numberLines:true} {5}
// Capture the global this context.
const outerThis = this;

const myFunc = () => {
  console.log(this === outerThis);    // true
}

myFunc();
```

In the below example, the _output_ will be _NaN_ every second. This is due to the _behavior_ of `js÷setInterval()` function. By default, the _this_ context inside this function is set to the _global_ object thereby resulting in _NaN_.

```js {numberLines:true} {4,10}
const counter = {
  count: 0,
  increment() {
    setInterval(function() {
      console.log(++this.count);
    }, 1000);
  }
}

counter.increment();    // Output: NaN NaN NaN NaN NaN etc.
```

To _get around this behavior_, you can _use an arrow function_ inside the _setInterval_ function like below and the _this_ context will now _correctly point_ to the _counter variable_.

```js {numberLines:true} {4,10}
const counter = {
  count: 0,
  increment() {
    setInterval(() => {
      console.log(++this.count);
    }, 1000);
  }
}

counter.increment();    // Output: 1 2 3 4 5 etc.
```

### _this_ within classes

Since _class bodies_ are by _default_ in an _implicit strict mode_, if you _assign_ a class method to a _variable_ and then _call_ that variable, the _intented receiver is lost as usual_. 

```js {numberLines:true} {13,18}
// By default, class bodies are in implicit strict mode!
class Person {
  constructor(first) {
    this.first = first;
  }

  sayHello() {
    console.log(`Hello! my name is ${this.first}.`);
  }
}

const person = new Person("John");
person.sayHello();    // Hello! my name is John.

// Error: Cannot read property 'first' of undefined!
// due to class body being in implicit strict mode.
const greet = person.sayHello;
greet();              
```

You can try the _following options_ as shown in the below example to _get around this behavior_:

- _Hard-bind_ the _person_ object to the _sayHello()_ method while _assigning_ to the _greet_ variable
- A _second variation_ is to _bind_ the method _within the constructor_
- Alternatively, you can _also use_ the _class fields feature_

```js {numberLines:true} {8,17-20,27}
// By default, class bodies are in implicit strict mode!
class Person {  
  constructor(first) {
    this.first = first;

    // Another variation of the hard-binding is to bind
    // within the constructor like so.
    this.sayHello = this.sayHello.bind(this);
  }

  // Normal class method.
  sayHello() {
    console.log(`Hello! my name is ${this.first}.`);
  }

  // Another variation is to use the class fields feature like so.
  sayHello = () => {
    console.log(`Hello! my name is ${this.first}.`);
  }
}

const person = new Person("John");
person.sayHello();    // Hello! my name is John.

// Hard-binding the `person` object to the `sayHello` method
// is one option to prevent losing the receiver.
const greet = person.sayHello.bind(person);
greet();              
```

And _that's it for this blog on javascript's this keyword_. These notes are _mental models_ from [Marius Schulz's](https://mariusschulz.com/) amazing [egghead course on this topic](https://egghead.io/courses/understand-javascript-s-this-keyword-in-depth).

