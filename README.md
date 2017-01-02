# *JavaScript - Understanding The Weird Parts*
My JavaScript notes. Still editing...

## Conceptual Aside
* **Syntax parser** <br/>
A program that reads your code and determines what it does and if its grammar is valid.
* **Lexical environment** <br/>
Where something sits physically in the code you write.
* **Scope** <br/>
Where a variable is available in your code.
* **Execution context** <br/>
A wrapper to help manage the code that is running. <br/>
There are lots of lexical environments, and which one is currently running is managed via execution contexts. Every line of JavaScript code is run in an “execution context.” The JavaScript runtime environment maintains a stack of these contexts, and the top execution context on this stack is the one that’s actively running.
* **Executable code** <br/>
There are three types of executable code: global code, function code, and eval code. Global code is code at the top level of your program that’s not inside any functions, function code is code that’s inside the body of a function, and eval code is global code evaluated by a call to eval.
* **Object** <br/>
A collection of name value pairs.
* **Invocation** <br/>
Running a function.
* **Synchronous** <br/>
One at a time.
* **Asynchronous** <br/>
More than one at a time.
* **Dynamic typing** <br/>
In JavaScript, variables can hold different types of values because it's all figured out during execution.
* **Primitive type** <br/>
A type of data that represents a single value. There are 6 primitive types in JavaScript: undefined, null, boolean, number, string, symbol (used in ES6).
Both *undefined* and *null* represent lack of existence, but never set a varibale to *undefined*, set it to *null*, leave *undefined* to the engine.


## The global object
All JavaScript runtimes have a unique object called the global object. Its properties include built-in objects like Math and String, as well as extra properties provided by the host environment. In browsers, the global object is the window object. In Node.js, it’s just called the “global object”. 

## Context vs. Scope
Every function invocation has both a scope and a context associated with it. Fundamentally, scope is function-based while context is object-based. In other words, scope pertains to the variable access of a function when it is invoked and is unique to each invocation. Context is always the value of the this keyword which is a reference to the object that “owns” the currently executing code.

## Variable Scope
A variable can be defined in either local or global scope, which establishes the variables’ accessibility from different scopes during runtime. Any defined global variable, meaning any variable declared outside of a function body will live throughout runtime and can be accessed and altered in any scope. Local variables exist only within the function body of which they are defined and will have a different scope for every call of that function. There it is subject for value assignment, retrieval, and manipulation only within that call and is not accessible outside of that scope.

## "this" Context
Context is most often determined by how a function is invoked. When a function is called as a method of an object, *this* is set to the object the method is called on:
```javascript
var obj = {
    foo: function() {
        return this;   
    }
};

obj.foo() === obj; // true
```
The same principle applies when invoking a function with the *new* operator to create an instance of an object. When invoked in *this* manner, the value of this within the scope of the function will be set to the newly created instance:
```javascript
function foo() {
    alert(this);
}

foo() // window
new foo() // foo
```

## Execution Context
JavaScript is a single threaded language, meaning only one task (command) can be executed at a time. When the JavaScript interpreter initially executes code, it first enters into a global execution context by default. Each invocation of a function from this point on will result in the creation of a new execution context.

This is where confusion often sets in, the term “execution context” is actually for all intents and purposes referring more to scope and not context as previously discussed. It is an unfortunate naming convention, however it is the terminology as defined by the ECMAScript specification, so we’re kinda stuck with it.

Each time a new execution context is created it is appended to the top of the execution stack. The browser will always execute the current execution context that is atop the execution stack. Once completed, it will be removed from the top of the stack and control will return to the execution context below.

An execution context can be divided into a creation and execution phase. In the creation phase, the interpreter will first create a variable object (also called an activation object) that is composed of all the variables, function declarations, and arguments defined inside the execution context. From there the scope chain is initialized next, and the value of this is determined last. Then in the execution phase, code is interpreted and executed.

## The Scope Chain
For each execution context there is a scope chain coupled with it. The scope chain contains the variable object for every execution context in the execution stack. It is used for determining variable access and identifier resolution. For example:
```javascript
function first() {
    second();
    function second() {
        third();
        function third() {
            fourth();
            function fourth() {
                // do something
            }
        }
    }   
}
first();
```
Running the preceding code will result in the nested functions being executed all the way down to the *fourth* function. At this point the scope chain would be, from top to bottom: fourth, third, second, first, global. The *fourth* function would have access to global variables and any variables defined within the *firs*t, *second*, and *third* functions as well as the functions themselves.

Name conflicts amongst variables between different execution contexts are resolved by climbing up the scope chain, moving locally to globally. This means that local variables with the same name as variables higher up the scope chain take precedence.

To put it simply, each time you attempt to access a variable within a function’s execution context, the look-up process will always begin with its own variable object. If the identifier is not found in the variable object, the search continues into the scope chain. It will climb up the scope chain examining the variable object of every execution context looking for a match to the variable name. For example:
```javascript
function b() {
    console.log(myVar);
};
function a() {
    var myVar = 2;
    b();
};
var myVar = 1;
a();
// output is 1
```

```javascript
function a() {
    var myVar = 2;
    b();
    function b() {
        console.log(myVar);
    };
};
var myVar = 1;
a();
// output is 2
```
## Closures
Accessing variables outside of the immediate lexical scope creates a closure. In other words, a closure is formed when a nested function is defined inside of another function, allowing access to the outer functions variables. Returning the nested function allows you to maintain access to the local variables, arguments, and inner function declarations of its outer function. This encapsulation allows us to hide and preserve the execution context from outside scopes while exposing a public interface and thus is subject to further manipulation. A simple example of this looks like the following:
```javascript
function foo() {
    var localVariable = 'private variable';
    return function() {
        return localVariable;
    }
}

var getLocalVariable = foo();
getLocalVariable() // "private variable"
```
One of the most popular types of closures is what is widely known as the module pattern; it allows you to emulate public, private, and privileged members:
```javascript
var Module = (function() {
    var privateProperty = 'foo';

    function privateMethod(args) {
        // do something
    }

    return {

        publicProperty: '',

        publicMethod: function(args) {
            // do something
        },

        privilegedMethod: function(args) {
            return privateMethod(args);
        }
    };
})();
```
The module acts as if it were a singleton, executed as soon as the compiler interprets it, hence the opening and closing parenthesis at the end of the function. The only available members outside of the execution context of the closure are your public methods and properties located in the return object (*Module.publicMethod* for example). However, all private properties and methods will live throughout the life of the application as the execution context is preserved, meaning variables are subject to further interaction via the public methods.

Another type of closure is what is called an immediately-invoked function expression (IIFE) which is nothing more than a self-invoked anonymous function executed in the context of the window:
```javascript
(function(window) {
          
    var foo, bar;

    function private() {
        // do something
    }

    window.Module = {

        public: function() {
            // do something 
        }
    };

})(this);
```
This expression is most useful when attempting to preserve the global namespace as any variables declared within the function body will be local to the closure but will still live throughout runtime. This is a popular means of encapsulating source code for applications and frameworks, typically exposing a single global interface in which to interact with.


## Hoisting
Hoisting is JavaScript's default behavior of moving declarations to the top. It's WRONG to say that variable and function declarations are physically moved to the top your coding. What happens is the variable and function declarations are put into memory during the compile phase, but stays exactly where you typed it in your coding.  

#### JavaScript Declarations are Hoisted
In Javascript, a variable can be declared after it has been used.
```javascript
x = 5; // assign 5 to x
console.log(x);
var x; // declare x
```
Declaration of x is hoisted and set (variable setup) equal to 'undefined'. 

#### Initializations are Not Hoisted
JavaScript only hoists declarations, not initializations.
```javascript
var x = 5; // initialized x
y = 7; // assign 7 to y
console.log(x+y); // 12
var y; // declare y
```
```javascript
var x = 5; // initialized x
console.log(x+y); // NaN
var y = 7; // initialized y
```

## Scoping and Hoisting
Scoping is quite confusing in JavaScript. Consider the following C program:
```c
#include <stdio.h>
int main() {
  int x = 1;
	printf("%d, ", x); // 1
	if (1) {
		int x = 2;
		printf("%d, ", x); // 2
	}
	printf("%d\n", x); // 1
}
```

Because C has block-level scope. However, JavaScript has function-level scope.
```javascript
var x = 1;
console.log(x); // 1
if (true) {
	var x = 2;
	console.log(x); // 2
}
console.log(x); // 2
```

In JavaScript, blocks, such as if statements, do not create a new scope, only functions do. <br/>
If you must create temporary scopes within a function, do the following:
```javascript
var x = 1;
console.log(x); // 1
if (x) {
	(function () {
		var x = 2;
		console.log(x); // 2
	}());
}
console.log(x); // 1
```

## "Let"
The following two examples demostrate *let* which uses block-scoping:
```javascript
var a = 1;
var b = 2;
if (a < b) {
    var c = true;
};
console.log(c); // true
```
```javascript
var a = 1;
var b = 2;
if (a < b) {
    let c = true;
};
console.log(c); // c is not defined
```

## Event Queue
Aonther list that sits on JavaScript engine besides execution context. When the execution stack is empty, then JavaScript priority looks at event queue and looks for something to be there. If something is there, it looks the see if a particular function should run when the event is triggered. For example, if click event is on event queue and when execution stack is empty, *clickHandler()* is added to the stack at the event of click.
```javascript
function wait() {
    var ms = 3000 + new Date().getTime();
    while (new Date() < ms) {}
    console.log('finished function')
}

function clickHandler() {
    console.log('click event!');
}

document.addEventListener('click', clickHandler);

wait();
console.log('finish execution');
```
