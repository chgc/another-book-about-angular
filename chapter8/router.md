## 基本設定

Angular 設定路由的方式有兩種，`RouterModule.forRoot` 跟 `RouterModule.forChild`，這兩個方法都接受 `Routes`，而 `forRoot` 還可以額外傳入設定參數

`forRoot` 與 `forChild` 的差異是，`forRoot` 是只會註冊一次，而 `forChild` 可以有很多次，通常會出現在 `FeatureModule` 裡

通常設定路由的格式長得如此

```typescript
const appRoutes: Routes = [
  { path: 'crisis-center', component: CrisisListComponent },
  { path: 'hero/:id',      component: HeroDetailComponent },
  {
    path: 'heroes',
    component: HeroListComponent,
    data: { title: 'Heroes List' }
  },
  { path: '',
    redirectTo: '/heroes',
    pathMatch: 'full'
  },
  { path: '**', component: PageNotFoundComponent }
];

@NgModule({
  imports: [
    RouterModule.forRoot(
      appRoutes,
      { enableTracing: true } // <-- debugging purposes only
    )
    // other imports here
  ],
  ...
})
export class AppModule { }
```

如上面所說， `forRoot` 與 `forChild` 第一個參數是接受一個 Route 陣列，而 `Route` 這型別的介面是

```typescript
interface Route { 
  path?: string
  pathMatch?: string
  matcher?: UrlMatcher
  component?: Type<any>
  redirectTo?: string
  outlet?: string
  canActivate?: any[]
  canActivateChild?: any[]
  canDeactivate?: any[]
  canLoad?: any[]
  data?: Data
  resolve?: ResolveData
  children?: Routes
  loadChildren?: LoadChildren
  runGuardsAndResolvers?: RunGuardsAndResolvers
}
```

- `path` 為路徑設定

  - 可以為空白
  - 可以設定參數，格式為 `:params` ，可以透過 `ActivatedRoute.paramMap` 取得
  - 可以設定 `**` ，但必須放在最後

- `pathMatch` 路徑比對的方式，有 `prefix` 與 `full` ，預設值為 `prefix`

  - `prefix` 只會比對起始網址是否吻合，例如 `team/:id` 這條件 `/team/11/user` 就會吻合
  - `prefix` 當 `path` 為空白時，任何網址都會符合此路徑設定，當遇到此設定方式時，記得要放在其他路徑設定條件的後面
  - `full` 則會完整比對，所以空白路徑，就只有 `/` 根網址才會符合條件

- `matcher` 當 `path` 與 `pathMatch` 這兩者對於需求來說不足時，可以自定義的路徑比對規則

  ```typescript
   export function htmlFiles(url: UrlSegment[]) {
      return url.length === 1 && url[0].path.endsWith('.html') ? ({consumed: url}) : null;
   }
   
   export const routes = [{ matcher: htmlFiles, component: AnyComponent }];
  ```

- `component` 設定對應 path 要顯示的 `component`

- `redirectTo` 符合條件時要轉址的位置

- `outlet` 可指定要顯示在哪一個 `router-outlet` 上，畫面上可存在多個 `router-outlet`

  - 比較常見的使用情境是，共用網址參數，分別顯示在不同區域

    ```typescript
    [{
       path: 'parent/:id',
       children: [
         { path: 'a', component: MainChild },
         { path: 'b', component: AuxChild, outlet: 'aux' }
       ]
    }]
    ```

    ```html
    <router-outlet></router-outlet>
    <router-outlet name="aux"></router-outlet> // 對應 outlet 設定名稱
    ```

- `canActivate` 可設定一個陣列的檢查規則(service)，來決定是否可以進入這個路徑

- `canActivateChild` 可設定一系列的檢查規則(service)，來決定是否可以進入這個路徑下的子路由

- `canDeactivate` 可設定一系列檢查規則(service)，來決定是否可以離開這個路徑

- `canLoad` 可設定一系列的檢查規則(service)，來決定是否可以離開這個路徑

- `data` `component` 可透過 `ActivatedRout` 取得路由所設定的資料

- `resolve` `component` 可透過 `ActivatedRout` 取得路由所設定的非同步資料，會等非同步完成後才會進入 component

- `runGuardsAndResolvers` 設定 `guards` 和 `resolvers` 的觸發時間點。有三種模式可以設定

  1. `paramsChange` 預設行為，只有在`matrix ` 參數改變時才會觸發
  2. `paramsOrQueryParamsChange` 除了 `matrix` 參數，`query` 參數改變時也會觸發
  3. `always` 每次觸發，`onSameUrlNavigation`設定為 `always` 時，這個選項沒設定時，第二次觸發時，是不會觸發 `resolver` 事件的

- `children` 子路由設定

- `loadChildren` 設定要延遲載入 `module` 位置，Angular CLI 也會根據這個拆出獨立的 js 檔案

## RouterModule 參數

* `enableTracing` 在 console 地方顯示路由事件
* `useHash` 使用 URL fragment 替代 history API
* `initialNavigation` 停止初始瀏覽事件
* `errorHandler` 設定自訂錯誤處理方法
* `preloadingStrategy ` 設定預先載入策略
* `onSameUrlNavigation ` 設定路由遇到瀏覽相同網址時應該要做什麼事情，`reload` | `ignore` ，預設: `ignore`。這裡要留意的是此設定只會觸發路由事件的重整，並不會使 `component` 重新建立，這表示並不會觸發 `ngOnInit` 事件
* `paramsInheritanceStrategy` 設定路由在子路由時處理 `params`、`data`、`resolve` 資料的方法
  * `emptyOnly` 預設值，只會繼承沒有設定路徑或是 component 的規則
  * `always` 無條件的繼承父層的參數

