### 1. Explain event delegation

Event delegation is a technique involving adding event listeners to a parent element instead of adding them to the descendant elements. The listener will fire whenever the event is triggered on the descendant elements due to event bubbling up the DOM. The benefits of this technique are:

- Memory footprint goes down because only one single handler is needed on the parent element, rather than having to attach event handlers on each descendant.
- There is no need to unbind the handler from elements that are removed and to bind the event for new elements.

### 2. Explain how `this` works in JavaScript

The value of `this` varies based on how the function is called.

1. If the new keyword is used when calling the function, this inside the function is a brand new object.
2. If apply, call, or bind are used to call/create a function, this inside the function is the object that is passed in as the argument.
3. If a function is called as a method, such as obj.method() — this is the object that the function is a property of.
4. If a function is invoked as a free function invocation, meaning it was invoked without any of the conditions present above, this is the global object. In a browser, it is the window object. If in strict mode ('use strict'), this will be undefined instead of the global object.
5. If multiple of the above rules apply, the rule that is higher wins and will set the this value.
6. If the function is an ES2015 arrow function, it ignores all the rules above and receives the this value of its surrounding scope at the time it is created.

### 3. Can you give an example of one of the ways that working with `this` has changed in ES6?

ES6 allows you to use arrow functions which uses the enclosing lexical scope (used to explain how variables and functions declared in an outer scope are accessible in an inner scope). This is usually convenient, but does prevent the caller from controlling context via .call or .apply—the consequences being that a library such as jQuery will not properly bind this in your event handler functions. Thus, it's important to keep this in mind when refactoring large legacy applications.

### 4. Explain how prototypal inheritance works

All JavaScript objects have a **proto** property with the exception of objects created with Object.create(null), that is a reference to another object, which is called the object's "prototype". When a property is accessed on an object and if the property is not found on that object, the JavaScript engine looks at the object's **proto**, and the **proto**'s **proto** and so on, until it finds the property defined on one of the **proto**s or until it reaches the end of the prototype chain. This behavior simulates classical inheritance, but it is really more of delegation than inheritance.

Example:

```
// Parent object constructor.
function Animal(name) {
  this.name = name;
}

// Add a method to the parent object's prototype.
Animal.prototype.makeSound = function () {
  console.log('The ' + this.constructor.name + ' makes a sound.');
};

// Child object constructor.
function Dog(name) {
  Animal.call(this, name); // Call the parent constructor.
}

// Set the child object's prototype to be the parent's prototype.
Object.setPrototypeOf(Dog.prototype, Animal.prototype);

// Add a method to the child object's prototype.
Dog.prototype.bark = function () {
  console.log('Woof!');
};

// Create a new instance of Dog.
const bolt = new Dog('Bolt');

// Call methods on the child object.
console.log(bolt.name); // "Bolt"
bolt.makeSound(); // "The Dog makes a sound."
bolt.bark(); // "Woof!"
```

### 5. Explain function borrowing

Function borrowing allows us to use the methods of one object on a different object without having to make a copy of that method and maintain it in two separate places. It is accomplished through the use of .call(), .apply(), or .bind(), all of which exist to explicitly set this on the method we are borrowing.

```
function findO() {
  var args = Array.prototype.slice.call(arguments)
  return args.filter(a => a.includes('o'))
}

findO("orchid", "tulip", "rose", "lilac")
=> [ 'orchid', 'rose' ]
```

### 6. What do you think of AMD vs CommonJS

Both are ways to implement a module system, which was not natively present in JavaScript until ES2015 came along. CommonJS is synchronous while AMD (Asynchronous Module Definition) is obviously asynchronous. CommonJS is designed with server-side development in mind while AMD, with its support for asynchronous loading of modules, is more intended for browsers.

I find AMD syntax to be quite verbose and CommonJS is closer to the style you would write import statements in other languages. Most of the time, I find AMD unnecessary, because if you served all your JavaScript into one concatenated bundle file, you wouldn't benefit from the async loading properties. Also, CommonJS syntax is closer to Node style of writing modules and there is less context-switching overhead when switching between client side and server side JavaScript development.

I'm glad that with ES2015 modules, that has support for both synchronous and asynchronous loading, we can finally just stick to one approach. Although it hasn't been fully rolled out in browsers and in Node, we can always use transpilers to convert our code.

### 7. Explain why the following doesn't work as an IIFE: function foo(){ }();. What needs to be changed to properly make it an IIFE?

IIFE stands for Immediately Invoked Function Expressions. The JavaScript parser reads function foo(){ }(); as function foo(){ } and ();, where the former is a function declaration and the latter (a pair of parentheses) is an attempt at calling a function but there is no name specified, hence it throws Uncaught SyntaxError: Unexpected token .

Here are two ways to fix it that involves adding more parentheses: (function foo(){ })() and (function foo(){ }()). Statements that begin with function are considered to be function declarations; by wrapping this function within (), it becomes a function expression which can then be executed with the subsequent (). These functions are not exposed in the global scope and you can even omit its name if you do not need to reference itself within the body.

You might also use void operator: void function foo(){ }();. Unfortunately, there is one issue with such approach. The evaluation of given expression is always undefined, so if your IIFE function returns anything, you can't use it. An example:

```
const foo = void (function bar() {
  return 'foo';
})();

console.log(foo); // undefined
```

### 8. What's the difference between a variable that is: null, undefined or undeclared? How would you go about checking for any of these states?

Undeclared variables are created when you assign a value to an identifier that is not previously created using var, let or const. Undeclared variables will be defined globally, outside of the current scope. In strict mode, a ReferenceError will be thrown when you try to assign to an undeclared variable. Undeclared variables are bad just like how global variables are bad. Avoid them at all cost! To check for them, wrap its usage in a try/catch block.

```
function foo() {
  x = 1; // Throws a ReferenceError in strict mode
}

foo();
console.log(x); // 1
```

A variable that is undefined is a variable that has been declared, but not assigned a value. It is of type undefined. If a function does not return any value as the result of executing it is assigned to a variable, the variable also has the value of undefined. To check for it, compare using the strict equality (===) operator or typeof which will give the 'undefined' string. Note that you should not be using the abstract equality operator to check, as it will also return true if the value is null.

```
var foo;
console.log(foo); // undefined
console.log(foo === undefined); // true
console.log(typeof foo === 'undefined'); // true

console.log(foo == null); // true. Wrong, don't use this to check!

function bar() {}
var baz = bar();
console.log(baz); // undefined
```

A variable that is null will have been explicitly assigned to the null value. It represents no value and is different from undefined in the sense that it has been explicitly assigned. To check for null, simply compare using the strict equality operator. Note that like the above, you should not be using the abstract equality operator (==) to check, as it will also return true if the value is undefined.

```
var foo = null;
console.log(foo === null); // true
console.log(typeof foo === 'object'); // true

console.log(foo == undefined); // true. Wrong, don't use this to check!
```

### 9. What is a closure, and how/why would you use one?

A closure is the combination of a function and the lexical environment within which that function was declared. The word "lexical" refers to the fact that lexical scoping uses the location where a variable is declared within the source code to determine where that variable is available. Closures are functions that have access to the outer (enclosing) function's variables—scope chain even after the outer function has returned.
