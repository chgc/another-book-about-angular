# NgModules

NgModules 可以根據功能性來組織應用程式的程式碼，讓應用程式的程式碼更容易整理及擴充。

NgModules 是一個 Class 掛上 `@NgModule` 裝飾器，`@NgMoudle` 接受一個 metadata 物件用來設定 `@NgModule`。

```typescript
@NgModule({
  // metadata
  ...
})
export class AppModule {}
```

## 模組化

對於組織程式或是擴充既有程式，模式化是一個很有效的方式，Angular 本身也是使用模組化的方式，將類似的功能整理在一起，例如：`FormsModule` 、 `HttpModule`  等，甚至第三方套件也採用一樣的模式做發佈，讓我們可以簡單的安裝並使用。

