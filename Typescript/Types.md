# Types

- [Types](#types)
  - [Reference](#reference)
  - [Types In Transcript](#types-in-transcript)
  - [Any type](#any-type)
    - [`any` type checking](#any-type-checking)
    - [strictNullChecks](#strictnullchecks)
    - [Non-null Assertion Operator (Postfix `!`)](#non-null-assertion-operator-postfix-)
  - [`never`](#never)
  - [`unknown`](#unknown)
  - [`typeof` and `instanceof`](#typeof-and-instanceof)
  - [(DATA) Type Annotations](#data-type-annotations)
  - [Functions](#functions)
    - [Anonymous Functions](#anonymous-functions)
  - [Object Types](#object-types)
  - [Undefined](#undefined)
  - [Intersection Types `&`](#intersection-types-)
  - [Union Type (`|`)](#union-type-)
  - [Type Aliases](#type-aliases)
  - [Interface](#interface)
  - [Type Assertions (`as`)](#type-assertions-as)
      - [`(expr as any) as T`](#expr-as-any-as-t)
    - [Type Declaration VS Type Assertion](#type-declaration-vs-type-assertion)
    - [Genetics VS Type Assertion](#genetics-vs-type-assertion)
  - [Literal Types](#literal-types)
      - [`boolean` literals](#boolean-literals)
    - [Literal Inference](#literal-inference)
  - [Less Common Primitives](#less-common-primitives)
  - [`Symbol(....)`](#symbol)
  - [Tuple Types](#tuple-types)
    - [Tuple's Javascript array destructuring](#tuples-javascript-array-destructuring)
    - [`readonly` Tuple Types](#readonly-tuple-types)
## Reference 
[EveryDayType](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)  
[Type Inference & Annotation](https://learn.appointment-bot.webzyno.com/docs/typescript/type-inference-annotation)

## Types In Transcript

- Primitive Types： `number, string, boolean, undefined, null, ES6 介紹的 symbol, and`void
- Object Types : `JSON`,`Array<T>` or `T[]`, instance of the Class, Enum and Tuple
- Function Types
- Literal Type
  - Object Literal 
  - boolean Literal
- `any`、`never` and `unknown`
- union (`|`) 與 intersection (`&`) 
- Generic Types

## Any type

you can use `any` whenever you don’t want a PARTICULAR value to cause typechecking errors.

> When a value is of type `any`, you can access any properties of it (which will in turn be of type `any`), call it like a function, assign it to (or from) a value of any type, or pretty much anything else that’s syntactically legal

```typescript
let obj: any = { x: 0 };

// None of the following lines of code will throw compiler errors.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```
- Using `any` disables all further type checking, and it is assumed you know the environment better than TypeScript.
- The `any` type is useful **when you don’t want to write out a long type just to convince TypeScript that a particular line of code is okay.**
- When you don’t specify a type, and TypeScript can’t infer it from context, the compiler will typically default to `any`.



If programmer doesn't initialize the variable, typescript infers it as `any` type
```typescript
// without initializing
// you can assign any value to variable 
let messageToSend;
anuVariable = 'NineFiveTwoSeven';
anyVariable = 9527;


let absoluteNothing: undefined = undefined;
let literallyAbsoluteNothing: null = null;

// Warning
absoluteNothing = 123;
literallyAbsoluteNothing = "I can't live in this variable now...";
```

Variable with Type Annotation when it has no value to be initialized to prevent from assigning it to other variable as `any` type
```typescript
// With Type Annotation 
let canBeNullableString: string;

canBeNullableString = 'hello';

canBeNullableString = undefined;    // error
canBeNullableString = null;         // error

let canBeNullableString: string;

// error
let myString = canBeNullableString;
canBeNullableString = 'hello';
```

### `any` type checking
 
To signal absent or uninitialized value (`null` and `undefined`)

```typescript
// typescript infers them as any type
let notingUndefined = undefined; 
let nothingNull = null;
```

### strictNullChecks

**With strictNullChecks on, when a value is `null` or `undefined`, you will need to test for those values before using methods or properties on that value.**

we can use narrowing to check for values that might be `null`
```typescript
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

### Non-null Assertion Operator (Postfix `!`)

TypeScript also has a special syntax for removing `null` and `undefined` from a type without doing any explicit checking. Writing `!` after any expression is effectively a type assertion that the value isn’t `null` or `undefined`
```typescript
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

It’s important to only use `!` when you know that the value can’t be `null` or `undefined`.


## `never`

`never` is not `void` and it built for function that **having not returned value** (e.g. throw Error) or **infinity loop**

```typescript
// Not have a reachable end point
function error(message: string): never {
  throw new Error(message);
}

// error inferred return type is never
function fail() {
  return error("Something failed");
}

// Not have a reachable end point
function infiniteLoop(): never {
  while (true) {}
}
```

## `unknown`

What is the Difference btw `any` and `unknown`
- **`unknown` which is the type-safe counterpart of `any`**. 

在 TypeScript 中，僅有 `Any` 與 `Unknown` 型別可以被存放任意值，作為更嚴謹的 `Unknown` 型別，就會有更嚴格的規範

- Anything is assignable to `unknown`, but `unknown` isn't assignable to anything but itself and `any` without a type assertion or a control flow based narrowing.   
Likewise, **no operations are permitted on an `unknown` without first asserting or narrowing to a more specific type.**

- **`unknown` type can be assigned to other types**
```typescript
// Anything is assignable to unknown
let value: unknown = 3;
value = "Ian";
value = undefined;
value = null;
value = true;


let value1: unknown = value;   // anything is assignable to unknown
let value2: any = value;       // anything is assignable to any

// Only unknown is assignable to itself
let value3: number = value;   // Error 
let value4: boolean = value;  // Error
let value5: string = value;   // Error

let vAny: any = 10;          // We can assign anything to any
let vUnknown: unknown =  10; // We can assign anything to unknown just like any 

let s1: string = vAny;       // Any is assignable to anything 
let s2: string = vUnknown;   // Invalid; we can't assign vUnknown to any other type (without an explicit assertion)
vAny.method();     // OK; anything goes with any
vUnknown.method(); // Not Ok ; we don't know anything about this variable

let value: unknown;
let value2: any;
value2.substring(); // OK (may cause accidents)
value.substring();  // Error : Object is of type 'unknown'
```

## `typeof` and `instanceof`

通常檢測原始型別都是為用 `typeof`
```typescript
typeof value === ''
```

通常廣義物件或類別（Class）**建造出來的物件則是會用`instanceof`
```typescript
someObject instanceof ObjectBelongingClass
```


## (DATA) Type Annotations 

you can optionally add a type annotation to explicitly specify the type of the variable via `let` or `const`:
```typescript
let myName: string = "Alice";
```
- TypeScript doesn’t use _types on the left_-style declarations like `int x = 0;` Type annotations will always go after the thing being typed (`variable : type`).

**Wherever possible, TypeScript tries to automatically infer the types.**   
For example, the type of a variable is inferred based on the type of its initializer:
```typescript
// Typescript knows that
// 'myName' inferred as type string
let myName = "Alice";
```

## Functions

TypeScript allows you to specify the types of both the input and output values of functions.

```typescript
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

When a parameter has a type annotation, arguments (of types) to that function will be checked:
```typescript
// This would be a runtime error if executed!
greet(42);
```

### Anonymous Functions

Anonymous functions are a little bit different from function declarations. 

When a function appears in a place where TypeScript can determine how it’s going to be called, the parameters of that function are automatically given types.
- This process is called _contextual typing_ because the context that the function occurred within informs what type it should have.   
```typescript
// No type annotations here, 
// but TypeScript can spot the bug
const names = ["Alice", "Bob", "Eve"];
 
// Contextual typing for function
names.forEach(function (s) {
  console.log(s.toUppercase());
  // Property 'toUppercase' does not exist on type 'string'. 
  // Did you mean 'toUpperCase'?
});
 
// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUppercase());
  // Property 'toUppercase' does not exist on type 'string'. /// Did you mean 'toUpperCase'?
});
```
- TypeScript used the types of the `forEach` function, along with the inferred type of the array, to determine the type s will have.   

## Object Types

The most common sort of type is an object type. 
> To define an object type, we simply list its properties and their types.

```typescript
type obj = { x : number , y : number }
function printCoord(pt : obj){
  //...
}

// or 
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 3, y: 7 });
```
- **The type part of each property is also optional. If you don’t specify a type, it will be assumed to be `any`.**


## Undefined

In JavaScript, if you access a property that doesn’t exist, you’ll get the value `undefined` rather than a runtime error.   
Because of this, when you read from an optional property, you’ll have to check for `undefined` before using it.
```typescript
function printName(obj: { first: string; last?: string }) {
  // Error - might crash if 'obj.last' wasn't provided!
  console.log(obj.last.toUpperCase());
  // Object is possibly 'undefined'.
  
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
 
  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
```
## Intersection Types `&`

An intersection type represents an entity that is of all types. 
```typescript
function extend <A, B> (a: A, b: B): A & B {
  
  Object.keys(b).forEach(key => {
    a[key] = b[key]
  })

  return a as A & B
}
```
## Union Type (`|`)

A union type is a type formed from two or more other types, representing values that may be any one of those types. 

```typescript
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");

// Error
printId({ myID: 22342 });
// object type is not number | string typ
```

**TypeScript will only allow an operation if it is valid for every member of the union.**    
- For example, if you have the union `string | number`, you can’t use methods that are only available on string:
```typescript
function printId(id: number | string) {
  console.log(id.toUpperCase());
  // Property 'toUpperCase' does not exist on type 'string | number'.
  // Property 'toUpperCase' does not exist on type 'number'.
}
```

Using Narrowing occurs when TypeScript can deduce a more specific type for a value based on the structure of the code.
```typescript
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}

// narrowing with array and non-array
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}
```

If every member in a union has a property in common, you can use that property without narrowing. 
- For example, both arrays and strings have a slice method
```typescript 
// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```
- The union `number | string` is composed by taking the union of the values from each type. 

Notice that given two sets with corresponding facts about each set, only the intersection of those facts applies to the union of the sets themselves. 

For example, if we had a room of tall people wearing hats, and another room of Spanish speakers wearing hats, after combining those rooms, the only thing we know about every person is that they must be wearing a hat.

## Type Aliases
**A type alias is exactly that - a name for any type or group of types.**   

```typescript 
// Type named Point
type Point = {
  x: number;
  y: number;
};
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

You can actually use a type `alias` to give a name to any type at all, not just an object type.    
```typescript
// creating ID type 
type ID = number | tring;
```
**Note that aliases are only aliases - you cannot use type aliases to create _different/distinct versions_ of the same type.**


When you use the alias, it’s exactly as if you had written the aliased type.  
```typescript
type UserInputSanitizedString = string;
 
function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}
 
// Create a sanitized input
let userInput = sanitizeInput(getInput());
 
// Can still be re-assigned with a string though
userInput = "new input";
```

```typescript
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;

type Human = Male | Female

// tuple 
type PetList = [Dog, Pet]

// using type instead getName(n : string | NameResolver)   
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    } else {
        return n();
    }
}
```

## Interface 

Differences Between Type Aliases and Interfaces.  
Almost all features of an interface are available in type, **the key distinction is that a `type` cannot be re-opened to add new properties vs an interface which is always extendable.**

Extending an interface
```typescript
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
```   

Extending a type via intersections(`&`)
```typescript
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: boolean 
}

const bear = getBear();
bear.name;
bear.honey;
```     

Adding new fields to an existing interface
```typescript
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```        

A type cannot be changed after being created
```typescript 
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}
// Error: Duplicate identifier 'Window'.
```
        
## Type Assertions (`as`)

**Sometimes you will have information about the type of a value that TypeScript can’t know about.**
- **Like a type annotation, type assertions are removed by the compiler and won’t affect the runtime behavior of your code.**

For example, if you’re using `document.getElementById`,TypeScript only knows that this will return some kind of `HTMLElement`, but you might know that your page will always have an `HTMLCanvasElement` with a given ID.        
In this situation, you can use a type assertion to specify a more specific type:  
```typescript 
const myCanvas = 
  // typescript doesn't know "main_canvas"
  document.getElementById("main_canvas") as HTMLCanvasElement;
```
You can also use the angle-bracket syntax (except if the code is in a `.tsx` file), which is equivalent:
```typescript
const myCanvas = 
  <HTMLCanvasElement>document.getElementById("main_canvas");
```
Because type assertions are removed at compile-time, there is no runtime checking associated with a type assertion.  
There won’t be an exception or `null` generated if the type assertion is wrong.

**TypeScript only allows type assertions which convert to a more specific or less specific version of a type.**

This rule, it prevents impossible coercions like:
```typescript
const x = "hello" as number;

// Eoor
Conversion of type 'string' to type 'number' may be a mistake because neither type 
sufficiently overlaps with the other. If this was intentional, 
convert the expression to 'unknown' first.
```

#### `(expr as any) as T`

Sometimes this rule can be too conservative and will disallow more complex coercions that might be valid.   

If this happens, you can use two assertions, first to `any` (or `unknown`), then to the desired type:
```typescript
const a = (expr as any) as T;
```

### Type Declaration VS Type Assertion


```typescript 
interface Animal {
    name: string;
}
interface Cat {
    name: string;
    run(): void;
}

const animal: Animal = {
    name: 'tom'
};

// ERROR
// Cat is subclass of animal 
let tom: Cat = animal;
```
### Genetics VS Type Assertion
```typescript
// Assertion
function getCacheData(key: string): any {
    return (window as any).cache[key];
}

interface Cat {
    name: string;
    run(): void;
}

const tom = getCacheData('tom') as Cat;
tom.run();

// generic
function getCacheData<T>(key: string): T {
    return (window as any).cache[key];
}

interface Cat {
    name: string;
    run(): void;
}

const tom = getCacheData<Cat>('tom');
tom.run();
```

## Literal Types

**In addition to the general types `string` and `number`, we can refer to specific strings and numbers in type positions**.

```typescript
// let and var represent any possible string
// Type of variable changingString is Hello World
let changingString = "Hello World";
changingString = "Olá Mundo";
changingString;
      
let changingString: string

// const has a literal type representation 
const constantString = "Hello World";
const constantString: "Hello World"

// By themselves, literal types aren’t very valuable:
let x: "hello" = "hello";
// OK (allow only one value ("hello") assignable  )
x = "hello";

x = "howdy";
// Type '"howdy"' is not assignable to type '"hello"'.
```
- It’s not much use to have a variable that can only have one value

By combining literals into unions, you can express a much more useful concept. For example, **functions that only accept a certain set of known values**:
```typescript 
// Alignment only allows three value (left, right or center)
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");

// error
Argument of type '"centre"' is not assignable to 
parameter of type '"left" | "right" | "center"'.
```

Numeric literal types work the same way:
```typescript
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}

// it also can be combined with non-literal types:
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");

configure("automatic");
// Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

#### `boolean` literals

There are only two boolean literal types, and as you might guess, they are the types `true` and `false`. 

The type boolean itself is actually just an alias for the `union true | false.`

### Literal Inference

When you initialize a variable with an object, TypeScript assumes that the properties of that object might change values later. For example ::
```typescript
// Typescript knows obj.counter type is number 
const obj = { counter: 0 };

if (someCondition) {
  // properties of obj changed 
  obj.counter = 1;
}
```

```typescript
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
// Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```
- `req.method` is inferred to be `string`, not `"GET"`.   
   The call of `handleRequest` which could assign a new string like `"GUESS"` to `req.method`, TypeScript considers this code to have an error.

There are two ways to work around this.        
- You can change the inference by adding a TYPE ASSERTION in either location:
```typescript
// Change 1 :
const req = { url: "https://example.com", method: "GET" as "GET" };

// Change 2
handleRequest(req.url, req.method as "GET");
```
- Change 1 means _I intend for `req.method` to always have the literal type `"GET"`_, preventing the possible assignment of `"GUESS"` to that field after. 
- Change 2 means _I know for other reasons that `req.method` has the value `"GET"`_.

You can use `as const` to convert the entire object to be type literals:
```typescript
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```
The `as const` suffix acts like const but for the type system, ensuring that all properties are assigned the literal type instead of a more general version like `string` or `number`.


## Less Common Primitives

From ES2020 onwards, there is a primitive in JavaScript used for very large integers, `BigInt`:
```typescript
// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);
 
// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;
```

## `Symbol(....)`
There is a primitive in JavaScript used to create a globally **unique reference** via the function `Symbol()`

```typescript
// difference reference 
const firstName = Symbol("name");
const secondName = Symbol("name");
 
if (firstName === secondName) {
  /**
    * This condition will always return 'false' 
    * since the types 'typeof firstName' 
    * and 'typeof secondName' have no overlap.
    */
}
```
## Tuple Types

**A tuple type is another sort of Array type that knows exactly how many elements it contains, and exactly which types it contains at specific positions.**  
unlike array it contains different data type.
```typescript 
pe StringNumberPair = [string, number];
```
- `StringNumberPair` is a tuple type of `string` and `number`.  
- To the type system, StringNumberPair describes arrays whose 0 index contains a `string` and whose 1 index contains a `number`.

Tuple Types Like `ReadonlyArray`, it has no representation at runtime, but is significant to TypeScript.  
```typescript 
function doSomething(pair: [string, number]) {
  const a = pair[0];
       
  const a: string

  const b = pair[1];
       
  const b: number
  // ...
}
 
doSomething(["hello", 42]);
```

If we try to index past the number of elements, we’ll get an error.
```typescript 
function doSomething(pair: [string, number]) {
  // ...
 
  const c = pair[2];
  // Tuple type '[string, number]' of length '2' has no element at index '2'.
}
```

### Tuple's Javascript array destructuring

We can also destructure tuples using JavaScript’s array destructuring.
```typescript
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;
 
  console.log(inputString);
  const inputString: string
 
  console.log(hash);
  const hash: number
}
```

Tuple types are useful in heavily convention-based APIs, **where each element’s meaning is obvious.**  
- This gives us flexibility in whatever we want to name our variables when we destructure them.  

However, since not every user holds the same view of what’s obvious, it may be worth reconsidering whether using objects with descriptive property names may be better for your API.

Other than those length checks, simple tuple types like these are equivalent to types which are versions of Arrays that declare properties for specific indexes, and that declare length with a numeric literal type.
```typescript 
interface StringNumberPair {
  // specialized properties
  length: 2;
  0: string;
  1: number;
 
  // Other 'Array<string | number>' members...
  slice(start?: number, end?: number): Array<string | number>;
}
```

Another thing you may be interested in is that tuples can have optional properties by writing out a question mark (`?` after an element’s type).    

Optional tuple elements can only come at the end, and also affect the type of length.
```typescript 
type Either2dOr3d = [number, number, number?];
 
function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;
  const z: number | undefined
  
  console.log(`Provided coordinates had ${coord.length} dimensions`);
  
  (property) length: 2 | 3
}
```
Tuples can also have rest elements, which have to be an array/tuple type.
```typescript 
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```
1. `StringNumberBooleans` describes a tuple whose first two elements are `string` and `number` respectively, but which may have any number of `booleans` following.
2. `StringBooleansNumber` describes a tuple whose first element is string and then any number of `booleans` and ending with a `number`.
3. `BooleansStringNumber` describes a tuple whose starting elements are any number of `booleans` and ending with a `string` then a `number`.

A tuple with a rest element has no set length     
it only has a set of well-known elements in different positions.   
```typescript 
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```

Why might optional and rest elements be useful? Well, it allows TypeScript to correspond tuples with parameter lists. 

Tuples types can be used in rest parameters and arguments, so that the following:
```typescript 
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```
is basically equivalent to:
```typescript 
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```
This is handy when you want to take a variable number of arguments with a rest parameter, and you need a minimum number of elements, but you don’t want to introduce intermediate variables.

### `readonly` Tuple Types

tuples types have `readonly` variants, and can be specified by sticking a `readonly` modifier in front of them - just like with array shorthand syntax.
```typescript 
function doSomething(pair: readonly [string, number]) {
  // ...
}

function doSomething(pair: readonly [string, number]) {
  pair[0] = "hello!" ;
  
  Cannot assign to '0' because it is a read-only property.
}
```

**Tuples tend to be created and left un-modified in most code, so annotating types as readonly tuples when possible is a good default.** 

This is also important given that array literals with const assertions will be inferred with `readonly` 
tuple types.   
```typescript 
let point = [3, 4] as const;
 
// here `distanceFromOrigin` never modifies its elements, but expects a mutable tuple.  
function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}


// But we assign a const parameter  
distanceFromOrigin(point);

Argument of type 'readonly [3, 4]' is not assignable to parameter of type '[number, number]'.
The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.
```