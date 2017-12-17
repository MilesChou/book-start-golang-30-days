# Array Type

Go 語言的世界裡，陣列為固定長度，元素型態與長度都是陣列型態的一部分。

## 宣告

使用 `[n]type` 來宣告一個陣例，其中 `n` 是數字， `type` 則為型態，下面簡單的範例：

```go
package main

import "fmt"

func main() {
	var arr [5]int
	arr[0] = 5
	arr[1] = 4
	
	fmt.Println(arr)  // [5 4 0 0 0]
}
```

這裡宣告 arr 的型態是 `[5]int` ，因「元素型態與長度都是陣列型態的一部分」，所以 `[10]int` 與 `[5]int` 會是不同的型態。

另外因為後面三個元素並沒有指定新值，但可以看到它的初值是 `0` ，也就是 `zero value` 。

宣告給值的話要使用 `:=` 指定，也可以使用不固定長度 `[...]` 來宣告，它會依後面給值的數量來決定陣列長度：

```go
package main

import "fmt"

func main() {
	arr1 := [3]int{1, 2, 3}
	arr2 := [5]int{1, 2, 3}
	arr3 := [...]int{1, 2, 3, 4, 5}
	
	fmt.Println(arr1)  // [1 2 3]
	fmt.Println(arr2)  // [1 2 3 0 0]
	fmt.Println(arr3)  // [1 2 3 4 5]
}
```

如果存取陣列超過範圍時，會出現 `out of bounds` 的編譯錯誤

```go
package main

import "fmt"

func main() {
	arr := [...]int{1, 2, 3, 4, 5}

	fmt.Println(arr[9])  // invalid array index 9 (out of bounds for 5-element array)
}
```

## 複製

陣列的內容是值，所以也可以複製給另一個變數，如：

```go
package main

import "fmt"

func main() {
	arr := [...]int{1, 2, 3, 4, 5}
	
	var arrCopy [5]int
	
	arrCopy := arr

 	fmt.Println(arr)         // [1 2 3 4 5]
	fmt.Println(arrCopy)     // [1 2 3 4 5]
	
	arr[0] = 10

	fmt.Println(arr)         // [10 2 3 4 5]
	fmt.Println(arrCopy)     // [1 2 3 4 5]
	
	var arrErr [10]int
	arrErr = arrCopy         // cannot use arrCopy (type [5]int) as type [10]int in assignment
}
```

最後一行是型態不一致的錯，型態與長度相同，才有辦法複製值。

## 比較

陣列可以用 `==` 與 `!=` 來比較內容，一樣型態與長度相同才能比較。

```go
package main

import "fmt"

func main() {
	arr := [...]int{1, 2, 3, 4, 5}
	arrCopy := arr

 	fmt.Println(arr)             // [1 2 3 4 5]
	fmt.Println(arrCopy)         // [1 2 3 4 5]
	fmt.Println(arr == arrCopy)  // true
	
	arr[0] = 10

	fmt.Println(arr)             // [10 2 3 4 5]
	fmt.Println(arrCopy)         // [1 2 3 4 5]
	fmt.Println(arr == arrCopy)  // false
}
```

## 巢狀陣列

當宣告陣列時， `int` 是一種型態，所以我們在前面加上 `[n]` 即成為 `int` 的陣列型態。 

同樣地， `[n]int` 也是一種型態，在前面加上 `[m]` 就會成為 `[n]int` 的陣列型態。

下面是一個巢狀陣列的例子：

```go
package main

import "fmt"

func main() {
	var arr [3][2]int
	fmt.Println(arr)    // [[0 0] [0 0] [0 0]]
}
```
 
上面可以觀察到，這是一個有 3 個 `[n]int` 元素的陣列。

巢狀陣列也可以宣告同時指定初值：

```go
package main

import "fmt"

func main() {
	arr1 := [3][2]int{{1, 2}, {3, 4}, {5,6}}
	fmt.Println(arr1)    // [[1 2] [3 4] [5 6]]
	
	arr2 := [...][2]int{{1, 2}, {3, 4}, {5,6}}
	fmt.Println(arr2)    // [[1 2] [3 4] [5 6]]

	arr3 := [...][...]int{{1, 2}, {3, 4}, {5,6}}  // use of [...] array outside of array literal
	fmt.Println(arr3)
}
```

其中 arr3 不能這樣宣告的原因是：因為長度也是型態的一部分，宣告陣列時元素的型態必須是確定的，所以 `[2]int` 才能拿來做最外層陣列的元素型態， 而 `[...]int` 不行。

## 走訪

陣列除了可以用 for + `len()` 來走訪外，也可以使用 [`for range`][For statements] ：

```go
package main

import "fmt"

func main() {
	arr := [3]int{1, 2, 3}
	for index, element := range arr {
		fmt.Printf("%d => %d\n", index, element)
	}
}
```

## 今日回顧

今天學習了陣列宣告的基本，也多了解了兩個關鍵字 `for` `range` 和一個 function `len()` 。

## 參考資料

* [身為複合值的陣列](https://openhome.cc/Gossip/Go/Array.html)
* [Index expressions](https://golang.org/ref/spec#Index_expressions)
* [For statements][]

[For statements]: https://golang.org/ref/spec#For_statements
