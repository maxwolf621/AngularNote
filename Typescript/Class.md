# Class 

- [Class](#class)
  - [Class](#class-1)
  - [Interface](#interface)
  - [Abstract Class](#abstract-class)

## Class 

```typescript
class Animal {
    public name: string;
    constructor(name: string) {
        this.name = name;
    }    
    sayHi() {
        return `My name is ${this.name}`;
    }
}
class Cat extends Animal{
    constructor(name: string) {
        super(name); 
        console.log(this.name);    
    }
    sayHi() {
        return 'Meow, ' + super.sayHi(); 
    }
    sayMeow(){
        return 'Meow~';
    }
}
let c = new Cat('Tom'); // Tom
console.log(c.sayHi()); // Meow, My name is Tom
```

## Interface 

```typescript
interface Meow{
    sayMeow(): any;
}
interface Hi{
    sayHi() :any;
}
class Cat extends Animal implements Meow, Hi{
    constructor(name: string) {
        super(name); 
        console.log(this.name);    
    }

    // Implementations
    sayHi() {
        return 'Meow, ' + super.sayHi(); 
    }
    sayMeow(){
        return 'Meow~';
    }
}
let c = new Cat('Tom'); // Tom
console.log(c.sayHi()); // Meow, My name is Tom
```

## Abstract Class

```typescript
abstract class Animal {
    
    public name: string;
    
    constructor(name: string) {
        this.name = name;
    }    
    
    sayHi() {
        return `My name is ${this.name}`;
    }
}
class Cat extends Animal{
    constructor(name: string) {
        super(name); // 呼叫父類別的 constructor(name)
        console.log(this.name);    
    }
    sayHi() {
        return 'Meow, ' + super.sayHi(); // 呼叫父類別的 sayHi()    
    }
    sayMeow(){
        return 'Meow~';
    }
}
let c = new Cat('Tom'); // Tom
console.log(c.sayHi()); // Meow, My name is Tom
```