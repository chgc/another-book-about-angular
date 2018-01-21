# MobX

MobX 是 React 社群裡面一個新的火紅的狀態管理，是通過透明的函式型反應式程式設計 (Transparently applying Functional Reactive Programming, TFRP) 使得狀態管理變得更為簡單和提高可擴展性，在這裡我們會透 `mobx-angular` 來讓 Angular 和 MobX 做連接。

如果讀者曾經用過 Angular + NgRx 或 React + Redux 或 Vue + Vuex 都可以很快的切換到 MobX 的，就算沒有用過的也沒關係，因為 MobX 的簡單也是可以快速上手的。

## 核心概念

### Observable states (可觀察的狀態)

  這相當於 Redux 的初始狀態，而在 Vuex 的 States 就是做類似的操作，不同於 Redux，MobX 不是對「值」而是對「屬性」。

### Action functions (行為函式)

  在 Redux 的世界中，除了一般的 Actions，還會有 Reducers，而在 Vuex 就是 Actions 和 Mutations

### Computed values (計算值)

  Redux 本身沒有類似的機制，所以會過 `reselect` 來執行，也就是說這會相當於 Reselect 的 Selector，更明確的叫做 `getters`，而在 Vuex 的 Getters 就是做類似的操作

## 起手式

從一個簡單的計數器開始，透過按鈕觸發的方式來了解 MobX 的基本操作。

建立一個計數器的應用程式

```bash
$ ng new angular-mobx-go
$ cd angular-mobx-go
```

安裝相依性函式庫

```bash
$ yarn add mobx mobx-angular

# 也安裝一下 UI 的函式庫
$ yarn add @angular/material @angular/cdk
```

```ts
// ...

import { MobxAngularModule } from 'mobx-angular';

// ...

@NgModule({
  imports: [
    // ...
    MobxAngularModule,
    // ...
  ],
  // ...
})
export class AppModule {}
```

```ts
// counter/counter.component.ts
import { Component } from '@angular/core';

import { CounterStore } from './counter.store';

@Component({
  selector: 'playground--counter',
  templateUrl: './counter.component.html',
  styleUrls: ['./counter.component.css']
})
export class CounterComponent {
  constructor(public $c: CounterStore) {}
}
```

```ts
// counter/counter.module.ts
import { NgModule } from '@angular/core';
import { MatButtonModule } from '@angular/material';
import { MobxAngularModule } from 'mobx-angular';

import { CounterComponent } from './counter.component';
import { CounterStore } from './counter.store';

@NgModule({
  imports: [
    MobxAngularModule,
    MatButtonModule
  ],
  declarations: [CounterComponent],
  providers: [CounterStore]
})
export class CounterModule {}
```

```ts
// counter/counter.store.ts
import 'rxjs/add/observable/of';
import 'rxjs/add/operator/delay';
import 'rxjs/add/operator/filter';

import { Injectable } from '@angular/core';
import { observable, action, computed } from 'mobx';
import { Observable } from 'rxjs/Observable';

@Injectable()
export class CounterStore {
  @observable value: number = 0;

  @action
  increment(): void {
    this.value++;
  }

  @action
  decrement(): void {
    this.value--;
  }

  @action
  incrementAsync(): void {
    setTimeout(() => this.increment(), 1000);
  }

  @action
  decrementAsync(): void {
    Observable.of(null)
      .delay(1000)
      .subscribe(() => this.decrement());
  }

  @action
  incrementIfOdd(): void {
    if (Math.abs(this.value % 2) === 1) {
      this.increment();
    }
  }

  @action
  decrementIfEven(): void {
    Observable.of(null)
      .filter(() => this.value % 2 === 0)
      .subscribe(() => this.decrement());
  }

  @action
  startCount() {

  }

  @action
  cancelCount() {

  }

  @computed
  get evenOrOdd(): string {
    return this.value % 2 === 0 ? 'even' : 'odd';
  }
}
```
