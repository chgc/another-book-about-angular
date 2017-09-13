# 進階用法

## Interceptor

`Interceptor` 是 `@angular/common/http` 內一個很重要的功能，這功能可以讓我們在後端間建立一個關卡，用來監控或加工 request 與  response

基本 interceptor 架構如下

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

 Interceptor 需註冊到 `@NgModules`裡， 這裡有使用 multi，這表示我們可以寫多個 Interceptor ，套用順序會依註冊的順序

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


* HttpHandler.handle：接受 `HttpRequest` 為引數，回傳結果可以視為 Response

```typescript
  class HttpHandler {
    handle(req: HttpRequest<any>): Observable<HttpEvent<any>>
  }
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

* HttpEvent：一個型別代表以下的狀態

```typescript
  type HttpEvent = HttpSentEvent | HttpHeaderResponse | HttpResponse<T>| HttpProgressEvent | HttpUserEvent<T>;
```


### Immutable 特性

`HttpRequest`是 immutable 物件，所以必須透過內建的方法 `clone()` 用來改變值並會回傳一個新的 HttpRequest，以下為改變 HttpRequest header 的範例程式

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

可修改的內容，請參閱上述的 HttpRequest Class

### 使用範例

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

#### Authentication

```typescript
@Injectable()
export class TokenInterceptor implements HttpInterceptor {
  constructor(public auth: AuthService) {}
  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // 每一個 Http Request 都會加上 Token 資訊
    request = request.clone({
      setHeaders: {
        Authorization: `Bearer ${this.auth.getToken()}`
      }
    });
    return next.handle(request);
  }
}
```

```typescript
export class JwtInterceptor implements HttpInterceptor {
  constructor(public auth: AuthService) {}
  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    
    return next.handle(req).do((event: HttpEvent<any>) => {
      if (event instanceof HttpResponse) {
       
      }
    }, (err: any) => {
      if (err instanceof HttpErrorResponse) {
        if (err.status === 401) {
          // 處理未授權時應有的後續動作
        }
      }
    });
  }
}
```



## Progress events

如何簡單又快速的做到上傳檔案進度監控的動作呢? 以前需要自己實作 XHR ，現在直接使用  HttpClient.request 就可以了，方法如下

```typescript
const req = new HttpRequest('POST', '/upload/file', file, {
  reportProgress: true,
});
```

`HttpRequest` 的 constructor 接受這些參數

```typescript
constructor(method: string, url: string, 
            third?: T|{
        		headers?: HttpHeaders,
        		reportProgress?: boolean,
        		params?: HttpParams,
        		responseType?: 'arraybuffer'|'blob'|'json'|'text',
        		withCredentials?: boolean,
        	} |null, 
        	fourth?: {
	        	headers?: HttpHeaders,
    	    	reportProgress?: boolean,
        		params?: HttpParams,
        		responseType?: 'arraybuffer'|'blob'|'json'|'text',
        		withCredentials?: boolean,
      		})
```

當建立完要送出的 HttpRequest  物件後，可透過 `httpClient.request` 的方法去執行，並取得 HttpResponse 的結果回來，這裡的模式很類似於 interceptor 的運作模式

```typescript
http.request(req).subscribe(event => {
  // Via this API, you get access to the raw event stream.
  // Look for upload progress events.
   if (event.type === HttpEventType.DownloadProgress) {
     console.log("Download progress event", event);
   }
   if (event.type === HttpEventType.UploadProgress) {
     console.log("Upload progress event", event);
     // This is an upload progress event. Compute and show the % done:
     const percentDone = Math.round(100 * event.loaded / event.total);
     console.log(`File is ${percentDone}% uploaded.`);
   }
   if (event.type === HttpEventType.Response) {
     console.log("response received...", event.body);
   }  
});
```



