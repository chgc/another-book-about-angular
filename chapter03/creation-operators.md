# Creation Operators

## ajax

透過 Observable.ajax 可以呼叫 api 並取回資料，底層是使用 XMLHttpRequest 的方式實作。

```typescript
Observable.ajax('/products');
Observable.ajax({ url: 'products', method: 'GET' }); // 可接受 option 物件
```

參數物件內有以下設定可使用

- url
- body
- method：GET、POST、PUT、PATCH、DELETE
- async
- headers
- crossDomain： true / false
- createXHR
- resultSelector

**使用範例**

```typescript
Observable
  .ajax('https://jsonplaceholder.typicode.com/posts')
  .map(e => e.response)
  .subscribe(posts => console.log(posts));
```



## bindCallback

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

## bindNodeCallback

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

## create

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

## defer

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

## from

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

## fromEvent

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

## fromEventPattern

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

## fromPromise

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

## interval

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

## never

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

## empty

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

## throw

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

## of

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

## repeat

重複執行原有的 Observable n 次

![](images/repeat.png)

**使用介面**

```typescript
repeat(count: number): Observable
```

- count：重複次數

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

## repeatWhen

根據 notifications 來決定重複的時機，可以利用 `complete` 或 `error` 來取消重複執行的動作

![](images/repeatWhen.png)

**使用介面**

```typescript
repeatWhen(notifier: function(notifications: Observable): Observable): Observable
```

- notifier：決定是否要重複執行原有的 Observable

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

## range

一個 Observable 經設定開始與數量後，可以產生一系列的數列資料

![](images/range.png)

**使用介面**

```typescript
range(start: number, count: number, scheduler: Scheduler): Observable
```

- start：起始值，預設：0
- count： 數列長度，預設：0

**使用範例**

```typescript
var numbers = Rx.Observable.range(1, 10);
numbers.subscribe(x => console.log(x));
```

## timer

是 interval + delay 組合功能的 Observable

![](images/timer.png)

**使用介面**

```typescript
timer(initialDelay: number | Date, period: number, scheduler: Scheduler): Observable
```

- initialDelay：延遲時間
- period：發送資料頻率 (ms)

**使用範例**

```typescript
// 延遲 3 秒後，每隔 1 秒發送一次
var numbers = Rx.Observable.timer(3000, 1000);
numbers.subscribe(x => console.log(x));
```

