# Interface

介面（interface）跟一般 Java 所熟知的介面意義是一樣的：定義實體（instance）的行為。

## 定義與實作

介面定義方法很簡單，只要定義傳入與傳出就行了，比方說 Generator 有個行為叫 `Name` ，我們可以這樣定義介面：

```go
type Namer interface {
	Name(gender string, firstNameNum int) (name string, err error)
}
```

介面要如何實作？只要實體有實作這個介面就行了，不管是正常的 `Generator` 實作，或是一隻鴨子 `Duck` ：

```go
type Generator struct {
	rand     *rand.Rand
	Resource NamesResource
}

func (generator *Generator) Name(gender string, firstNameNum int) (name string, err error) {
	switch gender {
	case "":
		name = generator.LastName() + generator.firstName(firstNameNum)
	case "male":
		name = generator.LastName() + generator.firstNameMale(firstNameNum)
	case "female":
		name = generator.LastName() + generator.firstNameFemale(firstNameNum)
	default:
		err = errors.New(fmt.Sprintf("gender '%s' is invalid", gender))
	}

	return
}

type Duck struct {}

func (generator Duck) Name(gender string, firstNameNum int) (name string, err error) {
	return "我是隻醜小鴨", nil
}
```

## 多型

介面也是一種型態，所以 Slice 或 Array 我們可以定義型態是介面，如：

```go
var namer1 Namer = Duck{}
var namer2 Namer = &Generator{}

arr := []Namer{
    namer1,
    namer2,
}

fmt.Println(arr)
```

這裡必須注意，因為 `Generator` 實作的時候是使用傳址，所以介面這邊是要使用 `&` 取位址； Duck 則是傳值，所以不需要使用 `&` 取位址。

這邊會發現，我們可以丟很多不一樣的實體到一個 Slice 裡，只要有實作 `Namer` 就能丟。那有沒有一個型態是可以丟所有的實體？有的，它叫 `interface{}` ：

```go
arr := []interface{}{
    Duck{},
    &Generator{},
    123,
    func(){},
}

fmt.Println(arr)
```

事實上， `fmt.Println()` 的參數，其實也是定義 `interface{}` ，所以才能接所有種類的參數。

## 判斷型別與轉型

比方說下面這段程式，定義了一個 slice 裡面有一層 slice ，但執行後會發現它出錯了：


```go
arr := []interface{}{
    []int{1, 2, 3},
}

fmt.Println(arr[0])     // [1 2 3]
fmt.Println(arr[0][0])  // invalid operation: arr[0][0] (type interface {} does not support indexing)
```

它說 `interface{}` 不支援索引取值。

其實這是強型態的特色，宣告 `interface{}` ，它就把這個變數當成是 `interface{}` ，因此它不能當作 slice 操作。除非要拜託它把變數當成 slice ：

```go
arr := []interface{}{
    []int{1, 2, 3},
}

fmt.Println(arr[0])             // [1 2 3]
fmt.Println(arr[0].([]int)[0])  // 1
```

`var.(type)` 這是轉型的操作。如果可行的話，它會轉型成 `type` ，然後後面就可以繼續接 `type` 能做的操作。

它跟 Map 一樣會回傳第兩個值是 ok ：

```go
arr := []interface{}{
    []int{1, 2, 3},
    func() string { return "func" },
}

fmt.Println(arr[0].([]int)[0])  // 1
fmt.Println(arr[1].([]int)[0])  // cannot call non-function arr[1].([]int) (type []int)
```

因此可以這樣處理

```go
arr := []interface{}{
    []int{1, 2, 3},
    func() string { return "func" },
}

fmt.Println(arr[0].([]int)[0]) // 1

if a, ok := arr[1].([]int); ok {
    fmt.Println(a[0])
} else if f, ok := arr[1].(func() string); ok {
    fmt.Println(f())  // func
}
```

或是使用 `switch` ：

```go
arr := []interface{}{
    func() string { return "func" },
}

switch v := arr[0].(type){
case []int:
    fmt.Println(v[0])
case func() string:
    fmt.Println(v())
}
```

> `value.(type)` 只能用在 `switch` 。

## 型別轉換

上面的情況是強制轉型，不過也有的情況是單純的型別轉換

```go
type A interface {
	foo()
}

type B interface {
	foo()
}

type Duck struct {}

func (duck Duck) foo() {
	fmt.Println("Hello")
}

func main() {
	var some1 A = Duck{}
	var some2 B = some1

	some1.foo()
	some2.foo()
}
```

型別也能繼承，如：

```go
type Parent interface {
	foo()
}

type Child interface {
	Parent
	bar()
}

type Duck struct {}

func (duck Duck) foo() {
	fmt.Println("Hello Foo")
}

func (duck Duck) bar() {
	fmt.Println("Hello Bar")
}

func main() {
	var some1 Child = Duck{}
	var some2 Parent = some1
	// Parent 不能直接轉回 Child
	// var some3 Child = some2
	var some3 Child = some2.(Duck)
	var some4 Child = some2.(Child)

	some1.bar()
	some2.foo()
	some3.bar()
	some4.bar()
}
```

中間會發現， `Parent` 不能轉回 `Child` 了，除非再強制轉型 `Duck` 或是 `Child` 。

因為這是強型態的特色，但只要轉換過去有達到介面實作的條件就能正常轉換了；而實作的條件最一開始就有提了：它不會管是不是繼承或是結構，而是只要實體有實作介面的行為就可以了。

## 參考資料

* [介面入門](https://openhome.cc/Gossip/Go/Interface.html)
* [介面轉換與繼承](https://openhome.cc/Gossip/Go/InterfaceCastInheritance.html)
* [element.(T) 型態測試](https://openhome.cc/Gossip/Go/Element.Type.html)
