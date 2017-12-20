# Anonymous Function

[昨天][Day 11]在使用 callback 的時候，有用到匿名函式。今天來看一下匿名函式的其他細節。

下面是一個簡單匿名函式的使用方法：

```go
package main

import "fmt"

func main() {
	addFunc := func(a, b int) int {
		return a + b
	}

	sum := addFunc(1, 2)

	fmt.Println(sum)    // 3
}
```

如果覺得還要存放一個變數太麻煩，這也可以省略

```go
package main

import "fmt"

func main() {
	sum := func(a, b int) int {
		return a + b
	}(1, 2)

	fmt.Println(sum)    // 3
}
```

又或是，想要從另一個函式取得匿名函式：

```go
package main

import "fmt"

func getFunc() func(a, b int) int {
	return func(a, b int) int {
		return a + b
	}
}

func main() {
	sum := getFunc()(1, 2)

	fmt.Println(sum) // 3
}
```

再來這是組合技

```go
package main

import "fmt"

func getFunc() func(a, b int) int {
	return func(a, b int) int {
		return func(a, b int) int {
			return a + b
		}(a, b)
	}
}

func main() {
	sum := getFunc()(1, 2)

	fmt.Println(sum) // 3
}
```

## Closure

閉包是指，變數被關在某個區塊內。比方說剛剛的例子調整一下：

```go
package main

import "fmt"

func getFunc() func(a, b int) int {
	base := 10

	return func(a, b int) int {
		return base + func(a, b int) int {
			return a + b
		}(a, b)
	}
}

func main() {
	sum := getFunc()(1, 2)

	fmt.Println(sum) // 13
}
```

這樣的結果會跟下面這個例子的結果一樣：

```go
package main

import "fmt"

func getFunc() func(a, b int) int {
	base := 10

	return func(a, b int) int {
		return func(a, b int) int {
			return base + a + b
		}(a, b)
	}
}

func main() {
	sum := getFunc()(1, 2)

	fmt.Println(sum) // 13
}
```

我們可以發現，雖然匿名函式內容是封閉的，但 `base` 變數卻能夠被關進匿名函式裡，甚至是「匿名函式的匿名函式裡」，這就是閉包的特性。

最後，因為 Go 有取址運算，我們能拿得到變數真正的位址。那我們來看看在各階段裡面的位址為何：

```go
package main

import "fmt"

func getFunc() func(a, b int) int {
	base := 10
	fmt.Printf("In getFunc()   %p = %d\n", &base, base)

	return func(a, b int) int {
		fmt.Printf("In getFunc() closure   %p = %d\n", &base, base)

		return func(a, b int) int {
			fmt.Printf("In getFunc() closure's closure   %p = %d\n", &base, base)

			return base + a + b
		}(a, b)
	}
}

func main() {
	sum := getFunc()(1, 2)

	fmt.Println(sum) // 13
}
```

最後輸出：

```
In getFunc()   0xc420010058 = 10
In getFunc() closure   0xc420010058 = 10
In getFunc() closure's closure   0xc420010058 = 10
13
```

這裡可以發現在三個地方的 base 變數位址都是 `0xc420010058` ，這也是所謂「變數被關在某個區塊內」所代表的意思。

## 參考資料

* [匿名函式與閉包](https://openhome.cc/Gossip/Go/Closure.html)

[Day 11]: /docs/day11.md
