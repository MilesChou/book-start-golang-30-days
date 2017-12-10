# Hello World

學習程式的第一隻程式當然就是 Hello World 了，[官方首頁][]有提供 Hello World 原始碼：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

> 未來有原始碼，都將會在[這個專案](https://github.com/MilesChou/book-start-golang-30-days)裡更新。

討論原始碼細節前，我們先想辦法讓它可以在[昨天][]建好的環境執行。我們先建個目錄，在裡面新增一個檔案叫 `main.go` ，然後切換到目錄，把上面的內容輸入到檔案裡，接著下 `go run main.go` ：

    $ mkdir -p /path/to/helloworld
    $ cd /path/to/helloworld
    # 輸入程式碼
    $ vim main.go
    # 執行程式碼
    $ go run main.go
    Hello, 世界

順利的話，應該就會如上面的範例一樣，看到 `Hello, 世界` 。恭喜你，寫出第一隻 Go 程式了。

## 這之中到底做了什麼呢？

### go run

首先從指令開始看起：

    $ go run main.go
    Hello, 世界

指令 `go run` 所做的事正是直譯，也就是直接拿原始碼編譯，同時執行。

### package

接著來看原始碼：

```go
package main
```

第一行 `package` 指的是定義套件名稱。每個 `.go` 原始碼開頭都必須要宣告 `package` 。

`main` 套件是有特殊意義的套件名，它是程式的起始點。執行程式的時候，將會從 main 套件開始。

可以試著把 main 名字換成其他名字，再執行一次，將會出現錯誤訊息：

    $ go run main.go
    go run: cannot run non-main package

它說，不能跑非 main 的套件。這個概念與大多數 PHP 框架的 `index.php` 類似，是所有 request 的起始點。

### import

緊接著這行程式碼：

```go
import "fmt"
```

`import` 表示要引用套件，而 `fmt` 套件是 Go 內建的處理格式化輸入輸出函式庫。

Hello World 的目的是要輸出文字，所以我們需要這個函式庫。

### func

最後這裡是定義函式，也就是要開始寫流程了。

```go
func main() {
	fmt.Println("Hello, 世界")
}
```

`func` 定義了程式流程，供其他函式呼叫使用。上面的程式碼可以看到兩個函式，一個是現正定義的 `main` ，另一個則是 `fmt` 套件所提供的 `Println` 函式，這是把後面帶入的文字印出來，然後再另外加一個換行。

Go 語言有套件庫的概念，同時的函式也有能見度的規範。 Go 採用比較特別的方法：開頭大寫的函式是 public ，不同的套件庫可以呼叫 public func ；開頭小寫的則是 private ，只限套件庫內部使用。

上例 `Println` 是屬於 `fmt` 套件的 public func ，因此雖然套件庫不同（`main` 與 `fmt`），仍然可以正常呼叫。

而 `func main` 比較特別，它會搭配 `package main` 一起使用。前面提到 `package main` 是所有程式的進入點，而 `go run` 會把 `package main` 的 `func main` 拿出來呼叫。

最後總結一下： `go run main.go` 實際上就是執行 `fmt.Println("Hello, 世界")` ，於是就跑出 `Hello, 世界` （和換行）了。

## 今日回顧

* 今天成功寫了第一隻 Go 程式了
* 學習了 Go 的指令
  + `go run`
* 學習了 Go 的關鍵字（3/25）
  + `package`
  + `import`
  + `func`

## 參考資料

* [The Go Programming Language](https://golang.org)
* [語言技術：Go 語言](https://openhome.cc/Gossip/Go/index.html) | 良葛格學習筆記

[昨天]: /docs/day02.md
[官方首頁]: https://golang.org
