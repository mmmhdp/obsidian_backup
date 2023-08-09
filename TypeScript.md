DATA TYPES
There are three main primitives in JavaScript and TypeScript.
- `boolean` - true or false values
- `number` - whole numbers and floating point values
- `string` - text values like "TypeScript Rocks"

There are also 2 less common primitives used in later versions of Javascript and TypeScript.

- `bigint` - whole numbers and floating point values, but allows larger negative and positive numbers than the `number` type.
- `symbol` are used to create a globally unique identifier

let firstName: string = "Dylan";
let firstName = "Dylan";

`any` is a type that disables type checking and effectively allows all types to be used.

`unknown` is a similar, but safer alternative to `any`.
TypeScript will prevent `unknown` types from being used.

`never` effectively throws an error whenever it is defined.

`undefined` and `null` are types that refer to the JavaScript primitives `undefined` and `null` respectively.

ARRAYS
const names: string[] = [];
names.push("Dylan");

The `readonly` keyword can prevent arrays from being changed.
const names: readonly string[] = ["Dylan"];

TypeScript can infer the type of an array if it has values.

A **tuple** is a typed [array](https://www.w3schools.com/js/js_arrays.asp) with a pre-defined length and types for each index.

Tuples are great because they allow each element in the array to be a known type of value.

// define our tuple  
let ourTuple: [number, boolean, string];  
  
// initialize correctly  
ourTuple = [5, false, 'Coding God was here'];

A good practice is to make your **tuple** `readonly`.

Tuples only have strongly defined types for the initial values:
// define our tuple  
let ourTuple: [number, boolean, string];  
// initialize correctly  
ourTuple = [5, false, 'Coding God was here'];  
// We have no type safety in our tuple for indexes 3+  
ourTuple.push('Something new and wrong');  
console.log(ourTuple);
// define our readonly tuple  
const ourReadonlyTuple: readonly [number, boolean, string] = [5, true, 'The Real Coding God'];

**Named tuples** allow us to provide context for our values at each index.
const graph: [x: number, y: number] = [55.2, 41.3];

Since tuples are arrays we can also destructure them.
const graph: [number, number] = [55.2, 41.3];  
const [x, y] = graph;