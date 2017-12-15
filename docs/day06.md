# Types (2/3) －－ 變數宣告與常數宣告

原本想說變數宣告有點複雜，想把 Type 一次寫完再來寫，不過看來是不行的，還是得先經過這一關。

## 變數宣告

宣告變數使用 `var` 關鍵字，下面宣告了 num 變數為 int 型態，並給初始值為 10：

```go
var num int = 10
```

使用 IDE 會提示說， `int` 可以省略，因為 Go 會自動推斷 10 的型態為 `int` ，因此也可以這樣宣告：

```go
var num = 10
```

如果不給初始值的話，預定義變數都會有預設值， Go 語言稱之為零值（[The zero value][]） ， `int` 的零值是 0 ，所以下面兩行宣告是等價的：

```go
var num int = 0
var num int
```

Go 可以一次宣告多個變數，下面的型態分別會推斷為 `string` 、 `int` 、 `float64`：

```go
var name, age, height = "Miles", 18, 169.9
```

也可以分多行宣告

```go
var (
	name = "Miles"
	age = 18
	height = 169.9
)
```

多行宣告並指定型態與指定初始值

```go
var (
	name string = "Miles"
	age uint = 18
	height float32 = 169.9
)
```

多行宣告並指定型態不指定初始值

```go
var (
	name string
	age uint
	height float32
)
```

## 短變數宣告

在 func 裡，如果要宣告變數同時指定初值，可以使用[短變數宣告][Short variable declarations]：

```go
name := "Miles"
age := 18
height := 169.9
```

> 這裡就如同 PHP 的 `$name = 'Miles'` 一樣，宣告變數同時給值

一樣可以寫成一行

```go
name, age, height := "Miles", 18, 169.9
```

## 常數宣告

宣告變數使用 `const` 關鍵字，下面宣告了 num 變數為 int 型態，並給值為 10：

```go
const num int = 10
```

這時 num 會是不可變的常數，試圖指定新值會報錯。

除了常數一定要給值外，其他宣告的方法都跟變數一樣，如一次宣告多個常數

```go
const name, age, height = "Miles", 18, 169.9
```

多行宣告

```go
const (
	name = "Miles"
	age = 18
	height = 169.9
)
```

多行宣告並指定型態

```go
const (
	name string = "Miles"
	age uint = 18
	height float32 = 169.9
)
```

## 宣告後沒使用會？

變數宣告了就是要用，不然要幹嘛？如果宣告了一個 num 變數沒使用， Go 會在編譯時期出錯：

```
./hello.go:8:6: num declared and not used
```

常數則可以宣告但不使用。

## 今日回顧

* 學習了兩個關鍵字（5/23）
  + `var`
  + `const`

## 參考資料

* [變數宣告、常數宣告][] | 良葛格學習筆記
* [The zero value][] | The Go Programming Language Specification
* [Short variable declarations][] | The Go Programming Language Specification

[變數宣告、常數宣告]: https://openhome.cc/Gossip/Go/VariableConstantDeclaration.html
[The zero value]: https://golang.org/ref/spec#The_zero_value
[Short variable declarations]: https://golang.org/ref/spec#Short_variable_declarations
