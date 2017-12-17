# Slice Type

Slice 跟[陣列][Day 7]使用起來很像，而最大的不同是，陣列是值， Slice 是參考到一個陣列。 

## 建立

要建立一個全新的 Slice 有兩種方法，一個是使用 `make` 函式：

```go
package main

import "fmt"

func main() {
	slice := make([]int, 5)

	fmt.Println(slice)  // [0 0 0 0 0]
}
```

另一個方法則是指定初值，雖然用法跟陣列很像，但形態不一樣就不能拿來一起比較。

```go
package main

import "fmt"
import "reflect"

func main() {
	slice := []int{1, 2, 3, 4, 5}

	fmt.Println(slice)  // [1 2 3 4 5]

	arr := [...]int{1, 2, 3, 4, 5}

	fmt.Println(reflect.TypeOf(slice))  // []int
	fmt.Println(reflect.TypeOf(arr))    // [5]int
	fmt.Println(arr == slice)           // invalid operation: arr == slice (mismatched types [5]int and []int)
}
```

Slice 是參考到一個陣列，可以看下面這個範列了解：

```go
package main

import "fmt"

func main() {
	slice := []int{1, 2, 3, 4, 5}
	ref := slice

	fmt.Println(slice)  // [1 2 3 4 5]
	fmt.Println(ref)    // [1 2 3 4 5]

	slice[0] = 100

	fmt.Println(slice)  // [100 2 3 4 5]
	fmt.Println(ref)    // [100 2 3 4 5]

	ref[4] = 500

	fmt.Println(slice)  // [100 2 3 4 500]
	fmt.Println(ref)    // [100 2 3 4 500]
}
```

## 操作

我們可以對 Slice 做一些操作，如 `len` 函式可以查長度， `cap` 可以查參考的陣列有多少容量： 

```go
package main

import "fmt"

func main() {
	slice := []int{1, 2, 3}

	fmt.Println(len(slice))  // 3
	fmt.Println(cap(slice))  // 3
}
```

`append` 函式可以追加新元素在 Slice 最後面，下面是一個小範例：

```go
package main

import "fmt"

func main() {
	slice1 := []int{1, 2, 3}

	fmt.Println(slice1)       // [1 2 3]
	fmt.Println(len(slice1))  // 3
	fmt.Println(cap(slice1))  // 3

	slice2 := append(slice1, 10)

	fmt.Println(slice2)       // [1 2 3 10]
	fmt.Println(len(slice2))  // 4
	fmt.Println(cap(slice2))  // 6

	slice3 := append(slice2, 20)

	fmt.Println(slice3)       // [1 2 3 10 20]
	fmt.Println(len(slice3))  // 5
	fmt.Println(cap(slice3))  // 6

	slice3[0] = 100
	fmt.Println(slice1)       // [1 2 3]
	fmt.Println(slice2)       // [100 2 3 10]
	fmt.Println(slice3)       // [100 2 3 10 20]
}
```

上面可以觀察到 `slice1` 加入新元素產生出 `slice2` 有發生長度與容量的變化（長度 + 1 ，容量 * 2），並且最後面 `slice1` 與 `slice2` 的值並沒有參考到同個陣列。

另外 `slice2` 加入新元素產生出 `slice3` 只有長度 + 1 而已，最後面的值也有參考到同個陣列。

由此可知，當 Slice 新增元素超過了容量的時候，它會產生新的陣列，且容量有兩倍，給新的 Slice 參考；而容量夠用的時候，則不會產生新陣列。

`copy` 函式可以複製內容到另一個 Slice 裡，如下：

```go
package main

import "fmt"

func main() {
	src := []int{1, 2, 3}
	dst1 := make([]int, 2)
	dst2 := make([]int, 3)
	dst3 := make([]int, 4)

	fmt.Println(src)     // [1 2 3]
	fmt.Println(dst1)    // [0 0]
	fmt.Println(dst2)    // [0 0 0]
	fmt.Println(dst3)    // [0 0 0 0]

	copy(dst1, src)
	copy(dst2, src)
	copy(dst3, src)

	fmt.Println(src)     // [1 2 3]
	fmt.Println(dst1)    // [1 2]
	fmt.Println(dst2)    // [1 2 3]
	fmt.Println(dst3)    // [1 2 3 0]

	src[0] = 100

	fmt.Println(src)     // [100 2 3]
	fmt.Println(dst1)    // [1 2]
	fmt.Println(dst2)    // [1 2 3]
	fmt.Println(dst3)    // [1 2 3 0]
}
```

複製時，即使長度不一還是會執行成功，只是會沒有複製完全。


## 從陣列或 Slice 產生 Slice

除了從頭建一個新的 Slice 外，也可以從陣列或 Slice 上產生新的 Slice ，以下是簡單的範例

```go
package main

import "fmt"

func main() {
	arr := [...]int{1, 2, 3, 4, 5}

	slice := arr[1:4]

	fmt.Println(slice)        // [2 3 4]
	fmt.Println(len(slice))   // 3
	fmt.Println(cap(slice))   // 4

	slice2 := slice[1:3]

	fmt.Println(slice2)       // [3 4]
	fmt.Println(len(slice2))  // 2
	fmt.Println(cap(slice2))  // 3

	slice2[0] = 300

	fmt.Println(arr)          // [1 2 300 4 5]
	fmt.Println(slice)        // [2 300 4]
	fmt.Println(slice2)       // [300 4 5]
}
```

`[1:4]` 代表的意思是，從「第 1 個元素開始，到第 4 個元素，不含第 4 個元素」，因此會取得 `[2 3 4]` 三個元素。而容量會從第 1 個元素開始，一直到結尾，以上例來說就是 `4` 。

下一個 `[1:3]` 相信就不難懂了，不過它會從 `[2 3 4]` 這個 Slice 取元素，所以會取到的是 `[3 4]` ，容量是 `3`。

最後，因為沒有使用 `append` 函式，所以它們都參考到第一個陣列。

另外也可以使用 `[:]` 來取得全部陣列的內容。

## 參考資料

* [底層為陣列的 slice](https://openhome.cc/Gossip/Go/Slice.html)

[Day 7]: /docs/day07.md
