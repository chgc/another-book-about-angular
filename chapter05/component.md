# Component

Component 是畫面主要的組成元素，Angular 主要是透過 component 來提供畫面資料及動作，一個畫面可以由一個或是多個 components 組合，每一個 component 間也可以跟彼此直接或間接溝通

# 基本架構

重新溫習一下在 `Angular 概觀 - 架構` 小節描述的 component 架構，一個基本的 component 會長這樣

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <h1>{{title}}</h1>
    <h2>My favorite hero is: {{myHero}}</h2>
    `
})
export class AppComponent {
  title = 'Tour of Heroes';
  myHero = 'Windstorm';
}
```

一個完整的 component 會包含 `metadata` 與 `class` 的部分，`metadata` 有以下設定屬性，其中一定要設定的屬性是 `selector` 與 `template` / `templateUrl`

```typescript
@Component({ 
  changeDetection?: ChangeDetectionStrategy
  viewProviders?: Provider[]
  moduleId?: string
  templateUrl?: string
  template?: string
  styleUrls?: string[]
  styles?: string[]
  animations?: any[]
  encapsulation?: ViewEncapsulation
  interpolation?: [string, string]
  entryComponents?: Array<Type<any> | any[]>
  preserveWhitespaces?: boolean
  // inherited from core/Directive
  selector?: string
  inputs?: string[]
  outputs?: string[]
  host?: {...}
  providers?: Provider[]
  exportAs?: string
  queries?: {...}
})
```

至於其他的項目，可針對需求作設定，相關說明如下

- **selector** - 設定使用該component 要使用的 css selector，注意: selector 不可以重複

- **styleUrls** - 設定外部 stylesheets 檔案位置

- **styles** - 將 style 寫在 metadata 內 (inline)

- **templateUrl** - 設定外部 template 檔案位置

- **template** - 將 template 寫在 metadata 內 (inline)

- **animations** - 動畫寫在此處

- **changeDetection** - 設定此 component 是用的 changeDetection 規則於此，`ChangeDetectionStrategy.OnPush` | `ChangeDetectionStrategy.Default`

- **encapsulation** - 設定此 component style style 封裝規則，有三種模式
  - `ViewEncapsulation.Emulated` (預設值)
  - `ViewEncapsulation.Native`
  - `ViewEncapsulation.None`

- **viewProviders** - list of providers available to this component and its view children

- **providers** - list of providers available to this component and its children

- **preserveWhitespaces** - 設定是否要保留空白，預設值是 `true`，關閉保留空白的好處是在 AOT 產生的檔案大小可以在小一點，移除空白的是有條件的
  - 移除 template 開頭與結尾的空白 (trimmed)
  - 移除 HTML Element 之間的空白，例如: `<button> Action 1 </button>      <button>Action2</button>` 會變成 `<button> Action 1 </button><button>Action2</button>`
  - 遇到文字型的 element，像 `<span>` 內部連續的空白會移除並使用一個空白代替，`<span>           some text     </span>` 變成 `<span> some text </span>`
  - 遇到 `<pre>` 或是 `<textarea>` 內部的空白會忽略不處理

- **entryComponents** - 設定需要動態載入的 component 物件

- **exportAs** - 設定此 component 輸出的名稱，可定義多個名稱，當外部使用 `template-variable` 取得時，則可以取得該 component class 內建的公開屬性與方法

  ```html
  <app-parent #f="parent"></app-parent>
  ```

  ```typescript
  @Component({
    ...
    exportAs: 'parent'
  })
  ...
  ```

  ​

- **host** - map of class property to host element bindings for events, properties and attributes

- **inputs** - list of class property names to data-bind as component inputs

- **outputs** - list of class property names that expose output events that others can subscribe to

- **queries** - configure queries that can be injected into the component

- **interpolation** - 自訂 component 樣板上標示變數的符號 ; 可將預設的 `{{ }}` 換成其他的符號

- **moduleId** - 當使用 ES/CommonJS 建置 Angular 專案時，須設定 module id ，如使用 Angular CLI 則不需要設定這個項目

## 

剩下的，就靠想像力了

