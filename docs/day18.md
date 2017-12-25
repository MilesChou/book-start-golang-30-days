# Random

如果要產生假資料的話，亂數產生器是必要的。

今天先建立一個中文字的資料結構，然後再由 Go 產生亂數來選擇中文字，最後再經由 Command 輸出。

## 分析

首先要了解亂數是怎麼產生的，查了一下[資料][math/rand 和 crypo/rand 的差別]，看來 `math/rand` 就夠用了。

因為要給種子，所以查了一下時間 `time` 微秒的取法，結果找到奈秒的：

```go
t := time.Now().UnixNano()

fmt.Println(t)
```

另外[官方範例](https://golang.org/pkg/math/rand/)有展示 `math/rand` 的兩種用法，一種是全域的：

```go
rand.Seed(time.Now().UnixNano())

fmt.Println(rand.Int())
```

另一種是用種子取得結構後，再開始產生亂數：

```go
t := time.Now().UnixNano()
r1 := rand.New(rand.NewSource(t))
r2 := rand.New(rand.NewSource(t))

fmt.Println(r1.Int())
fmt.Println(r1.Int())
fmt.Println(r1.Int())
fmt.Println(r2.Int())
fmt.Println(r2.Int())
fmt.Println(r2.Int())
```

輸出如下：

```
6097576665619044690
3051760752526126731
1042372804046134795
6097576665619044690
3051760752526126731
1042372804046134795
```

亂數看來沒問題了，再來可以建一個模組存放這些中文的靜態資料。

## 開工

首先建目錄 `provider` ，在下面先建資源檔 `resource.go` ：

```go
package provider

var names = []string{
	"金太郎",
	"金城武",
	"金智賢",
}
```

這個檔比較沒什麼問題，下一個建主要產生器 `generator` ：

```go
package provider

import (
	"math/rand"
	"time"
)

type Generator struct {
	rand *rand.Rand
}

func (generator *Generator) Name() string {
	length := len(names)

	return names[generator.rand.Intn(length)]
}

func Create() Generator {
	r := rand.New(rand.NewSource(time.Now().UnixNano()))

	return Generator{
		rand: r,
	}
}
```

這裡說明一下， `Create()` 是建立 `Generator` 結構，類似工廠方法。

`Generator` 則有著亂數產生器，可以自行建立亂數，也可以依亂數取得資源內的內容。

其中 `rand.Intn(length)` 會回傳 0 ~ length 中的一個數字。剛好可以拿來當 key 。

原始碼的 Action 函式如下：

```go
func(c *cli.Context) error {
    num, err := strconv.Atoi(c.String("num"))

    if err != nil {
        return err
    }

    fmt.Println("Generate " + strconv.Itoa(num))

    generator := provider.Create()

    for i := 0; i < num; i++ {
        fmt.Println(generator.Name())
    }

    return nil
},
```

## 展示

使用 go run 執行：

```bash
$ go run main.go generate
Generate 10
金城武
金太郎
金智賢
金智賢
金智賢
金太郎
金智賢
金智賢
金智賢
金太郎
```

詳細程式可以參考 [PR Day 18](https://github.com/MilesChou/namer/pull/3)

## 參考資料

* [math/rand 和 crypo/rand 的差別][]

[math/rand 和 crypo/rand 的差別]: http://lihaoquan.me/2016/10/15/rand-in-go.html
