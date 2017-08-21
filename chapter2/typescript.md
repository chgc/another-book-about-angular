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

TypeScript 很聰明的可以判斷目前所選取的資料的型別，如果使用到不屬於該型別可以使用的方法時，在開發時就會顯示錯誤提示

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

`void` 這型別是告訴 TypeScript 說，目前的這個方法不會回傳任何資料，或是變數只能接受 `null` 或 `undefined`

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

`never` 是一個很特殊的型別，也是在 TypeScript 2.0 所新增的型別之一。`never` 代表**不會有任何值發生**。使用情境可以是用來判斷一個方法是否有涵蓋所有的可能性

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

ES6 (ES2015) 推出了兩種新方法 `let` 和 `const`  可以定義變數，同時也建議不要再使用 `var` 來定義變數。

在介紹 `let` 與 `const` 之前，先解釋一下為什麼不建議再使用 `var` 的原因

### No more Var

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



