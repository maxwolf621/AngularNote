# Structural Directive

- [`*` Structural Directives](https://ithelp.ithome.com.tw/articles/10195273)

在Angular我們利用利用`*`來實現Structural Directive  
```html
<div *ngIf="hero" >{{hero.name}}</div>

<!-- is equivalent to -->
<ng-template [ngIf]="hero">
  <div>{{hero.name}}</div>
</ng-template>
```
- `*`會將`ngIf`改為一個Attribute Bind(`[..] = ".."`) 
- `*ngIf`隱藏掉的物件，和我們使用CSS去show、hide在意義上是完全不同的
  - 如果我們需要在show或hide物件的同時執行一些特殊的指令，可以用Lifecycle Hooks來撰寫此時要做的事情。
  - **因為這可以避免過多的dom元素拖累網頁效能，若單純使用css去hide、show元素(只是隱藏而已)，所有的監聽器、物件依舊會在背景執行，這會讓效能變得不佳**
- `ng-template`並不會一開始就顯示在畫面上，而是通過Directive操作裡面的dom並將要顯示的template添加在dom之中

## 在`.html`套用Multiple Structural Directives 

**我們可以將許多`屬性Directives`寫在同一個host element上，但同一個host element只能夠有一個Structural Directives**   
所以在一般的狀態下,**如果需要兩個TAGS,則會將HTML利用一些不會影響結構的TAG來做多層的Structural Directives控制**   

```html
<!-- 
利用HTML tag
  ( <div> ... </die> 以及 
    <span> ... </span>)
進行多層Structural Directives 
*ngIf 以及 *ngFor 控制
-->
<div *ngIf="hero">
    <span *ngFor="hero of heroes">{{hero.name}} </span>
</div>
```

但有時候狀況不允許任何多餘的TAG在裡面，e.g. 下拉選單`select`  
若有一個區域選單`select`裡面的`option`內容要由`*ngFor`來產生，但是當`city`未選擇時，`select`又希望不要有任何下拉選單, 這時後我們在`option`上的確同時會需要放許多個結構指令，但**Angular不允許同一個標籤上放兩個Structural Directives**如果我們多包一層`span`去包裡面的內容，會發現因為`select`內不允許`span`標籤，會造成讀不到`option`下拉選單，即便已經選擇了`city`
```html 
<div>
  Pick your favorite hero
  (<label>
    <input type="checkbox" 
           checked (change)="showSad = !showSad">
    show sad
  </label>)
</div>

<select [(ngModel)]="hero">
  <!-- two span tags-->
  <span *ngFor="let h of heroes">
    <span *ngIf="showSad || h.emotion !== 'sad'">
      <option [ngValue]="h">{{h.name}} ({{h.emotion}})</option>
    </span>
  </span>
</select>
```
![image](https://user-images.githubusercontent.com/68631186/129431539-5f8ffb4c-92e4-4dd8-b8eb-b78a687f6ba6.png)

這時候就可以改用`<ng-container *structural directive"...">`來實現多層結構指令
```html
<select [(ngModel)]="hero">
  <ng-container *ngFor="let h of heroes">
    <ng-container *ngIf="showSad || h.emotion !== 'sad'">
      <option [ngValue]="h">{{h.name}} ({{h.emotion}})
      </option>
    </ng-container>
  </ng-container>
</select>
```
![image](https://user-images.githubusercontent.com/68631186/129431979-33264c35-0cf9-4998-aaaa-c03294e83fc4.gif)

## Custom Structural Directive

```typescript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';
/**
 * Add the template content to the DOM 
 * unless the condition is true.
 */
@Directive({ selector: '[appUnless]'})
export class UnlessDirective {
  private hasView = false;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef) { }

  @Input() set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}
```
```html
<p *appUnless="condition" class="unless a">
  (A) This paragraph is displayed because the condition is false.
</p>
```
