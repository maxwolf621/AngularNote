###### tags: `Angular`

# Typescript & Angular 

- [Typescript & Angular](#typescript--angular)
  - [Declare a type](#declare-a-type)
  - [Constructor](#constructor)
    - [Optional parameter (`?:`)](#optional-parameter-)
    - [Access-modifier parameter in constructor](#access-modifier-parameter-in-constructor)
  - [Property (Setter and Getter)](#property-setter-and-getter)
  - [import a module](#import-a-module)
  - [Arrow function](#arrow-function)
  - [Interface](#interface)
  - [Cohesion](#cohesion)
  - [With Angular](#with-angular)
  - [More Object-Orient Program way](#more-object-orient-program-way)
  - [Service and dependency Injection](#service-and-dependency-injection)
    - [Dependency Injection](#dependency-injection)

## Declare a type

Basic Syntax   
`variable : DataType`  

```typescript
S : string = "what up";
N : number = 1234;

arr0: any[] = [];
arr : any[] = [1,2,3,4,5];

s = "what up";
N = 1234;
```

## Constructor

```typescript
class Point{
    private x;
    private y;
    constructor(x : number , y : number)
}
```

### Optional parameter (`?:`)
```typescript
class Point{
    private x ;
    private y ;
    constructor(x? : number , y? : number){
        //..
    }
}
```

### Access-modifier parameter in constructor 
```typescript
class Point{
    constructor(private _x? : number , private _y? : number){
        //..
    }
}
```


## Property (Setter and Getter)

```typescript
class Point{

    // Attributes ...
    set x(value){
        //..
        this._x = value ;
    }
    
    get x(){
        return this._x;
    }
}

let point = new Point(1,2);
let x = point.x; // getter
point.x = 3 ;    // setter
```

## import a module 

Syntax   
`import { "moduleName" } from 'Where_The_Module_File_Locates'`  

For example
```typescript
import { Point } from './'
```

## Arrow function

```typescript
let log = function(message){
    console.log(message);
}
```

Arrow Function's Syntax `(parameter) => { function body }`
```typescript
let doLong = (message) => {
    console.log(message)
}

// or
let doLong = (message) => console.log(message)
```

## Interface

The following makes the code not so flexible
```typescript
let drawPoint = (point) =>{
    //...
}

drawPoint({
    x:1,
    y:2
})

let drawPoint = (point : {x:number , y:number})=>{
    //....
}
```

Using `interface`
```typescript
interface Point{
    x:number,
    y:number,
    draw: () => void
}

let drawPoint = (point : Point) =>{
    //...
}

// As object 
drawPoint({
    x:1,
    y:2
})
```


## Cohesion

The following causes Cohesion Problem
```typescript
// Parameter pointA and pointB would have same values
let GetDistance = (pointA : Point , pointB : Point){
    //...
}
drawPoint({
    x:1,
    y:2
})
```

To deal with it we need `class`
```typescript
class Point{
    x:number;
    y:number;
    
    // getDistance from this.Point to another_Point
    getDistance(another_Point : Point){
        //...
    }
}
```


## With Angular

[Tutorial](https://www.youtube.com/watch?v=k5E2AVpwsko)

Angular components 
- Data
- Logic
- HTML Template

- Module in Angular
  > Group of components


```typescript
// A course.component.ts

@Component({
    // selector : id , 
    // template: let the id wear something on
    selector : 'courses',
    template : "<h2>Courses</h2>"
})
export class CoursesComponent{
    
}
```

app.component.html
```html
<!-- selector will find out this
and `template` will replace it with <h2>Courses</h2> 
-->
<course></course>
```

Add the component to Module
```typescript
// A app.module.ts
import {AppComponent} from './app.component';
import {CoursesComponent} from './courses.component'
@NgMoudle({
    declarations:[
        AppComponent,
        // using CLI to generate component
        //     this component will automatically be added
        CoursesComponent
    ],

})
export class AppModule {}
```


Using `ng g c COMPONENTNAME`in CLI to generate a new component

```bash
ng g c course
create src/app/course/course.component.css
create src/app/course/course.component.html
create src/app/course/course.component.spec.ts
create src/app/course/course.component.ts
update src/app/app.module.ts
```


## More Object-Orient Program way

```typescript
import { Component } from '@angular/core'

@Component({
    selector : 'courses',
    template : '<h2>{{getTitle()}}}</h2>'
})
export class CourseComponent{
    title = "list of courses"
    getTitle(){
        return this.title;
    }
}
```

With items
```typescript
import { Component } from '@angular/core'

@Component({
    selector : 'courses',
    template : '
        <h2>{{title}}}</h2>
        <ul>
            <li *ngFor="let course of courses"></li>
            {{course}}
        </ul>
        '

})
export class CourseComponent{
    title = "list of courses"
    courses = ["course1, course2, course3"]
}
```



## Service and dependency Injection

[More Details](Dependency%20Injection.md)

- Service
  > Decoupling

COMPONENT : `coursecomponent.ts`
```typescript
import { Component } from '@angular/core'

@Component({
    selector : 'courses',
    template : '
        <h2>{{title}}}</h2>
        <ul>
            <li *ngFor="let course of courses"></li>
            {{course}}
        </ul>
        '

})
export class CourseComponent{
    title = "list of courses
    courses;
    
    constructor(){
        // create service
        let service = new CourseService();
        courses = service.getCourses;
    }
}
```


SERVICE : `courses.service.ts`
```typescript
export class CoursesService{
    courses = ["course1, course2, course3"]
    
    getCourses()
    {
        return this.courses;
    }
}
```

### Dependency Injection 

**To shared singleton instance of the module for other modules**

Without Dependency Injection A object of `CourseComponent` is invoked and also a new instance of `CourseServce` is invoked too 

```typescript
import { Component } from '@angular/core'
import { CourseService } from '...'
@Component({
    selector : 'courses',
    template : '
        <h2>{{title}}}</h2>
        <ul>
            <li *ngFor="let course of courses"></li>
            {{course}}
        </ul>
        '
})
export class CourseComponent{
    title = "list of courses
    courses;
    
    constructor(){
        // Not flexible 
        //  it will also cause multiple instances 
        let service = new CourseService();
        this.courses = service.getCourses();
    }
}
```

To share the same instance with modules which need to use `CourseService`, we must provide `CourseService` to array of `providers` in `@ngModule` 
```typescript
import {AppComponent} from './app.component';
import {CoursesComponent} from './courses.component'
@NgModule({
    declarations:[
        AppComponent,
        // using CLI to generate component
        //     this will automatically be added
        CoursesComponent
    ],
    // Dependencies (Singpleton Instance)
    providers:[
        CourseService
    ],
    // ....
})
export class AppModule {}
```

Once `providers:[...]` in `@NgModule` has `CourseService` in it.

Say a module (e.g. `CourseComponent`) want to use Component `CourseService` then ...
```typescript
constructor(service : CourseService){
    /**
      * Dependence injection to this constructor by Providers
      * from AppModule
      */
    this.courses = service.getCourses();
}
```
