# Reactive Form

Reactive Form 是 Angular 另外一種方式來專寫表單，讓我們能用程式的方式來控制表單的所有行為，只需要將表單元素綁定一個 `FormControlName`，這樣就可以透過程式來取得表單的所有資訊

```html
<input type="text" formControlName="firstName" />
```

```typescript
export class DemoComponent {
	firstNameControl = new FormControl();  
}
```

如同 Template-Driven Form，Reactive Form 也有基本三元素: `FormGroup`、`FormControl` 和 `FormArray`

我們可以透過這三元素來完成表單結構的組成，拿之前所使用的範例來轉換成 Reactive Form 模式

**Template Drive Form 樣式**

```html
<!-- Template driven Form -->
<form #f="ngForm" (ngSubmit)="login(f)" novalidate>
  <label for="username">UserName</label>
  <input type="text" name="username" ngModel required />
  <label for="password">Password</label>
  <input type="password" name="password" ngModel required/>
  <button type="submit" [disabled]="f.invalid">Login</button>
</form>
```
**Reactive Form 樣式**

```html
<!-- Model drive Form (Reactive Form) -->
<form [formGroup]="formData" (ngSubmit)="login()" novalidate>
  <label for="username">UserName</label>
  <input type="text" name="username" formControlName="userName" />
  <label for="password">Password</label>
  <input type="password" name="password" formControlName="password" />
  <button type="submit" [disabled]="formData.invalid">Login</button>
</form>
```

```typescript
export class DemoComponent {
  formData = new FormGroup({
    userName: new FormControl('', Validators.required),
    password: new FormControl('', Validators.required)
  });
}
```

這兩者模式在一個簡單的表單需求下，可能看不出 `Reactive Form`的強大，但當表單的使用者互動性很高時，這時候 Reactive Form 就可以展現出他的強大，可以用以下的範例來做個火力展示

## 範例

有一個可以搜尋 wikipedia 的輸入欄位，可以邊打邊取得搜尋結果

```html
<form [formGroup]="formData">
    <div>
        <label for="search">搜尋</label>
        <input type="text" name="search" formControlName="search" />
    </div>
</form>
<ul>
    <li *ngFor="let item of searchResult$ | async">
        {{ item }}
    </li>
</ul>
```

```typescript
import { mergeMap } from 'rxjs/operators';

export class DemoComponent {
  searchResult;
  
  formData = new FormGroup({
    search: new FormControl()
  });
  
  this.formData.get('search').valueChanges
    .pipe(
       mergeMap(inputValue => this.wikiService.search(inputValue))
  	)
    .subscribe(searchResult=>{
      this.searchResult = searchResult;
  });
}
```

**程式說明**

* `pipe` 是 RxJS 5.5 版本以後提供的方法，可以讓我們以串接的方式將各個運算式使用逗號的方式連結再一起，這樣的模式提供我們更彈性的方式封裝 RxJS 運算式
* `FormGroup.get(path)` 可以取得所指定的 `FormControl`
* `valueChanges`  是 FormControl 內所提供的 Observable 型別的屬性，可以讓我們監聽資料異動

以上的程式碼可以說是完成基本的 Reactive Form 了，接下來一些比較特殊的需求，我們就不用再異動到 HTML 的部分了，直接修改 component class 即可

# FormBuilder

`FormBuilder` 是一個可以快速建立 FormGroup 的 Helper，這個 Helper 可以可以快速的建立 `group`、`control` 和 `array` 

用法範例

```typescript
export class DemoComponent {
  formData = this.fb.group({
    search: [],
    subGroup: this.fb.group({
      ...
    }),
    subArray: this.fb.array({
      
    })
  });

  constructor(private fb: FormBuilder){}
}
```



# Reactive Form 的成員

以下分別列出 `FormGroup`、`FormControl` 和 `FormArray` 常用的屬性及方法

- FormGroup: 容器，用來包 FormControl、FormArray

- - contructor(value, validatorOrOpts)

  - - 可設定值，驗證，及 updateOn 選項

  - Control 的控制

  - - registerControl vs addControl

    - - registerControl 註冊 Control 到 Group 內並回傳 Control實體，而且不會觸發 valueChanges跟 validation
      - addControl 新增 Control 到  Group 內，不會回傳東西而且會觸發 valueChanges跟 validation

    - controls: FormGroups 下的 control 陣列清單

    - removeControl ： 移除 control

    - setControl: 替換 control

    - `contains` VS  `get`

    - - contains:  檢查 control 是否為 enabled
      - get: 從 FormGroup 內取得 control，不限定狀態

    - setValue vs patchValue

    - - setValue 會替換整個 FormGroup 內 controls 的值，所以要 setValue 的物件必須與 FormGroup 相符

      - patchValue 可以設定 FormGroup 內部分 controls 的值，patchValue 傳入的物件欄位數量不一定要和 FormGroup 相符

      - options: {onlySelf?: boolean, emitEvent?: boolean}

      - - onlySelf: 只會觸發自身的 Validation
        - emitEvent: 是否會觸發 valueChanges 事件

    - reset: 重置 FormGroup 值與狀態

  - value vs getRawValue()

  - - value: 只取出非 disabled 的 control 資料
    - getRawValue(): 取得 FormGroup 內所有 control 的資料

  - updateValueAndValidity(): 重新觸發驗證

  - get(<path>) 取得 Group內的 control

  - - 如果有sub group 裡的 control 時，可以這樣子使用 get(‘subgroup.controlName’) 或是 get([‘subgroup’, ‘controlName’]) 取得 control


- FormControl: 

- - 設定初始值及狀態

  - - new FormControl(value)

    - new FormControl({value: ’123’, disabled: true})

    - - control.value; // 123
      - control.status // disabled

    - new FormControl(‘’, Validators.required)

    - - control.value; // <空白>
      - control.status // invalid

    - new FormControl(‘’, {  validators: [Validators](https://angular.io/api/forms/Validators).required,  asyncValidators: myAsyncValidator, updateOn: ‘blur’ })

  - setValue vs patchValue 的行為同 FormGroup

  - - emitModelToViewChange: 是否觸發 onChange 事件 (預設值: true)
    - emitViewToModelChange: 是否觸發 ngModelChange 事件 (預設值: true)

  - registerOnChange(fn): 註冊一個 callback 到 FormControl 的 change 事件 (自訂 FormControl 用)

  - registerOnDisabledChange(fn): 註冊一個callback 到 FormControl disabled events (自訂 FormControl 用)

  - 錯誤處理

  - - setError

    - - 手動設定錯誤訊息

      - - setError({‘myError’: true}) 

      - 清除錯誤訊息

      - - setError(null)

    - hasError

    - - hasError(‘myError’)

    - getError: boole

  - enabled(opts?) & disabled(opts?)

    - opts: {
      ​    onlySelf?: boolean;
      ​    emitEvent?: boolean;
      } = {}
    - onlySelf: 只會觸發自身的 Validation
    - emitEvent: 是否會觸發 valueChanges 事件


- FormArray:  陣列用來裝 FormGroup/ FormControl

- - 基本操作

  - - push: 新增 control 到最後面
    - insert: 新增 control 至特定位置
    - removeAt: 移除 control
    - length: 取得目前有的 control 數量
    - setValue/patchValue/reset : 給予一個陣列的值，陣列內的資料會依序更新到 FormArray 內的 control 

# 小結

Reactive Form 的強大是需要搭配 RxJS 跟表單的複雜度才能顯示出來，如果一個表單的互動性越高，使用 Reactive Form 的模式開發效益愈高，當然簡單的表單也是可以使用

Template-Driven Form 與 Reacitve Form 並沒有說誰比較優，誰比較不優，兩者都可以完成工作，所以就看個人或是團隊如何決定了，多一個選擇總是好的，不是嗎?



# 參考資料

* [Reactive Form](https://angular.io/guide/reactive-forms)
* [Form Builder](https://angular.io/api/forms/FormBuilder)