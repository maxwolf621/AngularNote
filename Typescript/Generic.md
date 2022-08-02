# Generic 

- [Generic](#generic)
  - [Generic Constraints (`extends`)](#generic-constraints-extends)
    - [Multiple generic](#multiple-generic)
  - [Generic Interface](#generic-interface)

```typescript
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

// These declarations are available
createArray<string>(3, 'x'); 
createArray<number>(3, 5); 
createArray(3, 'x'); 
```

## Generic Constraints (`extends`)

Not all `T` has the `length()` methods
```typescript
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);
    return arg;
}   
```
 
Prevent from such problem by using `<T extends Class_Name>`
```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}
```

### Multiple generic

```typescript
function copyFields<T extends U, U>(target: T, source: U): T {
    for (let id in source) {
        target[id] = (<T>source)[id];
    }
    return target;
}

let x = { a: 1, b: 2, c: 3, d: 4 };

copyFields(x, { b: 10, d: 20 });
```

## Generic Interface

```typescript
interface CreateArrayFunc {
    <T> (length: number, value: T): Array<T>;
}

let createArray: CreateArrayFunc;

createArray = function <T> (length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```
