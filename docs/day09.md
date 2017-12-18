# Map Type

許多語言都有提供 key-value 存放方法的 map 結構， Go 使用內建型態 `map` 實作。 

`map` 型態的表示方法為： `map[keyType]valueType` ， `map` 是關鍵字， `keyType` 必須是可比較（[Comparable][Comparison operators]）的型態，如 `string` 、 `int` 等， `valueType` 則是內容形態。

## 建立

建立 Map 資料型態也是用 `make` ，設定與取值的方法跟大部分的語言（如 PHP ）都很像，範例如下：

```go
package main

import "fmt"

func main() {
	score := make(map[string]int)

	fmt.Println(score)  // map[]
	
	score["Miles"] = 80
	score["Chou"] = 60
	
	fmt.Println(score)          // map[Miles:80 Chou:60]
	fmt.Println(score["Miles"]) // 80
	fmt.Println(score["Chou"])  // 60
}
```

如果有初值的話，設定的方法很像 JSON ：

```go
package main

import "fmt"

func main() {
	score := map[string]int{
		"Miles": 80,
		"Chou": 60,
	}

	fmt.Println(score)  // map[]

	score["Miles"] = 80
	score["Chou"] = 60

	fmt.Println(score)          // map[Miles:80 Chou:60]
	fmt.Println(score["Miles"]) // 80
	fmt.Println(score["Chou"])  // 60
}
```

值得一提的是，宣告值最後一行 `"Chou": 60,` 的逗號是必要要加的。

這個寫法如果不給初值的話，就會跟使用 `make` 方法結果一樣：

```go
score := make(map[string]int)

score := map[string]int{}
```

`map` 跟 `slice` 一樣是使用參考，比方說：

```go
package main

import "fmt"

func main() {
	score := map[string]int{
		"Miles": 80,
		"Chou": 60,
	}

	ref := score

	fmt.Println(score)          // map[Miles:80 Chou:60]
	fmt.Println(ref)            // map[Miles:80 Chou:60]

	score["Someone"] = 0

	fmt.Println(score)          // map[Chou:60 Someone:0 Miles:80]
	fmt.Println(ref)            // map[Someone:0 Miles:80 Chou:60]
}
```
 
> 試了幾次，它的順序應該是不固定的。

## 操作

取值使用 `[]` 指定 key ，事實上它會回傳兩個值，如果 key 存在，會回傳值與 true ； key 不存在則回傳零值與 false ：

```go
package main

import "fmt"

func main() {
	score := map[string]int{
		"Miles": 80,
		"Chou": 60,
	}

	var value int
	var ok bool

    value, ok = score["Miles"]

	fmt.Println(value)     // 80
	fmt.Println(ok)        // true

    value, ok = score["Nobody"]

	fmt.Println(value)     // 0
	fmt.Println(ok)        // false
}
```

移除 key 使用 `delete` 函式：

```go
package main

import "fmt"

func main() {
	score := map[string]int{
		"Miles": 80,
	}

	var value int
	var ok bool

    value, ok = score["Miles"]

	fmt.Println(value)     // 80
	fmt.Println(ok)        // true

    delete(score, "Miles")
	value, ok = score["Miles"]

	fmt.Println(value)     // 0
	fmt.Println(ok)        // false
}
```

## 參考資料

* [成對鍵值的 map](https://openhome.cc/Gossip/Go/Map.html)
* [Comparison operators][]

[Comparison operators]: https://golang.org/ref/spec#Comparison_operators
