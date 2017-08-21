# TypeScript

為什麼需要了解 TypeScript 呢? 因為我們會用 TypeScript 來開發 Angular 應用程式，基於這個理由，我們得了解這個由微軟開發出來的 TypeScript。

TypeScript 是什麼? 

> TypeScript is a typed superset of JavaScript that compiles to plain JavaScript.
>
> Any browser. Any host. Any OS. Open source



這一個章節，我們會詳細的介紹 TypeScript 的功能



##  Basic Types

任何程式語言，都要去掌握該語言能使用的資料型別，TypeScript 也不例外，由於 TypeScript 的本質是 JavaScript，所以基本型別跟 JavaScript 是一樣的，但  TypeScript 提供更多的型別可以讓我們在開發時更加的方便。

### Boolean

單純的資料型別，只有 `true` 和 `false ` 兩種狀態，在 JavaScript 與 TypeScript 內的型別為 `boolean`

```typescript
let isDone: boolean = false;
```

### Number

數字型別在 JavaScript 或是在 TypeScript 裡，皆為浮點數型態，型別為 `number`

```typescript
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744
```

### String

TypeScript 的文字可以用單引號 (`'`) 或是 雙引號來表示 (`"`)，但通常文字格式建議使用單引號來表示文字形資料

```typescript
let color: string = "blue";
color = 'red';
```

另外一種表示文字的方式，稱為 `template strings` ，這種的表示方式是使用反單引號搭配 `${ expr } ` 來顯示變數

```typescript
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ fullName }.

I'll be ${ age + 1 } years old next month.`;
```

上述的寫法等同於以下寫法

```typescript
let sentence: string = "Hello, my name is " + fullName + ".\n\n" +
    "I'll be " + (age + 1) + " years old next month.";
```

### Array

TypeScript 的陣列表示，可以像 JavaScript 一樣，使用 `[]` ，當然也可以指定陣列型別

```typescript
let list: number[] = [1, 2, 3];
```

另外一種表示方式

```typescript
let list: Array<number> = [1, 2, 3];
```

### Tuple

Tuple 允許定義一個型別陣列，用來定義陣列內每一個元素的型別。

```typescript
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
```

TypeScript 很聰明的可以判斷目前所選取的資料的型別，如果使用到不屬於該型別可以使用的函數時，在開發時就會顯示錯誤提示

```typescript
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

同樣的， TypeScript 也可以檢查在設定變數資料時，如果餵與的資料型別與原始定義的資料型別不符合時，一樣會出現錯誤提示

```typescript
x[3] = "world"; // OK, 'string' can be assigned to 'string | number'

console.log(x[5].toString()); // OK, 'string' and 'number' both have 'toString'

x[6] = true; // Error, 'boolean' isn't 'string | number'
```

### Enum

Enum 是另外一個有寫過後端的人，非常喜歡的一種型別，TypeScript 當然也支援這個型別

```typescript
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```

如果沒有指定初始的資料時，預設會從 0 開始往下編，但如果一開始用給予初始值時，就會從初始值往後編

```typescript
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
```

另外一種情境是，每一個選項都分別代表不同的值，這時就需要將每一個選項都賦予代表的數值

```typescript
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;
```

假設我們有一個數值，想要透過某 Enum 型別去顯示所對應的選項文字是什麼時，可以用以下的方式達成

```typescript
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];

alert(colorName);
```

而在 TypeScript 2.4 版，TypeScript 介紹了另外一種 String Enum 的表示法，可以使用文字作為 Enum 選項的初始值，範例如下

```typescript
enum Colors {
    Red = "RED",
    Green = "GREEN",
    Blue = "BLUE",
}
    
 console.log(Colors["RED"]); //output: Red
```

### Any

雖然 TypeScript 是賦予 JavaScript 型別的一種語言，但有時候我們使用的第三方套件沒有提供結構定義檔時，我們只能告訴 TypeScript 說，目前所使用的變數，他可能是任何型別，這樣在編譯的過程中，就不會跳出型別錯誤的提示，雖然很方便，在開發專案時，我們的建議還是少用 `any` 的型別

```typescript
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean

let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.

let list: any[] = [1, true, "free"];

list[1] = 100;
```

### Void

`void` 這型別是告訴 TypeScript 說，目前的這個函數不會回傳任何資料，或是變數只能接受 `null` 或 `undefined`

```typescript
function warnUser(): void {
    alert("This is my warning message");
}

let unusable: void = undefined;
```

### Null and Undefined

`null` 與 `undefined` 各屬獨立的型別，他需要搭配其他的型別，才能展現出本身的價值，TypeScript 在 2.0 版本之後，介紹了 `--strictNullChecks` 的參數，當開啟這個參數時，TypeScript 就會檢查是否有 `null` 或是 `undefined` 被指定到不符合或是不允許型別組合的情形發生。

例如說，有一個變數是 `string` 型別，當指定 undefined 至該變數時，TypeScript 就會有錯誤提示。

```typescript
// --strictNullChecks ON
let name: string;
name = undefined ; // Error

let name: string | undefined;
name = undefined ; // OK
name = null ; // Error

let name: string | undefined | null;
name = undefined ; // OK
name = null ; // ok
```

`string | undefined` 是型別集合的概念，這會在後面的內容內介紹到，這裡就先知道就好。

>  官方強烈的建議要將 `strictNullChecks` 的功能給打開，這樣可以在開發時期，避免許多不必要的錯誤。

### Never

`never` 是一個很特殊的型別，也是在 TypeScript 2.0 所新增的型別之一。`never` 代表**不會有任何值發生**。使用情境可以是用來判斷一個函數是否有涵蓋所有的可能性

```typescript
// Inferred return type is number
function move1(direction: "up" | "down") {
    switch (direction) {
        case "up":
            return 1;
        case "down":
            return -1;
    }
    return error("Should never get here");
}

function error(message: string): never {
    throw new Error(message);
}

// or Function returning never must have unreachable end point
function infiniteLoop(): never {
    while (true) {
    }
}
```

### Type assertions

TypeScript 提供了型別轉型的方法，使用方式有 `<T>` 或是 `as T`，`T` 為要轉換的型別，主要目的是告訴 TypeScript 說，「我很確定這個物件或是變數的型別是我說的這個，所以請使用我指定的型別做後續的判斷」

```typescript
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;

// or use as-
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```



## Variable Declarations

ES6 (ES2015) 推出了兩種定義變數的新方式 `let` 和 `const` ，同時也建議不要再使用 `var` 來定義變數。

在介紹 `let` 與 `const` 之前，先解釋一下為什麼不建議再使用 `var` 的原因

### No more Var

在 ES5 以前的年代裡，都使用 `var` 來定義變數

```javascript
var a = 10;
```

當然也可以在 function 內使用 `var` 定義變數

```javascript
function f() {
    var message = "Hello, world!";

    return message;
}
```

或是這樣子定義變數，在 function 內所定義的其他 function 也可取得該變數的值

```javascript
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}
var g = f();
g(); // 回傳 11
```

但是，這樣子的定義方式，當 function 結構越複雜時，就越難追蹤該變數值的變化，例如以下範例，當寫程式或是接手程式的人，對於 JavaScript 的特性不熟悉時，就會有迷惑的情形發生

```javascript
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns '2'
```

而 `var` 還有另外一個奇怪的特性，就是 `scoping rule`，其實嚴格的說，應該是 JavaScript Hoisting 的特性造成的，根據 MDN 的解釋

> 提升 ( Hoisting ) 是您在 JavaScript 文檔中找不到的項目。提升是在 JavaScript 中需要思考程式片段的前後關係（特別是於創建和執行階段）通常是如何進行的地方，而且，提升可能會引起誤解。例如，提升使變數和函數的宣告被移動到您編寫的程式區塊頂端，但這並非發生於您編寫程式時，此發生於編譯階段，變數和函數的宣告提升會於放入記憶體中時處理，但在您編寫的程式碼中，仍然保留於您所鍵入的位置。

簡單的說，變數或是函數的宣告，都會自動被提升到函數或是程式碼的最上面，這個規則有好有壞，先說好的部分

```javascript
catName("Chloe");

function catName(name) {
  console.log("My cat's name is " + name);
}
```

上列的程式碼，即使函數是在呼叫函數之後才定義的，基於`提升`原則，這段程式碼還是可以正常運作的

但是，以下的程式碼範例就是顯示 `提升` 原則造成的思考邏輯上的混淆

```javascript
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }
    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```

上列的程式碼在 JavaScript 的編譯器裡面，會被轉換成這樣，也就能解釋為什麼第二次呼叫時，會回傳 `undefined` 了

```javascript
function f(shouldInitialize: boolean) {
    var x;
    if (shouldInitialize) {
        x = 10;
    }
    return x;
}
```

簡單的說，用 `var` 定義的變數，沒有真正的 `scope` 觀念，`var` 所認知的 `scoping` 定義是根據函數為 `scoping` 的界線，有人稱之為 `function-scoping`，這就延伸出另外一個很常見的錯誤，**var in for loop**

```javascript
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}

// or

for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

這兩段程式碼跑出來的結果，是我們真正所預期看到的結果嗎? 可以自己跑看看就知道了。

這也是為什麼在 ES6 以後的變數定義，不建議使用 `var` 而一律使用 `let`  或是 `const`來定義變數了。



### let

### const









## Interface

## Class

## Functions

## Generics

## Enums

## Type Inference

## Type Compatibility

## Advanced Types

## Symbols

## Iterators and Generators

## Modules

## Decorators



