# 進階用法

## Interceptor

`Interceptor` 是 `@angular/common/http` 內一個很重要的功能，這功能讓我們可以在後端間建立一個關卡，用來監控或加工進出的邀請與回應

一個基本的寫法如下

```typescript
import {Injectable} from '@angular/core';
import {HttpEvent, HttpInterceptor, HttpHandler, HttpRequest} from '@angular/common/http';

@Injectable()
export class NoopInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req);
  }
}
```

然後當一個 Interceptor 完成後，在註冊到 `@NgModules`裡

```typescript
import {NgModule} from '@angular/core';
import {HTTP_INTERCEPTORS} from '@angular/common/http';

@NgModule({
  providers: [{
    provide: HTTP_INTERCEPTORS,
    useClass: NoopInterceptor,
    multi: true,
  }],
})
export class AppModule {}
```

* HttpRequest： An outgoing HTTP request with an optional typed body. Immutable 物件

  ```typescript
  class HttpRequest<T> {
    constructor(method: string, url: string, third?: T|{...}, fourth?: {...})
    body: T|null
    headers: HttpHeaders
    reportProgress: boolean
    withCredentials: boolean
    responseType: 'arraybuffer'|'blob'|'json'|'text'
    method: string
    params: HttpParams
    urlWithParams: string
    url: string
    serializeBody(): ArrayBuffer|Blob|FormData|string|null
    detectContentTypeHeader(): string|null
    clone(update: {...}): HttpRequest<any>
  }
  ```

* HttpHandler.handle：接受 `HttpRequest` 為引數，回傳結果可以視為 Response

  ```typescript
  class HttpHandler {
    handle(req: HttpRequest<any>): Observable<HttpEvent<any>>
  }
  ```

* HttpEvent：一個型別代表以下的狀態

  ```typescript
  type HttpEvent = HttpSentEvent | HttpHeaderResponse | HttpResponse<T>| HttpProgressEvent | HttpUserEvent<T>;
  ```



### 順序

由於在註冊於 `@NgModules` 時有使用 multi，這表示我們可以寫多個 ineterceptor，套用的順序會依註冊的順序



### Immutable

`HttpRequest` 與 `HttpResponse` 都是 immutable 物件，所以必須透過內建的方法用來改變值，每一個方法都會回傳一個新的物件，以下為改變 HttpRequest header 的範例程式

```typescript
import {Injectable} from '@angular/core';
import {HttpEvent, HttpInterceptor, HttpHandler, HttpRequest} from '@angular/common/http';
 
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor() {}
 
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {    
    // 改變現有 HttpRequest 的 header 內容
    const authReq = req.clone({headers: req.headers.set('Authorization', 'myAuthHeaderInfo')});
    // 傳入新的 HttpRequest 物件替代原本的
    return next.handle(authReq);
  }
}
```



### 範例

#### Logging

```typescript
import 'rxjs/add/operator/do';
 
export class TimingInterceptor implements HttpInterceptor {
  constructor(private auth: AuthService) {}
 
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
  	const started = Date.now();
    return next
      .handle(req)
      .do(event => {
        if (event instanceof HttpResponse) {
          const elapsed = Date.now() - started;
          console.log(`Request for ${req.urlWithParams} took ${elapsed} ms.`);
        }
      });
  }
}
```

#### Caching

```typescript
abstract class HttpCache {
  /**
   * Returns a cached response, if any, or null if not present.
   */
  abstract get(req: HttpRequest<any>): HttpResponse<any>|null;

  /**
   * Adds or updates the response in the cache.
   */
  abstract put(req: HttpRequest<any>, resp: HttpResponse<any>): void;
}
```

```typescript
@Injectable()
export class CachingInterceptor implements HttpInterceptor {
  constructor(private cache: HttpCache) {}
 
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
  	// Before doing anything, it's important to only cache GET requests.
    // Skip this interceptor if the request method isn't GET.
    if (req.method !== 'GET') {
      return next.handle(req);
    }
 
    // First, check the cache to see if this request exists.
    const cachedResponse = this.cache.get(req);
    if (cachedResponse) {
      // A cached response exists. Serve it instead of forwarding
      // the request to the next handler.
      return Observable.of(cachedResponse);
    }
 
    // No cached response exists. Go to the network, and cache
    // the response when it arrives.
    return next.handle(req).do(event => {
      // Remember, there may be other events besides just the response.
      if (event instanceof HttpResponse) {
      	// Update the cache.
      	this.cache.put(req, event);
      }
    });
  }
}
```



## Progress events

```typescript
const req = new HttpRequest('POST', '/upload/file', file, {
  reportProgress: true,
});
```

```typescript
http.request(req).subscribe(event => {
  // Via this API, you get access to the raw event stream.
  // Look for upload progress events.
  if (event.type === HttpEventType.UploadProgress) {
    // This is an upload progress event. Compute and show the % done:
    const percentDone = Math.round(100 * event.loaded / event.total);
    console.log(`File is ${percentDone}% uploaded.`);
  } else if (event instanceof HttpResponse) {
    console.log('File is completely uploaded!');
  }
});
```



