# Template Driven Form

有寫過 AngularJS 的人一定知道 `ng-model` 這讓人又愛又恨的功能，雙向繫結允許我們可以很簡單的收集表單上的資料，同樣的效果不同的實踐方式，一樣存活在 Angular 的框架裡，搭配 `[(ngModel)]` 就可以完成雙向繫結的效果，但是這裡就不探討底層的運作原理，Angular 內的雙向繫結沒有 AngularJs 效能上的問題，可以安心使用

```html
<input type="text" [(ngModel)]="firstName" />
```

`NgModel` 在 Angular 裡面有另外一個功能，那就是賦予 Form Element 擁有 FormControl 的功能

```html
<input type="text" name="firstName" ngModel #firstName="ngModel" />
```

透過樣版變數取得 NgModel 物件，所取得的物件本身具有 FormControl 的所有功能及特性，包括驗證狀態，資料等

表單當然不可能這麼簡單，以下是一個很常見的登入表單

```html
<form #f="ngForm" (ngSubmit)="login(f)" novalidate>
  <label for="username">UserName</label>
  <input type="text" name="username" [(ngModel)]="userInfo.username" />
  <label for="password">Password</label>
  <input type="password" name="password" [(ngModel)]="userInfo.password" />
  <button type="submit">Login</button>
</form>
```

或是透過 ngForm 取得資料(建議使用這種方式)

```html
<form #f="ngForm" (ngSubmit)="login(f)" novalidate>
  <label for="username">UserName</label>
  <input type="text" name="username" ngModel required />
  <label for="password">Password</label>
  <input type="password" name="password" ngModel required/>
  <button type="submit" [disabled]="f.invalid">Login</button>
</form>

```

這兩者都可以取得表單上所輸入的資料

`ngForm` 會產生一個最上層的 FormGroup ，FormGroup 下會涵括所有的 FormControl 跟子FormGroup，之後將 範本變數所接受的 FormGroup 物件傳到 Component Class 時，就可以取得畫面上表單資料及表單的驗證狀態等資訊

![](images/2017-11-11_15-21-22.png)

上圖列出 FormGroup 有的屬性與方法可供使用

* control：是取得自己的狀態
* controls：FormGroups 所包含的 FormControls
* dirty： 資料使否有異動
* disabled：停用狀態
* enabled：啟用狀態
* errors：FormControl 的錯誤訊息
* valid：是否通過所有的驗證規則
* value：FormControp 下 FormControls/FormGroups 的資料集
* valueChanges：可持續監控 FormGroups/FormControls 值的變化
* statusChanges: 可持訊監控 FormGroups/FormControls 狀態的變化

如果想要將 `NgModel` 群組化呢? 這時可以透過 `NgModelGroup` 的方式來完成

```html
 <div ngModelGroup="name" #nameCtrl="ngModelGroup">
   <input name="first" [ngModel]="name.first" minlength="2">
   <input name="last" [ngModel]="name.last" required>
</div>
```

這樣可以輸出以下的結果

```
{
  name: {first: '123', last: '456'}
}
```

## 驗證

Template-Driven Form 的所有驗證規則都是標示在 html 上，例如

```html
<input type="text" name="first" #first=ngModel required />
```

請留意 `required` 表示這個 input 欄位為必須填寫的欄位，這時可以透過 NgControl 的方式取得該欄位的驗證狀態，取得方式是

```html
{{ first.errors | json }}
// 顯示的錯誤訊息
{
  "required": true
}
```

還有其他內建的驗證規則，規則如下

* min - 設定輸入值得最小值，必須大於或等於此設定
* max - 設定輸入值得最大值，必須小於或等於此設定
* required - 必需輸入資料
* requiredTrue - 必須為真
* email - 驗證輸入資料是符合 email 格式
* minLength - 輸入值的最小長度
* maxLength - 輸入值的最大長度
* pattern - 利用 Regex 來制定驗證規則
* nullValidator - 沒有作用的驗證規則，只會回傳 null
* compose - 合併多個驗證規則再一起
* composeAsync - 同上，是針對非同步的驗證規則



## 自訂驗證

製作自訂驗證規則可以透過 directive 的方式來完成

```typescript
@Directive({
  selector: '[appForbiddenName]',
  providers: [{provide: NG_VALIDATORS, useExisting: ForbiddenValidatorDirective, multi: true}]
})
export class ForbiddenValidatorDirective implements Validator {
  @Input() forbiddenName: string;
 
  validate(control: AbstractControl): {[key: string]: any} {
    const nameRe = new RegExp(this.forbiddenName, 'i');
    const forbidden = nameRe.test(control.value);
    return forbidden ? {'forbiddenName': {value: control.value}} : null;
  }
}
```

* selector: 定義要在 html 上使用的屬性名稱
* providers: 註冊這個 Validator 到 NG_VALIDATORS 的集合中，記得一定要加上`multi: true`
* 實作 Validator 介面
  * validate 只會回傳 `{錯誤訊息: value}` 或是 null 兩種結果

## 狀態樣式

一個物件可以針對以下的狀態，分別對應到 css class上

* valid - .ng-valid
* invalid - .ng-invalid
* pending - .ng-pending
* pristine - .ng-pristine
* dirty - .ng-dirty
* untouched - .ng-untouched
* touched - .ng-touched

所以我們可以透過 css 的方式來針對這些狀態來顯示樣式

```css
.ng-valid[required], .ng-valid.required  {
  border-left: 5px solid #42A948; /* green */
}

.ng-invalid:not(form)  {
  border-left: 5px solid #a94442; /* red */
}
```

這也會延伸一些控制狀態的方法，例如使用 reset() 來將表單的狀態回歸的初始值，或是執行 `markAsTouched` 相關的方法來改變狀態



## 小結

Template-Driven Form 是一個可以快速開發表單的方式，但是遇到表單比較複雜，或是需要動態調整表單內容時，Template-Driven 的開發方式就會感到有點吃力，或是在維護上比較困難了。

所以 Angular 團隊推出另外一種表單的開發方式，Reactive Form，將會在下一章節介紹



## 參考資料

* [NgModel](https://angular.io/api/forms/NgModel)
* [NgModelGroup](https://angular.io/api/forms/NgModelGroup)
* [Validators](https://angular.io/api/forms/Validators)