# NPM package.json

`package.json` 是 npm 用來管理套件，任務，發佈等相關資訊，因為 Angular CLI 也是使用 NPM 來管理套件，所以還是花點時間了解 `package.json` ，以下是 Angular CLI 所產生出來的 `package.json` 檔案。

```
{
  "name": "ng-router",
  "version": "0.0.0",
  "license": "MIT",
  "scripts": {
    ...
    "e2e": "ng e2e"
  },
  "private": true,
  "dependencies": {
    ...
  },
  "devDependencies": {
    ...
  }
}
```

# 設定檔結構

* **name** (必填)：套件名稱
  * 規則
    * 長度必須在214字元以下，包含 `scope` 名稱
    * 名稱的第一個字元不能是 `.` 和 `_`
    * 名稱內不能有大寫字元
    * 因為名稱會成為網址的一部分，所以名稱不能含有任何 `non-URL-safe`  字元
  * 小技巧
    * 不使用與 `node` 核心套件相同的名稱 
    * 名稱內不要放 `js` 或是 `node` ，預設就認定是 `js` 套件，如果真的要設定，可以設定在 `engines`參數
    * 因為名稱會被引用 (required 或是 import)，所以名稱應該要簡短但同時又賦有意義
    * 在命名前，先檢查 npm 套件資料庫裡是否有想相同的名稱 (如果有要發佈到 npm 資料庫中)

* **version**(必填)：套件版本
  * 版本命名方式採 **semver** 模式

* **license**：專案授權模式

* **scripts**：設定 npm 可以執行的指令
  * scripts 有自己的執行生命週期，生命週期如下
    * preinstall、install、postinstall：安裝套件
    * preuninstall、uninstall、postuninstall ：移除套件
    * pretest、test、posttest：執行 `npm test` 指令
    * prestop、stop、poststop：執行 `npm stop`指令
    * prestart、start、 poststart：執行 `npm start`指令
    * prerestart、restart、postrestart：執行 `npm restart`指令，如果沒有指定 `restart` 動作時，則會執行 `stop` 跟 `start`指令

* **private**：當設定為 `true` 時，npm 就不會執行任何 `publish`的動作

* **dependencies**：定義**使用**此專案所依賴的套件及版本

  * 指定的方式可以直接使用套件名稱、tarball 或是  git 的網址

  * 版本指定方式，有很多方式，以下一一列出說明

    * `version`：指定特定版本

    * `>version`：版本必須大於 `version`

    * `>=version`：版本必須大於或等於 `version`

    * `<version`：版本必須小於 `version`

    * `<=version`：版本必須小於或等於 `version`

    * `~version`：大致相同於此版本

    * `^version` ：相容於此版本

    * `1.2.x` ：1.2.0, 1.2.1 等, 但不是 1.3.0

    * `http://...`：指定套件的網址，會從該網址下載套件並安裝

    * `*` 任何版本

    * `""` ：空白，效果等同於 `*`

    * `version1 - version2` 等同於 `>=version1 <=version2`.

    * `range1 || range2` 如果 range1 或 range2 都不符合時，則忽略此套件

    * `git…` : 格式與範例如下

      ```
      <protocol>://[<user>[:<password>]@]<hostname>[:<port>][:][/]<path>[#<commit-ish> | #semver:<semver>]
      ```

      ```
      git+ssh://git@github.com:npm/npm.git#v1.0.27
      git+ssh://git@github.com:npm/npm#semver:^5.0
      git+https://isaacs@github.com/npm/npm.git
      git://github.com/npm/npm.git#v1.0.27
      ```

    * `user/repo`：使用 GitHub Repo

      ```
      "express": "expressjs/express",
      "mocha": "mochajs/mocha#4727d357ea",
      "module": "user/repo#feature\/branch"
      ```

    * `tag`

    * `path/path/path` ：套件來源是本機某資料夾

      ```
      ../foo/bar
      ~/foo/bar
      ./foo/bar
      /foo/bar
      ```

      ​

  * 範例

    ```
    { "dependencies" :
      { "foo" : "1.0.0 - 2.9999.9999"
      , "bar" : ">=1.0.2 <2.1.2"
      , "baz" : ">1.0.2 <=2.3.4"
      , "boo" : "2.0.1"
      , "qux" : "<1.0.0 || >=2.3.1 <2.4.5 || >=2.5.2 <3.0.0"
      , "asd" : "http://asdf.com/asdf.tar.gz"
      , "til" : "~1.2"
      , "elf" : "~1.2.3"
      , "two" : "2.x"
      , "thr" : "3.3.x"
      , "lat" : "latest"
      , "dyl" : "file:../dyl"
      }
    }
    ```

    ​

* **devDependencies**：定義**開發**此專案所需的其它套件及版本

  * 版本設定的方式與 `dependencies` 相同
  * 可以使用 `npm link` 的方式使用套件




# 應用技巧

## 更新套件




# 參考資料

* [package.json](https://docs.npmjs.com/files/package.json)
* [scripts](https://docs.npmjs.com/misc/scripts)
* [semver](https://docs.npmjs.com/misc/semver)