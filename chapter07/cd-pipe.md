# Pipe 與 ChangeDetection

Angular 會透過 `Change Detection` 方法執行的過程中，去檢查 data-bound值的變化，而 `Change Detection` 會在每一次 DOM Event 後被觸發，例如按下鍵盤的鍵、滑鼠的移動、伺服器的回應等事件，這個過程是需要付出相對的成本，為了效能，Angular 會盡量降低 `Change Detection` 的次數。

所以 Pipe 會採用最簡單又快速的判斷規則，[ChangeDetectionStrategy.OnPush](https://angular.io/docs/ts/latest/api/core/index/ChangeDetectionStrategy-enum.html)。

這表示當 Pipe 如果用在陣列上，就有機會出現不在預期內的顯示結果，如以下的範例

```typescript
import {Pipe, PipeTransform} from '@angular/core';

import {Flyer} from './app.component';

@Pipe({name: 'flyingHeroes'})
export class FlyingHeroesPipe implements PipeTransform {
  transform(allHeroes: Flyer[]) {
    return allHeroes.filter(hero => hero.canFly);
  }
}
```

component

```typescript
@Component({
  selector: 'app-root',
  template: `
     <input type="text" #box
          (keyup.enter)="addHero(box.value); box.value=''"
          placeholder="hero name">
    <button (click)="reset()">Reset</button>
    <div *ngFor="let hero of heroes | flyingHeroes">
      {{hero.name}}
    </div>
`,
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  heroes: any[] = [];
  canFly = true;
  constructor() {
    this.reset();
  }

  addHero(name: string) {
    name = name.trim();
    if (!name) {
      return;
    }
    let hero = {name, canFly: this.canFly};

    // this.heroes.push(hero); // 這個不會更新畫面，因為不符合 OnPush 的條件
    
    this.heroes = [...this.heroes, hero]; // 因會產生一個新的 Array Object, 所以會觸發 CD
  }

  reset() {
    this.heroes = HEROS.slice();
  }
}
```

