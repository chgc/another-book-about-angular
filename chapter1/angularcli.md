---
--i18n-formattypora-root-url: ./
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

### 參數

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
  * 給 `component selector` 使用的前綴詞
  * 可事後由 `.angular-cli.json` 設定

* **routing**

  * `--routing`  預設值：`false`
  * 建立路由模組

* **skip-commit**

  * `--skip-commit` (alias: `-sc`)  預設值：`false`
  * 忽略第一次建立時的 git 簽入動作

* **skip-git**

  * `--skip-git` (alias: `-sg`) 預設值：`false`
  * 不建立 git Repository

* **skip-install**

  * `--skip-install` (alias: `-si`) *預設值：`false`*
  * 建立新專案時，不執行安裝套件動作 (install packages)

* **skip-test**

  * `--skip-tests (aliases: `-st) 預設值：`false`
  * 不建立 `spec` 測試檔案
  * 不包含 `e2e` 測試功能

* **source-dir**

  * `--source-dir` (alias: `-sd`) 預設值： src
  * 主專案程式存放的資料夾名稱
  * 可事後由 `.angular-cli.json` 設定 (app[0].root)

* **style**

  * `--style` 預設值：css
  * 支援以下格式
    * css
    * scss
    * less
    * sass
    * styl (stylus)
  * 可事後由 `.angular-cli.json` 設定 (defaults.styleExt)

* **verbose**

  * `--verbose` (alias: `-v`) 預設值：`false`
  * 顯示更多輸出資訊


## serve

`ng serve` 指令會進行專案建置並啟動網頁伺服器

可以搭配使用的參數說明如下

### 參數

* **host**
  * `--host` (aliases: `-H`) 預設值: localhost
  * 預設指監聽 localhost
* **hmr**
  * `—hmr` 預設值: false
  * 啟動模組熱拔插功能
* **live-reload**
  * `--live-reload` (aliases: `-lr`) 預設值: true
  * 使用 `live-reload` 監控異動，當有異動時則會重整畫面。
* **public-host**
  * `--public-host` (aliases: `--live-reload-client`)
  * 指定一個其他使用者可以瀏覽的網址。
* **disable-host-check**
  * `--disable-host-check`  預設值: false
  * 不檢查連線使用者連線是否來自允許的網域。
* **open**
  * `--open` (aliases: `-o`) 預設值: false
  * 在預設瀏覽器打開網址
* **port**
  * `--port` (aliases: `-p`) 預設值: 4200
  * 網站伺服器所使用的連接阜
* **ssl**
  * `--ssl`
  * 網站伺服器使用 `HTTPS`
* **ssl-cert**
  * `--ssl-cert` (aliases: `-`) 
  * 指定 SSL 憑證位置給網站伺服器使用
* **ssl-key**
  * `--ssl-key`
  * 指定 SSL 金鑰位置給網站伺服器使用
* **aot**
  * `--aot`
  * 使用 `AOT` 模式建置專案
* **base-href**
  * `--base-href` (aliases: `-bh`)
  * 設定 `<base>` 網址
* **deploy_url**
  * `--deploy-url` (aliases: `-d`)
  * 設定部署網址
* **environment**
  * `--environment` (aliases: `-e`)
  * 指定環境變數檔
* **extract-css**
  * `--extract-css` (aliases: `-ec`)
  * 是否將 `global style`內設定的 css 檔案建置為 css 檔案而非 js 檔案
* **i18n-file**
  * `--i18n-file`
  * 指定多國語系檔位置
* **i18n-format**
  * `--i18n-format`
  * 設定多國語系檔格式
* **locale**
  * `--locale`
  * 設定語系
* **output-hashing**
  * `--output-hashing` (aliases: `-oh`)
  * 設定輸出檔案檔名模式(cache-busting hashing mode)，可使用的參數如下
    * none
    * all
    * media
    * bundles
* **output-path**
  * `--output-path` (aliases: `-op`) 
  * 設定建置輸出位置
* **poll**
  * `--poll`
  * 設定檢查檔案異動頻率 (微秒)
* **progress**
  * `--progress` (aliases: `-pr`) 預設值: true
  * 顯示建置進度
* **sourcemap**
  * `--sourcemap` (aliases: `-sm`, `sourcemaps`)
  * 輸出 souremap
* **target**
  * `--target` (aliases: `-t`, `-dev`, `-prod`) 預設值: development
  * 設定建置模式 (development模式，production模式)
* **vendor-chunk**
  * `--vendor-chunk` (aliases: `-vc`) 預設值: true
  * 是否將 `vendor` 單獨建置成獨立的檔案
* **common-chunk**
  * `--common-chunk` (aliases: `-cc`) 預設值: true
  * 是否將重複性質的程式碼單獨建置成獨立的檔案
* **verbose**
  * `--verbose` (aliases: `-v`) 預設值: false
  * 是否要顯示更多的資訊
* **watch**
  * `--watch` (aliases: `-w`)
  * 當檔案異動時，重新建置專案

### 備註

當執行 `ng serve` 時，所有的過程都是在記憶體裡完成的，所以不會有實體檔案的輸出。

## generate

## build

## lint

## e2e

## xi18n

## doc

# 設定篇

# 應用篇

