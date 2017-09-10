# 基本用法

Angular 在 4 版的末期推出了 `HttpClientModule`，這一個 module 讓使用者在操作 Http 動作時，更簡單了，也額外提供一些原本要自己實作的 progress events 的功能，也簡化了 Http interceptor 的實作方式，在這一章節，會著重於基本的 HttpClient 的使用方式

# 安裝

`HttpClientModule` 是在 `@angular/common/http`  下，我們需要 import 這個 `HttpClientModule` 才能使用 `HttpClinet` 

```typescript
// app.module.ts:
 
import {NgModule} from '@angular/core';
import {BrowserModule} from '@angular/platform-browser';
 
// Import HttpClientModule from @angular/common/http
import {HttpClientModule} from '@angular/common/http';
 
@NgModule({
  imports: [
    BrowserModule,
    // Include it under 'imports' in your application module
    // after BrowserModule.
    HttpClientModule,
  ],
})
export class MyAppModule {}
```



# 語法

在 `Http` 時代，當發出一個請求回傳的結果型態都是 `response` ，所以如果要從內取出結果，都需要經過一個轉換的動作，而 `HttpClient` 內建就會做完轉換的動作，預設是會做 `response.json()` ，如果需要改變轉換方式，可以再請求時就加入請求參數內

```typescript
// Http 時代
http.get('...').map(res=> res.json()) // 這樣才能取出回傳結果

// HttpClient 時代
// 預設: 回傳json 物件
http.get('...') // 即可取得回傳結果

// 如果回傳型態是文字型，會做 response.text() 的轉型
http.get('...', {responseType: 'text'})
```

## 型別

HttpClient 另外一個新增功能是 `型別`，可以透過指定型別的方式，讓 response 回傳時即可賦予型別

```typescript
interface ItemsResponse {
  results: string[];
}

http.get<ItemsResponse>('/api/items').subscribe(data => {
  this.results = data.results;
});
```



## Response 物件

當需要取得完整的回傳 response 物件，設定方式類似取文字的，設定 ` observe` 為 `response` 即可

```typescript
http.get('...', {observe: 'response'})
```



## 錯誤處理

因為 `HttpClient` 回傳依舊是 `Observable`，所以處理方式與使用 `Http` 相同，可以透過 `catch` 處理錯誤，或是在 `subscribe` 的 error 區塊處理都可以

```typescript
http
  .get<ItemsResponse>('/api/items')
  .subscribe(
    data => {...},
    (err: HttpErrorResponse) => {
      if (err.error instanceof Error) {
        // client端或網路發生錯誤時
        console.log('An error occurred:', err.error.message);
      } else {
        // 回傳非 2xx 狀態碼時
        console.log(`Backend returned code ${err.status}, body was: ${err.error}`);
      }
    }
  );
```



## 送出資料

基本使用方式

```typescript
http.post(url, body).subscribe();
```

需設定其他參數，例如 header

```typescript
http.post(url, body,{
    headers: new HttpHeaders().set('Authorization', 'my-auth-token'),
}).subscribe();
```

這裡要注意的地方是，`HttpHeaders` 是一個 Immutable 物件，所以必須透過方法才可以設定裡面的值

```typescript
class HttpHeaders {
  constructor(headers?: string|{[name: string]: string | string[]})
  has(name: string): boolean
  get(name: string): string|null
  keys(): string[]
  getAll(name: string): string[]|null
  append(name: string, value: string|string[]): HttpHeaders
  set(name: string, value: string|string[]): HttpHeaders
  delete(name: string, value?: string|string[]): HttpHeaders
}
```

設定網址參數的方式與設定 header 的方式雷同

```typescript
http
  .post('/api/items/add', body, {
    params: new HttpParams().set('id', '3'),
  })
  .subscribe();
// 送出網址會長得這樣： /api/items/add?id=3
```

`HttpParams` 也是 Immutable 物件，相關操作方法如下

```typescript
class HttpParams {
  constructor(options: {...})
  has(param: string): boolean
  get(param: string): string|null
  getAll(param: string): string[]|null
  keys(): string[]
  append(param: string, value: string): HttpParams
  set(param: string, value: string): HttpParams
  delete(param: string, value?: string): HttpParams
  toString(): string
}

```



# 進階技巧

## interceptor

## progress events



# 安全性



