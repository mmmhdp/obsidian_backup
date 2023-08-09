- Writing into an HTML element, using `innerHTML`.
- Writing into the HTML output using `document.write()`.
- Writing into an alert box, using `window.alert()`.
- Writing into the browser console, using `console.log()`.


<p id="demo"></p>  
  
<script>  
document.getElementById("demo").innerHTML = 5 + 6;  
</script><p id="demo"></p>  

Using document.write() after an HTML document is loaded, will **delete all existing HTML**:
<script>  
document.write(5 + 6);  
</script>

<script>  
window.alert(5 + 6);  
</script>

<button onclick="window.print()">Print this page</button>\

|Keyword|Description|
|---|---|
|var|Declares a variable|
|let|Declares a block variable|
|const|Declares a block constant|
|if|Marks a block of statements to be executed on a condition|
|switch|Marks a block of statements to be executed in different cases|
|for|Marks a block of statements to be executed in a loop|
|function|Declares a function|
|return|Exits a function|
|try|Implements error handling to a block of statements|

The `var` keyword was used in all JavaScript code from 1995 to 2015.

The `let` and `const` keywords were added to JavaScript in 2015.

The `var` keyword should only be used in code written for older browsers.

In computer programs, variables are often declared without a value. The value can be something that has to be calculated, or something that will be provided later, like user input.

A variable declared without a value will have the value `undefined`.

The keyword `const` is a little misleading.

It does not define a constant value. It defines a constant reference to a value.

Because of this you can NOT:

- Reassign a constant value
- Reassign a constant array
- Reassign a constant object

But you CAN:

- Change the elements of constant array
- Change the properties of constant object

You can use the JavaScript `typeof` operator to find the type of a JavaScript variable.

The `typeof` operator returns the type of a variable or an expression:
typeof ""             // Returns "string"  
typeof "John"         // Returns "string"

It is a common practice to declare objects with the const keyword.
const car = {type:"Fiat", model:"500", color:"white"};

## Objects 

const person = {  
  firstName: "John",  
  lastName : "Doe",  
  id       : 5566,  
  fullName : function() {  
    return this.firstName + " " + this.lastName;  
  }  
};

## Common HTML Events

Here is a list of some common HTML events:

|Event|Description|
|---|---|
|onchange|An HTML element has been changed|
|onclick|The user clicks an HTML element|
|onmouseover|The user moves the mouse over an HTML element|
|onmouseout|The user moves the mouse away from an HTML element|
|onkeydown|The user pushes a keyboard key|
|onload|The browser has finished loading the page|

FULL LIST OF EVENTS
https://www.w3schools.com/jsref/dom_obj_event.asp

**Template literals** allows multiline strings:
### Example

let text =  \`The quick \`;

## Interpolation

**Template literals** provide an easy way to interpolate variables and expressions into strings.

The method is called string interpolation.

The syntax is:

${...}

## Arrays

const _array_name_ = [_item1_, _item2_, ...]
const cars = new Array("Saab", "Volvo", "BMW");

// Create an array with one element:  
const points = [40];

// Create an array with 40 undefined elements:  
const points = new Array(40);

const numbers = [45, 4, 9, 16, 25];  
let txt = "";  
numbers.forEach(myFunction);  
  
function myFunction(value, index, array) {  
  txt += value + "<br>";  
}

## JavaScript Array map()

The `map()` method creates a new array by performing a function on each array element.

The `map()` method does not execute the function for array elements without values.

The `map()` method does not change the original array.

This example multiplies each array value by 2:

### Example

const numbers1 = [45, 4, 9, 16, 25];  
const numbers2 = numbers1.map(myFunction);  
  
function myFunction(value, index, array) {  
  return value * 2;  
}

## Conditional (Ternary) Operator

JavaScript also contains a conditional operator that assigns a value to a variable based on some condition.

### Syntax

_variablename_ = (_condition_) ? _value1_:_value2_

## Conditionals 

if (_condition1_) {  
  //  _block of code to be executed if condition1 is true_} else if (_condition2_) {  
  //  _block of code to be executed if the condition1 is false and condition2 is true_  
} else {  
  //  _block of code to be executed if the condition1 is false and condition2 is false_}

## Different Kinds of Loops

JavaScript supports different kinds of loops:

- `for` - loops through a block of code a number of times
- `for/in` - loops through the properties of an object
- `for/of` - loops through the values of an iterable object
- `while` - loops through a block of code while a specified condition is true
- `do/while` - also loops through a block of code while a specified condition is true

The `break` statement "jumps out" of a loop.

The `continue` statement "jumps over" one iteration in the loop.

## Essential Set Methods

|Method|Description|
|---|---|
|new Set()|Creates a new Set|
|add()|Adds a new element to the Set|
|delete()|Removes an element from a Set|
|has()|Returns true if a value exists in the Set|
|forEach()|Invokes a callback for each element in the Set|
|values()|Returns an iterator with all the values in a Set|
|Property|Description|
|size|Returns the number of elements in a Set|

---

## How to Create a Set

You can create a JavaScript Set by:

- Passing an Array to `new Set()`
- Create a new Set and use `add()` to add values
- Create a new Set and use `add()` to add variables

// Create a Set  
const letters = new Set();  
  
// Add Values to the Set  
letters.add("a");  
letters.add("b");  
letters.add("c");

## Essential Map Methods

|Method|Description|
|---|---|
|new Map()|Creates a new Map|
|set()|Sets the value for a key in a Map|
|get()|Gets the value for a key in a Map|
|delete()|Removes a Map element specified by the key|
|has()|Returns true if a key exists in a Map|
|forEach()|Calls a function for each key/value pair in a Map|
|entries()|Returns an iterator with the [key, value] pairs in a Map|
|Property|Description|
|size|Returns the number of elements in a Map|

---

## How to Create a Map

You can create a JavaScript Map by:

- Passing an Array to `new Map()`
- Create a Map and use `Map.set()`



const fruits = new Map([  
  ["apples", 500],  
  ["bananas", 300],  
  ["oranges", 200]  
]);

## Throw, and Try...Catch...Finally

The `try` statement defines a code block to run (to try).

The `catch` statement defines a code block to handle any error.

The `finally` statement defines a code block to run regardless of the result.

The `throw` statement defines a custom error.

<p id="demo"></p>  
  
<script>  
try {  
  adddlert("Welcome guest!");  
}  
catch(err) {  
  document.getElementById("demo").innerHTML = err.message;  
}  
</script>

## The finally Statement

The `finally` statement lets you execute code, after try and catch, regardless of the result:

### Syntax

try {  
  _Block of code to try  
_}  
catch(_err_) {  
  _Block of code to handle errors  
_}  
finally {  
  _Block of code to be executed regardless of the try / catch result  
_}

# JavaScript Arrow Function

Arrow functions allow us to write shorter function syntax:

let myFunction = (a, b) => a * b;
### Before Arrow:

hello = function() {  
  return "Hello World!";  
}

### With Arrow Function:

hello = () => {  
  return "Hello World!";  
}

It gets shorter! If the function has only one statement, and the statement returns a value, you can remove the brackets _and_ the `return` keyword:

### Arrow Functions Return Value by Default:

hello = () => "Hello World!";


Note: This works only if the function has only one statement.

If you have parameters, you pass them inside the parentheses:
Arrow Function With Parameters:
hello = (val) => "Hello " + val; 

## JavaScript Class Syntax

Use the keyword `class` to create a class.

Always add a method named `constructor()`:

### Syntax

class ClassName {  
  constructor() { ... }  
}

class Car {  
  constructor(name, year) {  
    this.name = name;  
    this.year = year;  
  }  
}

## Class Methods

Class methods are created with the same syntax as object methods.

Use the keyword `class` to create a class.

Always add a `constructor()` method.

Then add any number of methods.

### Syntax

class ClassName {  
  constructor() { ... }  
  method_1() { ... }  
  method_2() { ... }  
  method_3() { ... }  
}

## Modules

JavaScript modules allow you to break up your code into separate files.

This makes it easier to maintain a code-base.

Modules are imported from external files with the `import` statement.

Modules also rely on `type="module"` in the <script> tag.

### Example

<script type="module">  
import message from "./message.js";  
</script>


## Don't Use new Object()

- Use `""` instead of `new String()`
- Use `0` instead of `new Number()`
- Use `false` instead of `new Boolean()`
- Use `{}` instead of `new Object()`
- Use `[]` instead of `new Array()`
- Use `/()/` instead of `new RegExp()`
- Use `function (){}` instead of `new Function()`

### Example

let x1 = "";             // new primitive string  
let x2 = 0;              // new primitive number  
let x3 = false;          // new primitive boolean  
const x4 = {};           // new object  
const x5 = [];           // new array object  
const x6 = /()/;         // new regexp object  
const x7 = function(){}; // new function object

## Use === Comparison

The `==` comparison operator always converts (to matching types) before comparison.

The `===` operator forces comparison of values and type:

### Example

0 == "";        // true  
1 == "1";       // true  
1 == true;      // true  
  
0 === "";       // false  
1 === "1";      // false  
1 === true;     // false