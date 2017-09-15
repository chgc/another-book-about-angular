# 測試

在測試的角度來看 HttpClient，做法比以前要測試 Http 來的簡單很多了，不在需要自己弄一個 `MockBackend` 去模擬一個回傳的結果。Angular 一樣提供一個 `HttpClientTestingModule` 供使用，透過 `HttpTestingController` 的設定要反應的網址及回傳結果

## HttpClientTestingController

```typescript
// import HttpClientTestingModule
beforeEach(() => {
  TestBed.configureTestingModule({
    ...,
    imports: [
      HttpClientTestingModule,
    ],
  })
});
...
// 取得 HttpClient 與 HttpTestingController service
it('expects a GET request', inject([HttpClient, HttpTestingController], (http: HttpClient, httpMock: HttpTestingController) => {
  
  // 呼叫 /data 並預期回傳一個物件 {name: 'Test Data'}  
  http
    .get('/data')
    .subscribe(data => expect(data['name']).toEqual('Test Data'));
 
  // 設定 HttpTestingController 要反應的 request 網址
  const req = httpMock.expectOne('/data');
 
  // 預期請求模式為 GET
  expect(req.request.method).toEqual('GET');
   
  // 下一步，送出要回傳的結果
  req.flush({name: 'Test Data'});   
}));

afterEach(inject([HttpTestingController], (httpMock: HttpTestingController) => {
   // 假設之後沒有任何請求了，結束動作
  httpMock.verify();
}));
```

如果要處理多次 request 時，原本的 `expectOne` 只接受一次的呼叫，表示 0 次呼叫或是 2 次以上的呼叫都會丟出例外錯誤，這時既要使用 `match` 來代替 `expectOne`

```typescript
// Expect that 5 pings have been made and flush them.
const reqs = httpMock.match('/ping');
expect(reqs.length).toBe(5);
reqs.forEach(req => req.flush());
```

`HttpClientTestingModule` 除了 `expectOne` 、`match` 外，還有哪些方法可以使用呢?

```typescript
class HttpTestingController {
  match(match: string|RequestMatch|((req: HttpRequest<any>) => boolean)): TestRequest[]
  expectOne(url: string, description?: string): TestRequest
  expectNone(url: string, description?: string): void
  verify(opts?: {ignoreCancelled?: boolean}): void
}
```

* match：符合設定條件，不限制呼叫次數

  * RequestMatch: 

    ```typescript
    interface RequestMatch { 
      method?: string
      url?: string
    }
    ```

    ​

* expectOne：符合呼叫條件，但只能呼叫一次，如果沒有不呼叫或是呼叫超過 1 次時，則會丟出例外錯誤

* expectNone：符合呼叫條件，不能被呼叫，如果有被呼叫時，則會丟出例外錯誤

* verify：確認之後不會有任何呼叫行為，如果在這之後還有任何呼叫行為，則會丟出例外錯誤

  * 預設是呼叫  `CancelledRequest`，如果將 `ignoreCancelled` 設定為 `false` 時，當 `CancelledRequest` 發生時，也會丟出例外錯誤

## TestRequest

`HttpTestingController` 會回傳 `TestRquest` 物件或陣列，而 `TestRequest` 有以下的功能

```typescript
class TestRequest {
  constructor(request: HttpRequest<any>, observer: Observer<HttpEvent<any>>)
  get cancelled(): boolean
  request: HttpRequest<any>
  flush(body: ArrayBuffer|Blob|string|number|Object|(string|number|Object|null)[]|null, opts: {...}): void
  error(error: ErrorEvent, opts: {...}): void
  event(event: HttpEvent<any>): void
}
```

* flush ：回傳正常的 response 結果

* error：回傳失敗的 response 狀態

* event：送出 `HttpEvent`，例如 progress event

  * HttpProgressEvent

    ```typescript
    interface HttpProgressEvent { 
      type: HttpEventType.DownloadProgress|HttpEventType.UploadProgress
      loaded: number
      total?: number
    }
    ```

    ​



