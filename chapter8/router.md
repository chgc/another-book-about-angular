# Router

Angular 也有內建路由機制，透過路由機制，我們可以做到頁面至頁面的轉換、Lazy loading 模組、或是路由 Guard 等功能，這章節會一一介紹路由的功能

## 基本設定

Angular 設定路由的方式有兩種，`RouterModule.forRoot` 跟 `RouterModule.forChild`，這兩個方法都接受 `Routes`，而 `forRoot` 還可以額外傳入設定參數

`forRoot` 與 `forChild` 的差異