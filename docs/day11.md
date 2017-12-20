# First class function

[昨天][Day 10]定義好的函式，可以當作變數來使用，如：

```go
package main

import "fmt"
import "reflect"

func add(a, b int) int {
	return a + b
}

func main() {
	add2 := add

	fmt.Println(add(1, 2))               // 3
	fmt.Println(add2(1, 2))              // 3
	fmt.Println(reflect.TypeOf(add))     // func(int, int) int
	fmt.Println(reflect.TypeOf(add2))    // func(int, int) int
}
```

上面可以看到 `add` 函式也可以當作一個變數來操作，熟悉 Javascript 一定對這個寫法不陌生。

上面同時也看得到變數的型態，因此我們也能定義一個變數是函式型態（Function Type）：

```go
package main

import "fmt"
import "reflect"

func add(a, b int) int {
	return a + b
}

func main() {
	var add2 func(int, int) int

	fmt.Println(add2)          // <nil>

	add2 = add

	fmt.Println(add(1, 2))     // 3
	fmt.Println(add2(1, 2))    // 3
}
```

因為它是變數，所以也可以成為其他函式的傳入值，就像 Javascript callback 一般：

```go
package main

import "fmt"

func cmd(a, b int, callback func(int, int) int) int {
	return callback(a, b)
}

func add(a, b int) int {
	return a + b
}

func main() {
	var add2 func(int, int) int

	add2 = add

	fmt.Println(cmd(1, 2, add))      // 3
	fmt.Println(cmd(1, 2, add2))     // 3
}
```

看到一堆 `func(int, int) int` 會覺得很冗長，我們可以使用 `type` 來定義新的型態：

```go
package main

import "fmt"

type addFunc func(int, int) int

func cmd(a, b int, callback addFunc) int {
	return callback(a, b)
}

func add(a, b int) int {
	return a + b
}

func main() {
	var add2 addFunc

	add2 = add

	fmt.Println(cmd(1, 2, add))      // 3
	fmt.Println(cmd(1, 2, add2))     // 3
}
```

## 匿名函式

除了直接宣告函式傳入之外，也可以使用匿名函式：

```go
package main

import "fmt"

type addFunc func(int, int) int

func cmd(a, b int, callback addFunc) int {
	return callback(a, b)
}

func main() {
	var add2 addFunc

	add2 = func(a, b int) int {
		return a + b
	}

	fmt.Println(cmd(1, 2, add2))     // 3
}
```

或者直接 inline 會更簡潔：

```go
package main

import "fmt"

type addFunc func(int, int) int

func cmd(a, b int, callback addFunc) int {
	return callback(a, b)
}

func main() {

    // 3
	fmt.Println(cmd(1, 2, func(a, b int) int {
        return a + b
    }))
}
```

## 參考資料

* [一級函式](https://openhome.cc/Gossip/Go/FirstClassFunction.html)
* [匿名函式與閉包](https://openhome.cc/Gossip/Go/Closure.html)
* [Function types][]

[Function types]: https://golang.org/ref/spec#Function_types
[Day 10]: /docs/day10.md
