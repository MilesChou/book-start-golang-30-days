# Predeclared Type

[中華小當家][]的劉昴星曾說過：「鍋子是火燄的化身」，使用鍋子也是中華廚師的必學基礎之一；而在一個程式語言裡，資料型別是資料的化身，同樣也是重要的基礎功。廚師練好基礎功，能煮出佳餚；開發者練好基礎功，才有辦法寫出千變萬化的應用程式。

Go 的資料型別有 11 種，今天先介紹 *Predeclared Type*　，它們也是「有名稱的型態（Named Type）」。

## Boolean types

最簡單的型別－－ `bool` ，它只有兩個預定義的常數 `true` 和 `false` 。

## Numeric types

`numeric` 型態包含了 integer （整數）、 float（浮點數）、 complex（複數）三種。

整數又分帶號 `int` 與不帶號 `uint` 兩類，也可以直接指定大小 `int8` 、 `int16` 、 `int32` 、 `int64` 或是不帶號的 `uint8` 、 `uint16` 、 `uint32` 、 `uint64` ，這些相信一看就知道佔了多少容量（bit）。至於 `int` 和 `uint` 會使用哪一個要看平台實作決定，有可能是 32 bit 也有可能是 64 bit。

而另外還有兩個整數型態： `rune` 是 `int32` 的別名， `byte` 是 `int8` 的別名。

float 有 `float32` 與 `float64` 兩種，但沒有 `float` 。

complex 則表示複數，以 RE + IMi 的方法表示，如：

```go
10+5i
```

大小則有分 `complex64` 與 `complex128` 兩種。

Go 內建的 math 套件提供常數取得各型態的最大值和最小值，除了解整數範圍外，也有助於實作上的判斷。

如：

```go
package main

import "fmt"
import "math"

func main() {
	fmt.Println(math.MinInt8)
	fmt.Println(math.MaxInt8)
	fmt.Println(math.MinInt16)
	fmt.Println(math.MaxInt16)
	fmt.Println(math.MinInt32)
	fmt.Println(math.MaxInt32)
	fmt.Println(math.MinInt64)
	fmt.Println(math.MaxInt64)
	
	// Uint 最小值是 0
	fmt.Println(math.MaxUint8)
	fmt.Println(math.MaxUint16)
	fmt.Println(math.MaxUint32)
	fmt.Println(math.MaxUint64)
	
	// Float 的表示是最小非 0 浮點數
	fmt.Println(math.SmallestNonzeroFloat32)
	fmt.Println(math.MaxFloat32)
	fmt.Println(math.SmallestNonzeroFloat64)
	fmt.Println(math.MaxFloat64)
}
```

需要注意的是，不同的數字型態，是不能直接摻在一起操作的。如 int8 不能跟 uint8 相加。另外， int 有可能是 32 位元，但 int 也不能跟 int32 相加。

## String types

Go 語言字串都是 UTF-8 字元集編碼，它可以正常的處理多國語言。字串可以使用雙引號 `"` 或反引號 `` ` `` 定義，也可以相加，如：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello 雙引號")
	fmt.Println(`Hello 反引號`)
	fmt.Println("雙引號" + `反引號`)
}
```

## 今日回顧

今天是蹲馬步的基本功，後面其他型態將會使用這些基本型態炒出各式各樣的菜色。

## 參考資料

* [The Go Programming Language Specification](https://golang.org/ref/spec#Types)
* [認識預定義型態](https://openhome.cc/Gossip/Go/PreDeclaredType.html)

[中華小當家]: https://zh.wikipedia.org/wiki/%E4%B8%AD%E8%8F%AF%E5%B0%8F%E7%95%B6%E5%AE%B6
