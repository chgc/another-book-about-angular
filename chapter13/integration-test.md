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

## Providing the dependencies

## Getting the dependencies

## providing stubs

## Route parameters

## RouterOutlet components

## Shallow component

## Attribute directives

## Asynchronous operations





