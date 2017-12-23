# Method

結構可以定義同類型的資料，而同類型的資料通常又會有同類型的行為。因為 Go 有 [Anonymous Function][Day 12]  ，有寫過 Javascript 可能第一個想到的解法是這樣：

```go
package main

import "fmt"

type People struct {
	name  string
	age   int
	Hello func(other People) string
}

func main() {
	var miles People

	other := People{name: `Someone`}
	miles = People{`Miles`, 18, func(other People) string {
		return `Hi! ` + other.name + `, I am ` + miles.name
	}}

	fmt.Println(miles.Hello(other)) // Hi! Someone, I am Miles
}
```

但明顯的缺點是，它需要動態給匿名函式的值，且需要閉包才能取得 `miles` 變數，這樣一來函式就很難共用，二來得先宣告 `miles` ，才能把匿名函式指定給 `miles` ，整個程式會變得有點混亂。

不如，我們把函式定義改一下：

```go
package main

import "fmt"

type People struct {
	name  string
	age   int
	Hello func(self People, other People) string
}

func main() {
	hello := func(self People, other People) string {
		return `Hi! ` + other.name + `, I am ` + self.name
	}

	other := People{name: `Someone`}
	miles := People{`Miles`, 18, hello}

	fmt.Println(miles.Hello(miles, other)) // Hi! I am Miles
}
```

第一個傳的值就是結構變數，在函式裡就可以使用 self 取得結構的資料。這做法有點類似 [Python 定義類別][]裡的 `self` ，只是需要自己手動傳值。

事實上 Go 有提供更好的寫法：

```go
package main

import (
	"fmt"
)

type People struct {
	name  string
	age   int
}

func (people People) Hello(other People) string {
	return `Hi! ` + other.name + `, I am ` + people.name
}

func main() {
	other := People{name: `Someone`}
	miles := People{`Miles`, 18}

	fmt.Println(miles.Hello(other)) // Hi! Someone, I am Miles
}
```

它的概念是定義一個函式 `func` ，而開頭就宣告這是哪個結構所使用的，以及裡面將會用什麼變數來代表結構本身 `(people People)` （傳值或傳址都可以），後面就跟平常定義函數一樣。

這樣的寫法，就會把 `Hello` 函式跟 `People` 綁定在一起，用起來就會非常像物件導向的寫法了。

## 參考資料

* [結構與方法](https://openhome.cc/Gossip/Go/Method.html)
* [Python 定義類別][]

[Python 定義類別]: https://openhome.cc/Gossip/Python/Class.html
[Day 12]: /docs/day12.md
