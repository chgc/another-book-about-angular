# Pipe

Angular Pipe 是一個很強大的資料顯示轉型的工具，在 HTML 樣板中，我們可以透過 Pipe 的幫忙，快速的將原始資料轉換成我們想要顯示的樣式，且不會異動到原始資料的內容。

哆啦 A 夢有個很厲害的道具，格列佛隧道（縮小隧道），人從大的一邊進入，從小的一邊出來，就會變成小人，再倒著走回去就會恢復原狀，這跟 Pipe 有異曲同工之妙。

![格列佛隧道](https://i.imgur.com/s3JSD8P.jpg)

我們可以把 Pipe 想像成一個隧道，在資料通過這個管道之後，會將資料轉換成我們想要的樣子。

Angular 有內建了一些 Pipe 像是 `DatePipe`、`UpperCasePipe`、`LowerCasePipe`、`CurrencyPipe` 和 `PercentPipe`及其他的 [Pipe](https://angular.io/api?query=pipe)，方便我們做日期、字串、幣別或數字的樣式轉換。當然 Angular 也允許讓我們自訂 Pipe 的功能，讓我們可以實現我們想要的格列佛隧道。

* [內建 PIPE](default-pipe.md)
* [自訂 PIPE](custom-pipe.md)
* [Pipe 與 ChangeDetection](cd-pipe.md)
* [Pure and Impure Pipes](pure-impure.md)

