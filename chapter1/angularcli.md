---
typora-root-url: ./
typora-copy-images-to: images
---

# Angular CLI

Angular CLI 是 Angular Team 與社群一起合作創造出來的工具，Angular CLI (以下簡稱為 CLI) 提供一系列的指令，讓開發者可以從建置專案、開發、測試、一直到部屬，都可以透過 CLI 的幫助下完成。



# 安裝篇

我們可以透過 `npm` 安裝 CLI，打開命令視窗輸入以下的指令 (windows/mac 皆適用)

> npm install -g @angular/cli

![](/images/npminstall.jpg)

當安裝完成後，可以透過 `ng -v` 的指令來檢查安裝的版本

![](/images/2017-08-16_19-56-50.jpg)

當看到這個畫面時，就表示 Angular CLI 安裝成功了

# 指令篇

以下命令都需要透過 `ng` 使用，範例如下

> ng <command>

## new

`ng new [專案名稱]` 建立新的 Angular 專案，預設會建立一個與專案名稱相同的資料夾，並在該資料夾內初始化 Angular 專案

參數

* **directory**

  * `--directory` (alias: `-dir`)  預設值 ： `[專案名稱]` 
  * 設定建立專案的隸屬資料夾名稱

* **dry-run**

  * `--dry-run` (alias: `-d`)  預設值：`false`
  * 顯示建立專案時會產生的檔案清單，但不會實際產生實體檔案

* **inline-style**

  * `--inline-style` (alias: `-is`)  預設值：`false`
  * 使用 `inline-style` 模式

* **inline-template**

  * `--inline-template` (alias: `-it`)  預設值：`false`
  * 使用 `inline-template` 模式

* **minimal**

  * `--minimal` 預設值：`false`

  * 產生最小單位的 Angular 專案

  * 最小單位的 Angular 專案結構

    ![2017-08-16_21-41-33](/images/2017-08-16_21-41-33.jpg)

  * 完整的 Angular 專案架構

    ![2017-08-16_21-43-07](/images/2017-08-16_21-43-07.jpg)

* **prefix**

  * `--prefix` (alias: `-p`)  預設值: app
  * 給所有 `component selector` 使用的前綴詞
  * 可事後由 `.angular-cli.json` 設定

 

## serve

## generate

## build

## lint

## e2e

## xi18n

## doc

# 設定篇

# 應用篇

