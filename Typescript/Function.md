# Functions
- [Functions](#functions)
  - [Function as parameter in other function](#function-as-parameter-in-other-function)
    - [Function tpe expressions ( `fn:( ... ) => returnType`)](#function-tpe-expressions--fn----returntype)
    - [use a type alias to name a function type](#use-a-type-alias-to-name-a-function-type)
  - [Call Signatures](#call-signatures)
  - [Construct(`new`) Signatures](#constructnew-signatures)
  - [Generic Functions](#generic-functions)
  - [Inference](#inference)
  - [Constraints `T extends A` (Generic Function)](#constraints-t-extends-a-generic-function)
    - [Specifying Type Arguments](#specifying-type-arguments)
    - [Use Fewer Type Parameters](#use-fewer-type-parameters)
    - [Type Parameters Should Appear Twice](#type-parameters-should-appear-twice)
  - [Optional Parameters](#optional-parameters)
    - [Optional Parameters in Callbacks](#optional-parameters-in-callbacks)
  - [Function Overloads](#function-overloads)
    - [Writing Good Overloads](#writing-good-overloads)
    - [Declaring `this` in a Function](#declaring-this-in-a-function)
  - [Other Types to Know About](#other-types-to-know-about)
    - [`void`](#void)
    - [object](#object)
    - [`unknown`](#unknown)
    - [never](#never)
    - [`Function`](#function)
  - [Rest Arguments](#rest-arguments)
  - [`void` return type](#void-return-type)
  - [Call Signatures](#call-signatures-1)
  - [Construct(`new`) Signatures](#constructnew-signatures-1)
  - [Generic Functions](#generic-functions-1)
  - [Inference](#inference-1)
  - [Constraints](#constraints)
    - [Specifying Type Arguments](#specifying-type-arguments-1)
    - [Use Fewer Type Parameters](#use-fewer-type-parameters-1)
    - [Type Parameters Should Appear Twice](#type-parameters-should-appear-twice-1)
  - [Optional Parameters](#optional-parameters-1)
    - [Optional Parameters in Callbacks](#optional-parameters-in-callbacks-1)
  - [Function Overloads](#function-overloads-1)
    - [Writing Good Overloads](#writing-good-overloads-1)
    - [Declaring `this` in a Function](#declaring-this-in-a-function-1)
  - [Other Types to Know About](#other-types-to-know-about-1)
    - [`void`](#void-1)
    - [object](#object-1)
    - [`unknown`](#unknown-1)
    - [never](#never-1)
    - [`Function`](#function-1)
  - [Rest Arguments](#rest-arguments-1)
  - [`void` return type](#void-return-type-1)


## Function as parameter in other function 
### Function tpe expressions ( `fn:( ... ) => returnType`)

```typescript 
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

function printToConsole(s: string) {
  console.log(s);
}

greeter(printToConsole);
```
- The syntax `(a: string) => void` means _a function with one parameter, named `a`, of type `string`, that doesn’t have a return value (`void`)_.  
- Just like with function declarations, if a parameter type isn’t specified, it’s implicitly `any`.  
The function type `(string) => void` means `(string : any) => void`

###  use a type alias to name a function type
```typescript
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

## Call Signatures
Functions can have properties in addition to being callable.   
However, the function type expression syntax doesn’t allow for declaring properties. 

If we want to describe something callable with properties, we can write a call signature in an object type:

```typescript
// object type 
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};

// callable object type 
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```

## Construct(`new`) Signatures

Some objects, like JavaScript’s Date object, can be called with or without `new`.
```typescript
type SomeConstructor = {
  new (s: string): SomeObject;
};

function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

You can combine call and construct signatures in the same type arbitrarily:
```typescript
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

## Generic Functions

It’s common to write a function where the types of the input relate to the type of the output, or where the types of two inputs are related in some way. 

```typescript 
function firstElement(arr: any[]) {
  // return input's first element as output
  return arr[0];
}
```
- This function does its job, but unfortunately has the return type `any`. It’d be better if the function returned the type of the array element.

Generics are used when we want to describe a correspondence between two values.   
We do this by declaring a type parameter in the function signature:
```typescript 
// function name<Generic>(p : Generic) : Generic | undefined
function firstElement<Type>(arr: Type[]):  Type | undefined {
  // return Specific Type or undefined
  return arr[0];
}
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
// u is of type undefined
const u = firstElement([]);
```

## Inference
    
The type was inferred - chosen automatically - by TypeScript.  
We can use multiple type parameters as well. 

For example, a standalone version of map would look like this:
```typescript 
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}

// 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```
- TypeScript could infer both the type of the Input type parameter (from the given `string` array), as well as the Output type parameter based on the return value of the function expression (`number`).

## Constraints `T extends A` (Generic Function)

Sometimes we want to relate two values, but can only operate on a certain subset of values. 
In this case, we can use a constraint to limit the kinds of types that a type parameter can accept.
- **We constrain the type parameter to that type by writing an `extends` clause:**


Let’s write a function that returns the longer of two values. 
```typescript
function longest<Type extends { length: number }> (a: Type, b: Type) 
{
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
 
// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");

// Error! Numbers don't have a 'length' property
// Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
const notOK = longest(10, 100);
```
- Because we constrained `Type` to `{ length: number }`, we were allowed to access the `.length` property of the a and b parameters.

Without the type constraint, we wouldn’t be able to access those properties because the values might have been some other type without a length property.
- the call to `longest(10, 100)` is rejected because the `number` type doesn’t have a `.length` property.

Working with Constrained Values
```typescript 
function minimumLength<Type extends { length: number }>( obj: Type, minimum: number): Type 
{
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length : minimum };
    // Type '{ length: number; }' is not assignable to type 'Type'.
    /** 
     * '{ length: number; }' is assignable to the constraint of type 'Type', 
     *  but 'Type' could be instantiated 
     *  with a different subtype of constraint '{ length: number; }'.
     */
  }
}
```
- `Type` is constrained to `{ length: number }`, and the function either returns `Type` or a value matching that constraint.  

The problem is that the function promises to return the same kind of object as was passed in, not just some object matching the constraint.    
If this code were legal, you could write code that definitely wouldn’t work:
```typescript 
// 'arr' gets value { length: 6 }
// minimum Length return type should be array[] not number 
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

### Specifying Type Arguments

TypeScript can usually infer the intended type arguments in a generic call, but not always.  
- For example
```typescript 
// Combine two arrays
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

Normally it would be an error to call this function with mismatched arrays:
```typescript
const arr = combine([1, 2, 3], ["hello"]);
// Type 'string' is not assignable to type 'number'.
```

If you intended to do this, however, you could manually specify Type:
```typescript
// <Type> : string | number 
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```
**Having too many type parametersor using constraints where they aren’t needed can make inference less successful, frustrating callers of your function.**  
Here are two ways of writing a function that appear similar:
```typescript 
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}
 
// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```
- `firstElement1` is a much better way to write this function. Its inferred return type is `Type`, but `firstElement2`’s inferred return type is `any` because TypeScript has to resolve the `arr[0]` expression using the constraint type, rather than “waiting” to resolve the element during a call.

> Rule: When possible, use the type parameter itself rather than _constraining_ it

### Use Fewer Type Parameters

Prevent from using too many type parameters 
```typescript
// one parameter and one function parameter
function filter1<Type>(arr: Type[], func : (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}

function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```
We’ve created a type parameter `Func` that doesn’t relate two values. 

That’s always a red flag, because it means callers wanting to specify type arguments have to manually specify an extra type argument for no reason. Func doesn’t do anything but make the function harder to read and reason about!

> Rule: Always use as few type parameters as possible

### Type Parameters Should Appear Twice   

Sometimes we forget that a function might not need to be generic:
```typescript 
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}
 
greet("world");
```

We could just as easily have written a simpler version:
```typescript 
function greet(s: string) {
  console.log("Hello, " + s);
}
```

**Type parameters are for relating the types of multiple values. If a type parameter is only used once in the function signature, it’s not relating anything.**

> Rule: If a type parameter only appears in one location, strongly reconsider if you actually need it

## Optional Parameters

Functions in JavaScript often take a variable number of arguments. 

- For example, the `toFixed` method of number takes an optional digit count:
```typescript 
function f(n: number) {
  console.log(n.toFixed());  // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

We can model this in TypeScript by marking the parameter as optional with `?`:
```typescript 
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```
- Although the parameter is specified as type `number`, the `x` parameter will actually have the type `number | undefined` because unspecified parameters in JavaScript get the value `undefined`.

Provide a parameter default instead
```typescript 
function f(x = 10) {
  // ...
}
```
Now in the body of `f`, `x` will have type `number` because any `undefined` argument will be replaced with `10`.  

**Note that when a parameter is optional, callers can always pass `undefined`, as this simply simulates a _missing_ argument**:
```typescript 
declare function f(x?: number): void;

// All OK
f();
f(10);
f(undefined);
```

### Optional Parameters in Callbacks

It’s very easy to make the following mistakes when writing functions that invoke callbacks with optional Parameters:
```typescript 
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```
What people usually intend when writing `index?` as an optional parameter is that they want both of these calls to be legal:
```typescript 
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```
What this actually means is that callback might get invoked with one argument.     
In other words, the function definition says that the implementation might look like this:
```typescript 
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // I don't feel like providing the index today
    callback(arr[i]);
  }
}
```

In turn, TypeScript will enforce this meaning and issue errors that aren’t really possible:
```typescript
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
  // Object is possibly 'undefined'.
});
```

In JavaScript, if you call a function with more arguments than there are parameters, the extra arguments are simply ignored. TypeScript behaves the same way.   

Functions with fewer parameters (of the same types) can always take the place of functions with more parameters.    

**When writing a function type for a `callback`, never write an optional parameter (`?`) unless you intend to call the function without passing that argument**   

## Function Overloads

In TypeScript, we can specify a function that can be called in different ways by writing overload signatures.    
To do this, write some number of function signatures (usually two or more), followed by the body of the function:
```typescript 
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
// No overload expects 2 arguments, 
// but overloads do exist that expect either 1 or 3 arguments.
```
- we wrote two overloads:   
one accepting one argument, and another accepting three arguments.  
These first two signatures are called the overload signatures.  
Then, we wrote a function implementation with a compatible signature.  
- Functions have an implementation signature, but this signature can’t be called directly.    
Even though we wrote a function with two optional parameters after the required one, it can’t be called with two parameters!   


**Overload Signatures and the Implementation Signature
This is a common source of confusion.** 

Often people will write code like this and not understand why there is an error:
```typescript 
// Overload Signature
function fn(x: string): void;

// Implementation Signature
function fn() {
  // ...
}
// Expected to be able to call with zero arguments
fn();
// Expected 1 arguments, but got 0.
```

The signature of the implementation is not visible from the outside.   

When writing an overloaded function, you should always have two or more signatures above the implementation of the function.

- The implementation signature must also be compatible with the overload signatures.


For example, these functions have errors because the implementation signature doesn’t match the overloads in a correct way:
```typescript
// first overload func
function fn(x: boolean): void;

// Argument type isn't right
function fn(x: string): void;
// This overload signature is not compatible with its implementation signature.
function fn(x: boolean) {}

function fn(x: string): string;
// Return type isn't right (should be void)

function fn(x: number): boolean;
// This overload signature is not compatible with its implementation signature.

function fn(x: string | number) {
  return "oops";
}
```

### Writing Good Overloads

Like generics, there are a few guidelines you should follow when using function overloads. 

> **Always prefer parameters with union types instead of overloads when possible.**

- For example 
```typescript
// function that returns length 
function len(s: string): number;
function len(arr: any[]): number;

function len(x: any) {
  return x.length;
}
```
This function is fine; we can invoke it with strings or arrays. However, we can’t invoke it with a value that might be a string or an array, because TypeScript can only resolve a function call to a single overload:
```typescript 
len(""); // OK
len([0]); // OK

len(Math.random() > 0.5 ? "hello" : [0]);
/**
 * No overload matches this call.
 * Overload 1 of 2, '(s: string): number', gave the following error.
 * Argument of type 'number[] | "hello"' is not assignable to parameter of type 'string'.
 * Type 'number[]' is not assignable to type 'string'.
 * Overload 2 of 2, '(arr: any[]): number', gave the following error.
 * Argument of type 'number[] | "hello"' is not assignable to parameter of type 'any[]'.
 * Type 'string' is not assignable to type 'any[]'.
 */
```
Because both overloads have the same argument count and same return type, we can instead write a non-overloaded version of the function:
```typescript 
function len(x: any[] | string) {
  return x.length;
}
```

### Declaring `this` in a Function

TypeScript will infer what the `this` should be in a function via code flow analysis, for example in the following:
```typescript 
const user = {
  id: 123,
  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```
- TypeScript understands that the function `user.becomeAdmin` has a corresponding `this` which is the outer object user. 

**The JavaScript specification states that you cannot have a parameter called `this`, and so TypeScript uses that syntax space to let you declare the type for `this` in the function body.**
```typescript 
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```
This pattern is common with callback-style APIs, where another object typically controls when your function is called. 

Note that you need to use function and not arrow functions to get this behavior:
```typescript
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(() => this.admin);
// The containing arrow function captures the global value of 'this'.
// Element implicitly has an 'any' type because type 'typeof globalThis' has no index signature.
```

## Other Types to Know About
There are some additional types you’ll want to recognize that appear often when working with function types. Like all types, you can use them everywhere, but these are especially relevant in the context of functions.

### `void`

`void` represents the return value of functions which don’t return a value.  

It’s the inferred type any time a function doesn’t have any return statements, or doesn’t return any explicit value from those return statements:
```typescript
// The inferred return type is void
function noop() {
  return;
}
```
In JavaScript, a function that doesn’t return any value will implicitly return the value `undefined`.    

However, `void` and `undefined` are not the same thing in TypeScript. 
> **`void` is not the same as `undefined`.**

### object

The special type `object` refers to any value that isn’t a primitive (`string`, `number`, `bigint`, `boolean`, `symbol`, `null`, or `undefined`). 

This is different from the empty object type `{ }`, and also different from the global type Object.  

It’s very likely you will never use Object.   
`object` is not Object. 
Always use `object`!     

Note that in JavaScript, function values are objects:   
They have properties, have `Object.prototype` in their prototype chain, are instanceof Object, you can call `Object.keys` on them, and so on.  

For this reason, ***function types are considered to be objects in TypeScript.***

### `unknown`

The `unknown` type represents any value. 
- This is similar to the `any` type, but is safer because 
***it’s not legal to do anything with an `unknown` value***:

```typescript 
function f1(a: any) {
  a.b(); // OK
}

function f2(a: unknown) {
  a.b();
  // Object is of type 'unknown'.
}
```
**This is useful when describing function types because you can describe functions that accept any value without having any values in your function body.**

Conversely, you can describe a function that returns a value of `unknown` type:
```typescript 
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
 
// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```
### never

The `never` type represents values which are never observed.

**In a return type, this means that the function throws an exception or terminates execution of the program.**
```typescript
function fail(msg: string): never {
  throw new Error(msg);
}
```

`never` also appears when TypeScript determines there’s nothing left in a `union`.
```typescript 
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

### `Function`

The global type `Function` describes properties like bind, call, apply, and others present on all function values in JavaScript.  

It also has the special property that values of type `Function` can always be called; these calls return `any`:
```typescript 
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```
- **This is an untyped function call and is generally best avoided because of the unsafe any return type.**

If you need to accept an arbitrary function but don’t intend to call it, the type `() => void` is generally safer.

**In addition to using optional parameters or overloads to make functions that can accept a variety of fixed argument counts, _we can also define functions that take an unbounded number of arguments using rest parameters_.**  

A rest parameter appears after all other parameters, and uses the `...` syntax:
```typescript 
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```
- In TypeScript, the type annotation on these parameters is implicitly `any[]` instead of `any`, and `any` type annotation given must be of the form `Array<T>` or `T[]`, or a tuple type

## Rest Arguments

Conversely, we can provide a variable number of arguments from an array using the spread syntax. 

For example, the `push` method of arrays takes any number of arguments:
```typescript 
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

In general, TypeScript does not assume that arrays are immutable.  
This can lead to some surprising behavior:
```typescript 
const args = [8, 5];
const angle = Math.atan2(...args);
// A spread argument must either have a tuple type or be passed to a rest parameter.
```
- Inferred type (of `args`) is `number[]` -- "an array with zero or more numbers", not specifically two numbers

In general a const context is the most straightforward solution:
- [`as const`](https://stackoverflow.com/questions/66993264/what-does-the-as-const-mean-in-typescript-and-what-is-its-use-case)
```typescript 
// Inferred as "2-length" tuple via as const
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```
- Using rest arguments may require turning on downlevelIteration when targeting older runtimes.

You can use ***parameter destructuring to conveniently unpack objects provided as an argument into one or more local variables in the function body.***
In JavaScript, it looks like this:
```typescript 
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

The type annotation for the object goes after the destructuring syntax:
```typescript 
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

This can look a bit verbose, but you can use a named type here as well:
```typescript 
// Same as prior example
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

## `void` return type

The `void` return type for functions can produce some unusual, but expected behavior.
- Contextual typing with a return type of `void` does not force functions to not return something. 

Another way to say this is a contextual function type with a `void` return type (`type vf = () => void`), when implemented, can **return any other value, but it will be ignored.**

Thus, the following implementations of the `type () => void` are valid:
```typescript 
type voidFunc = () => void;
 
const f1: voidFunc = () => {
  return true;
};
 
const f2: voidFunc = () => true;
 
const f3: voidFunc = function () {
  return true;
};
```

And when the return value of one of these functions is assigned to another variable, it will retain the type of `void`:
```typescript 
const v1 = f1(); 
const v2 = f2();
const v3 = f3();
```

This behavior exists so that the following code is valid even though `Array.prototype.push` returns a number and the `Array.prototype.forEach` method expects a function with a return type of `void`.
```typescript 
const src = [1, 2, 3];
const dst = [0];
 
src.forEach((el) => dst.push(el));
```

There is one other special case to be aware of, when a literal function definition has a `void` return type, that function must not return anything.
```typescript 
function f2(): void {
  // @ts-expect-error
  return true;
}
 
const f3 = function (): void {
  // @ts-expect-error
  return true;
};
```# Functions

## Function Type Expressions

```typescript 

function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}
 
function printToConsole(s: string) {
  console.log(s);
}
 
greeter(printToConsole);
```
- The syntax `(a: string) => void` means _a function with one parameter, named `a`, of type `string`, that doesn’t have a return value (`void`)_.  
Just like with function declarations, if a parameter type isn’t specified, it’s implicitly `any`.

- the parameter name is required.   
The function type `(string) => void` means `(string : any) => void`

Or use a type alias to name a function type
```typescript
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

## Call Signatures

Functions can have properties in addition to being callable.

However, the function type expression syntax doesn’t allow for declaring properties. 

If we want to describe something callable with properties, we can write a call signature in an object type:
```typescript
// object type 
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};

// callabe object type 
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```

## Construct(`new`) Signatures

Some objects, like JavaScript’s Date object, can be called with or without `new`.
```typescript
type SomeConstructor = {
  new (s: string): SomeObject;
};

function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

You can combine call and construct signatures in the same type arbitrarily:
```typescript
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

## Generic Functions

It’s common to write a function where the types of the input relate to the type of the output, or where the types of two inputs are related in some way. 

```typescript 
function firstElement(arr: any[]) {
  // return input's first element as output
  return arr[0];
}
```
- This function does its job, but unfortunately has the return type `any`. It’d be better if the function returned the type of the array element.

Generics are used when we want to describe a correspondence between two values.   
We do this by declaring a type parameter in the function signature:
```typescript 
// function name<Generic>(p : Generic) : Generic | undefined
function firstElement<Type>(arr: Type[]):  Type | undefined {
  // return Specific Type or undefined
  return arr[0];
}
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
// u is of type undefined
const u = firstElement([]);
```

## Inference

Note that we didn’t have to specify Type in this sample.    
The type was inferred - chosen automatically - by TypeScript.  

We can use multiple type parameters as well. 

For example, a standalone version of map would look like this:
```typescript 
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}

// 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```
- TypeScript could infer both the type of the Input type parameter (from the given `string` array), as well as the Output type parameter based on the return value of the function expression (`number`).

## Constraints

Sometimes we want to relate two values, but can only operate on a certain subset of values. 

In this case, we can use a constraint to limit the kinds of types that a type parameter can accept.

Let’s write a function that returns the longer of two values. 
To do this, we need a length property that’s a number. 
**We constrain the type parameter to that type by writing an `extends` clause:**
```typescript
function longest<Type extends { length: number }> (a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
 
// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");

// Error! Numbers don't have a 'length' property
// Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
const notOK = longest(10, 100);
```
- Because we constrained `Type` to `{ length: number }`, we were allowed to access the `.length` property of the a and b parameters.   
Without the type constraint, we wouldn’t be able to access those properties because the values might have been some other type without a length property.
- the call to `longest(10, 100)` is rejected because the `number` type doesn’t have a `.length` property.

Working with Constrained Values
```typescript 
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
    // Type '{ length: number; }' is not assignable to type 'Type'.
    /** 
     * '{ length: number; }' is assignable to the constraint of type 'Type', 
     *  but 'Type' could be instantiated 
     *  with a different subtype of constraint '{ length: number; }'.
     */
  }
}
```
- `Type` is constrained to `{ length: number }`, and the function either returns `Type` or a value matching that constraint.  

The problem is that the function promises to return the same kind of object as was passed in, not just some object matching the constraint.    
If this code were legal, you could write code that definitely wouldn’t work:
```typescript 
// 'arr' gets value { length: 6 }
// minimumuLength return type should be array[] not number 
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

### Specifying Type Arguments

TypeScript can usually infer the intended type arguments in a generic call, but not always.  
- For example
```typescript 
// Combine two arrays
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

Normally it would be an error to call this function with mismatched arrays:
```typescript
const arr = combine([1, 2, 3], ["hello"]);
// Type 'string' is not assignable to type 'number'.
```

If you intended to do this, however, you could manually specify Type:
```typescript
// <Type> : string | number 
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```
**Having too many type parametersor using constraints where they aren’t needed can make inference less successful, frustrating callers of your function.**  
Here are two ways of writing a function that appear similar:
```typescript 
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}
 
// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```
- `firstElement1` is a much better way to write this function. Its inferred return type is `Type`, but `firstElement2`’s inferred return type is `any` because TypeScript has to resolve the `arr[0]` expression using the constraint type, rather than “waiting” to resolve the element during a call.

> Rule: When possible, use the type parameter itself rather than _constraining_ it

### Use Fewer Type Parameters

Prevent from using too many type parameters 
```typescript
// one parameter and one function parameter
function filter1<Type>(arr: Type[], func : (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}

function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```
We’ve created a type parameter `Func` that doesn’t relate two values. 

That’s always a red flag, because it means callers wanting to specify type arguments have to manually specify an extra type argument for no reason. Func doesn’t do anything but make the function harder to read and reason about!

> Rule: Always use as few type parameters as possible

### Type Parameters Should Appear Twice   

Sometimes we forget that a function might not need to be generic:
```typescript 
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}
 
greet("world");
```

We could just as easily have written a simpler version:
```typescript 
function greet(s: string) {
  console.log("Hello, " + s);
}
```

**Type parameters are for relating the types of multiple values. If a type parameter is only used once in the function signature, it’s not relating anything.**

> Rule: If a type parameter only appears in one location, strongly reconsider if you actually need it

## Optional Parameters

Functions in JavaScript often take a variable number of arguments. 

- For example, the `toFixed` method of number takes an optional digit count:
```typescript 
function f(n: number) {
  console.log(n.toFixed());  // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

We can model this in TypeScript by marking the parameter as optional with `?`:
```typescript 
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```
- Although the parameter is specified as type `number`, the `x` parameter will actually have the type `number | undefined` because unspecified parameters in JavaScript get the value `undefined`.

Provide a parameter default instead
```typescript 
function f(x = 10) {
  // ...
}
```
Now in the body of `f`, `x` will have type `number` because any `undefined` argument will be replaced with `10`.  

**Note that when a parameter is optional, callers can always pass `undefined`, as this simply simulates a _missing_ argument**:
```typescript 
declare function f(x?: number): void;

// All OK
f();
f(10);
f(undefined);
```

### Optional Parameters in Callbacks

It’s very easy to make the following mistakes when writing functions that invoke callbacks with optional Parameters:
```typescript 
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```
What people usually intend when writing `index?` as an optional parameter is that they want both of these calls to be legal:
```typescript 
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```
What this actually means is that callback might get invoked with one argument.     
In other words, the function definition says that the implementation might look like this:
```typescript 
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // I don't feel like providing the index today
    callback(arr[i]);
  }
}
```

In turn, TypeScript will enforce this meaning and issue errors that aren’t really possible:
```typescript
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
  // Object is possibly 'undefined'.
});
```

In JavaScript, if you call a function with more arguments than there are parameters, the extra arguments are simply ignored. TypeScript behaves the same way.   

Functions with fewer parameters (of the same types) can always take the place of functions with more parameters.    

**When writing a function type for a `callback`, never write an optional parameter (`?`) unless you intend to call the function without passing that argument**   

## Function Overloads

In TypeScript, we can specify a function that can be called in different ways by writing overload signatures.    
To do this, write some number of function signatures (usually two or more), followed by the body of the function:
```typescript 
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
// No overload expects 2 arguments, 
// but overloads do exist that expect either 1 or 3 arguments.
```
- we wrote two overloads:   
one accepting one argument, and another accepting three arguments.  
These first two signatures are called the overload signatures.  
Then, we wrote a function implementation with a compatible signature.  
- Functions have an implementation signature, but this signature can’t be called directly.    
Even though we wrote a function with two optional parameters after the required one, it can’t be called with two parameters!   


**Overload Signatures and the Implementation Signature
This is a common source of confusion.** 

Often people will write code like this and not understand why there is an error:
```typescript 
// Overload Signature
function fn(x: string): void;

// Implementation Signature
function fn() {
  // ...
}
// Expected to be able to call with zero arguments
fn();
// Expected 1 arguments, but got 0.
```

The signature of the implementation is not visible from the outside.   

When writing an overloaded function, you should always have two or more signatures above the implementation of the function.

- The implementation signature must also be compatible with the overload signatures.


For example, these functions have errors because the implementation signature doesn’t match the overloads in a correct way:
```typescript
// first overload func
function fn(x: boolean): void;

// Argument type isn't right
function fn(x: string): void;
// This overload signature is not compatible with its implementation signature.
function fn(x: boolean) {}

function fn(x: string): string;
// Return type isn't right (should be void)

function fn(x: number): boolean;
// This overload signature is not compatible with its implementation signature.

function fn(x: string | number) {
  return "oops";
}
```

### Writing Good Overloads

Like generics, there are a few guidelines you should follow when using function overloads. 

> **Always prefer parameters with union types instead of overloads when possible.**

- For example 
```typescript
// function that returns length 
function len(s: string): number;
function len(arr: any[]): number;

function len(x: any) {
  return x.length;
}
```
This function is fine; we can invoke it with strings or arrays. However, we can’t invoke it with a value that might be a string or an array, because TypeScript can only resolve a function call to a single overload:
```typescript 
len(""); // OK
len([0]); // OK

len(Math.random() > 0.5 ? "hello" : [0]);
/**
 * No overload matches this call.
 * Overload 1 of 2, '(s: string): number', gave the following error.
 * Argument of type 'number[] | "hello"' is not assignable to parameter of type 'string'.
 * Type 'number[]' is not assignable to type 'string'.
 * Overload 2 of 2, '(arr: any[]): number', gave the following error.
 * Argument of type 'number[] | "hello"' is not assignable to parameter of type 'any[]'.
 * Type 'string' is not assignable to type 'any[]'.
 */
```
Because both overloads have the same argument count and same return type, we can instead write a non-overloaded version of the function:
```typescript 
function len(x: any[] | string) {
  return x.length;
}
```

### Declaring `this` in a Function

TypeScript will infer what the `this` should be in a function via code flow analysis, for example in the following:
```typescript 
const user = {
  id: 123,
  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```
- TypeScript understands that the function `user.becomeAdmin` has a corresponding `this` which is the outer object user. 

**The JavaScript specification states that you cannot have a parameter called `this`, and so TypeScript uses that syntax space to let you declare the type for `this` in the function body.**
```typescript 
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```
This pattern is common with callback-style APIs, where another object typically controls when your function is called. 

Note that you need to use function and not arrow functions to get this behavior:
```typescript
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(() => this.admin);
// The containing arrow function captures the global value of 'this'.
// Element implicitly has an 'any' type because type 'typeof globalThis' has no index signature.
```

## Other Types to Know About
There are some additional types you’ll want to recognize that appear often when working with function types. Like all types, you can use them everywhere, but these are especially relevant in the context of functions.

### `void`

`void` represents the return value of functions which don’t return a value.  

It’s the inferred type any time a function doesn’t have any return statements, or doesn’t return any explicit value from those return statements:
```typescript
// The inferred return type is void
function noop() {
  return;
}
```
In JavaScript, a function that doesn’t return any value will implicitly return the value `undefined`.    

However, `void` and `undefined` are not the same thing in TypeScript. 
> **`void` is not the same as `undefined`.**

### object

The special type `object` refers to any value that isn’t a primitive (`string`, `number`, `bigint`, `boolean`, `symbol`, `null`, or `undefined`). 

This is different from the empty object type `{ }`, and also different from the global type Object.  

It’s very likely you will never use Object.   
`object` is not Object. 
Always use `object`!     

Note that in JavaScript, function values are objects:   
They have properties, have `Object.prototype` in their prototype chain, are instanceof Object, you can call `Object.keys` on them, and so on.  

For this reason, ***function types are considered to be objects in TypeScript.***

### `unknown`

The `unknown` type represents any value. 
- This is similar to the `any` type, but is safer because 
***it’s not legal to do anything with an `unknown` value***:

```typescript 
function f1(a: any) {
  a.b(); // OK
}

function f2(a: unknown) {
  a.b();
  // Object is of type 'unknown'.
}
```
**This is useful when describing function types because you can describe functions that accept any value without having any values in your function body.**

Conversely, you can describe a function that returns a value of `unknown` type:
```typescript 
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
 
// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```
### never

The `never` type represents values which are never observed.

**In a return type, this means that the function throws an exception or terminates execution of the program.**
```typescript
function fail(msg: string): never {
  throw new Error(msg);
}
```

`never` also appears when TypeScript determines there’s nothing left in a `union`.
```typescript 
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

### `Function`

The global type `Function` describes properties like bind, call, apply, and others present on all function values in JavaScript.  

It also has the special property that values of type `Function` can always be called; these calls return `any`:
```typescript 
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```
- **This is an untyped function call and is generally best avoided because of the unsafe any return type.**

If you need to accept an arbitrary function but don’t intend to call it, the type `() => void` is generally safer.

**In addition to using optional parameters or overloads to make functions that can accept a variety of fixed argument counts, _we can also define functions that take an unbounded number of arguments using rest parameters_.**  

A rest parameter appears after all other parameters, and uses the `...` syntax:
```typescript 
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```
- In TypeScript, the type annotation on these parameters is implicitly `any[]` instead of `any`, and `any` type annotation given must be of the form `Array<T>` or `T[]`, or a tuple type

## Rest Arguments

Conversely, we can provide a variable number of arguments from an array using the spread syntax. 

For example, the `push` method of arrays takes any number of arguments:
```typescript 
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

In general, TypeScript does not assume that arrays are immutable.  
This can lead to some surprising behavior:
```typescript 
const args = [8, 5];
const angle = Math.atan2(...args);
// A spread argument must either have a tuple type or be passed to a rest parameter.
```
- Inferred type (of `args`) is `number[]` -- "an array with zero or more numbers", not specifically two numbers

In general a const context is the most straightforward solution:
- [`as const`](https://stackoverflow.com/questions/66993264/what-does-the-as-const-mean-in-typescript-and-what-is-its-use-case)
```typescript 
// Inferred as "2-length" tuple via as const
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```
- Using rest arguments may require turning on downlevelIteration when targeting older runtimes.

You can use ***parameter destructuring to conveniently unpack objects provided as an argument into one or more local variables in the function body.***
In JavaScript, it looks like this:
```typescript 
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

The type annotation for the object goes after the destructuring syntax:
```typescript 
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

This can look a bit verbose, but you can use a named type here as well:
```typescript 
// Same as prior example
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

## `void` return type

The `void` return type for functions can produce some unusual, but expected behavior.
- Contextual typing with a return type of `void` does not force functions to not return something. 

Another way to say this is a contextual function type with a `void` return type (`type vf = () => void`), when implemented, can **return any other value, but it will be ignored.**

Thus, the following implementations of the `type () => void` are valid:
```typescript 
type voidFunc = () => void;
 
const f1: voidFunc = () => {
  return true;
};
 
const f2: voidFunc = () => true;
 
const f3: voidFunc = function () {
  return true;
};
```

And when the return value of one of these functions is assigned to another variable, it will retain the type of `void`:
```typescript 
const v1 = f1(); 
const v2 = f2();
const v3 = f3();
```

This behavior exists so that the following code is valid even though `Array.prototype.push` returns a number and the `Array.prototype.forEach` method expects a function with a return type of `void`.
```typescript 
const src = [1, 2, 3];
const dst = [0];
 
src.forEach((el) => dst.push(el));
```

There is one other special case to be aware of, when a literal function definition has a `void` return type, that function must not return anything.
```typescript 
function f2(): void {
  // @ts-expect-error
  return true;
}
 
const f3 = function (): void {
  // @ts-expect-error
  return true;
};
```
