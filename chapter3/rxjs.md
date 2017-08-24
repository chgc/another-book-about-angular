# RxJS

RxJS 主要元素有

* Observable：代表一系列的資料或是事件
* Observer：用來監聽 Observable 所發生的值
* Subscription：代表 Observable 的訂閱ID，主要目的是用來取消 Observable
* Operators：pure function, 提供一系列的功能可以操作 Observable
* Subject：等同於 EventEmitter，處理 multicasting 的唯一方法
* Schedulers

在這章節會詳細的一一介紹及提供可能使用情境

# Observable

## 解析 Observable

Observable 的執行生命週期是建立 ( creating )、訂閱 ( subscribing) 、 執行 ( executing )、回收 ( disposing )，一個完整的 Observable 都會經歷這四個階段

* 建立 Observable

  ```typescript
  var observable = Observable.create(observer => {
    var id = setInterval(() => {
      observer.next('hi')
    }, 1000);
  });
  ```

  RxJS 提供很多方法可以用來建立 Observable，會在之後的內容逐一介紹

* 訂閱 Observable

  ```typescript
  observable.subscribe({
    next: x => console.log('Observer got a next value: ' + x),
    error: err => console.error('Observer got an error: ' + err),
    complete: () => console.log('Observer got a complete notification'),
  });
  ```

  `subscribe` 是訂閱 Observable 的動作，會接收一個 observer 物件 (包含 next、error、complete 三階段)，會是接收三個函式，分別處理 next、error、complete 三個階段。在執行 `subscribe` 時，會回傳一個 `subscription`，這個 `subscription` 是用來取消 Observable 使用的

* 執行 Observable

  * 剛剛有提過 subscribe 內會有三個階段要處理，分別為 next、error、complete
    * next：Observable 正常執行時，任何值都會傳給負責處理 next 階段的函式
    * error：當 Observable 發生錯誤時，就會進入 error 階段，負責的函式也會收到一個參數
    * complete：當 Observable 正常執行結束後，就會進入 complete 階段，不會接受到任何參數
  * 一但進入 error 或是 complete 階段，即使 Observable 後續還有任何資料的發生，next 階段都不會接受到任何資料了。( RxJS 有提供 operators 可以負責處理 error 階段的函式，並可以切回正常階段 )

* 回收 Observable

  * 訂閱 Observable 會收到一個 subscription，執行 `unsubscribe()` 即可取消 Observable 的動作

    ```typescript
    var observable = Observable.from([10, 20, 30]);
    var subscription = observable.subscribe(x => console.log(x));
    // Later:
    subscription.unsubscribe();
    ```

# Operators

Operators 都屬 `pure function`，RxJS 要用的好，很大的因素在於 operators 的掌握及熟悉度，對 operators 越熟悉，RxJS 的威力越可以展現出來，但不要急，花點時間練習就可以掌握了。

Operators 主要分為 `creation`、 `transformation`、 `filtering`、 `combination`、 `multicasting`、 `error handling`、 `utility` 等類別，每個類別的功能及目的都不太一樣，有些 operators 會是其他 operators 的混搭版，之後會更仔細的說明。

## Creation Operators

### ajax

透過 Observable.ajax 可以呼叫 api 並取回資料，底層是使用 XMLHttpRequest 的方式實作。

**使用介面**

```typescript
Observable.ajax('/products');
Observable.ajax({ url: 'products', method: 'GET' }); // 可接受 option 物件
```

參數物件內有以下設定可使用

* url
* body
* method：GET、POST、PUT、PATCH、DELETE
* async
* headers
* crossDomain： true / false
* createXHR
* resultSelector

**使用範例**

```typescript
Observable
  .ajax('https://jsonplaceholder.typicode.com/posts')
  .map(e => e.response)
  .subscribe(posts => console.log(posts));
```



### bindCallback

將 callback API 轉換成一個回傳 Observable 的函式

```typescript
bindCallback(func: function, selector: function, scheduler: Scheduler): function(...params: *): Observable
```

**使用範例**

```typescript
function callbackFn(value1, value2, callback) {
  // value1 output: 1;
  // value2 output: 2;
  callback(value1, value2);
}

let source = Observable.bindCallback(callbackFn, (value1, value2) => {
  // value1 receive: 1;
  // value2 receive: 2;
  return value1 + value2;
});

source(1, 2).subscribe(value => {
  // value output: 2;
  console.log(value);
});
```

### bindNodeCallback

`bindNodeCallback` 與 `bindCallback` 的用法是一樣的，但是針對 node.js-style callback

**使用範例**

```typescript
import * as fs from 'fs';
// fs.readFile('/etc/passwd', 'utf8', callback);

var readFileAsObservable = Observable.bindNodeCallback(fs.readFile);
var result = readFileAsObservable('./roadNames.txt', 'utf8');
result.subscribe(x => console.log(x), e => console.error(e));
```

### create

建立自訂 Observable，`create` 會送出一個 observer 物件，包含 `next`、`error`、`complete` 三個函式。

透過這三個函式來控制 Observable 資料的輸出

1. next：送出資料
2. error：送出錯誤訊息，之後就會結束 Observable
3. complete：結束 Observable，之後不會有任何輸出值

```typescript
var observable = Observable.create(function (observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
});
observable.subscribe(
  value => console.log(value),
  err => {},
  () => console.log('this is the end')
);
```

```typescript
const observable = Observable.create((observer) => {
  observer.error('something went really wrong...');
});

observable.subscribe(
  value => console.log(value), // will never be called
  err => console.log(err),
  () => console.log('complete') // will never be called
);
```



### defer

### empty

### from

### fromEvent

### fromEventPattern

### fromPromise

### generate

### interval

### never

### of

### repeat

### repeatWhen

### range

### throw

### timer

## Transformation Operators

## Filtering Operators

## Combination Operators

## Multicasting Operators

## Error Handling Operators

## Utility Operators

## Conditional and Boolean Operators

## Mathematical and Aggregate Operators



# Subject

# Schedulers



