# Struct

`struct` 是定義資料的集合，跟物件很像，也是把資料集合在一包。

## 定義

`struct` 可以使用 `type` 關鍵字定義，開頭的大小寫跟[函式][Day 10]一樣，會影響能見度；內部的成員名的定義也是一樣。

定義與使用的範例如下：

```go
package main

import "fmt"

type People struct {
	name string
	age  int
}

func main() {
	miles := People{`Miles`, 18}
	fmt.Println(miles)       // {Miles 18}
	fmt.Println(miles.name)  // Miles
	fmt.Println(miles.age)   // 18
	
	chou := People{age: 18, name: `Chou`}
	fmt.Println(chou)        // {Chou 18}

	part := People{name: `Part`}
	fmt.Println(part)        // {Part }

	empty := People{}
	fmt.Println(empty)       // { 0}

	var empty2 People
	fmt.Println(empty2)      // { 0}
}
```

上面可以注意

* `chou` 範例可以在指定值的時候改順序
* `part` 範例在指定部分值的時候，必須確定指明是哪個部分，比方說上例的 `name` ，即使順序一樣，沒有指明 `name` 的話一樣會出錯，比方說： `part := People{"Part"}` 這是不合法的範例
* `empty` 、 `empty2` 是零值範例

如果把一個結構指定給另一個結構時，它會使用複製：

```go
package main

import "fmt"

type People struct {
	name string
	age  int
}

func main() {
	miles := People{`Miles`, 18}
	copy := miles
	
	copy.name = `Copy`
	
	fmt.Println(miles) //  {Miles 18}
	fmt.Println(copy) //   {Copy 18}
}
```

函數傳遞也是如此，如果需要傳址，可以直接用指標

```go
package main

import "fmt"

type People struct {
	name string
	age  int
}

func changeName(people *People) {
	people.name = `Someone`
}

func main() {
	miles := People{`Miles`, 18}
	fmt.Println(miles)  //   {Miles 18}

	changeName(&miles)
	fmt.Println(miles)  //   {Someone 18}
}
```

使用 `new` 關鍵字的話，得到的會是指標：

```go
package main

import "fmt"

type People struct {
	name string
	age  int
}

func changeName(people *People) {
	people.name = `Someone`
}

func main() {
	miles := new(People)
	miles.name = `Miles`
	miles.age = 18
	fmt.Println(miles)  //   &{Miles 18}

	changeName(miles)
	fmt.Println(miles)  //   &{Someone 18}
}
```

結構的成員也可以結構，雖然無法直接把自己包起來，但可以使用指標，如：

```go
package main

import "fmt"

type People struct {
	name   string
	age    int
	friend *People
}

func main() {
	friend := new(People)
	friend.name = `Friend`

	miles := People{`Miles`, 18, friend}
	fmt.Println(miles)               // &{Miles 18 0xc42000a060}
	fmt.Println(miles.name)          // Miles
	fmt.Println(miles.friend.name)   // Friend
}
```

## 參考資料

* [結構入門](https://openhome.cc/Gossip/Go/Struct.html)

[Day 10]: /docs/day10.md