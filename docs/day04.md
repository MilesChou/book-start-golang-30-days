# Constants

Go 語言的常數有分幾種類型：

* *boolean constants* ，布林常數。
* *rune constants* ，表示字元的常數。
* *integer constants* ，整數常數
* *floating-point constants* ，浮點數常數
* *complex constants* ，複數常數
* *string constants* ，字串常數

這些常數都可以用實字（literal）表示，實字又分成下面幾種：

* *rune literal* ， Rune 實字
* *integer literal* ，整數實字
* *floating-point literal* ，浮點數實字
* *imaginary literal* ，虛數實字
* *string literal* ，字串實字

常數有可能是已定義型態（typed）或是未定型態（untyped），實字常數、 `true` 、 `false` 、 `iota` 都屬於未定型態。

另外較特別的是，常數運算式裡的運算元都是未定型態時，運算完的結果也會是未定型態。比方說，下面都是未定型態：

```go
10        // 10, Untyped integer constant.
10 + 20   // 30, Untyped integer constant.
10 / 20   // 0,  Untyped integer constant.
```

但如果有一個型態是確定的，那運算完的結果也會是確定的，如：

```go
int32(10)           // 10,  type int32
int32(10) + 20      // 30,  type int32
float64(10) / 20    // 0.5, type float64
```

## Boolean constants

布林常數是內建的常數，就只有兩個： `true` 和 `false`

原始碼實作也蠻有趣的（程式碼來自 [`builtin.go`](https://github.com/golang/go/blob/master/src/builtin/builtin.go#L16-L20)）：

```go
// true and false are the two untyped boolean values.
const (
	true  = 0 == 0 // Untyped bool.
	false = 0 != 0 // Untyped bool.
)
```

## Rune constants

Rune 常數使用 Rune 實字（rune literal）來表示，它其實是代表一個 Unicode 的整數。可以使用單引號 `'` 括住 Unicode 字元，或是 byte 值來表示，如下面的範例是輸出 `a` 的三種方法，與輸出 `中` 的三種方法：

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println(string('a'))
	fmt.Println(string('\141'))
	fmt.Println(string('\x61'))
	fmt.Println(string('中'))
	fmt.Println(string('\u4e2d'))
	fmt.Println(string('\U00004e2d'))
	fmt.Println(string('\n'))
}
```

> `string()` 函式為強制轉型字串

Byte 值的表示方法：

* 直接給字元 `a`
* `\` 開頭為八進制，後面必須是 3 個八進位的字元（`[0-9]{3}`）
* `\x` 開頭為十六進制表示，後面必須是 2 個十六進位的字元（`[0-9a-f]{2}`）

Unicode 表示方法：

* 直接給字元 `中`
* `\u` 開頭，後面必須是 4 個十六進位的字元（`[0-9a-f]{4}`）
* `\U` 開頭，後面必須是 8 個十六進位的字元（`[0-9a-f]{8}`）
* 跳脫字元： `\` 後面接 `a` `b` `f` `n` `r` `t` `v` `\` `'` `"` 。

## Integer constants

數字常數使用數字實字（integer literal）表示。數字實字有三種表示法：

* 十進位，跟大部分的程式碼一樣，為非 `0` 開頭的連續數字（`[1-9][0-9]+`）
* 八進位， `0` 開頭，後面接八進位數字（`0[0-9]+`）
* 十六進位， `0` 開頭，後面接八進位數字（`0[x|X][0-9a-fA-F]+`）

## Floating-point constants

浮點數常數使用浮點數實字（floating-point literal）表示，浮點數使用的兩種表示法：小數點與科學符號，下面是幾個例子可以參考：

```go
package main

import (
	"fmt"
)

func main() {
	// 10.0
	fmt.Println(10.)
	fmt.Println(10.0)
	fmt.Println(010.0)
	fmt.Println(10.e+0)
	fmt.Println(1E1)

	// 0.1
	fmt.Println(.1e+0)
	fmt.Println(.1E0)
	fmt.Println(.1)

	// 10.1
	fmt.Println(10.1)
	fmt.Println(1.01E1)
}
```

## Complex constants

複數常數為數字實字加虛數實字（imaginary literal）組合而成。

虛數實字的表示法為：

* 十進位 + 小寫 `i` ，如 `10i`
* 浮點數 + 小寫 `i` ，如 `1E1i`

而複數常數的範例如下：

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println(10 + 10i)
	fmt.Println(1E1 + 1E1i)
}
```

## String constants

字串常數使用字串實字（string literal）表示。如果是純字串，可以使用 `` ` `` 括要表示的字串，如：

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println(`\n`)
}
```

這樣就會輸出 `\n` 兩個字元

如果需要轉譯 rune 常數為字元的話，可以用雙引號 `"` 括要表示的字串，如：

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("這是\u4e2d\u6587")
}
```

這樣就會輸出 `這是中文`

## 今日回顧

今天先介紹基本的實字表示，再來要解釋變數型態應該就會比較好懂了。

## 參考資料

* [Constants][] | The Go Programming Language Specification

[Constants]: https://golang.org/ref/spec#Constants
