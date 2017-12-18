# 為什麼需要使用依賴注入方式

為什麼需要依賴注入? 簡單的說是因為要方便抽換模組功能。為什麼使用 DI 模式就可以簡單的抽換模組呢? 以下簡單的說明

情境如下

> 假設 Component 需要透過使用者授權服務，檢查使用者的授權狀態

程式碼大概會長得這樣

```typescript
export class LoginComponent {
  login(username, password){
    if(this.authService.login(username, password)){
      // success
    }else{
      // fail
    }
  }
}
```



## 沒有使用 DI 時

如果沒有 DI 的情況下，這時我們就需要在 constructor 裡，自己建立 authSerivce

```typescript
export class LoginComponent {
  authSerivce: AuthService;
  constructor(){
    this.authSerivce = new AuthService();
  }
  ...
}
```

這樣子的寫法有哪些缺點

1. 每當建立一個 class 都會建立一個獨立的 service 實體，這個當遇到 ngFor components 就會出現問題

2. 建構子內有邏輯程式

   ```typescript
   class myComponent{
     users;
     orders;
     
     constructor(){
       this.users = new users(new Orders(new LineImtes(new Discounts())));
       this.orders = new Orders(new LineImtes(new Discounts()));
     }
   }
   ```

   上列的程式碼當複雜時，就會變得無法維護

3. 不容易被測試

   通常在測試時，我們不希望直接去呼叫真正的後端服務，所以會透過 mock/spy 的方式替代真的服務，當服務是在 constructor 內建立時，這樣會讓測試變得比較困難

4. 隱藏的相依性

   如果 service 是直接在 constructor 下建立時，就不容易發現該 service 是否有相依其他的 class



## 使用 DI 時

```typescript
export class LoginComponent {
  authSerivce: AuthService;
  constructor(auth: AuthService){
    this.authSerivce = auth;
  }
  ...
}
```

而 DI 機制就會判斷該 Component Class 需要哪一個 service，進而指派所需的 service，詳細的運作流程會在下一小節說明，這樣的好處有

1. 避免在 constructor  內撰寫任何邏輯程式碼
2. Singleton Instance：可以共用同一個 service
3. Dependency Instantiation Order
4. Enable Testing
5. Explicitly State Collaborators



## 總結

Angular 內建的 DI 機制可以說是非常的完整，利用 DI 的幫忙，可以讓我們更容易管理 services 與 component 間的相依性，更容易測試

















