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

**使用介面**

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

**使用介面**

```typescript
bindNodeCallback(func: function, selector: function, scheduler: Scheduler): function(...params: *): Observable
```



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

![](images/create.png)

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

`defer` 是 Observable 工廠，`defer` 接受傳入一個會回傳 Observable 或是 Promise 的函式

![](images/defer.png)

**使用介面**

```typescript
defer(observableFactory: function(): SubscribableOrPromise): Observable
```

**使用範例**

```typescript
const clicksOrInterval = Observable.defer(function () {
  if (Math.random() > 0.5) {
    return Observable.fromEvent(document, 'click');
  } else {
    return Observable.interval(1000);
  }
});
clicksOrInterval.subscribe(x => console.log(x));
```

### from

接受陣列，Promise 或是可以 iterable 物件資料並轉換成 Observable (串流模式)

![](images/from.png)

**使用介面**

```typescript
from(ish: ObservableInput<T>, scheduler: Scheduler): Observable<T>
```

**使用範例**

```typescript
// 轉換陣列
var array = [10, 20, 30];
var result = Rx.Observable.from(array);
result.subscribe(x => console.log(x)); // 依序輸出 10, 20, 30
```

```typescript
// 轉換 generator function 
function* generateDoubles(seed) {
  var i = seed;
  while (true) {
    yield i;
    i = 2 * i; // double it
  }
}

var iterator = generateDoubles(3);
var result = Rx.Observable.from(iterator).take(10);
result.subscribe(x => console.log(x));
```

### fromEvent

轉換 Dom events 、Node EventEmitter events 或其他的事件至 Observable

![](images/fromEvent.png)

**使用介面**

```typescript
fromEvent(target: EventTargetLike, eventName: string, options: EventListenerOptions, selector: SelectorMethodSignature<T>): Observable<T>
```

**使用範例**

```typescript
var clicks = Rx.Observable.fromEvent(document, 'click');
clicks.subscribe(x => console.log(x));
```

### fromEventPattern

轉換 `addHandler` 和 `removeHandler` 至 Observable，會在 subscribe 時執行 addHandler 函式，而在 unsubscribe 時執行 removeHandler 函式

![](images/fromEventPattern.png)

**使用介面**

```typescript
fromEventPattern(addHandler: function(handler: Function): any, removeHandler: function(handler: Function, signal?: any): void, selector: function(...args: any): T): Observable<T>
```

**使用範例**

```typescript
function addClickHandler(handler) {
  console.log('add click event listener');
  document.addEventListener('click', handler);
}

function removeClickHandler(handler) {
  console.log('remove click event listener');
  document.removeEventListener('click', handler);
}

var clicks = Rx.Observable.fromEventPattern(
  addClickHandler,
  removeClickHandler
);
clicks.take(5).subscribe(x => console.log(x));
```

### fromPromise

轉換 promise 至 observable

**使用介面**

```typescript
fromPromise(promise: Promise<T>, scheduler: Scheduler): Observable<T>
```

**使用範例**

```typescript
var result = Rx.Observable.fromPromise(fetch('https://jsonplaceholder.typicode.com/users'));
result.subscribe(x => console.log(x), e => console.error(e));
```

### interval

建立一個會定時( ms )發送資料的 Observable

![](images/interval.png)

**使用介面**

```typescript
interval(period: number, scheduler: Scheduler): Observable
```

**使用範例**

```typescript
var numbers = Rx.Observable.interval(1000);
numbers.subscribe(x => console.log(x));
```

### never

建立一個不會發送任何資料的 Observable，主要為測試用途。

![](images/never.png)

**使用介面**

```typescript
never(): Observable
```

**使用範例**

```typescript
function info() {
  console.log('Will not be called');
}
var result = Observable.never().startWith(7);
result.subscribe(x => console.log(x), info, info);
```

### empty

直接回傳 `complete` 的 observable，主要為測試用途。

![](images/empty.png)

**使用介面**

```typescript
empty(scheduler: Scheduler): Observable
```

**使用範例**

```typescript
var interval = Rx.Observable.interval(1000);
var result = interval.mergeMap(x =>
  x % 2 === 1 ? Rx.Observable.of('a', 'b', 'c') : Rx.Observable.empty()
);
result.subscribe(x => console.log(x));
```

### throw

回傳 `error` 的 observable

![](images/throw.png)

**使用介面**

```typescript
throw(error: any, scheduler: Scheduler): Observable
```

**使用範例**

```typescript
var interval = Rx.Observable.interval(1000);
var result = interval.mergeMap(x =>
  x === 13 ?
    Rx.Observable.throw('Thirteens are bad') :
    Rx.Observable.of('a', 'b', 'c')
);
result.subscribe(x => console.log(x), e => console.error(e));
```

### of

回傳一系列資料的 Observable

![](images/of.png)

**使用介面**

```typescript
of(values: ...T, scheduler: Scheduler): Observable<T>
```

**使用範例**

```typescript
// Emit 10, 20, 30, then 'a', 'b', 'c', then start ticking every second.
var numbers = Rx.Observable.of(10, 20, 30);
var letters = Rx.Observable.of('a', 'b', 'c');
var interval = Rx.Observable.interval(1000);
var result = numbers.concat(letters).concat(interval);
result.subscribe(x => console.log(x));
```

### repeat

重複執行原有的 Observable n 次

![](images/repeat.png)

**使用介面**

```typescript
repeat(count: number): Observable
```

* count：重複次數

**使用範例**

```typescript
const source = Observable.of(1,2).repeat(2);
source.subscribe(x=> console.log(x)); // 輸出：1, 2, 1, 2
```

```typescript
// 兩次 repeat 會相乘等於重複 4 次
const source = Observable.of(1,2).repeat(2).repeat(2);
source.subscribe(x=> console.log(x)); // 輸出：1, 2, 1, 2, 1, 2, 1,2
```

### repeatWhen

根據 notifications 來決定重複的時機，可以利用 `complete` 或 `error` 來取消重複執行的動作

![](images/repeatWhen.png)

**使用介面**

```typescript
repeatWhen(notifier: function(notifications: Observable): Observable): Observable
```

* notifier：決定是否要重複執行原有的 Observable

**使用範例**

```typescript
let retried = false;
const source =
    Observable.of(1, 2).repeatWhen(function (notifications) {         
        return notifications.map(function (x) {            
        if (retried) {
            throw new Error('done');
        }
        retried = true;
        return x;
    }); })
source.subscribe(x => console.log(x));

```

### range

一個 Observable 經設定開始與數量後，可以產生一系列的數列資料

![](images/range.png)

**使用介面**

```typescript
range(start: number, count: number, scheduler: Scheduler): Observable
```

* start：起始值，預設：0
* count： 數列長度，預設：0

**使用範例**

```typescript
var numbers = Rx.Observable.range(1, 10);
numbers.subscribe(x => console.log(x));
```

### timer

是 interval + delay 組合功能的 Observable

![](images/timer.png)

**使用介面**

```typescript
timer(initialDelay: number | Date, period: number, scheduler: Scheduler): Observable
```

* initialDelay：延遲時間
* period：發送資料頻率 (ms)

**使用範例**

```typescript
// 延遲 3 秒後，每隔 1 秒發送一次
var numbers = Rx.Observable.timer(3000, 1000);
numbers.subscribe(x => console.log(x));
```



## Transformation Operators

### map

根據資料轉型的函式，將 Observable 發出的資料做轉型

![](images/map.png)

**使用介面**

```typescript
map(project: function(value: T, index: number): R, thisArg: any): Observable<R>
```

**使用範例**

```typescript
Observable.of(1, 2, 3, 4, 5)
    .map((value, idx) => {
      return value * (idx % 2 == 0 ? 1 : 2);
    })
    .subscribe(value => console.log(value)); // 輸出: 1, 4, 3, 8, 5
```

※ `map` 是 RxJS 使用最頻繁之一的 operators

### mapTo

送出一個固定的值

![](images/mapTo.png)

**使用介面**

```typescript
mapTo(value: any): Observable
```

**使用範例**

```typescript
Observable.of(1, 2, 3, 4, 5)
    .mapTo('Hi')
    .subscribe(value => console.log(value)); // 輸出：Hi, Hi, Hi, Hi, Hi
```

### concatMap

依序地結合兩個 Observable 發出的資料並做轉換

![](images/concatMap.png)

**使用介面**

```typescript
concatMap(project: function(value: T, ?index: number): ObservableInput, resultSelector: function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): any): Observable source
```

**使用範例**

```typescript
Observable.of(1, 2, 3, 4, 5)
    .concatMap(
        (value, idx) => {
          const _v = value * (idx % 2 == 0 ? 1 : 2);
          return Observable.of(_v);// 輸出：1, 4, 3, 8, 5
        },
        (outerValue, innerValue) => {
          console.log({outValue, innerValue}); // 輸出：{1,1},{2,4},{3,3},{4,8},{5,5}
          return outerValue * innerValue
        })
    .subscribe(value => console.log(value));// 輸出：1, 8, 9, 32, 25
```

### concatMapTo

依序地結合兩個 Observable 並送出一個固定的值

![](images/concatMapTo.png)

**使用介面**

```typescript
concatMapTo(innerObservable: ObservableInput, resultSelector: function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): any): Observable
```

**使用範例**

```typescript
Observable.of(1,2,3,4,5)
    .concatMapTo(
        Observable.of('Hi'),
        (outerValue, innerValue) => {
          return innerValue + ' ' + outerValue;
        })
    .subscribe(value => console.log(value)); // 輸出: Hi 1, Hi 2, Hi 3, Hi 4, Hi 5
```

### mergeMap

依資料發生的時序結合兩個 Observable 並做資料轉換

![](images/mergeMap.png)

**使用介面**

```typescript
mergeMap(project: function(value: T, ?index: number): ObservableInput, resultSelector: function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): any, concurrent: number): Observable
```

**使用範例**

```typescript
Observable.of(1, 2, 3, 4, 5)
    .mergeMap(
        (value, idx) => {
          const _v = value * (idx % 2 == 0 ? 1 : 2);
          return Observable.of(_v);// 輸出：1, 4, 3, 8, 5
        },
        (outerValue, innerValue) => {
          console.log({outerValue, innerValue}); // 輸出：{1,1},{2,4},{3,3},{4,8},{5,5}
          return outerValue * innerValue
        })
    .subscribe(value => console.log(value));// 輸出：1, 8, 9, 32, 25
```



### mergeMapTo

依資料發生的時序結合兩個 Observable 並送出一個固定的值

![](images/mergeMapTo.png)

**使用介面**

```typescript
 mergeMapTo(innerObservable: ObservableInput, resultSelector: function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): any, concurrent: number): Observable
```

**使用範例**

```typescript
Observable.of(1,2,3,4,5)
    .mergeMapTo(
        Observable.of('Hi'),
        (outerValue, innerValue) => {
          return innerValue + ' ' + outerValue;
        })
    .subscribe(value => console.log(value)); // 輸出: Hi 1, Hi 2, Hi 3, Hi 4, Hi 5
```

### switchMap

取最新發生的資料合併兩個 Observable 並做轉換

![](images/switchMap.png)

**使用介面**

```typescript
switchMap(project: function(value: T, ?index: number): ObservableInput, resultSelector: function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): any): Observable
```

**使用範例**

```typescript
const clicks = Rx.Observable.fromEvent(document, 'click');
const result = clicks.switchMap((ev) => Rx.Observable.interval(1000));
result.subscribe(x => console.log(x));
```

在每一次 click 動作重新執行 Observable.interval 的動作

### switchMapTo

就跟 `switchMap` 一樣，只是每次都回傳一樣的值

![](images/switchMapTo.png)

**使用介面**

```typescript
switchMapTo(innerObservable: ObservableInput, resultSelector: function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): any): Observable
```

**使用範例**

```typescript
var clicks = Rx.Observable.fromEvent(document, 'click');
var result = clicks.switchMapTo(Rx.Observable.interval(1000));
result.subscribe(x => console.log(x));
```

### exhaustMap

如果在上一個 Observable 動作未完成，又發生另一個 observable 動作，則這一個動作會被忽略。

![](images/exhaustMap.png)

**使用介面**

```typescript
exhaustMap(project: function(value: T, ?index: number): ObservableInput, resultSelector: function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): any): Observable
```

**使用範例**

```typescript
var clicks = Rx.Observable.fromEvent(document, 'click');
var result = clicks.exhaustMap(Rx.Observable.interval(1000));
result.subscribe(x => console.log(x));
```

### expand

![](images/expand.png)

**使用介面**

```typescript
expand(project: function(value: T, index: number), concurrent: number, scheduler: Scheduler): Observable
```

**使用範例**

```typescript
Observable.of(1, 2)
    .expand((value: number) => {
      if (value === 8)
        return Observable.empty();
      else
        return Observable.of(value * 2);
    })
    .subscribe(x => console.log(x)); // 輸出 1, 2, 4, 8, 2, 4, 8
```

### groupBy

根據條件將資料群組化輸出成獨立的 Observable

![](images/groupBy.png)

**使用介面**

```typescript
roupBy(keySelector: function(value: T): K, elementSelector: function(value: T): R, durationSelector: function(grouped: GroupedObservable<K, R>): Observable<any>): Observable<GroupedObservable<K, R>>
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
let groupData = [];
Observable.from(data).groupBy(x => x % 2, (ele)=> ele + '!').subscribe(g => {
  let group = {key: g.key, values: []};
  g.subscribe(value => {
    group.values.push(value);
  });
  groupData.push(group);
});
if (groupData) {
  console.log(groupData); // 輸出: [{key: 1, values: [1!, 3!, 5!]}, {key: 2, values: [2!, 4!]}]
}
```

### mergeScan

執行 scan 動作時，會回傳一個 observable 並與 outer observable 做 merge

**使用介面**

```typescript
mergeScan(accumulator: function(acc: R, value: T): Observable<R>, seed: *, concurrent: number): Observable<R>
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
Observable.from(data)
    .mergeScan((acc, one) => Observable.of(acc + one), 0)
    .subscribe(value => console.log(value)); // 輸出: 1, 3, 6, 10, 15
```

### pairwise

將前一次與本次發生的資料合併輸出

![](images/pairwise.png)

**使用介面**

```typescript
pairwise(): Observable<Array<T>>
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
Observable.from(data)
          .pairwise()
          .subscribe(value => {
  			console.log(value); // 輸出: [1,2], [2,3], [3,4], [4,5]
          });
```

### partition

根據條件分成符合與不符合條件的兩組 Observable

![](images/partition.png)

**使用介面**

```typescript
partition(predicate: function(value: T, index: number): boolean, thisArg: any): [Observable<T>, Observable<T>]
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
let parts = Observable.from(data).partition(x => x % 2 === 1);
parts[0].subscribe(value => {
  console.log(value); // 輸出: 1, 3, 5
});
parts[1].subscribe(value => {
  console.log(value); // 輸出: 2, 4
});
```

### pluck

取出特定的屬性，可以透過第二，及第三引數等，取得更深層的屬性

![](images/pluck.png)

**使用介面**

```typescript
 pluck(properties: ...string): Observable
```

**使用範例**

```typescript
import 'rxjs';
import {Observable} from 'rxjs/Observable';

const data = [{name: 'a', sex: 1}, {name: 'b', sex: 1}, {name: 'c', sex: 0}]

Observable.from(data)
          .pluck('name')
          .subscribe(value => console.log(value)); // 輸出: a, b, c
```

```typescript
import 'rxjs';
import {Observable} from 'rxjs/Observable';

const data = [
  {
    id: 1,
    company: {
      name: 'Romaguera-Crona',
      catchPhrase: 'Multi-layered client-server neural-net',
      bs: 'harness real-time e-markets'
    }
  },
  {
    id: 2,
    company: {
      name: 'Deckow-Crist',
      catchPhrase: 'Proactive didactic contingency',
      bs: 'synergize scalable supply-chains'
    }
  },
  {
    id: 3,
    company: {
      name: 'Romaguera-Jacobson',
      catchPhrase: 'Face to face bifurcated interface',
      bs: 'e-enable strategic applications'
    }
  }
];

Observable.from(data)
    .pluck('company', 'name')
    .subscribe(value => {
       console.log(value); // 輸出: Romaguera-Crona, Deckow-Crist, Romaguera-Jacobson
    }); 
```

### scan

類似 JavaScript 的 reduce，但是會保留之前的狀態

![](images/scan.png)

**使用介面**

```typescript
scan(accumulator: function(acc: R, value: T, index: number): R, seed: T | R): Observable<R>
```

**使用範例**

```typescript
const subScan = new Subject();
subScan.scan((acc: number, one: number) => acc + one, 0)
    .subscribe(value => console.log(value));
subScan.next(1); // 輸出: 1
subScan.next(1); // 輸出: 2
```

### buffer

根據 Observable 條件決定緩衝空間，當緩衝空間滿時，則輸出資料

![](images/buffer.png)

**使用介面**

```typescript
buffer(closingNotifier: Observable<any>): Observable<T[]>
```

**使用範例**

```typescript
Observable.interval(250)
    .take(10)
    .buffer(Observable.interval(1000))
    .subscribe(value => console.log(value)); // 輸出：[0,1,2], [3,4,5,6]

```

### bufferCount

根據資料數量條件決定緩衝空間，當緩衝空間滿時，則輸出資料

![](images/bufferCount.png)

**使用介面**

```typescript
bufferCount(bufferSize: number, startBufferEvery: number): Observable<T[]>
```

**使用範例**

```typescript
Observable.interval(300)
    .take(10)
    .bufferCount(3)
    .subscribe(value => console.log(value)); // 輸出: [0,1,2], [3,4,5],[6,7,8],[9]

Observable.interval(300)
    .take(10)
    .bufferCount(3,2)
    .subscribe(value => console.log(value)); // 輸出: [0,1,2], [2,3,4],[4,5,6],[6,7,8].[8,9]
```

### bufferTime

根據時間條件決定緩衝空間，當緩衝空間滿時，則輸出資料

![](images/bufferTime.png)

**使用介面**

```typescript
bufferTime(bufferTimeSpan: number, bufferCreationInterval: number, maxBufferSize: number, scheduler: Scheduler): Observable<T[]>
```

**使用範例**

```typescript
Observable.interval(300)
    .take(10)
    .bufferTime(1000)
    .subscribe(value => console.log(value));
```

### bufferToggle

根據開始與結束開關決定緩衝空間，當緩衝空間滿時，則輸出資料

![](images/bufferToggle.png)

**使用介面**

```typescript
bufferToggle(openings: SubscribableOrPromise<O>, closingSelector: function(value: O): SubscribableOrPromise): Observable<T[]>
```

**使用範例**

```typescript
// 0123456789
//    oc  oc
Observable.interval(250)
    .take(10)
    .bufferToggle(Observable.interval(1000), x => Observable.interval(500))
    .subscribe(value => console.log(value)); // 輸出： [3,4],[7,8]

```

### bufferWhen

根據結束開關決定緩衝空間，當緩衝空間滿時，則輸出資料

![](images/bufferWhen.png)

**使用介面**

```typescript
bufferWhen(closingSelector: function(): Observable): Observable<T[]>
```

**使用範例**

```typescript
Observable.interval(250)
    .take(10)
    .bufferWhen(() => Observable.interval(1000))
    .subscribe(value => console.log(value)); // 輸出：[0,1,2],[3,4,5,6],[7,8,9]
```

### window

根據 Observable 條件決定緩衝空間，當緩衝空間滿時，則輸出包含緩衝資料的新Observable

![](images/window.png)

**使用介面**

```typescript
window(windowBoundaries: Observable<any>): Observable<Observable<T>>
```

**使用範例**

```typescript
let i = 0;
let groupsData = [];
Observable.interval(250)
    .take(10)
    .window(Observable.interval(1000))
    .subscribe(value => {
      let group = {groupId: ++i, values: []};
      value.forEach(v => group.values.push(v));
      groupsData.push(group);      
    });
```

### windowCount

根據資料數量條件決定緩衝空間，當緩衝空間滿時，輸出包含緩衝資料的新Observable

![](images/windowCount.png)

**使用介面**

```typescript
windowCount(windowSize: number, startWindowEvery: number): Observable<Observable<T>>
```

**使用範例**

```typescript
let i = 0;
let groupsData = [];
Observable.interval(250)
    .take(10)
    .windowCount(3)
    .subscribe(value => {
      let group = {groupId: ++i, values: []};
      value.forEach(v => group.values.push(v));
      groupsData.push(group);      
    });
```



### windowTime

根據時間條件決定緩衝空間，當緩衝空間滿時，輸出包含緩衝資料的新Observable

**使用介面**

```typescript
windowTime(bufferTimeSpan: number, bufferCreationInterval: number, maxBufferSize: number, scheduler: Scheduler): Observable<Observable<T>>
```

**使用範例**

```typescript
let i = 0;
let groupsData = [];
Observable.interval(250)
    .take(10)
    .windowTime(1000)
    .subscribe(value => {
      let group = {groupId: ++i, values: []};
      value.forEach(v => group.values.push(v));
      groupsData.push(group);
      console.log(groupsData);
    });
```

### windowToggle

根據開始與結束開關決定緩衝空間，當緩衝空間滿時，輸出包含緩衝資料的新Observable

![](images/windowToggle.png)

**使用介面**

```typescript
windowToggle(openings: Observable<O>, closingSelector: function(value: O): Observable): Observable<Observable<T>>
```

**使用範例**

```typescript
let i = 0;
let groupsData = [];
Observable.interval(250)
    .take(10)
    .windowToggle(Observable.interval(1000), x => Observable.interval(500))
    .subscribe(value => {
      let group = {groupId: ++i, values: []};
      value.forEach(v => group.values.push(v));
      groupsData.push(group);
      console.log(groupsData);
    });
```

### windowWhen

根據結束開關決定緩衝空間，當緩衝空間滿時，輸出包含緩衝資料的新Observable

![](images/windowWhen.png)

**使用介面**

```typescript
windowWhen(closingSelector: function(): Observable): Observable<Observable<T>>
```

**使用範例**

```typescript
let i = 0;
let groupsData = [];
Observable.interval(250)
    .take(10)
    .windowWhen(()=> Observable.interval(1000))
    .subscribe(value => {
      let group = {groupId: ++i, values: []};
      value.forEach(v => group.values.push(v));
      groupsData.push(group);
      console.log(groupsData);
    });
```

## Filtering Operators

### debounce

根據另一個 observable 來決定延遲多久取資料

![](images/debounce.png)

**使用介面**

```typescript
debounce(durationSelector: function(value: T): SubscribableOrPromise): Observable
```

**使用範例**

```typescript
var clicks = Rx.Observable.fromEvent(document, 'click');
var result = clicks.debounce(() => Rx.Observable.interval(1000));
result.subscribe(x => console.log(x));
```



### debounceTime

根據設定時間決定延遲多久取資料

![](images/debounceTime.png)

**使用介面**

```typescript
debounceTime(dueTime: number, scheduler: Scheduler): Observable
```

**使用範例**

```typescript
var clicks = Rx.Observable.fromEvent(document, 'click');
var result = clicks.debounceTime(1000);
result.subscribe(x => console.log(x));
```

### audit

根據 observable 所設定取資料間隔時間，取得最新發生的資料

![](images/audit.png)

**使用介面**

```typescript
audit(durationSelector: function(value: T): SubscribableOrPromise): Observable<T>
```

**使用範例**

```typescript
Observable.interval(1000)
          .take(10)
          .audit(()=> Observable.interval(1200))
          .subscribe(value => console.log(value)) // 輸出：1, 3, 5,7
```

### auditTime

根據所設定取資料間隔時間，取得最新發生的資料

![](images/auditTime.png)

**使用介面**

```typescript
 auditTime(duration: number, scheduler: Scheduler): Observable<T>
```

使用範例

```typescript
Observable.interval(1000)
          .take(10)
          .audit(1200)
          .subscribe(value => console.log(value)) // 輸出：1, 3, 5,7
```

### sample

當 notifier 發出值時，則取得最新的資料

![](images/sample.png)

**使用介面**

```typescript
sample(notifier: Observable<any>): Observable<T>
```

**使用範例**

```typescript
var seconds = Rx.Observable.interval(1000);
var clicks = Rx.Observable.fromEvent(document, 'click');
var result = seconds.sample(clicks);
result.subscribe(x => console.log(x));
```

### sampleTime

依時間間隔取得最新的資料

![](images/sampleTime.png)

**使用介面**

```typescript
sampleTime(period: number, scheduler: Scheduler): Observable<T>
```

**使用範例**

```typescript
var clicks = Rx.Observable.fromEvent(document, 'click');
var result = clicks.sampleTime(70);
result.subscribe(x => console.log(x));
```

### throttle

### throttleTime

### distinct

過濾掉相同的值

**使用介面**

```typescript
distinct(keySelector: function, flushes: Observable): Observable
```

**使用範例**

```typescript
Observable.of(1, 1, 2, 2, 2, 1, 2, 3, 4, 3, 2, 1)
    .distinct()
    .subscribe(x => console.log(x));  // 1, 2, 3, 4
```

```typescript
interface Person {
    age: number,
    name: string
 }
 
 Observable.of<Person>(
     { age: 4, name: 'Foo'},
     { age: 7, name: 'Bar'},
     { age: 5, name: 'Foo'})
     .distinct((p: Person) => p.name)
     .subscribe(x => console.log(x));
```

### distinctUntilChanged

消除重複值到下一次資料不相同時

**使用介面**

```typescript
distinctUntilChanged(compare: function): Observable
```

**使用範例**

```typescript
Observable.of(1, 1, 2, 2, 2, 1, 1, 2, 3, 3, 4)
  .distinctUntilChanged()
  .subscribe(x => console.log(x)); // 1, 2, 1, 2, 3, 4
```

```typescript
interface Person {
   age: number,
   name: string
}
// 也可以給予函式作為比較差異的基準
Observable.of<Person>(
    { age: 4, name: 'Foo'},
    { age: 7, name: 'Bar'},
    { age: 5, name: 'Foo'})
    { age: 6, name: 'Foo'})
    .distinctUntilChanged((p: Person, q: Person) => p.name === q.name)
    .subscribe(x => console.log(x));

// displays:
// { age: 4, name: 'Foo' }
// { age: 7, name: 'Bar' }
// { age: 5, name: 'Foo' }
```



### distinctUntilKeyChanged

消除重複值到下一次設定比較欄位的資料不相同時

**使用介面**

```typescript
distinctUntilKeyChanged(key: string, compare: function): Observable
```

**使用範例**

```typescript
interface Person {
   age: number,
   name: string
}
// 也可以給予函式作為比較差異的基準
Observable.of<Person>(
    { age: 4, name: 'Foo'},
    { age: 7, name: 'Bar'},
    { age: 5, name: 'Foo'})
    { age: 6, name: 'Foo'})
   .distinctUntilKeyChanged('name', (x: string, y: string) => x.substring(0, 3) === y.substring(0, 3))
    .subscribe(x => console.log(x));

// displays:
// { age: 4, name: 'Foo' }
// { age: 7, name: 'Bar' }
// { age: 5, name: 'Foo' }
```

### elementAt

取得特定索引位置的資料

![](images/elementAt.png)

**使用介面**

```typescript
elementAt(index: number, defaultValue: T): Observable
```

**使用範例**

```typescript

interface Person {
    age: number,
    name: string
 }
 
 Observable.of<Person>(
     { age: 4, name: 'Foo'},
     { age: 7, name: 'Bar'},
     { age: 5, name: 'Foo'})
     .elementAt(1)
     .subscribe(x => console.log(x)); // 輸出：{ age: 7, name: 'Bar'}

```

### ignoreElements

忽略所有送出資料

![](images/ignoreElements.png)

**使用介面**

```typescript
ignoreElements(): Observable
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
Observable.from(data)
          .ignoreElements()
          .subscribe(value => console.log(value)); // 不會輸出任何資料
```

### filter

只有符合條件的資料可以通過

![](images/filter.png)

**使用介面**

```typescript
filter(predicate: function(value: T, index: number): boolean, thisArg: any): Observable
```

**使用範例**

```typescript

const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
Observable.from(data)
          .filter(x => x < 5)
          .subscribe(value => console.log(value)); // 輸出：1,2,3,4
```

### first

取得符合條件的第一筆資料

![](images/first.png)

**使用介面**

```typescript
first(predicate: function(value: T, index: number, source: Observable<T>): boolean, resultSelector: function(value: T, index: number): R, defaultValue: R): Observable<T | R>
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
Observable.from(data)
          .first(x => x > 5)
          .subscribe(value => console.log(value)); // 輸出: 6
```

### last

取得符合條件的最後一筆資料

![](images/last.png)

**使用介面**

```typescript
last(predicate: function): Observable
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
Observable.from(data)
          .last(x => x > 3)
          .subscribe(value => console.log(value)); // 輸出：5
```

### single

只允許一筆資料發生，如果有兩筆以上的資料，就會觸發錯誤

![](images/single.png)

**使用介面**

```typescript
single(predicate: Function): Observable<T>
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
Observable.from(data).single().subscribe(
    value => console.log(value), 
      err => console.error(err)); // 輸出: Sequence contains more than one element
```

### skip

忽略 n 筆資料後再輸出

![](images/skip.png)

**使用介面**

```typescript
skip(count: Number): Observable
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
Observable.from(data)
          .skip(3)
          .subscribe(value => console.log(value)); // 輸出 4, 5
```

### skipUntil

忽略資料直到 notifier 發出資料

![](images/skipUntil.png)

**使用介面**

```typescript
skipUntil(notifier: Observable): Observable<T>
```

**使用範例**

```typescript
Observable.interval(500)
    .take(10)
    .skipUntil(Observable.of('1').delay(3000))
    .subscribe(value => console.log(value)); // 輸出 5, 6, 7, 8, 9
```

### skipWhile

忽略資料直到不合條件時 發出資料

![](images/skipWhile.png)

**使用介面**

```typescript
skipWhile(predicate: Function): Observable<T>
```

**使用範例**

```typescript
Observable.interval(500)
    .take(10)
    .skipWhile(value => value < 4)
    .subscribe(value => console.log(value)); // 輸出: 4, 5, 6, 7, 8, 9
```

### take

設定要取得的資料筆數，滿足則會完成 Observable

![](images/take.png)

**使用介面**

```typescript
take(count: number): Observable<T>
```

**使用範例**

```typescript
Observable.interval(500)
    .take(5)    
    .subscribe(value => console.log(value)); // 輸出： 0, 1, 2, 3, 4
```

### takeLast

當 Observable 完成時，設定要取得最後所發生的 n 筆資料

**使用介面**

```typescript
takeLast(count: number): Observable<T>
```

**使用範例**

```typescript
const data = [1, 2, 3, 4, 5];
Observable.from(data)
    .takeLast(2)        
    .subscribe(value => console.log(value)); // 輸出: 4, 5
```

### takeUntil

持續取值直到 notifier 發生資料

![](images/takeUntil.png)

**使用介面**

```typescript
takeUntil(notifier: Observable): Observable<T>
```

**使用範例**

```typescript
Observable.interval(500)
    .take(10)
    .takeUntil(Observable.of('1').delay(3000))
    .subscribe(value => console.log(value)); // 輸出：0, 1, 2, 3, 4
```

### takeWhile

持續取值直到不符合條件時

![](images/takeWhile.png)

**使用介面**

```typescript
takeWhile(predicate: function(value: T, index: number): boolean): Observable<T>
```

**使用範例**

```typescript
Observable.interval(500)
    .take(10)
    .takeWhile(value => value < 4)
    .subscribe(value => console.log(value)); // 輸出：0, 1, 2, 3
```



## Combination Operators

### combineAll



![](images/combineAll.png)

**使用介面**

```typescript
combineAll(project: function): Observable
```

**使用範例**

```typescript
Observable.interval(1000)
    .take(2)
    .map(value => {
      return Observable.interval(1000)
          .map(i => `Result (${value}): ${i}`)
          .take(3);
    })
    .combineAll()
    .subscribe(value => console.log(value));
/* 輸出
  ["Result (0): 0", "Result (1): 0"]
  ["Result (0): 1", "Result (1): 0"]
  ["Result (0): 1", "Result (1): 1"]
  ["Result (0): 2", "Result (1): 1"]
  ["Result (0): 2", "Result (1): 2"] 
*/  
```

### combineLatest

合併兩個 Observable，並使用各 Observable 最後發生的資料

![](images/combineLatest.png)

**使用介面**

```typescript
combineLatest(observable1: ObservableInput, observable2: ObservableInput, project: function, scheduler: Scheduler): Observable
```

**使用範例**

```typescript
const firstTimer = Observable.timer(0, 1000); // emit 0, 1, 2... after every second, starting from now
const secondTimer = Observable.timer(500, 1000); // emit 0, 1, 2... after every second, starting 0,5s from now
const combinedTimers = Observable.combineLatest(firstTimer, secondTimer).take(4);
combinedTimers.subscribe(value => console.log(value));
/*
輸出: [ 0, 0 ],[ 1, 0 ],[ 1, 1 ],[ 2, 1 ]
*/
```

### concat

串接兩個 Observable 

![](images/concat.png)

**使用介面**

```typescript
concat(input1: ObservableInput, input2: ObservableInput, scheduler: Scheduler): Observable
```

**使用範例**

```typescript
const timer = Observable.interval(1000).take(4);
const sequence = Observable.range(1, 10);
const result = Observable.concat(timer, sequence);
result.subscribe(x => console.log(x));
/* 輸出:
0 -1000ms-> 1 -1000ms-> 2 -1000ms-> 3 -immediate-> 1 ... 10
*/
```

### concatAll

![](images/concatAll.png)

**使用介面**

```typescript
concatAll(): Observable
```

**使用範例**

```typescript
Observable.of('a', 'b')
    .map(x => Observable.range(1, 2).map(v => `{outer: ${x}, inner: ${v}`))
    .concatAll()
    .subscribe(x => console.log(x));
/* 輸出
{outer: a, inner: 1}
{outer: a, inner: 2}
{outer: b, inner: 1}
{outer: b, inner: 2}
*/
```

### merge

忽視 Observable 的資料發生順序，有資料產生就送出

![](images/merge.png)

**使用介面**

```typescript
merge(observables: ...ObservableInput, concurrent: number, scheduler: Scheduler): Observable
```

**使用範例**

```typescript
var timer1 = Rx.Observable.interval(1000).take(10);
var timer2 = Rx.Observable.interval(2000).take(6);
var timer3 = Rx.Observable.interval(500).take(10);
var concurrent = 2; // the argument
var merged = timer1.merge(timer2, timer3, concurrent);
merged.subscribe(x => console.log(x));
```

### mergeAll

基本上跟 merge 的功能是一樣的，只差在表示方式

![](images/mergeAll.png)

**使用介面**

```typescript
mergeAll(concurrent: number): Observable
```

**使用範例**

```typescript
var clicks = Rx.Observable.fromEvent(document, 'click');
var higherOrder = clicks.map((ev) => Rx.Observable.interval(1000).take(10));
var firstOrder = higherOrder.mergeAll(2);
firstOrder.subscribe(x => console.log(x));
```

### exhaust

當一個 Observable 尚未執行完成時，在期間有另一個 Observable 被執行，而這個 observable 會被忽略

![](images/exhaust.png)

**使用介面**

```typescript
exhaust(): Observable
```

**使用範例**

```typescript
const clicks = Rx.Observable.fromEvent(document, 'click');
const higherOrder = clicks.map((ev) => Rx.Observable.interval(1000).take(5));
const result = higherOrder.exhaust();
result.subscribe(x => console.log(x));
```

### forkJoin

等所有的 sources 都回傳資料後，再一起送出

**使用介面**

```typescript
forkJoin(sources: *): any
```

**使用範例**

```typescript
const data1 = [1, 2, 3];
const data2 = ['a', 'b', 'c'];
Observable
    .forkJoin(
        Observable.of(data1).delay(1000), Observable.of(data2).delay(2000))
    .subscribe(value => console.log(value)); 
/*
輸出：[
		[1, 2, 3],
		['a', 'b', 'c']
	  ]
*/
```

### race

採用最先發送資料者的資料

**使用介面**

```typescript
race(): Observable
```

**使用範例**

```typescript
Observable.of('outer')
    .delay(400)
    .race(Observable.of('inner'))
    .subscribe(value => console.log(value)); // 輸出: inner
```

### startWith

在 Observable 的資料發生前，插入要開始的資料

![](images/startWith.png)

**使用介面**

```typescript
startWith(values: ...T, scheduler: Scheduler): Observable
```

**使用範例**

```typescript
Observable.of('outer')
    .startWith('1','2','3')    
    .subscribe(value => console.log(value)); // 輸出: 1, 2, 3, outer
```

### switch

後面發生的 Observable 會接手並取消之前發生的 Observable

![](images/switch.png)

**使用介面**

```typescript
switch(): Observable<T>
```

**使用範例**

```typescript
// 每一次的 click 都會取消之前的 Observable，重新啟動新的 observable.interval(1000)
var clicks = Rx.Observable.fromEvent(document, 'click');
var higherOrder = clicks.map((ev) => Rx.Observable.interval(1000));
var switched = higherOrder.switch();
switched.subscribe(x => console.log(x));
```

### withLatestFrom

合併兩個 Observable, 並取每一個 observable 最後一次發生的資料當做資料轉換的根據

![](images/withLatestFrom.png)

**使用介面**

```typescript
withLatestFrom(other: ObservableInput, project: Function): Observable
```

**使用範例**

```typescript
Observable.interval(1000)
    .take(3)
    .withLatestFrom(
        Observable.timer(500, 250),
        (outer, inner) => `{outer:${outer}, inner: ${inner} }`)
    .subscribe(value => console.log(value)); 
/* 
輸出: 
{outer:0, inner:1}
{outer:1, inner:5}
{outer:2, inner:9}
*/
```

### zip

當監視的 Observable 都有回傳資料時，才會變成一組資料發出

**使用介面**

```typescript
zip(observables: *): Observable<R>
```

**使用範例**

```typescript

let age$ = Observable.of<number>(27, 25, 29);
let name$ = Observable.of<string>('Foo', 'Bar', 'Beer');
let isDev$ = Observable.of<boolean>(true, true, false);

Observable
    .zip(age$,
         name$,
         isDev$,
         (age: number, name: string, isDev: boolean) => ({ age, name, isDev }))
    .subscribe(x => console.log(x));

// outputs
// { age: 27, name: 'Foo', isDev: true }
// { age: 25, name: 'Bar', isDev: true }
// { age: 29, name: 'Beer', isDev: false }
```

### zipAll

**使用介面**

```typescript
zipAll(project: *): Observable<R> | WebSocketSubject<T> | Observable<T>
```

**使用範例**

```typescript
let age$ = Observable.of<number>(27, 25, 29);
let name$ = Observable.of<string>('Foo', 'Bar', 'Beer');

age$.map(() => name$)
    .zipAll(function(age, name) {
      return ({age, name});
    })
    .subscribe(value => console.log(value));
/* 輸出
{ age: 27, name: 'Foo'}
{ age: 25, name: 'Bar'}
{ age: 29, name: 'Beer'}
*/
```

## Multicasting Operators

## Error Handling Operators

## Utility Operators

## Conditional and Boolean Operators

## Mathematical and Aggregate Operators



# Subject

# Schedulers



# 進階應用技巧

## Observable.forEach

