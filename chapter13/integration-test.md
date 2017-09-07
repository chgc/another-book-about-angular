# 整合測試

整合測試主要的目的，是要測試 Component 的 template 的動作是否能如我們所預期的方式運作，而這是單元測試不能測試到的範圍

Angular 也貼心的準備 Helper 方法，`TestBed`，來幫助我們完成測試 Component 環境的設定

# TestBed

TestBed 是 Angular 提供的小幫手來建立測試用的 Module 環境，基本的用法如下

```typescript
import { async, ComponentFixture, TestBed } from '@angular/core/testing';
import { By } from '@angular/platform-browser';
import { DebugElement } from '@angular/core';

...
let component: VoteComponent;
let fixture: ComponentFixture<VoteComponent>;
let de: DebugElement;
let el: HTMLElement;
... 
beforeEach(async(() => {
   TestBed.configureTestingModule({
     declarations: [ VoteComponent ]  // declare the test component
   })
   .compileComponents(); // compile template and css
 }));

beforeEach(() => {
    fixture = TestBed.createComponent(VoteComponent);
    component = fixture.componentInstance; // 取得 component instance
    fixture.detectChanges(); // 執行 CD，這行會觸發 ngOnInit()，僅限第一次執行
  
    // query for the title <h1> by CSS element selector
    de = fixture.debugElement.query(By.css('p')); // 使用標轉的 CSS Selector 尋找並取得 debugElement
    el = de.nativeElement;
  });
```

`configuratTestingModule` 內的物件結構，與設定 `@NgModule` 是一樣的，在 `beforeEach` 內設定，可以確保每個測試案例不會受到其他測試案例結果的影響，當 `TestingModule` 設定完成後，可以透過 `createComponent` 的方式建立相對於 component 的 `fixtureComponent`，這個 `fixtureComponent` 將提供完整的 component 本身與對應的 template 內容

這裡有一個要留意的事情是，第一個 `beforeEach` 有使用 `async` 這個關鍵字讓包在裡面的函式變成非同步的處理方式，當 component 的 template 是單獨一個檔案時，因為有 IO 的非同步行為，所以需要 `async` 的幫忙讓非同步變成同步行為的處理方式，但是如果是使用 webpack 作為建置工具時，其實是不需要使用 `async` 的，因為 webpack 會將獨立的 html 檔案變成 inline template 的模式  

`fixture.detectChanges()` 是手動觸發 changeDetector 的方法，任何變數異動後要更新到 template 上時，都必須執行 `detectChanges()`，當然，也可以設定自動偵測異動並執行更新動作，透過以下的設定即可達成

```typescript
import { ComponentFixtureAutoDetect } from '@angular/core/testing';
...
TestBed.configureTestingModule({
  ...
  providers: [
    { provide: ComponentFixtureAutoDetect, useValue: true }
  ]
})
```

當這樣子設定完成後，之後的測試案例內，就不需要執行 `fixture.detectChages()` 了，但是請注意，預設的 `detechChages() ` 只為在非同步的事件觸發時才會被執行，例如 promise resolution、timers 或是 DOM Events，上述行為不包含直接修改變數值，因為這是屬於同步的行為，在這情況下，還是得自行執行 `fixture.detectChanges()`

```typescript
it('should still see original title after comp.title change', () => {
  const oldTitle = comp.title;
  comp.title = 'Test Title';
  // 改變 title 內容並不會觸發 template 顯示的更新，因屬於同步行為
  expect(el.textContent).toContain(oldTitle);
});

it('should display updated title after detectChanges', () => {
  comp.title = 'Test Title';
  fixture.detectChanges(); // 需手動觸發 detectChanges，更新 template 內容
  expect(el.textContent).toContain(comp.title);
});
```

所以，手動控制 detectChanges 會比開啟自動偵測機制來的好，我們就不需要去考慮什麼時候要自己執行 detectChanges，什麼時候不用，反正多執行也不會造成問題

# 測試範例

## property and class bindings

測試屬性 ( property ) 是一個很常見的測試情境，當一個變數值改變時，畫面上是否有正常顯示

```typescript
 it('should render total votes', () => {
    component.otherVotes = 20;
    component.myVote = 1;
	// totalValue = otherVotes + myVote;
    fixture.detectChanges();

    de = fixture.debugElement.query(By.css('.vote-count'));
    el = de.nativeElement;
    expect(el.innerText).toContain('21');
  });
```

```html
<span class="vote-count">{{ totalVotes }}</span>
```

寫法上要注意的還是 `fixture.detectChanges()`，記得要執行阿，不然測試會失敗

`debugElement` 本身有提供方法可以取得 `classes`、`style`、`attributes`、`properties` 等資訊，在這個測試案例，我們要測試當 `myVote == 1` 時，是否有 `hightlight` 的 css class 產生，而 debugElement.classes 是一個 keyValue 形式的物件，測試 Class 是否有正常的運作，測試案例可以這樣子寫

```html
<button class="upVote" [class.highlight]="myVote==1" (click)="upVote()">+</button>
```

```typescript
  it('should hightlight the upvote button is click', () => {
    component.myVote = 1;
    fixture.detectChanges();

    de = fixture.debugElement.query(By.css('.upVote'));    
    expect(de.classes['highlight']).toBeTruthy();
  });
```

## Event bindings

觸發事件的方法有兩種，一個是使用 debugElement 的 `triggerEventHandler` ，另外一種是使用 navtiveElement 轉型成 HTMLElement 後，操作 HTMLElement 的事件，這兩種方式都可以達到效果

```typescript
// 方法1
it('should click upVote and totalValue is 1', () => {
    const button = fixture.debugElement.query(By.css('.upVote'));
    button.triggerEventHandler('click', null);
    expect(component.totalVotes).toBe(1);
  });
```

```typescript
// 方法2
it('should click upVote and totalValue is 1', () => {
    de = fixture.debugElement.query(By.css('.upVote'));
    el = de.nativeElement;
    el.click();
    expect(component.totalVotes).toBe(1);
  });
```

到這個階段，或許會有一個問題，這個跟直接觸發 `component.upVote()` 後在檢查 `totalVotes` 的結果有什麼差別呢? 整合測試是要確保 template 上的行為是可以正常執行的，有時候函式在單元測試內是測試成功的，但是 template 上會因為沒有正常實作而造成測試失敗，這也是單元測試與整合測試的差異了。

## Dependencies

### Providing the dependencies

一個 Component 通常都會注入其他的 service，在測試時又該怎麼處理呢? 回想看看 `TestBed` 的功能是什麼，是設定一個測試用的 module，既然是 module，providers 和 imports 的動作就跟平常在設定 `@NgModules` 的方式是一模一樣的

假設 TodosComponent 有注入 TodoService，TodoService 有注入 HttpClient 服務。

```typescript
export class TodosComponent implements OnInit {
  todos = [];
  constructor(private service: TodoService) {}
  ...
}
```

測試檔案的內容於 `beforeEach` 的區塊，加上 imports 與 providers 兩個區塊，並將所需要的 service 與 modules 設定進去

```typescript
...
beforeEach(
    async(() => {
      TestBed.configureTestingModule({
        imports: [HttpClientModule],
        declarations: [TodosComponent],
        providers: [TodoService]
      }).compileComponents();
    })
  );
...
```

當這樣子設定完成後，providing service 的部分就已經完成了

### Getting the dependencies

Angular 設定 provider 的地方有兩個，`@NgModule` 與 `@Component` 內都可以設定 providers，因為設定位置的不一樣，所以取得的方式也會有所不同

如果 service 是設定在 `@NgModule` 內時，取得 service 的方式如下

```typescript
const service = TestBed.get(TodoService);
```

如果 service 是設定在 `@Component` 內時，取得 service 的方式如下

```typescript
const service = fixture.debugElement.injector.get(TodoService);
```

在整合測試時，我們還是不希望依賴外部引用的 service ，這裡的處理方式會跟單元測試的方式一樣，透過 `spyOn` 的方式控制 service 的行為

### providing stubs

有時候 component 所使用的 service 會遇到測試困難，例如路由。有時為了簡化測試的複雜度，會使用 `stubs` 的手法簡化，與其使用真的 service，不如自己建立一個簡單又符合目前所需的 service class 即可，也感謝 Angular 的 DI 機制，讓這一切變簡單了

```typescript
export class TodosComponent implements OnInit {
  constructor(private router: Router, private route: ActivatedRoute) {}

  ngOnInit() {
     this.route.params.subscribe(params => {
      if (params['id'] === 0) {
        this.router.navigate(['not-found']);
      }
    });
  }

  save() {
    this.router.navigate(['/dash']);
  }
}

```

Router 本身的功能很複雜，要測試的項目又很多，所以簡化的方式就是建立一個 RouterStub class 替換真的 Router

```typescript
class RouterStub {
  navigate(params) {}
}

class ActivatedRouteStub {
  params: Observable<any> = Observable.empty();
}
...
beforeEach(
  async(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientModule],
      declarations: [TodosComponent],
 	  providers: [          
          { provide: Router, useClass: RouterStub },
          { provide: ActivatedRoute, useClass: ActivatedRouteStub }
        ]
    }).compileComponents();
  })
);
```

這樣子的手法就可以大大的簡化測試的難度，這手法適用於其他第三方套件情境

## Route

### Navigation

由於在上一小節將 Router 與 ActivatedRoute 都用假的 class 替換掉了，所以這裡的測試就變簡單了

測試當某動作完成後，是否有正確的呼叫 router.navigate 函式，可以使用 `toHaveBeenCalledWith` 的方法來檢查

```typescript
 it('should redirect user to dash page', () => {
    const router = TestBed.get(Router);
    const spy = spyOn(router, 'navigate');

    component.save();
    // this.router.navigate(['dash']); 
    // 測試傳入引數是否正確
    expect(spy).toHaveBeenCalledWith(['dash']);
  });
```

### Parameters

測試路由參數的方式跟測試路由轉換的方式很類似，但還是要稍微修改一下 ActiveatedRouteStub 的內容，我們必須建立一個方法可以讓外部使用者將要設定路由參數傳入，修改如下

```typescript
class ActivatedRouteStub {
  private subject = new Subject();
  get params() {
    return this.subject.asObservable();
  }

  push(value) {
    this.subject.next(value);
  }
}
```

透過 RxJS Subject 的特性，可以很簡單的完成這個 `params.subscribe` 的功能，接下來就是測試在 `ngOnInit` 內的功能是否正常

```typescript
ngOnInit() {
  this.route.params.subscribe(params => {
    if (params['id'] === 0) {
      this.router.navigate(['not-found']);
    }
  });
}
```

當路由參數  id 是 0 時，會轉址到 `not-found` 的頁面

```typescript
it('should redirect user to NotFound page', () => {
    const router = TestBed.get(Router);
    const spy = spyOn(router, 'navigate');

    const route: ActivatedRouteStub = TestBed.get(ActivatedRoute);
    route.push({ id: 0 });

    expect(spy).toHaveBeenCalledWith(['not-found']);
});
```



## RouterOutlet components

`<router-outlet></router-outlet>` 是搭配路由設定顯示 Component 的標籤，一但沒有這個就無法正常地顯示 component 內容，那要怎麼確保這個標籤不會被誤刪呢? 就是寫個測試來保護他

```typescript
import { RouterOutlet } from '@angular/router';
...  
it('should have a route-outlet tag', () => {
    const de = fixture.debugElement.query(By.directive(RouterOutlet));
    expect(de).not.toBeNull();
  });
```



## Shallow component



## Attribute directives



## Asynchronous operations







