# Let's Golang

[前言](https://github.com/MilesChou/book-start-golang-30-days)

[Go][] 是 Google 所開發的程式語言，最近有很多新流行的 Server 端應用都是使用 Go 開發的，如 [Docker][] 或 [Drone CI][] 等；除此之外， Go 語言關鍵字少、程式結構相較簡單、加上內建開發工具（如編譯、測試、文件等等）都很完整，這對新手入門是非常友善的。

總和以上這些特質，有越來越多人已經入坑 Go 了，而今天開始，筆者也要入坑了！  

## 為何選擇 Go 語言？

其他語言也有優點，那為何選擇 Go 語言？首先，因為筆者興趣比較偏向後端語言；後端語言很多，而筆者比較想學靜態語言。

在候選名單中，有 C/C++ 、 Java 、 Rust ，但看了許多文章，最終選了 Go ，原因是：

1. C/C++ 雖然快，但 Go 的開發效率比較高，也比較容易上手
2. 有討論是說 Java 與 Go 的效能差不多， Go 的程式碼比較少，可維護性自然比較高
3. Rust 的即時運算效能高，比較適合開發系統或是 GUI 應用程式，但學習曲線較高，相反地 Go 好入門，且適合開發一般的網路應用程式
4. 更重要的是，後面有 Google 撐腰，相信未來還會有更多應用出爐

## 原本的 PHP 呢？

PHP 是目前筆者最熟悉的語言，當跟一個語言越熟，就越會了解它不適合的情境。眾所皆知的，它效能比較差，即使 7.0 的效能大大超過 5.x ，但還是無法跟 Java 等靜態語言相比；其次是，原生提供的函式庫並不擅長處理多執行緒；最後，大家最常使用的組合是 LAMP ，它非常適合設計成無狀態服務，但同時會反應另一個問題：因每次狀態都是從新建立，包括 DB 連線，那在大流量的情境下，該如何有效管理 DB connection pool 。

剛好 Go 能解決上述三個麻煩的問題，這也是我想學 Go 的主要原因。 

## 三十天目標

未來三十天預期會從環境建置開始，一邊學習，一邊使用 Go 寫出幾個簡單的 CLI 應用與 API Server 。

## 參考資料

* [值得注意的程式語言：D、Go、Rust](http://blog.cwchen.tw/programming/2016/12/02/the-languages-worth-noting-d-go-rust/) | Michael Talks
* [D、GO、Rust 誰會在未來取代 C？為什麼？](https://buzzorange.com/techorange/2015/11/18/which-can-replace-language-c/) | TechOrange - BuzzOrange

[Docker]: https://www.docker.com/
[Drone CI]: https://drone.io/
[Go]: https://zh.wikipedia.org/wiki/Go
