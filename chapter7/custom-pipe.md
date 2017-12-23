

# 自訂 Pipe

透過 CLI 產生 `Pipe` 是最快的，指令是

```typescript
ng g p "pipeName"
```

所產生出來的基本架構是

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'exponential'
})
export class ExponentialPipe implements PipeTransform {

  transform(value: number, exponent: string): number {
    let exp = parseFloat(exponent);
    return Math.pow(value, isNaN(exp) ? 1 : exp);
  }
}
```

- `transform` function 所回傳的值，會用來顯示在畫面上
- 第一個參數 `value` 是所要轉換的資料來源
- 第二個之後的參數可以用來接 Template 傳給 `pipe` 的參數值

如果要傳入多個參數的時後，transform 的地方值直接加上第3的參數或是使用 ...args 也是可以，而在 template 的使用方式則是 {% raw %}{{ value | xxpipe: 1_arg: 2_arg: 3:arg }}{% endraw %} 以此類推。