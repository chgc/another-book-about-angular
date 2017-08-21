# TypeScript 設定檔

在正式介紹 TypeScript 之前，先了解一下 `tsconfig.json` 檔案，這個設定檔讓我們能設定 TypeScript 編譯器的編譯專案的方式

以下為 Angular CLI 新增專案時，預設會產生的 `tsconfig.json` 

```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "outDir": "./dist/out-tsc",
    "sourceMap": true,
    "declaration": false,
    "moduleResolution": "node",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "target": "es5",
    "typeRoots": [
      "node_modules/@types"
    ],
    "lib": [
      "es2017",
      "dom"
    ]
  }
}
```

`tsconfig.app.json`

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "outDir": "../out-tsc/app",
    "module": "es2015",
    "baseUrl": "",
    "types": []
  },
  "exclude": [
    "test.ts",
    "**/*.spec.ts"
  ]
}

```

`tsconfig.spec.json`

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "outDir": "../out-tsc/spec",
    "module": "commonjs",
    "target": "es5",
    "baseUrl": "",
    "types": [
      "jasmine",
      "node"
    ]
  },
  "files": [
    "test.ts"
  ],
  "include": [
    "**/*.spec.ts",
    "**/*.d.ts"
  ]
}

```



# 設定細節

## compilerOptions

`compilerOptions` 可以說是 `tsconfig.json` 核心設定區塊，這區塊定義了 TypeScript 的編譯模式，及在開發模式下 TypeScript 如何檢查程式碼

* **outDir**：編譯後的輸出資料夾

* **sourceMap**：是否產生 `sourceMap`

* **declaration**：是否產生 `.d.ts` 檔案 

* **moduleResolution**： 設定解析模組的模式，有以下模式可供設定

  * AMD
  * System
  * ES6
  * Classic
  * Node

* **emitDecoratorMetadata**：Emit design-type metadata for decorated declarations in source

* **experimentalDecorators**：支援 ES decorators 

* **target**：目標編譯的 JavaScript 版本，有以下版本可供輸出使用

  * ES3
  * ES5
  * ES6/ES2015
  * ES2016
  * ES2017
  * ESNext

* **typeRoots**：設定含有型別定義檔的來源資料夾

* **types**：設定需要引入的型別定義檔的資料夾

* **lib**：需包含至編譯過程中的 Library 檔案，目前可使用的項目如下

  ► `ES5` 
  ► `ES6` 
  ► `ES2015` 
  ► `ES7` 
  ► `ES2016` 
  ► `ES2017` 
  ► `ESNext` 
  ► `DOM` 
  ► `DOM.Iterable` 
  ► `WebWorker` 
  ► `ScriptHost` 
  ► `ES2015.Core` 
  ► `ES2015.Collection` 
  ► `ES2015.Generator` 
  ► `ES2015.Iterable` 
  ► `ES2015.Promise` 
  ► `ES2015.Proxy` 
  ► `ES2015.Reflect` 
  ► `ES2015.Symbol` 
  ► `ES2015.Symbol.WellKnown` 
  ► `ES2016.Array.Include` 
  ► `ES2017.object` 
  ► `ES2017.SharedMemory` 
  ► `esnext.asynciterable` 

* **module** ：產生特定版本的模組程式碼

  * 有以下模式可供設定
    * None
    * CommonJS
    * AMD
    * System
    * UMD
    * ES6
    * ES2015
    * ESNext
  * 當設定為 `AMD` 或是 `System`時，才可以使用 `outFile` 的參數
  * 如果 `target` 設定為 `ES5` 或是更低版本時，可以使用 `ES6` 或 `ES2015`模式

* **baseUrl**：用於解析模組間關係時的根目錄

* **strictNullChecks**：啟動 `strict null` 檢查模式，詳細細節可以參閱 [TypeScript 2.0 新增功能](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html)

* **strict**：啟動多項 strict 項目，`noImplicitAny`、`noImplicitThis`、`alwaysStrict`  和 `strictNullChecks`，詳細細節可以參閱 [TypeScript 2.3 新增功能](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-3.html)

* **paths** ：設定基於 `baseUrl` 的路徑別名對應表，設定方式及使用方式如下

  ```json
  "baseUrl": "src",
  "paths": {
   "@env/*": ["environments/*"]
  }
  ```

  ```typescript
  import * as env from '@env/environment';
  ```



## 其他

- **exclude**：排除資料夾或檔案
- **include**：包含資料夾或檔案
- **files**：需要編譯的檔案清單
- **extends**：繼承 `tsconfig.json` 檔案，複寫或擴充所繼承的 `tsconfig.json` 設定檔



# 總結

TypeScript 隨著版本的演進，功能越來越強大，或許有些功能是我們用不到的，透過 `tsconfig.json` 的設定，讓就可以選擇我們所需的功能，花點時間了解 `TypeScript` 的設定方式，說不定就能大幅提升開發的品質及速度了。



# 參考資料

* [tsconfig](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
* [Compiler Options](https://www.typescriptlang.org/docs/handbook/compiler-options.html)