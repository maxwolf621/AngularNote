# QuickReview

- [QuickReview](#quickreview)
  - [Object Types as Function Parameter](#object-types-as-function-parameter)
    - [with (`?`) Optional Member](#with--optional-member)
    - [Union Type (`|`)](#union-type-)
  - [narrowing `undefined`](#narrowing-undefined)
  - [readonly](#readonly)
  - [Index Signatures](#index-signatures)
  - [Intersection Types `&`](#intersection-types-)
  - [Generic Type](#generic-type)
  - [assertions (`as`)](#assertions-as)
  - [Tuple Type](#tuple-type)
  - [Special Types](#special-types)
    - [`never`](#never)
    - [`unknown`](#unknown)
    - [`typeof variableName` and `instanceof className`](#typeof-variablename-and-instanceof-classname)
  - [Literal Types](#literal-types)
- [Function](#function)
  - [Function Type Expressions and function type alias](#function-type-expressions-and-function-type-alias)

## Object Types as Function Parameter
### with (`?`) Optional Member

```typescript
interface PaintOptions {
  shape: Shape;
  xPos?: number; // xPos is optional
  yPos?: number; // yPos is optional
}
 
// or
type PaintOptions {
  shape: Shape;
  xPos?: number; // xPos is optional
  yPos?: number; // yPos is optional
}

function paintShape(opts: PaintOptions) {
  // ...
}
 
const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

### Union Type (`|`) 

**TypeScript will only allow an operation if it is valid for every member of the union.**    
```typescript
function printId(id: number | string) {
  console.log(id.toUpperCase());
  
  Property 'toUpperCase' does not exist on type 'string | number'.
  Property 'toUpperCase' does not exist on type 'number'.
}
```

## narrowing `undefined`

Optional member or union member cause `undefined`. 
```typescript
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos; // let xPos: number
  let yPos = opts.yPos === undefined ? 0 : opts.yPos; // let yPos: number
  // ...
}
```

## readonly

**`readonly` doesn't imply that a value is totally immutable**

```typescript
interface Home {
  readonly resident: { name: string; age: number };
}
 
function visitForBirthday(home: Home) {
  home.resident.age++;
}
 
function evict(home: Home) {
  // But we can't write to the 'resident' property itself on a 'Home'.
  // It created a new address 
  home.resident = { 
    // Cannot assign to 'resident' because it is a read-only property.
    name: "Victor the Evictor",
    age: 42,
  };
}
```


## Index Signatures


- **This index signature states that when a `StringArray` is indexed with a `number`, it( will return a `string`.**   
- **An index signature property type must be either `string` or `number`.**   

A `string` index declares that `obj.property` is also available as `obj["property"]`.     
```typescript 
interface NumberDictionary {
  [index: string]: number;
  length: number; // ok
  
  name: string;
  // Property 'name' of type 'string' is not assignable to 'string' index type 'number'.
}


// union index signature
interface NumberOrStringDictionary {
  [index: string]: number | string;
  
  // index Signatures
  length: number; // ok, length is a number
  name: string;   // ok, name is a string
}

// readonly
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
 
let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
```


## Intersection Types `&`

```typescript
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;

function draw(circle: Colorful & Circle) {
    //....
}
 
// okay
draw({ color: "blue", 
       radius: 42 }); 
draw({ color: "red", 
        radius: 42 });
```


## Generic Type

```typescript 
function setContents<T>(box: Box<T>, newContents: T) {
  box.contents = newContents;
}


type OrNull<Type> = Type | null;

type OneOrMany<Type> = Type | Type[];
 
type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
           
type OneOrManyOrNull<Type> = OneOrMany<Type> | null
 
type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
               
type OneOrManyOrNullStrings = OneOrMany<string> | null;

function serContent<OrNull>(a : orNull){
    //...
}
```


## assertions (`as`)

using `as` if you already know the variable type (the type you know but typescript doesn't know)


## Tuple Type 

- A tuple with a rest element has no set length     
- It can has rest elements `...`
it only has a set of well-known elements in different positions. 
```typescript
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```
- **Tuples tend to be created and left un-modified in most code**, so annotating types as readonly tuples when possible is a good default.
```typescript
function doSomething(pair: readonly [string, number]) {
    pair[0] = "hello!" ;
    
    Cannot assign to '0' because it is a read-only property.
}
```

## Special Types 

### `never`

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

### `unknown`

What is the Difference btw `any` and `unknown` ? **`unknown` which is the type-safe counterpart of `any`**. 

- **Anything is assignable to `unknown`, but `unknown` isn't assignable to anything but itself** and `any` without a type assertion or a control flow based narrowing.   

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

###  `typeof variableName` and `instanceof className`

```typescript
typeof value === ''
someObject instanceof ObjectBelongingClass
```

## Literal Types

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

# Function

## Function Type Expressions and function type alias

```typescript 
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

function printToConsole(s: string) {
  console.log(s);
}

greeter(printToConsole);

type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

