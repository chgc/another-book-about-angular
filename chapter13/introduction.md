# 測試框架

透過 Angular CLI所建立的專案，會內建兩個測試 Framework，`Karma` 與 `Protractor`，兩者皆使用 `jasmine 測試語言`。

## Jasmine

Jasmine 是一個測試 JavaScript 語言框架，內建許多 API 能協助我們完成整個測試案例撰寫。

Jasmine 內建的 API 有很多，在這篇我們只介紹幾個比較常用的指令，詳細的說明可以到 Jasmine 網站查詢

一個完整的 Jasmine 測試案例如下

```typescript
describe("A suite is just a function", function() {
  var a;
  it("and so is a spec", function() {
    a = true;
    expect(a).toBe(true);
  });
});
   
```

在這個範例裡面基本上會含以下幾個元素

* describe：建立一個測試群組
* it：建立一個測試案例，一個測試群組內可以有多個測試案例
* expect：建立一個預期測試，一個測試案例內可以有多個預期測試 
* toBe：預期判斷式，有一系列類似的判斷式，一個預期測試只會有一個預期判斷式

Jasmine 當然不只有這些 API， 其實 jasmine 每一個測試群組有一個執行順序，當測試案例複雜時，就需要搭配其他的 API 完成，在之後的章節內在詳細介紹了

## Karma

Karma 是一個管理並執行測試的工具，有以下特色

1. 在真實環境上測試，例如瀏覽器、手機、平板、PhantomJS
2. 支援多種測試框架
3. 支援 Debugger
4. 可以與 CI 工具整合
5. Open source
6. 多種測試報表選擇
7. 可以安裝擴充套件

## Protractor

Protractor 是一套 End-to-End 測試框架，只要是透過模擬使用者操作網頁的動作來測試程式的正確性

Protractor 原本的目的是為了測試 AngularJS 應用測試，但是強大的功能也能讓我們當一般的 E2E 測試框架，以下是一個測試案例的範例

```typescript
describe('angularjs homepage todo list', function() {
  it('should add a todo', function() {
    browser.get('https://angularjs.org');

    element(by.model('todoList.todoText')).sendKeys('write first protractor test');
    element(by.css('[value="add"]')).click();

    var todoList = element.all(by.repeater('todo in todoList.todos'));
    expect(todoList.count()).toEqual(3);
    expect(todoList.get(2).getText()).toEqual('write first protractor test');

    // You wrote your first test, cross it off the list
    todoList.get(2).element(by.css('input')).click();
    var completedAmount = element.all(by.css('.done-true'));
    expect(completedAmount.count()).toEqual(2);
  });
});
```

沒錯，我們也可以使用 Jasmine 來寫測試案例給 Protractor 使用，搭配 protractor 的 api，我們就可以輕鬆的找到網頁上的物件並操作該物件了，細節的部分等到 e2e 測試章節再討論了