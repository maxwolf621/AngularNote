# Tuple

- [Tuple](#tuple)
  - [`never`](#never)
  - [`unknown`](#unknown)
    - [Difference btw `any` and `unknown`](#difference-btw-any-and-unknown)
  - [`typeof` and `instanceof`](#typeof-and-instanceof)

```typescript
// Declare a tuple type
let x: [string, number];

// Initialize 
x = ["string", 1]; 

x[0] = "string";
x[1] = 1;
```

## `never`

`never` built for function that **having not returned value** (e.g. throw Error) or **infinity loop**
- `never` is not `void`


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

[unkown vs any](https://stackoverflow.com/questions/51439843/unknown-vs-any)   


### Difference btw `any` and `unknown`

在 TypeScript 中，僅有 Any 與 Unknown 型別可以被存放任意值，作為更嚴謹的 Unknown 型別，就會有更嚴格的規範。  
**`unknown` which is the type-safe counterpart of any**. 
- Anything is assignable to `unknown`, but `unknown` isn't assignable to anything but itself and `any` without a type assertion or a control flow based narrowing.   
Likewise, **no operations are permitted on an `unknown` without first asserting or narrowing to a more specific type.**

```typescript
// Anything is assiable to unknown
let value: unknown = 3;
value = "Ian";
value = undefined;
value = null;
value = true;
let value1: unknown = value;   // anything is assiable to unkown
let value2: any = value;       // anything is assiable to any

// Only unkown is assiable to itself
let value3: number = value;   // Error 
let value4: boolean = value;  // Error
let value5: string = value;   // Error
```
```typescript
let vAny: any = 10;          // We can assign anything to any
let vUnknown: unknown =  10; // We can assign anything to unknown just like any 
let s1: string = vAny;       // Any is assignable to anything 
let s2: string = vUnknown;   // Invalid; we can't assign vUnknown to any other type (without an explicit assertion)
vAny.method();     // OK; anything goes with any
vUnknown.method(); // Not Ok ; we don't know anything about this variable

let value: unknown;
let value2: any;
value.substring();  // Error : Object is of type 'unknown'
value2.substring(); // OK (may cause accidents)
```

## `typeof` and `instanceof`

通常檢測原始型別都是為用 `typeof`
```typescript
typeof value === ''
```

通常**廣義物件或類別（Class）**建造出來的物件則是會用`instanceof`
```typescript
someObject instanceof ObjectBelongingClass
```
