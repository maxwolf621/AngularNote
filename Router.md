# Router

### Note Taking from
[Leo](https://ithelp.ithome.com.tw/articles/10207918)  
[Leo Router重點正理](https://ithelp.ithome.com.tw/articles/10209035)  
## Angular 的路由做了什麼？

- **負責重新配置頁面中應該顯示哪些 Component**
- 負責儲存頁面中 Component 的路由狀態
- **路由狀態定義了在某個路由的時候應該顯示哪些 Component**
- **路由狀態最重要的就是記錄路徑與 Component 之間的關係**



Root `app-routing.module.ts`
```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

const routes: Routes = [];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

Router會被注入到root `app.module.ts`中的`@ngModule.import`內
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

In `app.component.html`
```
<router-outlet></router-outlet>
```
- Angular 的路由機制靠的是我們放在 Template 裡的路由插座 - <router-outlet></router-outlet> 。


假如有兩個Routers
```typescript
import { AboutComponent } from './about/about.component';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  {
    path: 'home',
    component: HomeComponent
  },
  {
    path: 'about',
    component: AboutComponent
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes, {
      enableTracing: true,
      userHash : true
  })],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
- `enableTracing : true`在console印出listen router的trace


## Directive for Router

Router's directive 分成兩個
1. `<a></a>` 用的 RouterLinkWithHref 
2. 其他非`<a></a>`元素用的 RouterLink 

我們可以利用directive`<.... [routerLink]="'/path_name'">`在`.html`建立路徑達到轉頁
```
<ul>
  <li><a [routerLink]="'/home'">Home</a></li>
  <li><a [routerLink]="'/about'">About</a></li>
</ul>
```


## 萬用路由

```typescript
const routers: Routers:[
    {
        path: '**', 
        redirectTo: 'home', 
        pathMatch: 'full'
    }
];
```
- 當所在的路徑不屬於`const routers`內所有設定的路徑則統一redirect to `home` 這個路徑
- 路由的路徑設定是有順序性的,如果萬用路由放在前面, 其他在後面的routers會被略過,所以一般我們都把萬用路由放置`const routers` array的tail

## Router Design Pattern 子路由

設定子路由的優點
- 可讀性高的 url ，如： /articles/10207918 。
- 延遲載入。
- 減少撰寫重複的程式碼。
- 預處理層。


我們可以設計一個component將類似或者給特定使用者使用的components放入對應的component
![](https://i.imgur.com/WAdXz1e.png)


屆時我們可以將Router的路徑分成Base以及Child,如下
```typescript
{
  path: '',
  component: LayoutComponent,
  children: [
    {
      path: 'home',
      component: HomeComponent
    },
    {
      path: 'about',
      component: AboutComponent
    }
  ]
}
```
- 只要路由有設定children的路由，再加上在Template裡有放 <router-outlet></router-outlet> 的話, Angular 的路由機制就會自動幫你向下找。


## 非Root的router

實際應用上，通常我們會將相關的功能包裝成一個一個的 NgModule ，而每個 NgModule，也其實都可以有自己的 RoutingModule。

建立一個module以及FeatureRoutingModule給`feature` component使用
```console
# with --routing 則會另外建立一個FeatureRoutingModule的路由  
ng generate module feature --routing

ng generate component feature
```


```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { FeatureComponent } from './feature.component';

const routes: Routes = [
  {
    path: 'feature',
    component: FeatureComponent
  }
];

@NgModule({
  /**
   * only allow forChild
   */
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class FeatureRoutingModule { }
```
- 整個系統只有 AppRoutingModule 才會使用 forRoot ，其他的子路由模組都是使用`forChild`  


只要在`app.module.ts`內將該`FeatureModule.ts`註冊入`@ngModule.import`內就可見入路由路徑,由於路由是有順序性的所以不能放在root router module之前

例如
```typescript
imports: [
  BrowserModule,
  AppRoutingModule,
  FeatureModule   /* Feature Component的路徑會放在萬用路由的後面 */
]

// instead of

imports: [
  BrowserModule,
  FeatureModule
  AppRoutingModule,
]
```
- 所以AppRoutingModule總是會放在`@ngModule import`的tail

## Lazy Loading for 非root router module

Using `loadChildren`


In `FeatureRoutingModule.ts` 
```
const routes: Routes = [
  {
    path: '', // 將 feature 改成空字串
    component: FeatureComponent
  }
];
```

then add it in  `app-routing.module.ts`
```
// Angular 8+
{
  path: 'feature',
loadChildren: () => import('./feature/feature.module').then(module => module.FeatureModule)
}
```

## Pre-loading

預先載入跟延遲載入很像，差別只在於，延遲模組是要進入該路由的時候即時載入；預先載入是在頁面初始化的時候就把所有可延遲載入的模組透過背景非同步地下載，不會影響畫面的顯示或使用者的操作。

add new attribute `preLoadingStrategy : PreloadAllModules` in `@NgModule.imports` in root router module

```typescript
import { PreloadAllModules } from '@angular/router';

//....

@NgModule({
  imports: [RouterModule.forRoot(routes, {
    useHash: true,
    preloadingStrategy: PreloadAllModules
  })],
  exports: [RouterModule],
  providers: []
})
```

## path query parameter 

路由參數有時候我們會需透過路由傳遞參數，而傳遞參數的方法有兩種：

## 使用 Query String 表示法



Ex: `http://localhost:4200/products?id=101`


```typescript
this.router.navigate(['products'], {
    queryParams: {
      id: 101
    }
});
```

```html
<a [routerLink]="['products']" [queryParams]="{ id: 101 }">Prodcuts</a>
```


取得path的參數的方式
1. using `queryParams.subscribe((queryParams) => { ...})`
2. using `.snapshot.queryParams['..']`
```typescript
export class ProductsComponent implements OnInit {

  constructor(private route: ActivatedRoute) { }

  ngOnInit() {
  
      // 第一種方式（較推薦）
      this.route.queryParams.subscribe((queryParams) => {
        console.log(queryParams['id']);
      });
      
      // 第二種方式
      console.log(this.route.snapshot.queryParams['id']);
  
  }

}
```


## matrix URL notation 表示法

以`;`將parameters隔開

Ex：`http://localhost:4200/products;id=101`

```typescript
this.router.navigate(['path_name'], {queryParam_1 : xxx , queryParam_2 : yyy , ...});
this.router.navigate(['products', { id: 101 }]);
```

```html
<a [routerLink]="['products', { id: 101 }]">Products</a>
```

取得Path的參數的方式：
```typescript
export class ProductsComponent implements OnInit {

  constructor(private route: ActivatedRoute) { }

  ngOnInit() {
  
      // 第一種方式
      console.log(this.route.params['id']);
      
      // 第二種方式
      console.log(this.route.snapshot.params['id']);
  
  }

}
```

## [添加換頁效果](https://ithelp.ithome.com.tw/articles/10195347)

需要`BrowserAnimationsModule`這個動態效果模組。

```typescript
import { animate, AnimationEntryMetadata, state, style, transition, trigger } from '@angular/core';

// Component transition animations
export const slideInDownAnimation: AnimationEntryMetadata =
  trigger('routeAnimation', [
    state('*',
      style({
        opacity: 1,
        transform: 'translateX(0)'
      })
    ),
    transition(':enter', [
      style({
        opacity: 0,
        transform: 'translateX(-100%)'
      }),
      animate('0.2s ease-in')
    ]),
    transition(':leave', [
      animate('0.5s ease-out', style({
        opacity: 0,
        transform: 'translateY(100%)'
      }))
    ])
  ]);
```

然後在`src/app/heroes/hero-detail.component.ts`增加使用這個動態效果的綁定
```typescript
@HostBinding('@routeAnimation') routeAnimation = true;
@HostBinding('style.display')   display = 'block';
@HostBinding('style.position')  position = 'absolute';
```

## Child routing component

子路由元件與直接使用參數去定義路由，Router預設的狀況下，**若瀏覽器重新導航到相同的元件時，會重新使用該元件既有的實體，而不會重新創建**。因此在物件被重用的狀況下，該元件的ngOnInit只會被呼叫一次，即使是要顯示不同的內容資料。

但是**被創建的元件實體會在離開頁面時被銷毀並取消註冊**，因此在前面範例的heroes-routing.module.ts檔案中裡，所使用的導航設定方式，由於在瀏覽`HeroDetailComponent`之後，一定要先回到`HeroListComponent`，才能進入下一個Detail頁面。
```
const heroesRoutes: Routes = [
  { path: 'heroes',  component: HeroListComponent },
  { path: 'hero/:id', component: HeroDetailComponent }
];
```
這會造成因為在回到`HeroListComponent`時已把`HeroDetailComponent`刪除掉了，再選擇另一個英雄要查看細節時，又會再創立一個新的`HeroDetailComponent`。因此每次選擇不同的英雄時，組件都會重新創建。


如果想要保留頁面的狀態，就可以改使用子路由的方式來定義, for example 
```typescript
const crisisCenterRoutes: Routes = [
  {
    path: 'crisis-center',
    component: CrisisCenterComponent,
    children: [
      {
        path: '',
        component: CrisisListComponent,
        children: [
          {
            path: ':id',
            component: CrisisDetailComponent
          },
          {
            path: '',
            component: CrisisCenterHomeComponent
          }
        ]
      }
    ]
  }
];
```
- `CrisisDetailComponent`與`CrisisCenterHomeComponent`都是`CrisisListComponent`的子組件，因此在不同的子組件內的狀態得以被保存，直到頁面切換離開CrisisCenterComponent時才會將元件實體刪除，這可以讓我們在瀏覽不同CrisisDetail時，得以使用到Router預設的重用設定。

## 設定相對路徑

- `./` 是在目前的位置。
- `../` 在上一層的位置。

```typescript 
// Relative navigation back to the crises
this.router.navigate(['../', { id: crisisId, foo: 'foo' }], { relativeTo: this.route });
```