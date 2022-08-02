
# Object Types 
- [Object Types](#object-types)
  - [Reference](#reference)
  - [Object Types Introduction](#object-types-introduction)
  - [Optional Properties (`?`)](#optional-properties-)
  - [`readonly` Properties](#readonly-properties)
  - [Index Signatures](#index-signatures)
  - [Extending Types (`extends` and `&`)](#extending-types-extends-and-)
    - [Intersection Types (`&`)](#intersection-types-)
    - [Interfaces vs. Intersections](#interfaces-vs-intersections)
  - [Generic Object Types (`unknown` vs generic types `<T>`)](#generic-object-types-unknown-vs-generic-types-t)
    - [The Array Type](#the-array-type)
    - [The ReadonlyArray Type](#the-readonlyarray-type)


## Reference
[unkown vs any](https://stackoverflow.com/questions/51439843/unknown-vs-any)   


## Object Types Introduction

Object Types **GROUP** and **PASS** around data that is through objects   
```typescript 
// Using Object Types `{ object1, object2, ... , ... }`
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}

// via Interface
interface Person {
  name: string;
  age: number;
}
 
// via Type Alias 
type Person = {
  name: string;
  age: number;
};
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

## Optional Properties (`?`)

```typescript 
interface PaintOptions {
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

when we do under `strictNullChecks`, TypeScript will tell us they’re potentially `undefined`.
- For example
```typescript
function paintShape(opts: PaintOptions) {
  
  let xPos = opts.xPos; // (property) PaintOptions.xPos?: number | undefined
  let yPos = opts.yPos; // (property) PaintOptions.yPos?: number | undefined
  
  // ...
}
```


narrowing to handle `undefine`
```typescript 
// Handle `undefine` specifically
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos; // let xPos: number
  let yPos = opts.yPos === undefined ? 0 : opts.yPos; // let yPos: number
  // ...
}

// setting defaults for unspecified values 
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log("x coordinate at", xPos);// (parameter) xPos: number
  console.log("y coordinate at", yPos); // (parameter) yPos: number
  // ...
}
```

## `readonly` Properties

```typescript
interface SomeType {
  readonly prop: string;
}
 
function doSomething(obj: SomeType) {
  // We can read from 'obj.prop'.
  console.log(`prop has the value '${obj.prop}'.`);
 
  // But we can't re-assign it.
  obj.prop = "hello";

  // Cannot assign to 'prop' because it is a read-only property.
}
```

**`readonly` doesn't imply that a value is totally immutable**
```typescript
/**
  * update the value not the address 
  */
interface Home {
  readonly resident: { name: string; age: number };
}
 
function visitForBirthday(home: Home) {
  // We can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`);
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

```typescript
// Declare a tuple type
let x: [string, number];

// Initialize 
x = ["string", 1]; 

x[0] = "string";
x[1] = 1;
```

## Index Signatures

Sometimes you don’t know all the names of a type’s properties ahead of time(in advance), but you do know the shape of the values.    
In those cases you can use an index signature to describe the types of possible values   
```typescript
interface StringArray {
  // index : is number type
  // StringArray's properties : must be string type 
  [index: number]: string;
}
 
const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
const secondItem: string
```
- **This index signature states that when a `StringArray` is indexed with a `number`, it( will return a `string`.**   
- **An index signature property type must be either `string` or `number`.**   

It is possible to support both types of indexers.   
While `string` index signatures are a powerful way to describe the ***dictionary*** pattern, they also enforce that all properties match their return type.  

This is because a `string` index declares that `obj.property` is also available as `obj["property"]`.     
```typescript 
interface NumberDictionary {
  [index: string]: number;
  length: number; // ok
  
  name: string;
  // Property 'name' of type 'string' is not assignable to 'string' index type 'number'.
}
```

It's acceptable if the index signature is a union of the property types:
```typescript 
interface NumberOrStringDictionary {
  // indexed signature can be number or string
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string;   // ok, name is a string
}
```

Index signatures `readonly` in order to prevent assignment to their indices
```typescript 
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
 
let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
// Index signature in type 'ReadonlyStringArray' only permits reading.
```

## Extending Types (`extends` and `&`)

**The extends keyword on an interface allows us to effectively copy members from other named types, and add whatever new members we want.** 

This can be useful for cutting down the amount of type declaration boilerplate we have to write, and for signaling intent that several different declarations of the same property might be related. 

Interfaces can also extend from multiple types.
```typescript
interface Colorful {
  color: string;
}
 
interface Circle {
  radius: number;
}

// extend multiple types
interface ColorfulCircle extends Colorful, Circle {}
 
const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

### Intersection Types (`&`)

TypeScript provides another construct called intersection types that is mainly used to combine existing object types.

```typescript 
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;

function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}
 
// okay
draw({ color: "blue", 
       radius: 42 }); 
draw({ color: "red", 
        radius: 42 });
// Argument of type '{ color: string; radius: number; }' is not assignable to parameter of type 'Colorful & Circle'.
// Object literal may only specify known properties, but radius does not exist in type 'Colorful & Circle'. Did you mean to write 'radius'?
```

### Interfaces vs. Intersections

The principle difference between the two is how conflicts are handled, and that difference is typically one of the main reasons why you’d pick one over the other between an interface and a type alias of an intersection type.


Compiler units the same separate interface
```typescript
interface Cat {
    name: string;
}
interface Cat {
    age: number;
}

// compiler
interface Cat {
    name: string;
    age: number;
}
```

Interface can extend object type
```typescript
type Biological = {
  name: string
  age: number
};
interface Human extends Biological { 
  IQ: number; 
}
```


## Generic Object Types (`unknown` vs generic types `<T>`)

Use `unknown` instead of `any`, but that would mean that in cases where we already know the type of contents,assertions.
```typescript 
interface Box {
  contents: unknown;
}
 
let x: Box = {
  contents: "hello world",
};
 
// precautionary checks
if (typeof x.contents === "string") {
  console.log(x.contents.toLowerCase());
}
 
// or a type assertion
console.log((x.contents as string).toLowerCase());
```

One type safe approach would be to instead scaffold out different Box types for every type of contents.   
But that means we’ll have to create different functions, or overloads of functions, to operate on these types.
```typescript 
interface NumberBox {
  contents: number;
}
 
interface StringBox {
  contents: string;
}
 
interface BooleanBox {
  contents: boolean;
}

// Overloads
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}
```
That’s a lot of boilerplate.   

Instead, we can make a generic Box type which declares a generic type parameter.  
```typescript
// define generic type 
interface Box<Type> {
  contents: Type;
}
interface StringBox {
  contents: string;
}
 
let boxA: Box<string> = { contents: "hello" };
// (property) Box<string>.contents: string
 
let boxB: StringBox = { contents: "world" };
// (property) StringBox.contents: string

interface Apple {
  // ....
}
// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>;
```

This also means that we can avoid overloads entirely by instead using generic functions.
```typescript 
/**
 * function setContents(box: StringBox, newContents: string): void;
 * function setContents(box: NumberBox, newContents: number): void;
 * function setContents(box: BooleanBox, newContents: boolean): void;
 * function setContents(box: { contents: any }, newContents: any);
 * in to only one function 
 */
function setContents<T>(box: Box<T>, newContents: T) {
  box.contents = newContents;
}
```

It is worth noting that `type` aliases can also be generic.  
```typescript 
    interface             |         type 
interface Box<Type> {     |   type Box<Type> = {
  contents: Type;         |      contents: Type; 
}                         |   };  
```

**Since type aliases, unlike interfaces, can describe more than just object types, we can also use them to write other kinds of generic helper types.**
```typescript 
type OrNull<Type> = Type | null;
 
type OneOrMany<Type> = Type | Type[];
 
type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
           
type OneOrManyOrNull<Type> = OneOrMany<Type> | null
 
type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
               
type OneOrManyOrNullStrings = OneOrMany<string> | null;
```

### The Array Type

**Generic object types are often some sort of container type that work independently of the type of elements they contain.**     

the Array type. Whenever we write out types like `number[]` or `string[]`, that’s really just a shorthand for `Array<number>` and `Array<string>`.
```typescript
function doSomething(value: Array<string>) {
  // ...
}
 
let myArray: string[] = ["hello", "world"];
 
// either of these work!
doSomething(myArray);
doSomething(new Array("hello", "world"));
```

Array itself is a generic type.
```typescript 
interface Array<Type> {
  /**
   * Gets or sets the length of the array.
   */
  length: number;
 
  /**
   * Removes the last element from an array and returns it.
   */
  pop(): Type | undefined;
 
  /**
   * Appends new elements to an array, and returns the new length of the array.
   */
  push(...items: Type[]): number;
 
  // ...
}
```

### The ReadonlyArray Type

The `ReadonlyArray` is a special type that describes arrays that shouldn’t be changed.
```typescript 
function doStuff(values: ReadonlyArray<string>) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);
 
  // ...but we can't mutate 'values'.
  values.push("hello!");
  // Property 'push' does not exist on type 'readonly string[]'.
}
```
**Much like the readonly modifier for properties, it’s mainly a tool we can use for intent**. 

1. When we see a function that _RETURNS_ `ReadonlyArrays`, it tells us we’re not meant to change the contents at all.
2. when we see a function that _CONSUMES_ `ReadonlyArrays`, it tells us that we can pass any array into that function without worrying that it will change its contents.

Unlike Array, there isn’t a `ReadonlyArray` constructor that we can use.
- `ReadonlyArray` only refers to a type, but is being used as a value here.
```typescript 
new ReadonlyArray("red", "green", "blue");
```

Instead, we can assign regular Arrays to `ReadonlyArrays` type variable.
```typescript
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

Just as TypeScript provides a shorthand syntax for `Array<Type>` with `Type[]`, it also provides a shorthand syntax for `ReadonlyArray<Type>` with readonly `Type[]`.
```typescript 
function doStuff(values: readonly string[]) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);
 
  // ...but we can't mutate 'values'.
  values.push("hello!");
  // Property 'push' does not exist on type 'readonly string[]'.
}
```

Unlike the `readonly` property modifier, assignability isn’t bidirectional between regular Arrays and `ReadonlyArrays`.
```typescript 
let x: readonly string[] = [];
let y: string[] = [];
 
x = y;
y = x;
//  The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
```