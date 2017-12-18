# Function declarations

在我們 [Hello World][Day 3] 的練習裡，曾提到一點點函式的定義，今天要來詳解它。

## 定義函式

函式的定義語法如下：

```go
func <funcName>([param1 type1[, param2 type2 ...]]) ([return1 type1[, return2 type2 ...]]) {
	
	// Do something
	
    return [value1[, value2 ...]]
}
```

* `funcName` 為 function 名稱，首字大寫為 public 是外部套件可以共用的，小寫為 private 只有套件內部可以看得到。
* `param` 為傳入值。
* `return` 為回傳值。 Go 比較特別的是，它可以定義多個回傳值，而且還能定義它的名稱。定義名稱時，回傳的用法會有點特別。

先來個簡單的範例：

```go
package main

import "fmt"

func add(a int, b int) int {
	return a + b
}

func main() {
	sum := add(1, 2)

	fmt.Println(sum)     // 3
}
```

其中 `add` 函式， `a` 與 `b` 為相同型態，所以可以寫在一起，如下：

```go
func add(a, b int) int {
	return a + b
}
```

如果回傳值只有一個，而且沒有宣告名稱時，可以不用加括號 `()` 。下面是回傳兩個值的範例

```go
package main

import "fmt"

func add(a, b int) (int, bool) {
	return a + b, true
}

func main() {
	sum, ok := add(1, 2)

	fmt.Println(sum)    // 3
	fmt.Println(ok)     // true
}
```

這有點類似 [Map][Day 9] 取值的用法，第二個值是確認這個回傳是正確的。

如果有定義名稱的話， `return` 會把當下兩個變數的內容回傳出去。

```go
package main

import "fmt"

func add(a, b int) (sum int, ok bool) {
	sum = a + b
	ok = true 
	return
}

func main() {
	sum, ok := add(1, 2)

	fmt.Println(sum)    // 3
	fmt.Println(ok)     // true
}
```

常看過很多程式在一開始就定義 `result` ，在 `return` 的時候回傳出去。 Go 則是在規格上直接實作出來，覺得蠻有趣的。

傳回多個值的時候，必須照順序接值；如果不需要回傳值，可以用 `_` 略過：

```go
package main

import "fmt"

func add(a, b int) (sum int, ok bool) {
	sum = a + b
	ok = true 
	return
}

func main() {
	_, ok := add(1, 2)

	fmt.Println(ok)     // true
}
```

傳回多值通常會用在錯誤處理，如果使用不恰當的話，容易違反[單一職責原則][Refactoring Day 7]。

如果傳入值是不定的，可以用 `...` 來表示，如：

```go
package main

import "fmt"

func add(numbers ...int) (sum int) {
	sum = 0
	for _, num := range numbers {
		sum += num
	}

	return
}

func main() {
	sum := add(1, 2, 3, 4, 5)

	fmt.Println(sum)     // 15
}
```

裡面的 `numbers` 型態會是 [Slice][Day 8] `[]int` ，所以可以用 `for range` 走訪。

## 參考資料

* [函式入門](https://openhome.cc/Gossip/Go/Function.html)
* [Function declarations][]

[Function declarations]: https://golang.org/ref/spec#Function_declarations
[Refactoring Day 7]: https://github.com/MilesChou/book-refactoring-30-days/blob/master/docs/day07.md
[Day 3]: /docs/day03.md
[Day 8]: /docs/day08.md
[Day 9]: /docs/day09.md
