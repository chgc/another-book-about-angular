# Pure and Impure Pipes

Angular 的 Pipe 預設皆為 `Pure`，如果需要設定為 `Impure`的話，請這樣子設定

```typescript
@Pipe({
  name: 'flyingHeroesImpure',
  pure: false
})
...
```

至於什麼是 `Pure Pipe` ，什麼是 `Impure Pipe`

## Pure Pipe

這裡所指的 `Pure`，至針對 `Pipe` 所要轉換的值是否為 `Pure Change`，所謂的 `Pure Change` 是改變 primitive input value( String, Number, Boolean, Symbol) 或是改變  Object 參考的位址 (Date, Array, Function, Object)。

這規則與 [ChangeDetectionStrategy.OnPush](https://angular.io/docs/ts/latest/api/core/index/ChangeDetectionStrategy-enum.html) 是一樣的。在上面的例子中，因為 `heroes` 是一個陣列物件，如果是 `push`的行為並不會改變該陣列所參考的位址 (ByReference)，必須重新建立一個新的陣列物件，才會改變參考位址。



## Impure Pipe

`Impure Pipe` 就是 `Pure`的相反，也是 `ChangeDetectionStrategy` 預設的執行方式，只要有資料異動，就會觸發改變。

來調整一下上面的範例，來讓 array.push 也可以做到畫面更新顯示的功能。

```typescript
import {Pipe, PipeTransform} from '@angular/core';

import {Flyer} from './app.component';

@Pipe({name: 'flyingHeroes', pure: false})
export class FlyingHeroesPipe implements PipeTransform {
  transform(allHeroes: Flyer[]) {
    return allHeroes.filter(hero => hero.canFly);
  }
}
```

當這樣子設定為 `Impure`時，下面的 `push` 就可以使用而且畫面也會更新

```typescript
addHero(name: string) {
    name = name.trim();
    if (!name) {
      return;
    }
    let hero = {name, canFly: this.canFly};

    this.heroes.push(hero); 
  }
  
```

# 