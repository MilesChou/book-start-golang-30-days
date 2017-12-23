# Inheritance

[昨天][Day 14]學完怎麼在結構上加方法後，它就很像在設計物件導向程式了。今天來看看它怎麼實作繼承。

先把昨天最後的程式搬過來：

```go
package main

import (
	"fmt"
)

type People struct {
	name  string
	age   int
}

func (people People) Hello() string {
	return `Hi! I am ` + people.name
}

func main() {
	miles := People{`Miles`, 18}

	fmt.Println(miles.Hello()) // Hi! Someone, I am Miles
}
```

假設想要有新的結構是 `Taiwanese` 繼承 `People` ，寫法是這樣的：

```go
package main

import (
	"fmt"
)

type People struct {
	name  string
	age   int
}

type Taiwanese struct {
	People
	country  string
}

func (people People) Hello() string {
	return `Hi! I am ` + people.name
}

func main() {
	miles := Taiwanese{People{`Miles`, 18}, `Taiwan`}

	fmt.Println(miles)         // {{Miles 18} Taiwan}
	fmt.Println(miles.country) // Taiwan

	fmt.Println(miles.name)    // Miles
	fmt.Println(miles.age)     // 18
	fmt.Println(miles.Hello()) // Hi! I am Miles

	// 指定結構
	fmt.Println(miles.People.name)    // Miles
	fmt.Println(miles.People.age)     // 18
	fmt.Println(miles.People.Hello()) // Hi! I am Miles
}
```

它可以多重繼承，但如果成員重覆的話，就會出現 `ambiguous selector` 編譯錯誤

```go
package main

import (
	"fmt"
)

type People struct {
	name  string
	age   int
}

type Animal struct {
	name  string
}

type Taiwanese struct {
	People
	Animal
	country  string
}
```

## 覆寫成員與方法

剛剛第一個例子可以看到成員與方法是會被繼承下來的，它也可以被覆寫：

```go
package main

import (
	"fmt"
)

type People struct {
	name string
	age  int
}

type Taiwanese struct {
	People
	name    string
	country string
}

func (people People) Hello() string {
	return `Hi! I am ` + people.name
}

func (taiwanese Taiwanese) Hello() string {
	return `你好！我是` + taiwanese.name
}

func main() {
	miles := Taiwanese{People{`Miles`, 18}, `麥爾斯`, `Taiwan`}

	fmt.Println(miles.Hello())        // 你好！我是麥爾斯
	fmt.Println(miles.People.Hello()) // Hi! I am Miles
}
```

Taiwanese 看起來很像 People 了，不過它還是不能當作是 People 使用（多型），這要使用 interface 之後才能解決。

---

鐵人賽已過一半，明天要開始來實作應用程式了。

## 參考資料

* [結構與方法](https://openhome.cc/Gossip/Go/Method.html)

[Day 14]: /docs/day14.md
