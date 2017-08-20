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

# 細節

## compilerOptions

`compilerOptions` 可以說是 `tsconfig.json` 核心設定區塊，這區塊定義了 TypeScript 的編譯模式，及在開發模式下 TypeScript 如何檢查程式碼







# 參考資料

* [tsconfig](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
* [Compiler Options](https://www.typescriptlang.org/docs/handbook/compiler-options.html)