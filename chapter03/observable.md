# Observable

## 解析 Observable

Observable 的執行生命週期是建立 ( creating )、訂閱 ( subscribing) 、 執行 ( executing )、回收 ( disposing )，一個完整的 Observable 都會經歷這四個階段

- 建立 Observable

  ```typescript
  var observable = Observable.create(observer => {
    var id = setInterval(() => {
      observer.next('hi')
    }, 1000);
  });
  ```

  RxJS 提供很多方法可以用來建立 Observable，會在之後的內容逐一介紹

- 訂閱 Observable

  ```typescript
  observable.subscribe({
    next: x => console.log('Observer got a next value: ' + x),
    error: err => console.error('Observer got an error: ' + err),
    complete: () => console.log('Observer got a complete notification'),
  });
  ```

  `subscribe` 是訂閱 Observable 的動作，會接收一個 observer 物件 (包含 next、error、complete 三階段)，會是接收三個函式，分別處理 next、error、complete 三個階段。在執行 `subscribe` 時，會回傳一個 `subscription`，這個 `subscription` 是用來取消 Observable 使用的

- 執行 Observable

  - 剛剛有提過 subscribe 內會有三個階段要處理，分別為 next、error、complete
    - next：Observable 正常執行時，任何值都會傳給負責處理 next 階段的函式
    - error：當 Observable 發生錯誤時，就會進入 error 階段，負責的函式也會收到一個參數
    - complete：當 Observable 正常執行結束後，就會進入 complete 階段，不會接受到任何參數
  - 一但進入 error 或是 complete 階段，即使 Observable 後續還有任何資料的發生，next 階段都不會接受到任何資料了。( RxJS 有提供 operators 可以負責處理 error 階段的函式，並可以切回正常階段 )

- 回收 Observable

  - 訂閱 Observable 會收到一個 subscription，執行 `unsubscribe()` 即可取消 Observable 的動作

    ```typescript
    var observable = Observable.from([10, 20, 30]);
    var subscription = observable.subscribe(x => console.log(x));
    // Later:
    subscription.unsubscribe();
    ```

